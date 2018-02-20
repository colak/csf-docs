# Let’s Encrypt on WebFaction using _letsencrypt_webfaction_

There are different ways to setup Let’s Encrypt SSL certificates on WebFaction. One popular way has been by using Neil Pang’s well-maintained acme.sh script ([for example](https://github.com/content-strategy-forum/csf-docs/blob/master/Administrators/WebFaction/creating-ssl-certs-on-webfaction.md)).

Lately, however, WebFaction has been encouraging its customers to use a different method, the _letsencrypt_webfaction_ script method, based on the [will-in-wi](https://github.com/will-in-wi/letsencrypt-webfaction)’s work. This alternate methods employs a custom Ruby gem and a cron job, enabling automatic renewals of the certificates, and WebFaction says it’s more suited to the way they have things setup.

This tutorial explains the latter process.

## Install the letsencrypt_webfaction gems:

Tunnel into your WebFaction server using SSH. Get into your user directory if not already there:

```
cd ~
```

Then run the following command:

```
GEM_HOME=$HOME/.letsencrypt_webfaction/gems RUBYLIB=$GEM_HOME/lib gem2.2 install letsencrypt_webfaction
```

This installs the latest version of the script and it’s components (docs, gems, etc). 

Repeat this command in the future whenever you create new certs to ensure you have the latest version of the script.

## Edit .bash_profile file for testing

Now you’ll edit the _.bash_profile_ file to initially make it easy to manually install and test certificates.

Using what method you like, open the _.bash_profile_ file in your user directory. Here we use the nano editor:

```
nano ~/.bash_profile
```

Paste the following function lines to the bottom of the file:

```
function letsencrypt_webfaction {
    PATH=$PATH:$GEM_HOME/bin GEM_HOME=$HOME/.letsencrypt_webfaction/gems RUBYLIB=$GEM_HOME/lib /usr/local/bin/ruby2.2 $HOME/.letsencrypt_webfaction/gems/bin/letsencrypt_webfaction $*
}
```

Save and close the _.bash_profile_ file then run the following command to initialize the changes:

```
source ~/.bash_profile
```


## Install certificates

First a _nota bene_: A single certificate can apply to a given root domain and any number of subdomains on that root. Even if you don’t have any special subdomains, you’re likely to still need one for calls to ‘www’ if you prefer using Class B redirects away from it (in which case you will setup CNAMEs for in the WebFaction dashboard). So at minimum you might have a cert for adding ssl to:

* _domain.tld_
* _www.domain.tld_

But let’s say you want a subdomain for a blog too. It could be anything else or in addition to, but we’ll limit it here to just an extra subdomain. So, theoretically, you could have one certificate for all of the following (or more) if at the same root domain (note each additional subdomain needs the Class B handling):

* _domain.tld_
* _www.domain.tld_
* _sub.domain.tld_
* _www.sub.domain.tld_

Use a logical/consistent naming convention for your certificates per root domain. For example: ‘domain_cert’, ‘domain1_cert’, ‘domain2_cert’, etc.

In this case of handling Let’s Encrypt certs, each certficate must also have one _.yml_ configuration file associated with it. Again, use a logical pattern for naming the configuration files. For example: _domain.config.yml_, _domain1_config.yml_, _domain2_config.yml_, etc. Save all such configuration files in a folder under your user directory (e.g. ‘_sslconfigs_’).

So, first create the directory to hold the config files:

```
makdir ~/sslconfigs
```

Then open a new blank file for the intial configuration file (change “domain” to your root domain name):

```
nano ~/sslconfigs/domain.config.yml
```

Paste the following lines into it (then you’ll do some editing):

```
key_size: 4096
endpoint: 'https://acme-v01.api.letsencrypt.org/'
domains: [domain.tld,www.domain.tld,sub.domain.tld,www.sub.domain.tld]
public: ['~/webapps/appname/']
api_url: 'https://api.webfaction.com/'
cert_name: 'domain_cert'
quiet: false
letsencrypt_account_email: 'username@domain.tld'
username: 'xxxxx'
password: 'yyyyy'
```

File edit directives:

1. **key_size** and **endpoint**: Leave them alone.
2. **domains**: Edit the domains list so it only incudes the items you need, and correct the “domain” and “tld” values accordingly.
3. **public**: Change ‘appname’ to the name of your web app as it exists on WebFaction.
4. **api_url**: Leave this alone.
5. **cert_name**: Change “domain” accordingly.
6. **quiet**: Leave this alone.
7. **letsencrypt_account_email**: Use the email address you have on record with your main WebFaction account.
8. **username** and **password**: Use the username and password for your WebFaction account.

Exit and save the file. (I.e., `Ctrl+X` then `Enter`)

## Execute letsencrypt_webfaction

Now you’ll install the certificate

```
letsencrypt_webfaction --config ~/sslconfigs/domain_config.yml
```

The command should return the following message, indicating that the certificate is installed in the panel:

```
Your new certificate is now created and installed.
You will need to change your application to use the "domain_cert" certificate.
```

Add the `--quiet` parameter in your cron task to remove this message.

N.B.: The script uses the API to initialize Webfaction certificates, so there’s no need for copy/pasting certs into the WebFaction dashboard. Go to the dashboard and verify.


## Automate certificate renewal with a Cron Job

Now you’ll create a cron job to automate updating the certificates so you don’t have to remember to do it every 2 months. We’ll define it to renew certs every 2 months at midnight on the 1st of odd months (i.e. 1 Jan, 00:00; 1 Mar, 00:00; 1 May, 00:00; etc).

This works by adding a line to crontab for each configuration file you have, thus for each certificate (and it’s associated domain and subdomains).

Again at the command-line on WebFaction… Open crontab in the `nano` editor by typing the command:

```
EDITOR=nano crontab -e
```

**For each configuration file** you are adding at this point in time (you can add more in the future), add the following line (you’ll edit a couple things):

    0 0 1 1-11/2 *  PATH=$PATH:$GEM_HOME/bin GEM_HOME=$HOME/.letsencrypt_webfaction/gems RUBYLIB=$GEM_HOME/lib /usr/local/bin/ruby2.2ruby $HOME/.letsencrypt_webfaction/gems/bin/letsencrypt_webfaction --config ~/sslconfigs/domain_config.yml

File edit directives (after `--config`):

1. Edit the config folder name, ‘_sslconfigs_’, if you used something different.
2. Edit ‘domain’ on the config file name as needed. 
3. **Do not edit the other full paths!**

Example:

    0 1 2 */2 *   PATH=$PATH:$GEM_HOME/bin GEM_HOME=$HOME/.letsencrypt_webfaction/gems RUBYLIB=$GEM_HOME/lib /usr/local/bin/ruby2.2 $HOME/.letsencrypt_webfaction/gems/bin/letsencrypt_webfaction  --config ~/le_config/config.mysite1.yml  >> $HOME/logs/user/cron.log 2>&1
    
    0 1 2 */2 *   PATH=$PATH:$GEM_HOME/bin GEM_HOME=$HOME/.letsencrypt_webfaction/gems RUBYLIB=$GEM_HOME/lib /usr/local/bin/ruby2.2 $HOME/.letsencrypt_webfaction/gems/bin/letsencrypt_webfaction  --config ~/le_config/config.othersite.yml  >> $HOME/logs/user/cron.log 2>&1
    
    0 1 2 */2 *   PATH=$PATH:$GEM_HOME/bin GEM_HOME=$HOME/.letsencrypt_webfaction/gems RUBYLIB=$GEM_HOME/lib /usr/local/bin/ruby2.2 $HOME/.letsencrypt_webfaction/gems/bin/letsencrypt_webfaction  --config ~/le_config/config.mysite3.yml  >> $HOME/logs/user/cron.log 2>&1


## Configuration on WebFaction dashboard

Example with _domain.tld_, including one special domain, _sub.domain.tld_ (but you could leave it). Edit ‘domain’, ‘sub’, and ’tld’ accordingly.

1. In dashboard under **Domains**, create the following domains:
	1. domain.tld
	2. www.domain.tld
	3. sub.domain.tld
	4. www.sub.domain.tld
2. In dashboard under **Websites**, create 4 websites; an ‘http’ and ‘https’ for each of the two non-www domains:
	1. domain (http)
	2. domain_ssl (https)
	3. domain_sub (http)
	4. domain_sub_ssl (https)
3. In dashboard under **Applications**… What you do here depends on whether you actually use a ‘sub’ domain, and if it requires a different kind of application that the root domain. But generally speaking, you have this:
	1. domain_app (= domain and domain_ssl)
	2. domain_sub_app (= domain_sub and domain_sub_ssl)
4. In the _.htaccess_ file for root and sub domains… Again, this depends on the context of root and subdomains used. But, if .htaccess file exists, add the following lines for Class B redircts and force all calls to ‘https’:

    RewriteEngine On
    RewriteCond %{HTTP_HOST} ^www\.(.*)$ [NC]
    RewriteRule ^(.*)$ https://%1/$1 [R=301,L]
    RewriteCond %{HTTP:X-Forwarded-SSL} !on
    RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]
 
## Adjust for new web app

When all the above instructions are done and the working certificate is generated, return to the WebFaction dashboard and  ensure the secure (ssl) websites are aligned with the right apps:

1. domain_ssl = domain_app
2. domain_sub_ssl = domain_sub_app
configuration of your website "mysite_secure" (select in security "https"), and in the list of certificates, choose the one that secures your site, example: "mycertif".
