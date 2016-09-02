Our discussion boards are powered by Flarum and published at https://discussion.csf.commuity.

Flarum requires Composer to install and update the core, as well to install and update extensions. Likewise, there are some particular steps to be aware of in the dev-to-production workflow. This doc quickly outlines it all.

Considering how popular Composer is as a package manager, it wouldn't hurt to install it on your local setup whether you use it for Flarum or not. At least you'll have it when you do need it. But since you will need it if you're to help with discussion boards development, let's start with installing Compoer first.

Note: these instructions assume you already have a local AMP stack environment setup and working, and that you know how to clone a GitHub repository to yoru local machine. (Local dev setup and GitHub use is out of scope here, but you would need to have both in place before bothering with these instructions.) Also, these instructions assume an OS X operating system, but you could adjust if yours is different. Likewise, these instructions assume your development root is at /Users/username/Sites (where "username" is your actual username), but you could adjust for wherever your dev root is.

Installing Composer on local
First use the download commands, then use Composer's OS X install instructions for your local setup. 

Recommended: go the additional mile to make Composer available globally by moving it into your local path:
mv composer.phar /usr/local/bin/composer
Now Composer is ready. You'll use it to update Flarum core files and extensions when any updates are available.

Installing Flarum
While you can install Flarum from Flarum's source as described below, it's advised you simply clone the csf-discussion repository instead since working with that repo will be necessary anyway. But we'll walk through installing from source to be thorough.

To install from source, fire up Terminal and get into the local directory where you want to install the Flarum app. (In these instructions we'll use /Users/username/Sites/discussion). Then, using Flarum's install instructions, run this command:
composer create-project flarum/flarum . --stability=beta

If you want email notifications working locally, you'll need to configure that, as the Flarum instructions go on to explain. But since email notifications are already working on CSF's production server, you can skip this. (You would need to do it if you ran Flarum for a different project, of course.)

Updating Flarum
With respect to Composer, updating Flarum can mean two things:

Updating the existing version with same version (which will update all installed extension at once too).
Updating to the next/latest version. 

Updating existing version and extensions


Updating to new flarum version
The process to update from one beta release to the next, or to the first stable release, might vary, so we won't know what the command is until the next version is available.

In the meantime, you can install and update extensions individually...

Installing Flarum extensions on local
Installing extensions is as simple as using Terminal to gett into the Flarum install directory, then running:
composer require {dev}/flarum-ext-{name}
...where {dev} is the developers nick, and {name} is the name of the extension. 

For example: 
composer require s9e/flarum-ext-mediaembed