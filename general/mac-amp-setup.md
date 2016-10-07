# Setting up Apache, MySQL, and PHP on a Macintosh computer

_These instructions describe setting up a web development environment on a Mac using built-in and third-party software. Notable components are an **Apache** web server, a **MySQL** database server, and **PHP** — collectively, an **AMP** stack. You won’t have to use the database — e.g. maybe you like flat-file websites — but it will be setup and configured anyway just in case. We also cover how to upgrade components, when necessary, and demonstrate the basics of setting up a website with a content management system, putting all three components to work._ 

***

Written and edited by [Destry Wion](https://github.com/wion). 

Maintained to the latest macOS version.

| Published sections | OS version | Date of action |
|:---|:---|:---|
| First draft of Apache sections. | Sierra, 10.12 | 7 Oct 2016 |

General feedback is welcome about the usefulness, clarity, and accuracy of these instructions. 

Additionally, I am looking for three testers that would try the instructions under a controlled process, working at your own pace, and having my direct support along the way. In this case, you must have never set up an AMP stack on your Mac before, tend to shy away from the command-line, and do not identify as a developer. The testing will follow an atononymous, systematic procedure that I’ll communicate to you after you volunteer and I’ve asked you some questions.

***      

Contents:

{{TOC}}

***

## 1) Introduction

These instructions are written for Mac owners who do not identify as “developers”, have never used the built-in web server before, and maybe don’t know what the Terminal.app is. The instructions are written with a deliberate effort to teach rather than rush you through enigmatic commands.

### 1.1) Why might you care

You might set up a working web server on your macOS for a number of reasons, such as to:

* practice HTML, CSS, and other web standards and languages,
* build a flat-file prototype as design thinking and testing toward a dynamic product goal,
* try different CMSes before making a choice for a single one,
* maintain a staging platform to safely test new content and design modifications for a production website,
* collaborate with others as writer, designer, or developer on an open project via a repository hosting service (e.g. GitHub, BitBucket, Gitlab…), or
* all the above — and more.

Again, you do not have to be a developer, or need to write lines of code, to do these kinds of things. I am _not_ a developer. I work with words and UI layout. Yet I do need to install and test different web software packages on occasion. I do need to maintain staging sites for a few important production websites, whether side projects, for clients, or my own. And I do collaborate with others via GitHub, maintaining repositories and writng documentation for open source web publishing systems. You can’t write leading-edge documentation for products you can’t trial out of the box.

By doing all of those things, I: 

* remain competent with HTML and CSS, 
* get better using the command-line and learnig my Mac’s filesystem, 
* stay familiar with conventional tools and techniques used in distributed collaboration situations, and
* can talk about it all better in context when working with others.

### 1.2) What you will need and touch

Your Mac computer provides everything you need to establish a web development environment: 

* an Apache web server, 
* a PostgreSQL relational database, and 
* PHP to support dynamic scripting.

Making it all work requires nothing but running copy/paste commands in Terminal and making a few file edits.

We won’t be using the built-in PostgreSQL database, however. That would suggest an “APP” stack (an appropriate acronym, actually). I describe installing and using a MySQL database instead. I’ll address that choice more in the database section, but it’s basically a “best fit” decision (for the time being). 

I also touch on upgrading components when it’s _necessary_, and in this case I’ll elaborate on upgrading your PHP. By relying on Mac’s built-in components, whenever possible, we pass some of the responsibility to Apple to help us keep our components functional. Leveraging the built-in components is useful if, like me, you don’t live on the edge of your seat to upgrade software the moment a new version is released. We just want to write and edit, after all, not innovate the internet. Generally speaking, if the old dog is healthy, I don’t replace it. It’s only necessary to upgrade a given stack component when it’s a question of system security or said component stops being functionally compatable with related applications you rely on.

Of course, I’ll be using Terminal.app (the command-line client) along with my text editor of choice, TextMate. You can use whatever text editor you want, but you will need to use the command-line to follow these instructions. All commands are cut-’n’-paste with some explanation, no typing necessary. I’ll also be using [Homebrew](http://brew.sh/)’s utilities, so you’ll need `brew` installed too, and we’ll come back to doing that when it’s time.

### 1.3) You’re on your own, for the most part

A major reason for writing these instructions, admittedly, is so that I stay sharp on the process myself and have good instructions to refer to each time. But I wouldn’t write them so elaborately and make them public if I didn’t want others to succeed as well. I wish there had been instructions like these 12 years ago when I bought my first Macbook Pro. 

That said, I am a busy person like anyone else. I’ve chosen to invest my effort in maintaining the documentation based on feedback and user testing instead of providing one-on-one support to unlimited numbers of people who might have questions at every step. If you decide to follow these instructions, you’re on your own, just as I was 12 years ago. But if you follow the instructions to the letter, and your filesystem hasn’t been altered in any way that would make the instructions obsolete, you will succeed! 

Start from the beginning and work through each section in order. If you run into trouble, there are many knowledge bases to seek solutions from. [StackOverlow](http://stackoverflow.com/) is one that has helped me tremendously over the years.

Let’s get started.

## 2) Apache (the built-in web server)

You’ll set up and configure the built-in Apache web server and PHP before installing MySQL. You’ll begin by:

1. making a new web root directory,
2. configuring Apache, and
3. adding placeholder web project domains.

### 2.1) DocumentRoot (web root directory)

A ‘DocumentRoot’ is a place in your macOS filesystem where web documents are rendered dynamically by the Apache web server. If web documents are not in a web root directory, or in designated subdirectories (i.e. web domains, or ‘VirtualHost’ domains) then Apache won’t recognize them.

Over the history of Mac OS X (prior to macOS Sierra), there have been two DocumentRoot locations: system level and user level. 

### 2.1.1) System-level vs. user-level

In macOS Sierra, the system DocumentRoot is the supported default, located at _/Library/Webserver/Documents_. The leading foreslash indicates the path is relative to your hard-drive’s root (disk root). The fact it’s a “system” location means people sharing the computer, if that were the case, could all access and administer content there. From a web browser’s context, this location is represented as **http://localhost/**.

However, most Mac owners who do not need to share their computer prefer  using a DocumentRoot in their user directory instead, located at   _/Users/<username>/Sites_, where “<username>” is their account name. Because the _Sites_ folder is under the user directory, only the named user can access and manage the content there. 

The _Sites_ location used to be the default DocumentRoot in OS X up to Lion, then Apple moved away from it in later OS releases and stopped providing the _Sites_ directory. By that point, however, most of us Mac owners had become so used to the user location that we continue to use it, overriding Apple’s default configuration after each new OS upgrade. It has become somewhat a convention, and this is what I describe in these instructions.

### 2.1.2) Creating new web root and project directories 

You’ll start by creating the web root directory (_Sites_) and a couple of placeholder project directories inside it (_domain1_ and _domain2_). You’ll then put a test web document (_phpinfo.php_) inside both levels of the folder tree for use later when testing the server. In other words, you’re going to create this file tree structure:

	Sites
		domain1
			phpinfo.php
		domain2
		phpinfo.php

If you’re a relatively new Mac user, and certainly if you didn’t use OS X Lion or older, chances are good you don’t  have the _Sites_ directory and need to create it. 

Run the following ‘list directory contents’ command to see if the _Sites_ directory exists and if anything is in it:

```
ls ~/Sites
```

(**Note:** The tilde symbol (`~`) is common short notation in filesystems meaning the path to your user directory: `/Users/<username>`, where `<userame>` is your actual username. This notation is extremely useful when working on the command-line, as it helps make writing commands easier/shorter when in the user directory context. Throughout this document, `<username>` is used as a variable that you need to replace with your actual user name. For example, if your username is “kelly”, then your `~` directory path is _/Users/kelly_.) 

If the directory exists but is empty, you’ll just get a new command-line prompt ending with `$`. 

If it exists and there’s something in it, you’ll get a list of folder and file names (you might want to clean those up later if they are not your desired web documents).

But if a _Sites_ directory does not exist, you’ll see an error telling you as much. Create the directory by running the following ‘make directory’ command:

```
mkdir ~/Sites
```

Then run the following two commands one at a time, which create the placeholder subdirectories:

```
mkdir ~/Sites/domain1
mkdir ~/Sites/domain2
```

For the purpose of testing our locations later, you’ll add a test PHP file in the DocumentRoot and copy it to one of the subdirectories. First run the following command:

```
nano ~/Sites/phpinfo.php
```

The Nano editor will appear in the console. Copy (Cmd+C) the following line and paste it (Cmd+V) into the open nano file:

```
<?php phpinfo(); ?>
```

Then hit **Ctrl+X** keys to exit Nano editing mode, then hit **Y** key when asked to save file changes, then hit **Return** key to return to the command-line.

Finally, run the following command to copy the _phpinfo.php_ file to the _domain1_ subdirectory:

```
cp ~/Sites/phpinfo.php ~/Sites/domain1
```

### 2.2) Apache configuration: _<username>.conf_

To make Apache understand that your new DocumentRoot is at the user level in the _Sites_ directory, you need to create a user configuration file (_<username>.conf_) that contains the necessary directory mapping. 

**Attention:** make sure to correct the `<username>` variable in all instances it appears.

Run the following command, which opens a new file in the Nano editor:

```
nano /etc/apache2/users/<username>.conf
```

If that command won’t open the editor, then try this one with `sudo`, which gives it an extra boost with admin privileges:

```
sudo nano /etc/apache2/users/<username>.conf
```

(**Note:** avoid trying to use `sudo` when not required in a given command, because it’s like playing with a loaded gun; if you don’t know what you’re doing, your filesystem could get hurt.)

When the Nano file is open on the command-line, copy (Cmd+C) the following content and paste it (Cmd+V) into the file:

```
<Directory "/Users/<username>/Sites/">
AllowOverride All
Options Indexes MultiViews FollowSymLinks
Require all granted
</Directory>
```

Again, make sure you correct the `<username>` variable. Use the arrow keys to move through the lines of content in the Nano editor.

When ready, hit **Ctrl+X** keys to exit Nano editing mode, then hit **Y** key to save file changes, then hit **Return** key to return to the command-line.

Finally, you need to set the access permissions on the file. Run the following command (which does need `sudo`) and correct the user name:

```
sudo chmod 644 <username>.conf
```

You can verify the file was created by running the following and finding the file listed in the resulting output:

```
ls /etc/apache2/users
```

Onward to main Apache file configurations…

### 2.3) Apache configuration: _httpd.conf_

You’re now going to make a series of edits to the main Apache configuration file, _httpd.conf_. In total, you will:

1. turn on some DSO modules that are off by default, 
3. set ‘ServerAdmin’,
3. set the Apache ‘ServerName’, 
4. tell Apache where your new ‘DocumentRoot’ is, and
5. include needed configuration files.

**Notes about file editing via the command-line**

I’m going to assume you don’t have a lot of experience with using command-line text editors like Nano (the simpleton), Vim (the popular), or Emacs (the old), but if you do, great, just ignore this. Indeed, learning one takes a little time and practice, which is worth doing another day (go with Vim when you do), but it won’t help you now finish these instructions sooner. 

So with the exception of using Nano for a simple task I already walked you through in the previous section, you’re going to use your preferred GUI text editor. Further, you’ll open files in context of the editor via the command-line. There’s no cheating here by clicking around in your Dock and Finder, which is also an uproductive way to work.

There are multiple commands you can use to open a file from the command-line in context of a GUI application. This command, for example, opens a named file in the TextEdit.app, Mac’s default text editor (not my favorite app, to be sure):

```
open -e /path/to/file.ext
```

I will use that command in these instructions. It serves as a reliable constant that’s easy to describe since I don’t know what other text editor you might have installed. But following are a couple more commands just to give you an idea of options.  

This command also uses the `open` utility but allows you to name the text editor you want the file opened in. Yyou just have to replace `<TextEditorName>` with the name of the text editor application you want to use:

```
open -a /Applications/<TextEditorName>.app /path/to/file.ext
```

**Tip:** When needing to edit a command, use ← and → arrow keys to move across the command-line one character at a time, or in combination with the ⌥ key (i.e. ⌥+← or ⌥+→) to move one word at a time.

Or, if like me, you have TextMate installed, you can run this command to open the file using TextMate’s own command-line utility:

```
mate /path/to/file.ext
```

Other third-party editors might have such utlities too. I don’t know.

Moving on… 

**Open _httpd.conf_:**

Run the following command to open the _httpd.conf_:

```
open -e /etc/apache2/httpd.conf
```

Once opened, you should be looking at a file that begins with the following lines:

```
#
# This is the main Apache HTTP server configuration file.  It contains the
# configuration directives that give the server its instructions.
# See <URL:http://httpd.apache.org/docs/2.4/> for detailed information.
# In particular, see 
# <URL:http://httpd.apache.org/docs/2.4/mod/directives.html>
# for a discussion of each configuration directive.
```

Every line you see in the file that begins with an octothorpe (`#`) is a line that has been “commented out” (i.e. “turned off”), which prevents Apache from reading those lines. Conversely, lines that don’t start with an `#` are active (i.e. “turned on”) and readable by Apache. Thus the `#` is effectively the on/off switch for content in configuration files.

Most lines in the file are just text notes that should never be uncommented because Apache wouldn’t know how to read them and stop working as a result. Other lines are actual configuration lines that Apache recognizess when not commented out. The configuration items are organized top-to-bottom in a required order, so never re-arrange the order. Many configuration lines must be turned on for basic function, while others are optional to a server administrator’s needs. Our focus is on the configuration items needed for your local web server to work as intended, including the creation of ‘VirtualHost’ domains.

**Tip:** The note lines are the bulk of the configuration file, thus quite a lot to scroll through. They’re included as a kind of quick help for understanding the configuration settings, but not as a subtitute for the [official Apache docs](http://httpd.apache.org/docs/2.4/). Removing the note lines won’t hurt anything, but it’s worth pointing out that instructions like what you’re reading now often use line numbers to help you pinpoint configuration settings quicker. If you remove all the note bloat, the line numbers would be meaningless and confusing. On the other hand, the file would be much easier to scan, thus mentioning line numbers might not be necessary. In my opinion, all the note comments are an eyesore and subject for removal (and I remove them in my own files), while remaining header notes can be edited more concisely. But that’s out of scope here, and I do reference line numbers to help you.

#### 2.3.1) Activate DSO modules

Beginning around line 55 is where the list of Dynamic Shared Object (DSO) modules begins. Most of the modules will be commented out by default, while others will be active. You won’t turn any off, but you do need to ensure a few more are turned on by removing the `#` in front of them.

Make sure the following module are active. The line numbers should be correct, but treat them as approximations and ensure the correct modules are being addressed in each case. If any module is already active, skip to the next one:

* 103: `LoadModule include_module libexec/apache2/mod_include.so`
* 114: `LoadModule log_config_module libexec/apache2/mod_log_config.so`
* 160: `LoadModule vhost_alias_module libexec/apache2/mod_vhost_alias.so`
* 166: `LoadModule userdir_module libexec/apache2/mod_userdir.so`
* 168: `LoadModule rewrite_module libexec/apache2/mod_rewrite.so`

Continue…

#### 2.3.2) Set ‘ServerAdmin’

Around line 204 is the ‘ServerAdmin’ setting. Change the dummy email to a working email address that you receive email at, for example.

```
ServerAdmin you@gmail.com
```

Continue…

#### 2.3.3) Set the Apache ‘ServerName’

Around line 213 is the ‘ServerName’ setting. For individual Mac users like us, who are not running any networks, or whatever, this just needs to be your Mac’s IP address. 

To find your machine’s IP, hop back into Terminal and run the following command: 

```
ifconfig |grep inet
```

The output will look something like this:

```
	inet 127.0.0.1 netmask 0xff000000 
	inet6 ::1 prefixlen 128 
	inet6 fe80::1%lo0 prefixlen 64 scopeid 0x1 
	inet6 fe80::10d3:e27d:eed5:d9df%en1 prefixlen 64 secured scopeid 0x7 
	inet 192.169.0.75 netmask 0xffffff00 broadcast 192.169.0.266
	inet6 fe80::b44b:9c12:aa8c:bd6c%utun0 prefixlen 64 scopeid 0xa
```

Your Mac IP will be the numbers in the **second-to-last line** immediately after `inet`. In the example above, the Mac IP address is **192.169.0.75**. (The Mac IP is _not_ 127.0.0.1, which is your localhost address.)

Copy the IP number, jump back to line 213 in the main configuration file, and past the IP number as indicated below by the exes:

```
ServerName xxx.xxx.x.xx
```
 
Continue…

#### 2.3.4) Change ‘DocumentRoot’ path

In section C.1 you created a new web root directory at _~/Sites_ (i.e. _/Users/<username>/Sites_). Now you’re going to set the configuration paths that tell Apache to use that location. 

Around lines 237-8 will be the default ‘DocumentRoot’ settings:

```
DocumentRoot "/Library/WebServer/Documents"
<Directory "/Library/WebServer/Documents">
```

Change the paths in those two lines to _/Users/<username>/Sites_, where `<username>` is the name of your user directory:

```
DocumentRoot "/Users/<username>/Sites_"
<Directory "/Users/<username>/Sites_">
```

Continue…

#### 2.3.5) Include needed configuration files

In addition to the main configuration file that you’re editing, Apache will need two other configuration files for our intended purposes: _httpd-userdir.conf_ and _httpd-vhosts.conf_. The files already exist, you just need to uncomment the following lines:
 
* 493: Include /private/etc/apache2/extra/httpd-userdir.conf
* 499: Include /private/etc/apache2/extra/httpd-vhosts.conf

The two files work in relation to the DSO modules on lines 160 and 166 that you uncommented earlier.

You’re now done editing the main configuration file (_httpd.conf_). Hit Cmd+S to save your changes and close the file.

### 2.4) Apache configuration: _httpd-userdir.conf_

The _httpd-userdir.conf_ configuration file is required to force Apache to recognize the _Sites_ directory as the new user level DocumentRoot we desire for default. 

Run the following command to open the _httpd-userdir.conf_ file:

```
open -e /private/etc/apache2/extra/httpd-userdir.conf
```

This is a small file of about 20 lines. Look to make sure the ‘UserDir’ value around line 10 is uncommented and equal to:

```
UserDir Sites
```

Also make sure the ‘Control Access’ lines at bottom of file are uncommented and equal to:

```
Include /private/etc/apache2/users/*.conf
<IfModule bonjour_module>
       RegisterUserSite customized-users
</IfModule>
```

Hit Cmd+S to save your changes, if necessary, and close the file.

### 2.5) Apache configuration: _httpd-vhosts.conf_

The _httpd-vhosts.conf_ configuration file is needed to define the ‘VirtualHost’ containers that will make the placeholder subdirectories in the user-level DocumentRoot function like individual web domains (unique websites).

You can create as many such domains as you want. We’re only creating two in these instructions (_domain1_ and _domain2_) as examples to get you familiar with the process. The names are temporary until you change them to your needs at a later time.

Run the following command to open the _httpd-vhosts.conf_ file:

```
open -e /private/etc/apache2/extra/httpd-vhosts.conf
```

When open, replace the file’s entire contents with the content below. Yes, it’s okay. The process via keyboard is: 

1. highlight all the lines below, 
2. copy them to clipboard (**Cmd+C**), 
3. highlight all the lines in the default _httpd-vhosts.conf_ file, 
4. paste content from clipboard (**Cmd+V**), 
5. correct all instances of `<username>` with your actual username, and
5. save the changes (**Cmd+S**).

You could, of course, use your silly mouse or trackpad too. ;)

```
# VIRTUAL HOSTS
#
### VirtualHost Template Block
#
#<VirtualHost *:80>
#    DocumentRoot "/Users/<username>/Sites/<domainname>"
#    ServerName <domainname>.dev
#    ErrorLog "/private/var/log/apache2/<domainname>-error_log"
#    CustomLog "/private/var/log/apache2/<domainname>-access_log" common
#</VirtualHost>

### VirtualHost for domain1
#
#<VirtualHost *:80>
#    DocumentRoot "/Users/<username>/Sites/domain1"
#    ServerName domain1.dev
#    ErrorLog "/private/var/log/apache2/domain1-error_log"
#    CustomLog "/private/var/log/apache2/domain1-access_log" common
#</VirtualHost>

### VirtualHost for domain2
#
#<VirtualHost *:80>
#    DocumentRoot "/Users/<username>/Sites/domain2"
#    ServerName domain2.dev
#    ErrorLog "/private/var/log/apache2/domain2-error_log"
#    CustomLog "/private/var/log/apache2/domain2-access_log" common
#</VirtualHost>
```

With the new file content we have:

* cleaned out the needless note comments, 
* added a refined VirtualHost template block (be mindful of the variables, `<username>` and `<domainname>`), which you could feasibly delete since you have the following blocks as examples, and 
* added two VirtualHost blocks for your placeholder web domains in the new _Sites_ DocumentRoot. 

The two VirtualHost blocks are commented out until you’re ready to use them. Some notes about the four VirtualHost block lines:

`DocumentRoot` in this case is the unique path to the specific subdirectory domain.

`ServerName` is the domain name that will be seen in a web browser’s address bar. It can be whatever you want, but is ideally the same name as the associated DocumentRoot directory. Further, the ServerName (and DocumentRoot directory) are typically named after a matching production domain online, if one exists. The idea is you’re using the local site as a staging domain for the production site. A distinction is made between the two by adding the fake “.dev” extension to the local version. You could use any fake extension you wanted as long as it was not a real [Top-level domain](https://en.wikipedia.org/wiki/List_of_Internet_top-level_domains). For that reason, “.dev” is a good choice and perfectly suggestive of where it is and what it’s for. 

`ErroLog` and `CustomLog` are nothing to worry about. We set them only to keep Apache from complaining when running a configuration test. They can be useful for advanced users to troubleshoot server issues by lookig at logged events. But unless you know why you would look there, just ignore them.

### 2.6) Restart Apache

Whenever you edit an Apache configuration file — no matter which one — you need to restart the Apache server for the changes to take affect. Up to this point you’ve worked with four configuration files and made a lot of changes; it’s time to reboot Apache.

Run the following restart command (which does require `sudo`). If the server was off for any reason, this will also start it:

```
sudo apachectl restart
```

You’ll see a confirmation at the command-line that Apache was restarted successfully.

If your Apache server should ever turn off, you can start it again by running:

```
sudo apachectl start
```

Likewise, you can stop the Apache server with this command, though there’s no reason you should ever need to:

```
sudo apachectl stop
```

***

Theoretically speaking, you have now set up your Apache web server. Quickly check to make sure it’s more than just theory by going to the following link in your web browser (replacing “username” with your actual username):

http://localhost/~username/

You should see the default _index.html_ file in your new DocumentRoot location (i.e. _~/Sites_) where a proper congratulations awaits you for attaining this milestone.

But don’t linger, because there a few more things to do yet.

### 2.7) Apache configuration test

If you’re web server is working, by evidence of the default file check you just did, then you don’t have to worry about a configuration test too much. But it’s useful for finding any configuration discrepencies, whether critical or not.

First, run this command to output a report:

```
apachectl configtest
```

If you don’t have any problems, all you should get back is `Syntax OK` and you’re done with the test — and officially done with configuring Apache. /Applause!/

Hopefully this will be your situation, because by setting the `ServerAdmin` and `ServerName` values in the main _httpd.conf_ file, adding the `ErroLog` and `CustomLog` paths in the _httpd-vhosts.conf_ file, and commenting out the VirtualHost template block in the same file, you will avoid seeing some common config test errors that people often miss when setting up their AMP environments. 

I have no clue how many possible problems could be reported in a server configuration test (I suspect it could be a lot), but I did run into a few in the past, which I’ve accounted for in these instructions, as mentioned. You should not see those. 

Based on my experience, I can at least say the errors are generally helpful in figuring out where problems lie. It’s also the case that by fixing one problem you potentially eliminate multiple errors, depending on the nature of the problem, so if you can’t figure out one error, skip to the next and so on before killing yourself over a given one.

And keep in mind that we’re talking about your local web server here that only you have access to. You’re not administering the corporate network, or using your machine to host websites online. So if your test _index.html_ file renders okay but you still see some enigmatic error in your config test report, it’s probably nothing to lose sleep over.    

Remember that if you make changes to any Apache configuration files, you must restart the server as described in the previous section.

## 3) Mac _hosts_ file

The _hosts_ file has nothing to do with AMP software components; it’s an important file in your Mac’s filesystem. It must be used in combination with Apache if you plan to run VirtualHost directories, which is the assumption in these instructions, even if you never do. For each virtual directory you setup in the _httpd-vhosts.conf_ configuration file, you need to add the corresponding server name in the _hosts_ file too.

Open the hosts file in Nano. You’ll need to use `sudo` in the command, and you’ll be asked for your password becaue this is a system file:

```
sudo nano -e /private/etc/hosts
```

If you’ve never edited this file before, you should see something like this inside:

```
##
# Host Database
#
# localhost is used to configure the loopback interface
# when the system is booting.  Do not change this entry.
##
127.0.0.1       dev
127.0.0.1       localhost
255.255.255.255 broadcasthost
::1             localhost
```

Never edit any of those lines!

Use the down-arrow key to move the Nano cursor to the bottom of the file on a new line. If you have to use the right-arrow key to move to the end of the last line first then, so bit. Just make sure you have a couple line breaks at the bottom of the file without altering the lines already in the file.

Then copy (**Cmd+C**) and paste (**Cmd+V**) the following content into the bottom of file on the last line:

```
##
# My projects
127.0.0.1 domain1.dev
#127.0.0.1 domain2.dev
```

Then, as usual when using Nano, hit **Ctrl+X** keys to exit Nano edit mode, click **Y** when asked to save changes, and hit **Enter** key to return to the command-line.

You’ve simply added a few lines representing your virtual project domains. Remember that we set up two placeholder domains in the _httpd-vhosts.conf_ file and gave them the temporary server names _domain1.dev_ and _domain2.dev_. Those are the same server names that need used here.

Note that all of the lines except for _domain1.dev_ are commented out. The first two lines should _always_ be commented out; they’re just serving as a section header in the file. But you would need to uncomment your domain lines when ready to use them. I’ve left the _domain1.dev_ line active so you can test the web domain after configuring PHP, which you will do in the next section.

Finally, whenever you edit the _hosts_ file, you must flush the Directory Service cache afterward. Do this now by running the following command: 

```
dscacheutil -flushcache
```

Good job! Let’s dive into PHP.

## 4) PHP setup



### 4.1) upgrades and switching

MacOS Sierra ships with PHP 5.6.3. Regardless, you can check what version you have by running the following command: 

```
php -v`
```

We can gripe that Apple didn’t provide PHP 7 with Sierra, especially as PHP 7 was available before Sierra’s release, but we can’t rightly say that PHP 5.6 is outdated. In fact, because PHP 5.6 is the last of the “5” line, [security support for 5.6](http://php.net/supported-versions.php) will end at the same time as for PHP 7, in Decemember 2018. At that time, presumably, both PHP 5.6 and 7 will be end of life. So maybe Apple knows what’s it doing.

Outside of Apple-land, however, many of us need to use a flavor of PHP 7 because other software we might use (e.g. CMSes) functionally rely on it. So we must upgrade before Apple gets around to it, and if the PHP end-of-life dates are anything to guess by, Apple may take its sweet time. 

There are different PHP sources you could use to upgrade, such as the basic [Mac package from php.net](http://php.net/manual/en/install.macosx.php), the attractive [package from Homebrew](https://github.com/Homebrew/homebrew-php), the well-maintained [package from Liip](https://php-osx.liip.ch), and a few others. I describing usig Liip’s package, because that’s what I use.

Just so you know, I’m a bit of a lazy puritan. I don’t mind using Mac’s built-in version, and I don’t live to use the latest and greatest releases of software. As long as PHP is compatible with other applications I use and isn’t a radical security risk, I’m inclined to leave a sleeping dog alone. I only get around to upgrading when it’s functionally necessary. 

And that’s what happened… I use a particular CMS that evolved its core and now requires PHP 7 as a result. If I want to keep with maintaining my local websites using this CMS, I need to upgrade my PHP. No big deal. 

Liip installs its PHP packages to the side of Apple’s own binaries so the two versions don’t interfere with each other. That’s convenient for puritan’s like me, because with Liip’s packages to the side, I can switch back and forth between built-in and Liip as my slow upkeep pace desires. Plus Apple is doing half the work here, so even better still.

Of course, you don’t have to be a lazy switcher like me. You could simply use Liip forever more and continue to updgrade the second a new release is available. Whatever floats your boat.

### Installing PHP from Liip

What I recant here is basically a rendition of what you can [read from Liip’s own lips](https://php-osx.liip.ch). But English is obviously not the Swiss company’s native language (or maybe they need a tech writer), so my version might be easier to follow.

**Step 1:** Run the following command to download the _latest_ “current stable” version. (Edit the version number in the command accordingly if not already reflected here.) Do not use the “release candidate” version, which is not meant for production-grade development:

```
curl -s https://php-osx.liip.ch/install.sh | bash -s 7.0
```

This will install Liip’s files at the following locations. Do not be concerned about the “php5” in the paths, and do not alter path locations:

* Liip “packager” files: _/usr/local/packer_
* Liip PHP install: _/usr/local/php5_
* Liip’s new PHP binary: _/usr/local/php5/bin/php_
* Liip php.ini: _/usr/local/php5/php.d/99-liip-developer.ini_

By comparison, here’s where Apple’s native PHP files are located (exluding a “packager”). Leave these alone:

* PHP 5.6.3 install: _/usr/local/php5_
* The PHP binary: _/usr/local/php5/bin/php_
* php.ini: _/usr/local/php5/php.d/99-liip-developer.ini_


**Step 2:** Run the following command to open a new _.profile_ file under your user directory:

```
nano ~/.profile
```

If for some mystical reason you already had this file, the command will simply open it in the nano command-line editor. But it’s probably an empty file. 

Paste this line into the file (at bottom of file if there’s something already in there): 

```
export PATH=/usr/local/php5/bin:$PATH
```

Then hit Ctrl+X on your keyboard. Then hit the “Y” key to accept changes and exit.

This step does not influence the ability of the new PHP install to function. It simply takes care of a little suprise. If you didn’t do this step, the `php -v` command would output info for the older Apple binary install, and you might be confused by that feedback. By adding the new path in your user _.profile_ file, you make the command look in the right place for the version information. 

If/when you ever switch back to Apple’s binaries, simply remove the line or delete the file.

**Step 3:** 

If run `php -v` again to check your PHP version 

In _/etc/php.ini.default_ and _/etc/php.ini_, you need to set the time string to make local Textpattern install work without errors. Look for this string around line 910:

```
; date.timezone =
```

Uncomment and change it to:

```
date.timezone = Europe/Paris
```

## 5) Installing and updating MySQL

(Forthcoming.)

I’m a little sad about that decision, though, because I’m what you might call a puritan when it comes to software on my machine. Regardless of whether I have lots of space or not, I don’t like installing more applications than are necessary. There are exceptions, but I generally try to keep things “lightweight”. In this case, if Apple wants to give me a perfectly good database to use, I might use it and save myself the headache of installing something different. But the logic isn’t always that easy.

## 6) Configuration backup files

Odds are good that every time you upgrade your Mac's operating system to a new major release, your web environment will be thrown out of whack and not work. This happens because Apple upgrades the built-in software, which in turn overwrites the configuration files you modified with new built-in defaults. Generally this doesn’t effect any files you created, only the files you modified.  

This will not effect your MySQL database because Apple doesn’t provide one. Nor will it effect any version of PHP that you’ve installed to the side of Apple’s own built-in version, as talked about earlier. You should backup any PHP version when using Mac’s built-in binaries, however.

Although these instructions will be available to walk through configurations manually again, if needed, you can save some time by making backups of the configuration files, then revert to those files again after the macOS upgrade. The convention for backing up configuration files is to duplicate them in the same directory they belong, but with an arbitrary _.backup_ extension tacked on. The ‘copy file’ (`cp`) utility can do this easily. 

In the next sections are the configuration files you should backup. In each case, the list items are individual commands to run. Copy/paste the commands into the command-line one at a time and run them in series.

**Apache backup files:**

Note you need to change `<userame>` to your actual username in the fourth item.

1. `cp /etc/apache2/httpd.conf /etc/apache2/httpd.conf.backup`
2. `cp /private/etc/apache2/extra/httpd-userdir.conf /private/etc/apache2/extra/httpd-userdir.conf.backup`
3. `cp /private/etc/apache2/extra/httpd-vhosts.conf /private/etc/apache2/extra/httpd-vhosts.conf.backup`
4. `cp /etc/apache2/users/<username>.conf /etc/apache2/users/<username>.conf.backup`

**PHP backup files:**

(Forthcoming.)


## 7) Local website demonstration

(Forthcoming.)

# Appendix

Useful references.

## A) Find versions of stack components

Knowing what versions of software you have installed can be useful when making decisions about software compatibility and when to upgrade components. Versions for the built-in APP components will depend on which version of macOS you have and what the last Apple update may have provided. 

A default install of macOS Sierra 10.12 would provide: 

* Apache 2.4.23 
* PHP 5.6.3
* PostgreSQL 9.3.5

Those versions could change over time, however, and you might not keep track of them. You can easily find version information by running simple commands in Terminal.

### A.1) Apache version

To verify your Apache version, run either of the following commands, no need to run both:

```
apachectl -v
```

```
httpd -v
```

Either way, you’ll get output showing the Apache version and when the server was built:  

```
Server version: Apache/2.4.23 (Unix)
Server built:   Aug  8 2016 16:31:34
```

### A.2) PHP version

To verify your PHP version, run:

```
php -v
```

You’ll get output that begins with a line like this, which is the line you’re interested in, in fact:

```
PHP 7.0.7 (cli) (built: May 26 2016 16:01:05) ( NTS )
```

The output shown here is not reflecting PHP 5.6.3 that’s built-in with Sierra, but rather a newer version installed to the side that Apache is using.

### A.3) PostgreSQL version

These instructions didn’t describe using the PostgreSQL database, but you can run this command anyway to verify your version. In fact, there are two commands you could run…

This one outputs the PostgreSQL _server_ version:

```
postgres -V
```

And this one outputs the PostgreSQL _client_ version:

```
psql -V
```

You will likely see the same version number in each case, but there is a difference between the server and client components of a database, though nothing you need to be converned with here.

### A.4) MySQL version

Once MySQL is installed, you can run the following command, which uses the username (`-u`), password (`-p`), and version (`-V`) options. You’ll need to supply the _root_ user account password when asked:

``` 
mysql -uroot -p -V
```

You’ll get output similar to this:

```
mysql  Ver 14.14 Distrib 5.7.13, for osx10.11 (x86_64) using  EditLine wrapper
```

The output is a little confusing. In this example, the version number is **5.7.13**, not 14.14 as it would seem to indicate. Also the indication of “for osx10.11” (i.e. El Capitan) is irrelevant, even though were talking about Sierra 10.12. That simply reflects what version of OS was running when the database was installed (in this case via Homebrew). That information would change and reflect the new OS when the database was upgraded again.