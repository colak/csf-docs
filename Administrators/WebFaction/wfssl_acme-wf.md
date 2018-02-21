# Let’s Encrypt certs on WebFaction with _acme-webfaction_

There are different ways to setup Let’s Encrypt SSL certificates on WebFaction. [Neil Pang’s acme.sh application](https://github.com/content-strategy-forum/csf-docs/blob/master/Administrators/WebFaction/creating-ssl-certs-on-webfaction.md) was a common and general method before WebFaction added SSL/Certificates to the dashboard, and you can still use that reliably, it’s well-supported, but without auto-renewal benefit. 

Since then, other options have become available that are specifically for WebFaction and include the benefit of automatic renewal of certificates too. Two of these alternate methods are will-in-wi’s [letsencrypt_webfaction](https://github.com/will-in-wi/letsencrypt-webfaction) script, which uses Ruby gems, and Greg Brown’s [acme-webfaction](https://github.com/gregplaysguitar/acme-webfaction) script. You’ll find that WebFaction itself recommends one of these latter approaches, either via support or in the community.

As a former user of Pang’s general acme.sh method, I have opted to use Greg Brown’s _acme-webfaction_ approach, which I describe here. It seems to be a little simpler on the surface.

## Notes in advance

Two things to know to get off on the right foot.

### Scope of certificates

Something to keep in mind when you configure your cert assignments in the WebFaction dashboard… You can not have more than one (sub)domain per certificate, excluding the “www” configuration, which is allowed. For example, you can not apply a single cert to all of these:

- domain.tld
- www.domain.tld
- sub.domain.tld
- www.sub.domain.tld

That requires two certificates, one for _domain.tld_ and -_www.domain.tld_ and another for _sub.domain.tld_ and _www.sub.domain.tld_.

### _.well-known_ folder

The _.well-known_ folder is supposed to be created dynamically when running the script. It’s added to `~/webapps/appname/.well-known`. It is where the script places a code, which then sends an API request to the Let’s Encrypt server. The server API checks if the requested code can be generated on the domain. The whole point is to verify that you have control of the domain and have permission to be issuing certs on its behalf.

A lot of people seem to get errors related to this folder (this author included). However, it’s not (or rarely) a problem with the folder, per se, and more a problem with your site configuration in the WebFaction dashboard. But if you follow these instructions, you might make it through unscathed.

(Note: If your webapp is a Django or Rails app, see Greg Brown’s [usage notes](https://github.com/gregplaysguitar/acme-webfaction#usage) for different conditions.)

## Example setup in WebFaction dashboard

Let’s say you’re creating a cert for a new domain, **domain2.tld**. In the WebFaction dashboard you then create:

* 2 new domains (domain2.tld AND www.domain2.tld)
* 4 new websites (names here are sensible examples):
	* domain2
	* domain2_ssl
	* domain2_www
	* domain2_www_ssl
* 1 application (“domain2”, or whatever makes sense to you). Base it on whatever tech you need (e.g. php 7.2).

For the “www” domain (www.domain2.tld), configure it to be a CNAME that points to the other domain (domain2.tld).

For the four websites, assign them to their corresponding domains with or without “www”:

* domain2 —> domain2.tld
* domain2_ssl —> domain2.tld
* domain2_www —> www.domain2.tld
* domain2_www_ssl —> www.domain2.tld 

For security settings on the websites, make the two “ssl” websites as **Encrypted  website (https)** and assign, temporarily, to the **Shared certificate** option. You’ll change that later when your real certificate is created.

Assign all four websites to the “domain2” app.

Now, lets get a certificate.
 
## Install acme.sh

First, tunnel into your WebFaction server using SSH.

If you already used Neil Pang’s acme.sh script, you should have this installed already and just need to update it:

```
cd ~/src
acme.sh --upgrade
```

If you don’t already have acme.sh installed, install it now from your user directory: 

```
cd ~
curl https://get.acme.sh | sh
```

This will add acme.sh to `~/src`. 

Close and reopen your command-line client to enable using it. Only needed once.

## Install _acme-webfaction.py_

Next you need to install the _acme-webfaction_ python script. Run the following command:

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

## Run a test certificate

Let’s Encrypt gives you a max limit of 5 duplicate “real” certificates per week. You can’t just keep creating certs if you make mistakes or something goes wrong. If that happens, you’ll have to wait 7 days before you can request real ssl certs again. To ensure you’re not wasting requests, you can run unlimited test certs instead. Do this first to make sure everything is working correctly. 

Do a test now buy running the following (note the important `--test` parameter added), where `appname` is the name of your actual website’s app (discussed in example setup earlier), and `domain2.tld` is the actual domain:

```
acme.sh --test --issue -w ~/webapps/appname -d domain2.tld -d www.domain2.tld
``` 

If all goes well, a test certficate directory for the domain will be added to `~/.acme.sh` (i.e. `~/.acme.sh/domain2.tld`) and you will see certificate information as command-line output like this:

```
[Fri Feb 10 13:53:02 UTC 2017] Cert success.
-----BEGIN CERTIFICATE-----
MIIFAjCCA+qgAwIBAgISAyVE+7f/wnvVKi5El7xSsbrmMA0GCSqGSIb3DQEBCwUA
…etc
GKicLVeF6TTuWVTACY1QTY8T2eJMWEjbT0QvqzisJyXpp3e8+Xw=
-----END CERTIFICATE-----
[Fri Feb 10 13:53:02 UTC 2017] Your cert is in  /home/username/.acme.sh/domain2.tld/domain2.tld.cer 
[Fri Feb 10 13:53:02 UTC 2017] Your cert key is in  /home/username/.acme.sh/domain2.tld/domain2.tld.key
```

Don’t use any of that. It’s just a **test**. But that’s what you want to see. Now that you know the certificate can be issued without errors, you're ready to issue certificates for real.

**Note:** If you get an error when running the test, something like this:

```
domain2.tld:Verify error:Invalid response from http://domain2.tld/.well-known/acme-challenge/[some long string of characters]
```

It probably means your not configured correctly in the WebFaction dashboard. Have a look there again to check all is aligned and assigned appropriately.

You can also check to make sure a `.well-known` was added to our app folder (correct `appname` accordingly):

```
ls -a ~/webapps/appname/.well-known
``` 

If it’s there, fine. Leave it. This would further suggest your dashboard configuration is probably wrong somehow. 

If you make any chages in the dashboard and want to make another test (and you should until they actually work before making real ones), come back to the command-line and remove the faulty test cert folder (edit `domain2.tld` as needed):

```
rm -R ~/.acme.sh/domain2.tld
```

However, if you get the test cert as desired, described above. Your ready for the real thing.

## 6) Create the real certificate(s)

Before running the real certs request, remove the test certificate directory you just created if you haven’t already (edit `domain2.tld` as needed):

```
rm -R ~/.acme.sh/domain2.tld
``` 

Now the real thing! If you’re not already there, change into the _/src_ directory:

```
cd ~/src
```

Then run the same command as before except this time _without_ the `--test` option in the command (again, replace `appname` and `domain2.tld` accordingly):

```
acme.sh --issue -w ~/webapps/appname -d domain2.tld -d www.domain2.tld
```

You should see the same desired output as before with the test, except now you have real certificate(s) ready for use.

## Setup WebFaction certificate

Jump over to your WebFaction dashboard and create a new certificate for the one you just generated. You must use the copy/paste option. 

Here’s the entire process:

1. Click the **Add SSL certificate** drop-down menu button.
2. Then click the **Copy & paste certificate** option. (The copy/paste form appears.)
3. **Name** field: Add a name for your certificate. A good pattern is _domain2_cert_.
4. **Domains** and **Expiry date** fields: Ignore these. WebFaction fills them automatically from the certification files.
5. **Certificate** field: Copy the certificate hash that was output on the command-line from section #6, including the `BEGIN` and `END` lines, and paste it into this field.
6. **Private key** field: In the command-line output from section #6, you’ll be given a path to your cert key file. Use the `cat` command (`cat ~/.acme.sh/domain2.tld/domain2.tld.key`) to display the file’s contents on the command-line, which is the cert key hash. Copy the hash, including the `BEGIN` and `END` lines, and paste it into this field.
7. **Intermediates/bundle** field: _acme.sh_ will give you files for this too, and WebFaction says to add them if you have them. Repeat the `cat` command but using the new path to get the intermediates cert hash (`cat .acme.sh/domain2.tld/ca.cer`). Copy the hash, including the `BEGIN` and `END` lines, and paste it into this field.
8. Click **Save**

Your certificates will be added to the **SSL certificates** list, but not yet assigned to the website. You can see this by looking at the far right of the list item record; there will be nothing under the **Used on** cell.

Now go to the WebFaction **Websites** list. You’ll assign the new certs to the two websites for the domain having “ssl” in the names (i.e. _domain2_ssl_ and _domain2_www_ssl_). For each of the two websites, do the following:

1. Click the name of the website to open the properties form.
2. In the **Security** row:
		1. Make sure the **Encrypted website (https)** option is active
		2. Use the drop-down menu to change the “Shared certificate” option to the certificate name you gave in step 3 above (i.e. _domain2_cert_).
3. Click **Save** at bottom of the form.

Repeat for the other “ssl” website name.

You should now have working SSL certificates in your WebFaction website, which should be apparent in the front-end.

If you go back to the certificates list in the dashboard, you’ll be shown how long the certificates last until they need renewed.

You could stop here and renew certs manually every time, which is not a big deal, frankly. It only takes the time to drink a cappacino.

Or you can do the final step, create the cron job that renews them automatically.

## Create renewal cron job

Now we install the certificate so it’s bound to the one created in the dashboard, and set a cron job for it to renew automatically.

Run the following command (make edits as described underneath):

```
acme.sh --install-cert -d example.com -d www.example.com \
 --reloadcmd "WF_SERVER=WebXX WF_USER=user WF_PASSWORD=pass WF_CERT_NAME=certname acme_webfaction.py
```

Edit directives:

* `WF_SERVER`: Your webfaction server name (e.g. `WebXXX`), replacing “xxx” with your server number. The title-case on `Web` is _mandatory_.
* `WF_USER` and `WF_PASSWORD`: Your WebFaction control panel login credentials.
* `WF_CERT_NAME`: Name of the certificate you created in dashboard (previous step)

Now you have an acme.sh crontab entry that renews the certificate automatically, and, on renewal, will trigger _acme_webfaction.py_ to update the certificate via the WebFaction API.

You can test that it’s working by forcing a renewal. Run the following command from the crontab with `--force` appended (change `USER` to your user directory name):

```
"/home/USER/.acme.sh"/acme.sh --cron --home "/home/USER/.acme.sh" --force
```

If everything is working correctly, you should see the certificates renewed and the message "Reload success".