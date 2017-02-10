{{TOC}}

# Let’s Encrypt SSL Certificates on WebFaction

WebFaction (CSF’s web host) allows use of [Let’s Encrypt](https://letsencrypt.org/) to setup SSL certificates. The process first involves creating the new certificates, then using the [WebFaction control panel](https://my.webfaction.com) to upload them on the websites you need. 

This tutorial walks through the process as it was done to setup certs for https://csf.community and https://discussion.csf.community.

If you’ve already installed Let’s Encrypt certificates on WebFaction and just need to update them, jump to step #6 to create/renew certificates, then update them in step #7.

## About Let’s Encrypt

Let’s Encrypt provides free Secure Socket Layer (SSL) certificates. Because they are free, there are some [rate limits](https://letsencrypt.org/docs/rate-limits/) to be aware of:

* 20 certificates per week for registered domain (i.e. _domain.tld_)
* 100 names (e.g., subdomains) per certificate. (If combined with the above limit, you can issue certificates containing up to 2,000 unique subdomains per week.)
* 5 certificates per week for duplicate certificates (i.e. contain the exact same set of hostnames, ignoring capitalization and ordering of hostnames)

This means, even if you’re well under the first two limits, you have to be careful about the third rate limit, duplicate certs, because 5 is a low limit. You’re advised to use the staging (test) procedure when requesting or renewing certs, to make sure you don’t accidently use up your limit needlessly when walking through this process. The test procedure is covered in these instructions.

## 1) To ‘www’ or not to ‘www’ (Class B redirection)

This may not concern your situation, but just to cover bases… From the process of setting up SSL certificates for CSF, it was discovered that some of CSF’s domain configurations were amiss.

In our case, we don’t like using “www”. We prefer a Class B redirect. In other words, site visitors should land at _csf.community_, not _www.csf.community_, whether they added “www” or not in the URL. 

To set up a Class B on WebFaction, you must have a CNAME record in the WebFaction dashboard, **in addition** to using mode_rewrite in the app’s _.htaccess_ file to handle the Class B redirection. We didn’t realize the CNAME requirement before, and couldn’t proceed with creating Let’s Encrypt certifications until the CNAME records were setup.

At minimum, for every domain, we needed two website names, one for the target domain and one for the CNAME record. CSF’s, for example:

* **csf** = http://csf.community (target domain)
* **csf_www** = http://www.csf.community (CNAME record)

But to have SSL certificates on top of the Class B CNAME redirects, WebFaction requires _four_ website records — two for each domain type. We’ve created and named the records as:

* **csf** = [http://csf.community](http://csf.community) (Not secured)
* **csf_ssl** = [https://csf.community](https://csf.community) (Secured, our final destination)
* **csf_www** = [http://www.csf.community](http://www.csf.community) (CNAME record)
* **csf_www_ssl** = [https://www.csf.community](https://www.csf.community) (CNAME record)

The associated mod_rewrite rules in your _.htaccess_ file should be:

```
## Class B (no www) redirects
RewriteCond %{HTTP_HOST} ^www\.(.+)$ [NC]
RewriteRule ^(.*)$ http://%1/$1 [R=301,L]

## Redirects for http to https
RewriteCond %{HTTP:X-Forwarded-SSL} !on
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]
```

Now, no matter which one of the four URLs you click above, the destination should always end up being https://csf.community, as we desire for CSF. 

If you’re following these instructions and want the Class B redirects too, you need to have a similar set of four website records setup in the WebFaction dashboard _before_ proceeding with Let’s Encrypt. 

**WebFaction vocabulary:** In WebFaction parlance, a “domain” is what you would expect (e.g. _domain.tld_), and a “webapp” (or application) is any kind of software you have installed in your domain’s directory. In relation, a “website” is a name you give that domain record to conveniently reference it in the WebFaction dashboard. For example, the four bold names in the list above are website names, and _csf_ssl_ and _csf_www_ssl_ are the ones CSF needed to inform WebFaction about, as described in step #7.

## 2) Create a Let’s Encrypt webapp

Now you’ll create a new WebFaction application that can be used for creating the certificates you need now and in the future. You do not need to create a separate app for each of your websites!

In the **Applications** panel, under **Domains / Websites** of the WebFaction dashboard, create a new application as:

* name: “letsencrypt_validation”
* category: Custom
* type: Custom app (listening on port)   

Leave the **Open port** box unchecked.

When created, click the app’s name in the app list to see its properties, and make note of the **Port** number. You’ll need it later in step 5.

## 3) Install acme.sh

Neil Pang’s [acme.sh](https://github.com/Neilpang/acme.sh.git) is a “pure-shell” implementation of the [Automatic Certificate Management Environment](https://github.com/ietf-wg-acme/acme) (ACME). Pang’s acme.sh will organize your generated certificates and create a cron job to automatically check them for renewal. By adding an email address, it will notify you in advance of when certificates will expire.

Process:

Tunnel into the WebFaction server via SSH, then run the following series of commands, one at a time.

**Note:** The last command line below assumes you want to be notified by Let’s Encrypt in advance of cert expiration dates, in which case you would provide your email address, as indicated. This is probably a good idea with WebFaction because there is no auto-renewal of certificates with WebFaction; [you must manually upload them in the WebFaction dashboard](https://docs.webfaction.com/user-guide/websites.html?highlight=ssl#renew-a-certificate). See section 8 about renewal dates and notification intervals.):

1. `mkdir -p ~/src`
2. `cd ~/src`
3. `git clone 'https://github.com/Neilpang/acme.sh.git'`
4. `cd ./acme.sh`
5. `./acme.sh --install --accountemail username@emaildomain.tld`

The "install" routine creates a directory at _~/.acme.sh/_, in which all your certificates will go in respective sub-directories according to the domains you identify in the next section.   

**Important:** After doing this install, exit your SSH connection and reconnect again. 

## 4) Mount the letsencrypt_validation app

**Note:** This step takes your site offline while you do this. 

Before issuing certificates for a given domain using the _letsencrypt_validation_ app created in step #2, you need to mount that app on the associated websites for the domain in question. Using the csf.community domain as example, we would mount the _letsencrypt_validation_ app on each of the four domains described in step #1: csf, csf_ssl, csf_www, csf_www_ssl.

First go to the WebFaction dashboard and get into the **Websites** list. Then for each associated website, you’ll do the following:

1. Click the first associated “Website” name in the list. (This opens the properties form).
2. In the **Contents** field, delete the application that’s currently assigned by clicking the little “x”.
3. Click the **Add an application** drop-down menu button, then select the “_Reuse an existing application_” option.
4. In the resulting pop-up list, click the _letsencrypt_validation_ app, then click **Save**.
5. Click **Save** again at bottom of the website properties form.

Repeat the above for the other three websites (e.g. csf_ssl, csf_www, csf_www_ssl).

In a new browser tab, open and refresh the non-secured domains with and without “www” (i.e. _http://domain.tld_ and _http://www.domain.tld_) until you see a "502 Bad Gateway" error in each case. That's what you want to see!

If you don’t see the Bad Gateway error after a little time and refreshing, then you probably don’t have your domains/websites setup correctly in the WebFaction dashboard.

## 5) Run a certificate test 

Go back to your SSH connection with WebFaction at the commandline (reconnect if you need to)… You’re going to **test** that the certificates are working.

Change into the _/src_ directory:

`cd ~/src`

Then run the following single command, but replace `NNNNN` with the port number from step 2 and `domain.tld` with the appropriate domain in focus:

	Le_HTTPPort=NNNNN acme.sh --test --issue -d domain.tld -d www.domain.tld --standalone

This command instructs _acme.sh_ to bind to port `NNNNN` (the _letsencrypt_validation_ application port from step #2), thus proving to the Certificate Resource Authority (CSR) that you control the domains for which you are issuing a certificate. By doing it this way, you don’t have to separately create a CSR in the WebFaction dashboard. 

If all goes well, you'll see something like this at the end of the generated output:

```
[Fri Feb 10 13:53:02 UTC 2017] Cert success.
-----BEGIN CERTIFICATE-----
MIIFAjCCA+qgAwIBAgISAyVE+7f/wnvVKi5El7xSsbrmMA0GCSqGSIb3DQEBCwUA
           lots of other rows fill in here
GKicLVeF6TTuWVTACY1QTY8T2eJMWEjbT0QvqzisJyXpp3e8+Xw=
-----END CERTIFICATE-----
[Fri Feb 10 13:53:02 UTC 2017] Your cert is in  /home/username/.acme.sh/domain.tld/domain.tld.cer 
[Fri Feb 10 13:53:02 UTC 2017] Your cert key is in  /home/username/.acme.sh/domain.tld/domain.tld.key
```

You won’t use any of this, because it’s just a test, but that’s what you want to see. Now that you know the certificate can be issued without errors, you're ready to issue it for real.

But first remove the test certificate directory you just created:

`rm -R ~/.acme.sh/domain.tld` 

## 6) Create the real SSL certificate(s)

**Attention:** Unlike the test run before, this real run counts against the weekly rate limits set by Let’s Encrypt. If you make a mistake and have to redo it, the redo will count against your limit of 5 duplicate certification requests for the week. Let’s do this…

If you’re not already there, change into the _/src_ directory:

```
cd ~/src
```

Then run the Let’s Encrypt generation command again with the same port number used in the test, but _without_ the `--test` part in the command as before:

```
Le_HTTPPort=NNNNN acme.sh --issue -d domain.tld -d www.domain.tld --standalone
```

You should achieve the same result as previously with the test, except now you have real certificate(s) ready for installation.

## 7) In WebFaction, switch back to your original webapp

Before proceeding with installation certificates, you need to switch the web application back to the original application for the four websites — a reveral of what you did in step #4. **This brings your site back online.**

## 8) Install certificates on WebFaction

WebFaction allows you to either upload your certificates from your local server (e.g. macOS Apache server) or copy/paste them from somewhere else, such as your WebFaction server. We describe  the copy/paste process because we’ve already installed _acme.sh_ on the WebFaction server. (To do the upload process, you’d first have to install _acme.sh_ on your local server, generate the certs there as described above, and then use WebFaction’s file upload utility.) 

Go to the WebFaction dashboard, then to **Domains/Websites** > **SSL certificates**. If you haven’t added any certificates yet,  your certs list will be empty. Then:

1. Click the **Add SSL certificate** drop-down menu button.
2. Then click the **Copy & paste certificate** option. (The copy/paste form appears.)
3. **Name** field: Add a name for your certificate. As a pattern, we use the name of the website (from the four associated websites described in section #1) that reflects the domain we desire visitors to see (i.e. secured with Class B redirection) — _domain_ssl_.
4. **Domains** and **Expiry date** fields: Ignore these. WebFaction prevents any entry there anyway.
5. **Certificate** field: Copy the certificate hash that was output on the command-line from section #6, including the `BEGIN` and `END` lines, and paste it into this field.
6. **Private key** field: In the command-line output from section #6, you’ll be given a path to your cert key file. Use the `cat` command (`cat ~/.acme.sh/domain.tld/domain.tld.key`) to display the file’s contents on the command-line, which is the cert key hash. Copy the hash, including the `BEGIN` and `END` lines, and paste it into this field.
7. **Intermediates/bundle** field: _acme.sh_ will give you files for this too, and WebFaction says to add them if you have them. Repeat the `cat` command but using the new path to get the intermediates cert hash (`cat .acme.sh/domain.tld/ca.cer`). Copy the hash, including the `BEGIN` and `END` lines, and paste it into this field.
8. Click **Save**

Your certificates will be added to the list, but not yet assigned to the website. You can see this by looking at the far right of the list item record; there will be nothing under the **Used on** cell.

Now go to the WebFaction **Websites** list. You’ll assign the new certs to the two websites for the domain having “ssl” in the names (i.e. _domain_ssl_ and _domain_www_ssl_). For each of the two websites, do the following:

1. Click the name of the website to open the properties form.
2. In the **Security** row:
		1. Make sure the **Encrypted website (https)** option is active
		2. Use the drop-down menu to change the “Shared certificate” option to the certificate name you gave in step 3 above.
3. Click **Save** at bottom of the form.

Repeat for the other “ssl” website name.

You should now have working SSL certificates in your WebFaction website. If you go back to the certificates list in the dashboard, you’ll be shown how long the certificates last until they need renewed.

## 9) Redirect “http” calls to “https” only

If you don’t do this redirection, then a site visitor can access _http://domain.tld_ without SSL security applied and without getting any security errors. But if they go to _https://domain.tld_, then SSL is on and any problems (e.g. expired certificate) will show errors to the visitor.

Alternatively, you can create a mod_rewrite redirect in your _.htaccess_ that redirects all calls/visits to non-secured “http” to secured “https”. The rewrite rules for this were given in section #1. Here they are again:

```
## Redirects for http to https
RewriteCond %{HTTP:X-Forwarded-SSL} !on
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]
```

If you don’t want to use the redirect rules, just comment out the last two lines with a leading “#”, or remove the three lines altogether.

## 10) Renew certificates on WebFaction

Renewing certificates on WebFaction is the same as adding them, but without the extra labor of creating domains and binding ports.

First you’ll use acme.sh to generate new certs as described in section #6.

Then you’ll update the expiring certs in the WebFaction dashboard using the copy/paste method described in section #8. In this case, replace the old cert hashes with the new hashes. 

## 11) Certificate renewal dates and notifications

Let’s Encrypt certificates expire after 80 days, and [renewal notices](https://letsencrypt.org/docs/expiration-emails/) are sent to you three different times in advance: 20 days, 10 days, and finally 1 day, or until you renew your certs, whichever comes first. That’s all at Let’s Encrypt’s end.

But maybe you think the first 20-day notice is a little much, and the 10- and 1-day notices are enough. Or maybe just a 5-day notice is all you need to spring into action. While you can’t actually tell Let’s Encrypt when to notify you, you can make your renewal time  earlier than 80 days (not longer, sorry) to influence the notification pattern. Do this by running the following command, where “DD” is the number of days less than 80 that you want to renew your certs:

`acme.sh --renew -d mydomain.com --days DD --force`

The command will force update your installed acme.sh script by changing the default 80 renewal period to the earlier number of days you want.

For example, let’s say you don’t want a notice sooner than 5 days before the expiration period is up. To get a 5-day notice, you need to bump up the renewal date 15 days (i.e. DD=65) to cut into the  20- and 10-day notice periods (20-15=5 days). So run the command using 65:

`acme.sh --renew -d domain.tld --days 65 --force` 

Now the certificates expire after 65 days instead of the default 80, and the first renewal notice comes 5 days before they expire, and again at 1 day if you don’t renew certificates first. 

It works because Let’s Encrypt doesn’t know or care that you’ve changed the acme.sh script to renew your certifications earlier than 80 days. By forcing a renewal date to 65 days, you cut into Let’s Encrypt’s three notifications periods by 15 days; effectively bumping the first two out and creating a new first one at day 5.

**Note to CSF admins:** Certificates for CSF’s websites are currently set to expire after a provacative 69 days from renewal, which makes the first renewal notification arrive 9 days in advance.

## Keep your script updated!

The [acme.sh script](https://github.com/Neilpang/acme.sh) is continually being improved by it’s developer, Neil Pang, who is dedicted to it. You’re advised to update the script prior to using it each time you create certs for different domains. Upgrade using:

`acme.sh --upgrade`

If that doesn’t work, you probably have too old of version and need to reinstall the script using this:

`curl https://get.acme.sh | sh`

Thereafter, you should be able to update the script as normal with the previous command. 



   