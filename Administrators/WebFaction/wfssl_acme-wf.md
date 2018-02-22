# Let’s Encrypt certs on WebFaction with _acme-webfaction_

There are different ways to setup [Let’s Encrypt](https://letsencrypt.org) (LE) SSL certificates on WebFaction. [Neil Pang’s acme.sh shell](https://github.com/content-strategy-forum/csf-docs/blob/master/Administrators/WebFaction/wfssl-acme-pang.md) is one good way, though by itself you can’t automatically renew certificates on WebFaction because of the particular way the host is setup. In any case, acme.sh by itself is a general LE approach for many host situations, not just WebFaction. 

Other options now exist specifically for WebFaction and include automatic renewal of certificates. Two such alternate methods are [letsencrypt_webfaction](https://github.com/will-in-wi/letsencrypt-webfaction) (a Ruby gems method), and Greb Brown’ [acme-webfaction](https://github.com/gregplaysguitar/acme-webfaction) method, which employs a Python script to help with the auto-renewals.

As someone who liked using the acme.sh shell before, I’ve opted to describe the _acme-webfaction_ approach here, which seems less fiddly than the _letsencrypt_webfaction_ method. If you can follow Mr. Brown’s abbreviated intructions at the link above, feel free. My tutorial is considerably more thorough for non-dev types like myself. 

This tutorial is written with the assumption you have never set up Let’s Encrypt certs on WebFaction before, and you’re using the Mac Terminal.app.

## A) Insights

The three-mile high perspective.

### About Let’s Encrypt and rate limites

[Let’s Encrypt](https://letsencrypt.org) is a Certificate Resource Authority (CRA) that provides free Secure Socket Layer (SSL) certificates. You can read through LE’s documentation, but there’s a couple things worth highlighting here.

Be sure to understand [rate limits](https://letsencrypt.org/docs/rate-limits/), especially the limit about **5 certificates per week for duplicate certificates** (i.e. certs that contain the exact same set of hostnames, ignoring capitalization and ordering of hostnames).

This means you can’t create certs helter-skelter if you don’t have things setup correctly and need to keep doing it over. You’ll exceed your limit and have to wait 7 days before you can try to create your certs again. 

You’re advised to run test certs first to make sure everything works, which don’t apply against the limit. When all is fine, then run the real request and avoid hitting the limit. The test procedure is covered in these instructions.

### acme.sh

Neil Pang’s excellent [_acme.sh_](https://github.com/Neilpang/acme.sh.git) software is a highly-configurable shell implementation of the [Automatic Certificate Management Environment](https://github.com/ietf-wg-acme/acme) (ACME), and takes the place of using Certbot, which LE describes on its own site. 

Everything you need to know about it is in the commands in this tutorial, but you can read up on the full set of _acme.sh_ command [options and parameters](https://github.com/Neilpang/acme.sh/wiki/Options-and-Params) if you like.

### This method is not the end-game

WebFaction has been slow to support Let’s Encrypt “natively”. But it does seem to be in the works, according to [Ilias R.](https://community.webfaction.com/users/1146/iliasr), who commented on February 1, 2018, near the bottom of [this thread](https://community.webfaction.com/questions/21365/open-letter-to-webfaction-about-lets-encrypt-certification):

> Development of a full Letsencrypt integration within our panel is close to being finished, with mostly some minor fixes and touches left.

So don’t consider this _acme-webfaction_ script the end-all solution. There might be some easy push-button method in the dashboard eventually.

### Scope of certificates

At WebFaction, you can only use one certificate per (sub)domain, with and without “www”. For example, it would be nice to use one certificate for all of the following at the root “domain.tld”:

- domain.tld
- www.domain.tld
- sub.domain.tld
- www.sub.domain.tld

But the subdomain requires its own certificate, so in this case you need two certificates:

Cert 1 for:
* domain.tld
* www.domain.tld

Cert 2 for:
* sub.domain.tld
* www.sub.domain.tld

Et cetera.

### The _.well-known_ folder

Unless your webapp is a Django or Rails app (in which case see [Greg Brown’s usage notes](https://github.com/gregplaysguitar/acme-webfaction#usage)), then you’ll probably be using the _.well-known_ folder. This folder is created dynamically when running the script. It’s added to `~/webapps/appname/.well-known`.

The acme.sh script places a code in there, which then sends an API request to the Let’s Encrypt servers, which in turn checks to see if the requested code can be generated on your domain. The whole point is to verify that you have control of the domain and have permission to be issuing certs on its behalf.

A lot of people seem to get verification errors related to this folder (this author has). However, it’s rarely a problem with the folder (or the creation of it), per se, and more a problem with your WebFaction dashboard configurations. Because, let’s admit, it’s very easy to make soup in the WebFaction dashboard when creating and mixing “domains”, “websites”, “applications”, and “certifications”. You must double check and triple check how everything is lined up. But if you follow these instructions, you might make it through unscathed.

## B) Example setup in WebFaction dashboard

Speakign of dashboard setup, let’s say you’re creating your first cert for **domain.tld**. (This process would be the same if it was for a subdomain like **sub.domain.tld**.) You then create all of the following in their respective dashboard locations under **Domains / Websites**.

### Domains

Create two new domains:

* domain.tld
* www.domain.tld (create as a CNAME pointing to the other)

Explaining that parethentical note: 
If you don’t like using “www” in URLs, then a Class B redirect is what you need. In other words, you want site visitors to land at `https://domain.tld`, whether they add “www” in the URL or not. To set up a Class B situation on WebFaction, you use CNAME record in the WebFaction dashboard **and** use mode_rewrite in the app’s _.htaccess_ file to handle the Class B redirection.

For the CNAME part, simply configure the _www.domain.tld_ as the CNAME and point it to _domain.tld_.

The associated mod_rewrite rules in your _.htaccess_ file could  then be:

```
## Class B (no www) redirects
RewriteCond %{HTTP_HOST} ^www\.(.+)$ [NC]
RewriteRule ^(.*)$ http://%1/$1 [R=301,L]

## Redirects for http to https
#RewriteCond %{HTTP:X-Forwarded-SSL} !on
#RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]
```

The first set of rules redirects all calls to “www” to the domain without “www”, and the second set of rules (commented out here) redirects calls to “http” to “https” only.

**Attention:** For the time being, you should comment out the two lines for the second set of rules until the certificates are actually installed. Uncomment after certs are installed and working. At that point, no matter how a visitor gets to your site, they should only see the domain as `https://domain.tld`.

### Applications

Create the following 2 applications:

“**domain**” — (Or if a subdomain, use “sub_domain”.) You can really name it anything you want, but this will be the actual directory name on the server, `~/webapps/domain`, where your website files will be installed (e.g. a Textpattern installation). Create the application based on whatever tech you need for that domain (e.g. php 7.2).

“**le_validation**” — (Or if you want it more explicit, “**letsencrypt_validation**”.) You only create this app once and can re-use it in the future when creating new certs for other domains. 

Create the **le_validation** app as follows:

* **Category**: Custom
* **Type**: Custom app (listening on port)   
* Leave the **Open port** box unchecked.

### Websites

Create 4 new websites (following names are sensible examples, which keeps them together in the websites list):

* domain
* domain_ssl
* domain_www
* domain_www_ssl

Assign them to their corresponding domains with or without “www”, respectively:

* _domain_ and _domain_ssl_ —> _domain.tld_
* _domain_www_ and _domain_www_ssl_ —> _www.domain.tld_ 

For security settings on the websites, make the two “ssl” websites as **Encrypted  website (https)** and assign, temporarily, to the **Shared certificate** option. You’ll change that later when your real certificate is created.

Assign all four websites to the **le_validation** application. This is temporary until the certificate is created. Note this will take your site offline and show a **502 Bad Gateway** error. That’s what you want to see at both the http and https. Refresh pages until you do. Once the certificate is made and added, you’ll switch the 4 websites to the real application and everything will be right as rain, as they say.

Now, let’s create a certificate! 
 
## C) Install acme.sh

First, tunnel into your WebFaction server using SSH.

If you already used Neil Pang’s acme.sh script, you should have this installed already and just need to update it:

```
cd ~/src
acme.sh --upgrade
```

If you don’t already have **acme.sh** installed, install it now from your user directory: 

```
cd ~
curl https://get.acme.sh | sh
```

This will add acme.sh to `~/src`. 

Close and reopen your command-line client to enable using it. Only need to do this once upon first installation.

## D) Install _acme-webfaction.py_

Next you need to install Greg Brown’s _acme-webfaction_ python script. Run the following command:

```
wget https://raw.githubusercontent.com/gregplaysguitar/acme-webfaction/master/acme_webfaction.py
```

Move the file to your `~/bin` directory:

```
cp ./acme_webfaction.py ~/bin/
```

Change permissions on file:

```
chmod +x ~/bin/acme_webfaction.py
```

## E) Run a test certificate

As explained earlier, you should test before trying to create real certs. Do a test now buy running the following (note the important `--test` parameter added in the command), where `domain` is the name of your actual website’s desired application (discussed in example setup earlier), and `domain.tld` is the actual domain. (Chances are you named your app the same as the domain, so all instances of “domain” are the same. Remember to correct the `tld` extension too):

```
acme.sh --test --issue -w ~/webapps/domain -d domain.tld -d www.domain.tld
``` 

Note that we create the cert for both domain versions, with and without ‘www’.

If all goes well, a test certficate directory for the domain will be added to `~/.acme.sh` (i.e. `~/.acme.sh/domain.tld`) and you will see certificate information as command-line output like this:

```
[Fri Feb 10 13:53:02 UTC 2017] Cert success.
-----BEGIN CERTIFICATE-----
MIIFAjCCA+qgAwIBAgISAyVE+7f/wnvVKi5El7xSsbrmMA0GCSqGSIb3DQEBCwUA
…etc
GKicLVeF6TTuWVTACY1QTY8T2eJMWEjbT0QvqzisJyXpp3e8+Xw=
-----END CERTIFICATE-----
[Fri Feb 10 13:53:02 UTC 2017] Your cert is in  /home/username/.acme.sh/domain.tld/domain.tld.cer 
[Fri Feb 10 13:53:02 UTC 2017] Your cert key is in  /home/username/.acme.sh/domain.tld/domain.tld.key
```

Don’t use any of that! It’s just a **test**. But that’s what you want to see. Now that you know the certificate can be issued without problems or errors, you're ready to issue certificates for real.

**Note:** If you get an error when running the test, something like this:

```
domain.tld:Verify error:Invalid response from http://domain.tld/.well-known/acme-challenge/[some long string of characters]
```

It probably means your not configured correctly in the WebFaction dashboard. Have a look there again to check all is aligned and assigned appropriately.

You can also check to make sure a `.well-known` was added to our app folder (correct `appname` accordingly):

```
ls -a ~/webapps/appname/.well-known
``` 

If it’s there, fine. Leave it. This would further suggest your dashboard configuration is probably wrong somehow. 

If you make any chages in the dashboard, you should retest again. But first remove the faulty test cert folder:

```
rm -R ~/.acme.sh/domain.tld
```

However, if you get the test cert without errors, as desired and described above. Your ready for the real thing.

## F) Create the real certificate(s)

Before running the real certs request, remove the test certificate directory you just created if you haven’t already:

```
rm -R ~/.acme.sh/domain.tld
``` 

If you’re not already there, change into the _/src_ directory:

```
cd ~/src
```

Then run the same command as before except this time _without_ the `--test` option in the command. (Everything else stays the same; though, again, replace all instances of `domain` and `tld` accordingly):

```
acme.sh --issue -w ~/webapps/domain -d domain.tld -d www.domain.tld
```

You should see the same desired output as before with the test, except now you have real certificate(s) ready for use.

## G) Create WebFaction certificate

Jump over to your WebFaction dashboard again under **Domains / Websites** and create a new certificate for the one you just generated as follows:

1. Click the **Add SSL certificate** drop-down menu button.
2. Then click the **Copy & paste certificate** option. (The copy/paste form appears.)
3. **Name** field: Add a name for your certificate. A good pattern is _domain_cert_, where ‘domain’ is the name of the domain.
4. **Domains** and **Expiry date** fields: Ignore these. WebFaction fills them automatically from the certification files.
5. **Certificate** field: Copy the certificate hash that was output on the command-line, from the `BEGIN` and `END` lines, and paste it into this field.
6. **Private key** field: In the command-line output, you’ll be given a path to your cert key file. Use the `cat` command (`cat ~/.acme.sh/domain.tld/domain.tld.key`) to display the file’s contents on the command-line, which is the cert key hash. Copy the hash, again from the `BEGIN` and `END` lines, and paste it into this field.
7. **Intermediates/bundle** field: _acme.sh_ will give you files for this too, and WebFaction says to add them if you have them. Repeat the `cat` command but using the new path to get the intermediates cert hash (`cat .acme.sh/domain.tld/ca.cer`). Copy the hash again and paste it into this field.
8. Click **Save**

Your certificates will be added to the **SSL certificates** list, but not yet assigned to the website. You can see this by looking at the far right of the list item record; there will be nothing under the **Used on** cell.

Now go to the WebFaction **Websites** list. You’ll assign the new certs to the two websites for the domain having “ssl” in the names (i.e. _domain_ssl_ and _domain_www_ssl_). For each of the two websites, do the following:

1. Click the name of the website to open the properties form.
2. In the **Security** row:
		1. Make sure the **Encrypted website (https)** option is active
		2. Use the drop-down menu to change the “Shared certificate” option to the certificate name you gave in step 3 above (i.e. _domain_cert_).
3. Click **Save** at bottom of the form.

Repeat for the other “ssl” website name.

If you go back to the certificates list in the dashboard, you’ll be shown how long the certificates last until they need renewed.

## H) Switch to real app

Switch to the websites list in the dashboard and change the assigned application on all four website names from the temporary _le_validation_ app to your actual _domain_ app (or whatever your real application name is for the site).

You should now have working SSL certificates in your WebFaction website, which should be apparent in the front-end after propagating.

Give yourself a backslap.

***

Now, you could stop here and renew certs manually every time, which is not a big deal, frankly. It only takes the time to make and drink an espresso once every 2 months. In a nutshell you:

1. Update the acme.sh shell in case there were updates to it:
	1. `cd ~/src`
	2. `acme.sh --upgrade` 
2. Run a new test cert, if nervous (see section E). 
3. Run a real cert (see section F).
4. Update the ‘domain_cert’ record by copy/pasting new cert hashes into the respective fields.

Or you can do the final step below, create the cron job that renews them automatically, which is why we installed Greg Brown’s python script, and what distinguishes this method from a regular acme.sh shell method.

## Create renewal cron job

Here you install the certificate so it’s bound to the one created in the dashboard, and set a cron job for it to renew automatically.

Run the command below, making the these changes and correcting your domain names in the command:

* `WF_SERVER`: Your webfaction server name (e.g. `WebXXX`), replacing “xxx” with your server number. The title-case on `Web` is _mandatory_.
* `WF_USER` and `WF_PASSWORD`: Your WebFaction control panel login credentials.
* `WF_CERT_NAME`: Name of the certificate you created in dashboard in previous step (i.e. _domain_cert_, correcting for ‘domain`).

```
acme.sh --install-cert -d domain.tld -d www.domain.tld \
 --reloadcmd "WF_SERVER=WebXX WF_USER=user WF_PASSWORD=pass WF_CERT_NAME=certname acme_webfaction.py
```

Now you have an acme.sh crontab entry that renews the certificate automatically, and, on renewal, will trigger _acme_webfaction.py_ to update the certificate via the WebFaction API every 2 months (I think).

You can test that it’s working by forcing a renewal. (Keep in mind this counts against your weekly duplicate rate limit of 5.) Run the following command from the crontab with `--force` appended (change `USER` to your user directory name):

```
"/home/USER/.acme.sh"/acme.sh --cron --home "/home/USER/.acme.sh" --force
```

If everything is working correctly, you should see the certificates renewed and the message "Reload success".