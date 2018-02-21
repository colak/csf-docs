# Let’s Encrypt SSL Certificates on WebFaction

WebFaction allows use of [Let’s Encrypt](https://letsencrypt.org/) (LE) to setup SSL certificates on hosted domains. For some web hosts the process can be setup so certificates renew automatically, but for WebFaction it’s a kind of two-step manual process: 

1. first you create the new certificates using special software for the job, 
2. then you load the certificate data for your domains using WebFaction’s dashboard features for doing so. 

In fact, there are a few more steps for those setting up SSL certs on WebFaction for the first time. But once you’ve done it initially for one domain, it’s more or less the two-step process after that. **This tutorial walks through the entire process, and talks about renewing certificates too.**

If you’ve already installed LE certificates on WebFaction and just need a refresher on how to renew them, jump to section #10.

## About Let’s Encrypt (LE)

LE is a Certificate Resource Authority (CRA) that provides free Secure Socket Layer (SSL) certificates. You can read through LE’s documentation, but there’s a couple things worth highlighting here.

### LE rate limits

There are some LE [rate limits](https://letsencrypt.org/docs/rate-limits/) to be aware of. Be sure to read and understand those. Especially important to understand is the rate limit about **5 certificates per week for duplicate certificates** (i.e. contain the exact same set of hostnames, ignoring capitalization and ordering of hostnames).

This means even if you’re well under the other types of rate limits, you have to be careful about duplicate certs when creating or renewing them, because 5 is a low limit, and if you mess it up, you’ll have to wait 7 days before you can try again. 

You’re advised to use the staging (test) procedure when first setting up LE and requesting or renewing certs, to make sure it all works correctly, thus you don’t accidently use up your weekly limit needlessly before getting what you need. The test procedure is covered in these instructions.

### LE certificate renewal dates and notifications

LE certificates [expire after 90 days](https://letsencrypt.org/2015/11/09/why-90-days.html), and LE says they may even shorten that time in the future. 

Currently, LE recommends renewing certs after 60 days against the 90-day validity period. In relation to that, LE sends [3 renewal notifications](https://letsencrypt.org/docs/expiration-emails/) — 20 days, 10 days, and finally 1 day in advance — to warn you of the expiration date (notifications stop if you renew the certs before those time intervals). 

But to take advantage of the notifications, you must provide an email address when installing the certificate software we use, _acme.sh_, which is described in section #3.

Let’s begin.

## 1) Create CNAME records for Class B redirection on WebFaction

If you like using “www” in URLs, then this may not concern you. But if you don’t like using ‘www’, then a Class B redirect is what you need. In other words, you want site visitors to land at `https://domain.tld` (no www), whether they add “www” in the URL or not. 

To set up a Class B situation on WebFaction, you must have a CNAME record in the WebFaction dashboard, **in addition** to using mode_rewrite in the app’s _.htaccess_ file to handle the Class B redirection.

How it works:

For each domain on WebFaction that you want a Class B redirect for, you need two website names, one for the target domain and one for the CNAME record. For example, our CSF domain is setup like this:

* **d1** = http://domain1.tld (target domain)
* **d1_www** = http://www.domain1.tld (CNAME record)

But to have SSL certificates on top of the Class B CNAME redirects, WebFaction requires _four_ website records — two for each domain type:

* **d1** = http://domain1.tld (Not secured)
* **d1_ssl** = https://domain1.tld (Secured, our final destination)
* **d1_www** = http://www.domain1.tld (CNAME record)
* **d1_www_ssl** = https://www.domain1.tld (CNAME record) 

The associated mod_rewrite rules in your _.htaccess_ file should  then be:

```
## Class B (no www) redirects
RewriteCond %{HTTP_HOST} ^www\.(.+)$ [NC]
RewriteRule ^(.*)$ http://%1/$1 [R=301,L]

## Redirects for http to https
RewriteCond %{HTTP:X-Forwarded-SSL} !on
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]
```

The first set of rules redirect calls to “www” to the domain without “www”, and the second set of rules redirects calls to “http” to “https” only.

**Attention:** For the time being, you should comment out the two lines for the second set of rules until the certificates are actually installed. You’ll then uncomment them again in section #9 later. At that point, no matter how a visitor gets to your site, they should only land on and see the domain as `http://domain.tld`. 

If you’re following these instructions and want the Class B redirects too, you need to have a similar set of four website records setup in the WebFaction dashboard _before_ proceeding with the rest of these instructions. 

**Note on WebFaction semantics:** For those new to WebFaction’s parlance, a “domain” means what you would expect (e.g. _domain.tld_), and a “webapp” (or application) is any kind of software you have installed in your domain’s installation directory (e.g. `~/webapps/domain/index.php`). In relation, a “website” is a name you give that domain record to conveniently reference it in the WebFaction dashboard, and a given domain can logically have up to 4 names to represent the different possible states of the domain. Case in point, the 4 bold names in the list above are “website” names associated to the _csf.community_ domain, and _csf_ssl_ and _csf_www_ssl_ are the ones CSF needed to create the SSL certificates.

## 2) Create a LE webapp for WebFaction use

Before you can initially create and bind certificates to the “websites” of a given domain, you need to create a new WebFaction application for that purpose. You only do this once. You can reuse the application later as many times as you need for different domains.

Process:

In the WebFaction dashboard, go to **DOMAINS/WEBSITES** > **Applications**, and create a new application as:

* **Name**: letsencrypt_validation
* **Category**: Custom
* **Type**: Custom app (listening on port)   

Leave the **Open port** box unchecked.

## 3) Install _acme.sh_

Neil Pang’s [_acme.sh_](https://github.com/Neilpang/acme.sh.git) is a highly-configurable “pure-shell” implementation of the [Automatic Certificate Management Environment](https://github.com/ietf-wg-acme/acme) (ACME), and takes the place of using Certbot, which LE describes on it’s own site. 

See the full set of _acme.sh_ command [options and parameters](https://github.com/Neilpang/acme.sh/wiki/Options-and-Params) in the associated wiki. Spend some time looking them over, or proceed as we have below. Also keep in mind that acme.sh is actively worked on and the command options evolve. (These instructions have been updated 3 times just to keep up with the changing commands.)

Notes about adding certificates to WebFaction dashboard... WebFaction allows you to upload your certificates or copy/paste them. If you want to use the upload method, you might want to install _acme.sh_ on your local system. But if you want to use the copy/paste method, then you could install _acme.sh_ on your local system or on WebFaction’s server. It makes no difference in the latter case because you’ll be copying certificate data directly from the command-line either way. _We have chosen to use the copy/paste method, and we’ve installed _acme.sh_ on WebFaction’s server. The rest of these instructions are with that context in mind._ 

**Process:**

Tunnel into the WebFaction server via SSH, then run the following series of commands, one at a time. (Note about line 5: As described earlier about LE renewal dates and notifications, you need to add your email address in the last command. If you’d rather work by your own calendar without emails, then the 5th line is just `./acme.sh --install`.)

1. `mkdir -p ~/src`
2. `cd ~/src`
3. `git clone 'https://github.com/Neilpang/acme.sh.git'`
4. `cd ./acme.sh`
5. `./acme.sh --install --accountemail username@emaildomain.tld`

The "install" routine creates a directory at _~/.acme.sh/_, and your certificates will go in respective sub-directories there according to the domains they apply to.   

After you install _acme.sh_, exit your SSH connection with WebFaction and reconnect again.

### Handling email addresses after installing:

1.  To change your email address after installing _acme.sh_, use the `--update-account` option. (Need link to wiki page for this.)
2.  To remove your email address, you’ll need to uninstall _acme.sh_, and remove the `~/.acme.sh` folder, then re-install it all fresh again without using the `--accountemail` option. (A simpler command option for this is coming, but not ready yet as of 20 Feb 2017.)

## 4) Mount the letsencrypt_validation app 

Before issuing certificates for a given domain, you need to mount the _letsencrypt_validation_ app created in section #2 on the four associated ‘websites’ for the domain in question. 

As example, for the csf.community domain, we mounted the _letsencrypt_validation_ app on the 4 associated “websites”: d1, d1_ssl, d1_www, d1_www_ssl.

**Note:** This will take your website offline while you do this. You put it back online again in section #7.

Process:

In the WebFaction dashboard, go to **DOMAINS/WEBSITES** > **Websites**. Then for each associated website, you’ll do the following:

1. Click the first associated “Website” name in the list (e.g. csf). (This opens the properties form).
2. In the **Contents** field, delete the application that’s currently assigned by clicking the little “x”.
3. Click the **Add an application** drop-down menu button, then select the “_Reuse an existing application_” option.
4. In the resulting pop-up list, click the _letsencrypt_validation_ app, then click **Save**.
5. Click **Save** again at bottom of the website properties form.

Repeat the above steps for the other three websites.

In a new browser tab, open and refresh the non-secured domains with and without “www” (i.e. `http://domain.tld` and `http://www.domain.tld`) until you see a **502 Bad Gateway** error in each case. That's what you want to see!

If you don’t see the Bad Gateway error after a little time and refreshing, then you probably don’t have your domains/websites setup correctly in the WebFaction dashboard.

## 5) Run a certificate test 

Go back to Terminal with your SSH connection with WebFaction (reconnect if you need to)… You’re going to **test** that the certificates are working.

Change into the _/src_ directory:

```
cd ~/src
```

For certificates at WebFaction on an Apache server, you can test run and create new certificates using one of two modes: **webroot mode** (multiple domains) or **Apache mode**. The commands for these modes (and other non-usable modes in this case) are available in the [acm.sh instructions](https://github.com/Neilpang/acme.sh/wiki/How-to-issue-a-cert), but we describe the webroot mode here for multiple domains (i.e. with and without "www"). 

Run the following single webroot command, but replace `domain.tld` with the domain you’re creating certificates for, `user` with your WebFaction username, and `app` with the name of the WebFaction webapp for the domain:

```
acme.sh --test --issue -d domain.tld -d www.domain.tld -w /home/user/webapps/app
```

The `--test` parameter prevents this test run from being counted against LE’s limit of 5 duplicate cert requests per week, as described in the _About Let’s Encrypt_ section at head of this doc.

The `--issue` option, in not so technical terms, is the executive order from the command that says “create the certs”.

The `-d` option, and associated domain parameter, is how you indicate which domain to create the certificate for. Since we're specifically using the webroot mode in this case, you can only add as many of these as applies for the given domain name.

If all goes well, when running the command, you will see certificate information like the following at the end of the generated output:

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

You won’t use any of this, because it’s just a **test**, but that’s what you want to see. Now that you know the certificate can be issued without errors, you're ready to issue certificates for real.

First, remove the test certificate directory you just created, which is necessary:

```
rm -R ~/.acme.sh/domain.tld
``` 

## 6) Create the real SSL certificate(s)

**Note:** Unlike the test run before, this real run counts against the weekly rate limits described earlier. If you make a mistake and have to redo it, the redo will count against your limit of 5 duplicate certification requests for the week. 

If you’re not already there, change into the _/src_ directory:

```
cd ~/src
```

Then run the same command as before except this time _without_ the `--test` option in the command, remembering to change the dummy values here, of course:

```
acme.sh --issue -d domain.tld -d www.domain.tld -w ~/webapps/appname
```

You should achieve the same kind of output as with the test, except now you have real certificate(s) ready for use.

## 7) In WebFaction, switch back to your original webapp

Before proceeding with installing the certificates, you need to switch the mounted web application back to the original application for the four websites. In other words, a reversal of what you did in section #4. **This brings your site back online.**

## 8) Install certificates on WebFaction

In the WebFaction dashboard, go to **Domains/Websites** > **SSL certificates**. If you haven’t added any certificates yet, your certs list will be empty. Now you will create one.

Process:

1. Click the **Add SSL certificate** drop-down menu button.
2. Then click the **Copy & paste certificate** option. (The copy/paste form appears.)
3. **Name** field: Add a name for your certificate. A good pattern is _rootdomain_ssl_.
4. **Domains** and **Expiry date** fields: Ignore these. WebFaction fills them automatically from the certification files.
5. **Certificate** field: Copy the certificate hash that was output on the command-line from section #6, including the `BEGIN` and `END` lines, and paste it into this field.
6. **Private key** field: In the command-line output from section #6, you’ll be given a path to your cert key file. Use the `cat` command (`cat ~/.acme.sh/domain.tld/domain.tld.key`) to display the file’s contents on the command-line, which is the cert key hash. Copy the hash, including the `BEGIN` and `END` lines, and paste it into this field.
7. **Intermediates/bundle** field: _acme.sh_ will give you files for this too, and WebFaction says to add them if you have them. Repeat the `cat` command but using the new path to get the intermediates cert hash (`cat .acme.sh/domain.tld/ca.cer`). Copy the hash, including the `BEGIN` and `END` lines, and paste it into this field.
8. Click **Save**

Your certificates will be added to the **SSL certificates** list, but not yet assigned to the website. You can see this by looking at the far right of the list item record; there will be nothing under the **Used on** cell.

Now go to the WebFaction **Websites** list. You’ll assign the new certs to the two websites for the domain having “ssl” in the names (i.e. _domain_ssl_ and _domain_www_ssl_). For each of the two websites, do the following:

1. Click the name of the website to open the properties form.
2. In the **Security** row:
		1. Make sure the **Encrypted website (https)** option is active
		2. Use the drop-down menu to change the “Shared certificate” option to the certificate name you gave in step 3 above.
3. Click **Save** at bottom of the form.

Repeat for the other “ssl” website name.

You should now have working SSL certificates in your WebFaction website. If you go back to the certificates list in the dashboard, you’ll be shown how long the certificates last until they need renewed.

## 9) Redirect “http” calls to “https” only

As described in section #1, you can create a set of mod_rewrite rules in your _.htaccess_ that redirects all calls/visits to non-secured “http” to secured “https”. If you added the rules and commented out the lines, just uncomment them now so you have this:

```
## Redirects for http to https
RewriteCond %{HTTP:X-Forwarded-SSL} !on
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]
```

If you don’t want to use the redirect rules, just leave the two lines commented out. In that case, a site visitor can access `http://domain.tld` without SSL security applied and without getting any security errors. But if they go to `https://domain.tld`, then SSL will be active and any problems (e.g. expired certificate) will show errors to the visitor at that location.

## 10) Renew certificates on WebFaction

Before renewing certificates, update the _acme.sh_ script. It’s continually being revised/improved and you’ll want to ensure you have the latest working build:

```
cd ~/src
acme.sh --upgrade
```

Then run the following command to renew certs at the target domain:

```
acme.sh --renew -d domain.tld -d www.domain.tld --force
```

Then follow the copy/paste steps in section #8 to replace the old cert hashes with the new ones.