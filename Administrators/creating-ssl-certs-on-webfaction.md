# Installing LetsEncrypt SSL Certificates on WebFaction

[Let’s Encrypt](https://letsencrypt.org/) provides free Secure Socket Layer (SSL) certificates; those things that put the “s” in the “https://“ of your website URLs. Because they are free, there’s a limit to how many certificates you can get in a given period of time, and their validity periods have to be updated on a regular basis.

WebFaction, our web host, allows use of Let’s Encrypt to setup SSL certificates, but, at the time this doc is written, there is no easy process for doing via the WebFaction dashboard (though that does seem to be coming). In the meantime, you can set the SSL certificates up manually. 

Having learned the process between [this WebFaction community doc](https://community.webfaction.com/questions/19988/using-letsencrypt) and a ticket to WebFaction support, we can walk you through it now, as was done to setup https://csf.community and https://discussion.csf.community. 

## 1) Make sure your WebFaction “domains” and “websites” are correct

Some may laugh at this initial reality check, but from the process of setting up SSL certificates for CSF, it was discovered that some domain configurations were amiss. (We “blame” WebFaction’s unconventional dashboard design. But, all good.)

In our case, we don’t like using “www”. Site visitors should get _csf.community/_ (without “www”) whether they added “www” or not in the URL. To do this, we must have a CNAME record setup in the WebFaction dashboard, _in addition_ to using mode_rewrite in the app’s _.htaccess_ file to handle the Class B redirection. We didn’t realize this CNAME requirement before.

So, at minimum, for every domain, we need two website names, one for the target domain and one for the CNAME record. Ours, for example:

* **csf** = http://csf.community (target domain)
* **csf_www** = http://www.csf.community (CNAME record)

But to have SSL certificates on top of the Class CNAME redirects, WebFaction requires _four_ website records — two for each domain type. We’ve named them systematically as:

* **csf** = http://csf.community (Not secured)
* **csf_ssl** = https://csf.community (Secured, our final destination)
* **csf_www** = http://www.csf.community (CNAME record)
* **csf_www_ssl** = https://www.csf.community (CNAME record)

Now, no matter which one of those URLs you click, the destination should always end up being https://csf.community. 

So if you’re following these instructions and want the Class B redirects too, you need to have a similar set of four website records setup _before_ proceeding with Let’s Encrypt. 

**WebFaction vocabulary:** In WebFaction parlance, a “domain” means what you would expect, something of the form _domain.tld_. And a “website” is a name you give that domain record to conveniently reference it in the WebFaction dashboard. For example, CSF’s main site has the domain, _csf.community_, and we’ve named it “csf”. Finally, a “webapp” (or “application”) is any kind of software you have installed in your domain directory.

## 2) Create a new Let’s Encrypt webapp

Now you’ll create a new application that can be used for _all certificates now and in the future_. You do not need to create a separate app for each of your websites!

In the **Applications** panel, under **Domains / Websites** of the WebFaction dashboard, create a new application as:

* name: “letsencrypt_validation”
* category: Custom
* type: Custom app (listening on port)   

Leave the **Open port** box unchecked.

When created, click the app’s name in the app list to see its properties, and make note of the **Port** number. You’ll need it later in step 5.

## 3) Install acme.sh

Now you’re going to install **acme.sh**, a “pure-shell” implementation of the [Automatic Certificate Management Environment](https://github.com/ietf-wg-acme/acme) (ACME). This will organize all your certificates, when you generate them, and will create a cron job to automatically check your certificates for renewal. (Inspect your crontab to see it, if you want.)

Process:

Tunnel into the WebFaction server via SSH, then run the following series of commands, one at a time:

1. `mkdir -p $HOME/src`
1. `cd $HOME/src`
1. `git clone 'https://github.com/Neilpang/acme.sh.git'`
1. `cd ./acme.sh`
1. `./acme.sh --install`

The "install" routine creates a directory at _~/.acme.sh/_, in which all your certificates will go in respective sub-directories according to the domains you identify in the next step. 

After doing this install, exit your SSH connection and reconnect again. 

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

That was just a test certificate -- you can't actually use it. But now that you know that certificate can be issued without errors, you're ready to issue it for real. 

## 6) Create the real SSL certificate(s)

Delete the directory containing the test certificate created in step 5:

`rm -r $HOME/.acme.sh/domain.tld`

Presumably you’re still in _$HOME/src_, where you need to be (if not, get back in there), so then run the generation command again, but this time _without_ the `--test` part:

```
Le_HTTPPort=77777 acme.sh --issue -d domain.tld -d www.domain.tld --standalone
```

You should achieve the same result as previously with the test, except now you have a real, working certificate(s), ready for installation.

Before proceeding with installation, return to your **Websites** list in the WebFaction dashboard, and switch your site(s) back to your original application so that it's no longer serving the "letsencrypt_validation" custom app. This brings your site back online.

## 7) Tell WebFaction to install your certificates

Finally, create a [WebFaction Support Ticket](https://help.webfaction.com/) to have your SSL certificate installed. Use the following message in your ticket request:

> Please install the certificate under "~/.acme.sh/domain.tld”
for the website "mysite_ssl".

Replace "domain.tld” with your actual domain, and "mysite_ssl" with the name of your HTTPS website. 

Of course, do that for each domain you might have created certificates for.



   