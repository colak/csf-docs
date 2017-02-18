# Flarum: local install and development

Our [Content Strategy Discussion](https://discussion.csf.commuity) boards are powered by [Flarum](https://discuss.flarum.org/). Flarum requires [Composer](https://getcomposer.org) to install and update the core while in beta development, as well to install and update extensions. Likewise, there are some particular steps to be aware of in the dev-to-production workflow. This doc outlines getting setup on local development. See also **Flarum: local-to-production workflow**.

## Assumptions

These instructions make a couple of assumptions:

1. That you already have a local AMP stack environment setup and working, and that you know how to clone a GitHub repository to your local machine. Both are out of scope here, but you need to have them in place before continuing. 
2. That you’re using a Mac operating system, and that your development root is at _/Users/username/Sites/discussion_, where "username" is your actual username, and “discussion” is the Flarum install directory. But you could adjust for whatever operating system you use.

If you’re good to go, let’s hit it!

## Installing Composer

Flarum makes use of Composer, and may continue to do so into the future. If you want to fiddle with Flarum, you’ll need to install Composer.

First use [Composer’s download commands](https://getcomposer.org/download/), then use [Composer's OS X install instructions](https://getcomposer.org/doc/00-intro.md#installation-linux-unix-osx) for your local setup. 

**Recommended:** Go the additional mile to make Composer available globally by moving it into your local path:

`mv composer.phar /usr/local/bin/composer`

Now Composer is ready. You'll use it to update Flarum core files and extensions when any updates are available.

## Installing Flarum

While you can install Flarum from Flarum's source as described below, it's advised you simply clone the [csf-discussion repository](https://github.com/content-strategy-forum/csf-discussion) instead since working with that repo will be necessary anyway. But we'll walk through installing from source to be thorough.

To install from source, Use Terminal to get into the local directory where you want to install the Flarum app. (In these instructions we'll use the directory at _/Users/username/Sites/discussion_). Then, using [Flarum's install instructions](http://flarum.org/docs/installation/), run this command:

```
composer create-project flarum/flarum . --stability=beta
```

**Attention:** The above command is for the beta 5 release. Ensure the command is correct for any later version by verifying from Flarum’s install instructions!

If you want email notifications working locally, you'll need to configure that, as the Flarum instructions go on to explain. But since email notifications are already working on CSF's production server, you can skip this. (You would need to do it if you ran Flarum for a different project, of course.)

## Updating Flarum

With respect to Composer, updating Flarum can mean two things:

1. Updating the existing version with same version (which will update all installed extension at once too).
1. Updating to the next/latest version. 

### Updating existing version and extensions

To update the existing build version of Flarum, tunnel into the host server, change into the install directory, and run:

```
composer update
```

### Updating to new flarum version

Should an upgrade version require a different install command, it will be made clear at time of availability.

### Installing Flarum extensions individually

Again, tunnel into host server and change into install directory. Then run:

```
composer require {dev}/flarum-ext-{name}
```

...where `{dev}` is the developers nickname, and `{name}` is the name of the extension. For example, one extension CSF has installed is s9e’s extension called _mediaembed_, which is installed (and later udpated) using: 

```
composer require s9e/flarum-ext-mediaembed
```

