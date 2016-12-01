{{TOC}}

# Installing LetsEncrypt SSL Certificates on WebFaction

[Let’s Encrypt](https://letsencrypt.org/) provides free Secure Socket Layer (SSL) certificates; those things that put the “s” in the “https://“ of your website URLs. Because they are free, there are some [rate limits](https://letsencrypt.org/docs/rate-limits/) to be aware of:

* 20 certificates per week for registered domain (i.e. _domain.tld_)
* 100 names (e.g., subdomains) per certificate. (If combined with the above limit, you can issue certificates containing up to 2,000 unique subdomains per week.)
* 5 certificates per week for duplicate certificates (i.e. contain the exact same set of hostnames, ignoring capitalization and ordering of hostnames)

Even if you’re well under the first two limits, you have to be careful about the third rate limit, duplicate certs, because 5 is a low limit. This is why you should always use the staging (test) procedure when requesting or renewing certs, to make sure you don’t accidently use up your limit needlessly when walking through this process. The test procedure is covered in these instructions.

## Specifics regarding WebFaction hosting

WebFaction (CSF’s web host) allows use of Let’s Encrypt to setup SSL certificates, but at the time this doc is written, there is no easy process for doing it via the WebFaction dashboard. (Though there is talk in WF discussions that it may be coming.) In the meantime, you can set the SSL certificates up manually. 

The process described here is based on [this WebFaction community doc](https://community.webfaction.com/questions/19988/using-letsencrypt), support from WebFaction directly, and tips from the developer of the acme.she scipt, described later. 

This tutorial walks through the process as it was done to setup certs for https://csf.community and https://discussion.csf.community. 

## 1) Make sure your WebFaction “domains” and “websites” are correct

This may not concern your situation, but just for the record…

From the process of setting up SSL certificates for CSF, it was discovered that some of CSF’s domain configurations were amiss. (WebFaction’s unconventional dashboard design can do that. But, all good.)

In our case, we don’t like using “www”. Site visitors should land at _csf.community/_ (without “www”) whether they added “www” or not in the URL. This is called a Class B redirect. To set a Class B up on WebFaction, you must have a CNAME record in the WebFaction dashboard, _in addition_ to using mode_rewrite in the app’s _.htaccess_ file to handle the Class B redirection. We didn’t realize the CNAME requirement before, and couldn’t proceed with creating Let’s Encrypt certifications until the CNAME records were setup.

At minimum, for every domain, we needed two website names, one for the target domain and one for the CNAME record. CSF’s, for example:

* **csf** = http://csf.community (target domain)
* **csf_www** = http://www.csf.community (CNAME record)

But to have SSL certificates on top of the Class B CNAME redirects, WebFaction requires _four_ website records — two for each domain type. We’ve created and named the records as:

* **csf** = http://csf.community (Not secured)
* **csf_ssl** = https://csf.community (Secured, our final destination)
* **csf_www** = http://www.csf.community (CNAME record)
* **csf_www_ssl** = https://www.csf.community (CNAME record)

The associated mod_rewrite rules in your _.htaccess_ file should be:

```
## Class B (no www) redirects
RewriteCond %{HTTP_HOST} ^www\.(.+)$ [NC]
RewriteRule ^(.*)$ http://%1/$1 [R=301,L]

## Redirects for http to https
RewriteCond %{HTTP:X-Forwarded-SSL} !on
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]
```

Now, no matter which one of those URLs you click above, the destination should always end up being https://csf.community. 

If you’re following these instructions and want the Class B redirects too, you need to have a similar set of four website records setup in the WebFaction dashboard _before_ proceeding with Let’s Encrypt. 

**WebFaction vocabulary:** In WebFaction parlance, a “domain” is what you would expect (e.g. _domain.tld_), and a “webapp” (or application) is any kind of software you have installed in your domain’s directory. In relation, a “website” is a name you give that domain record to conveniently reference it in the WebFaction dashboard. For example, the four bold names in the list above are website names, and _csf_ssl_ and _csf_www_ssl_ are the ones CSF needed to inform WebFaction about, as described later in step 7.

## 2) Create a new Let’s Encrypt webapp

Now you’ll create a new WebFaction application that can be used for creating the certificates you need now and in the future. You do not need to create a separate app for each of your websites!

In the **Applications** panel, under **Domains / Websites** of the WebFaction dashboard, create a new application as:

* name: “letsencrypt_validation”
* category: Custom
* type: Custom app (listening on port)   

Leave the **Open port** box unchecked.

When created, click the app’s name in the app list to see its properties, and make note of the **Port** number. You’ll need it later in step 5.

## 3) Install acme.sh

Now you’re going to install **acme.sh**, a “pure-shell” implementation of the [Automatic Certificate Management Environment](https://github.com/ietf-wg-acme/acme) (ACME). This will organize all your certificates, when you generate them, and will create a cron job to automatically check your certificates for renewal (inspect your crontab to see it, if you want). By adding an email address, it will notify you in advance of when certificates will expire.

Process:

Tunnel into the WebFaction server via SSH, then run the following series of commands, one at a time (edit the email address in the last command to what you need):

1. `mkdir -p $HOME/src`
2. `cd $HOME/src`
3. `git clone 'https://github.com/Neilpang/acme.sh.git'`
4. `cd ./acme.sh`
5. `./acme.sh --install --accountemail username@emaildomain.tld`

The "install" routine creates a directory at _~/.acme.sh/_, in which all your certificates will go in respective sub-directories according to the domains you identify in the next section. The email allows you to be notified by Let’s Encrypt in advance of cert expiration times. 

Presumably you _want_ to be notified of your expiration dates, but if not, use this command instead of the last line above and you won’t get any:

`./acme.sh --install`

(See section 8 about renewal dates and notification intervals.)   

**Important:** After doing this install, exit your SSH connection and reconnect again. 

## 4) Mount the letsencrypt_validation app

Now you're ready to issue the certificates using the **letsencrypt_validation** app you created in step 2. **Attention:** This will take your site offline while you do this. 

Return to the WebFaction dashboard, then to the **Websites** region. For every website affiliated with the domain you want to make secure (in this example, the four websites described for CSF in step 1), change the assigned webapp to the _letsencrypt_validation_ app. 

In a new browser tab, open and refresh the non-secured domains (i.e. _http://domain.tld_ and _http://www.domain.tld_) until you see a "502 Bad Gateway" error. That's what you want to see!

If you don’t see the Bad Gateway error after a little time and refreshing, then you probably don’t have your domains/websites setup correctly in the WebFaction dashboard.

## 5) Run a certificate test 

Now, back to your SSH connection with WebFaction at the commandline (reconnect if you need to)… We’re going to **test** that the certificates are working.

First, change-directory into _/src_:

`cd $HOME/src`

Then run the following single command, where `NNNNN` is the port number from step 2 and `domain.tld` is the appropriate domain in focus:

	Le_HTTPPort=NNNNN acme.sh --test --issue -d domain.tld -d www.domain.tld --standalone

If you were doing more than one domain at once (i.e. `domain1.tld`, `domain2.tld`, etc), you could add as many `-d` entries as needed in series, or you can simply run the command separate for each domain. Whatever. For testing, it might be better to work with one domain at a time.

This command instructs _acme.sh_ to bind to port NNNNN (the letsencrypt_validation custom application port), thus proving to the Certificate Authority that you control the domains for which you are issuing a certificate. 

If all goes well, you'll see a return message in the command-line client indicating the new certificate was successfully issued and stored in _~/.acme.sh/domain.tld_.

That was just a test certificate -- you can't actually use it. But now that you know the certificate can be issued without errors, you're ready to issue it for real.

But first remove the directory containing the test certificate that was created during the test run:

`rm -r $HOME/.acme.sh/domain.tld` 

## 6) Create the real SSL certificate(s)

Now you’re going to create the real certificate(s). Unlike the test run before, this counts against the weekly rate limits set by Let’s Encrypt. If you make a mistake and have to redo it, the redo will count against your limit of 5 duplicate certification requests for the week.

If you’re not already there, change-directory into _/src_:

`cd $HOME/src`

Then run the Let’s Encrypt generation command again with the same port number, but this time _without_ the `--test` part:

```
Le_HTTPPort=NNNNN acme.sh --issue -d domain.tld -d www.domain.tld --standalone
```

You should achieve the same result as previously with the test, except now you have working certificate(s) ready for installation.

Before proceeding with installation, return to your **Websites** list in the WebFaction dashboard, and switch your site(s) back to your original application so that it's no longer serving the "letsencrypt_validation" custom app. This brings your site back online.

## 7) Install certificates on WebFaction

As of September 2016, you can now install certificates yourself in the WebFaction dashboard (whereas before you had to write a support ticket each time). See the [documentation for adding SSL certificates via the WebFaction dashboard](https://docs.webfaction.com/user-guide/websites.html#secure-sites-https).

***

Remove the following text when the new dashboard process has been tested and confirmed.

Finally, create a [WebFaction Support Ticket](https://help.webfaction.com/) to have your SSL certificate installed. Use the following message in your ticket request:

> Please install the certificate under "~/.acme.sh/domain.tld”
for the website "mysite_ssl".

Replace "domain.tld” with your actual domain, and "mysite_ssl" (if you named it that way) with the name of your HTTPS website. Of course, do that for each domain you might have created certificates for.

**Note:** at this point in time you still have to contact WebFaction as described when renewing the certificates too. In that case, just a support ticket again with the same details about domain directory and website name is all you need to do. No running commands yourself to update certificates. 

## 8) Certificate renewal dates and notifications

Let’s Encrypt certificates expire after 80 days (90?), and [renewal notices](https://letsencrypt.org/docs/expiration-emails/) are sent to you three different times in advance: 20 days, 10 days, and finally 1 day, or until you renew your certs, whichever comes first. That’s all at Let’s Encrypt’s end.

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



   