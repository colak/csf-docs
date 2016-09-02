# Flarum: local-to-production workflow

If you haven’t setup your local development for Flarum yet, see **Flarum: local development**.

Otherwise, this doc covers the workflow for updating production with your local development efforts.

All CSF workflow for web applications involves:

* your local dev setup (if you want to help contribute)
* CFS's associated GitHub repo (in this case, csf-discussion)
* The production location (in this case, discussion boards) 

The development workflow is (though maybe not without some kinks):

1. Update your local discussion database with production export file. 
1. Make code changes locally (including Composer installation/update of any new Flarum extensions).
1. Sync changes with the repo.
1. Freeze production site to new accounts and posts to finish remaining steps.
1. Update production with new local database (may not be necessary depending on the kinds of dev changes made and/or your last database update).
1. SFTP the necessary dev files to production (either static files under _/custom_ directory and the composer.lock file).
1. Run Composer to update all needed package components.
1. Unfreeze production site for normal use.

For those of you who work with Textpattern, most of this should seem clear enough. The Composer parts, however, might be new to you. Let's consider each step.

## 1) Update local database

An admin will need to export the production database and make the file available to anyone in web team who needs it.
You, if playing the game, import the database into your local MySQL server. Either via command-line or the [Sequal Pro](http://www.sequelpro.com) OS X app is highly recommended.

## 2) Make local dev changes

Dev changes will mainly be one of two things, or both: 

1. Install/update any desired Flarum extensions. (This should be by  admin concensus, not personal whim.)
1. Make any needed edits to UI in relation to extensions, or just UI evolutions in general via the presentation overrides found in Flarum’s admin-side.

Installing Flarum extensions was explained in **Flarum: local development**.

Editing Flarum presentation needs done in the admin-side of the Flarum front-end by anyone having "Admin" status in the boards (one of the user groups). For different reasons, assigned admins will be:

* CSF = does whatever it wants
* Destry = CSF
* Kevin = presentation tweaks, if interested enough

Flarum users, including administrators, don't actually have access to the core templates or CSS files, rather we are given a "Customize CSS" window to override CSS rules. This is found in **Administration** > **Appearance** > **Customize CSS**. That's where most of our presentation overrides for CSF's brand design are made. It's the same idea as working with Textpattern's Styles panel (if you’re familiar with Textpattern), except in Flarum you’re overriding the base CSS (written with LESS), not changing it. You basically need to use a browser's "Inspect source" tools to find the precise selectors to target, then override as best you can.

In the future there may be theming extension, or whatever, but I don't think we'll ever really have full control over Flarum presentation front-end. In the meantime, we use the override feature and any related UI extensions available to us.

**Important:** Whenever the override presentation rules are modified, a copy of the rules (top to bottom) should be made as a static file and added to the csf-discussion repo in the [overrides](https://github.com/content-strategy-forum/csf-discussion/tree/master/custom/css/overrides) directory. Use the same name but give it the latest number in sequence.

## 3) Sync changes with repo

Nearly all changes you’ll need to sync will be of three kind:

1. Static CSS and image files you add or remove under the [/custom](https://github.com/content-strategy-forum/csf-discussion/tree/master/custom) directory.
2. New extensions, installed via Composer and defined in the  _composer.lock_ file (extension files get added under the /vendors directory by developer name). See more on the _composer.lock_ file in #6.
3. The _composer.lock_ file, always.

You may also find there are a number of cache files that get generated over time and GitHub will expect to sync those too. You do not have to sync these, in fact they can be removed from your local install. (Indicate where to look for these.)
  
One thing needing done here is to ensure the _.gitignore_ file is configured to keep needless and senstive stuff out of the repo (thus out of public view). Sensitive files would include the _config.php_ file, and the production and development versions of the _.htaccess_ file (i.e. not the Flarum default file). Needless files might be the cache files mentioned above. (See if the cache files can be handled. Check back on it.)

## 4) Freeze the production site

While it’s unlikely to be a problem for discussions boards with little activity, updating production can interfere with any accounts being created at that time, or if someone was trying post a new discussion, etc. To avoid this mishap, we can do two things:

1. Temporarily turn off allowing new accounts in admin-side settings.
2. Temporarily swap in an _index.html_ file (in place of the default index.php file) to serve as a maintenance page.

When production is updated, these two actions are reversed, see step #8.

## 5) Update production CSS overrides with local changes

This has to be a manual copy/paste process from local admin-side to production admin-side.

## 6) SFTP dev files to production

When updating production, the only thing that really needs to be transferred over are:

1. The _composer.lock_ file (and especially if any extensions were installed in local).
2. The /custom directory (if anything in there changes, such as new overriding CSS file or images add and/or removed).
3. (Other? Don’t think so.)

## 7) Update Composer in production

Destry will need to do this step!

This step is necessary to update all dependencies that may have change regarding any new extensions. This is actually where the power of Composer comes in and why it needs installed on the production web server (which it is), because instead of having to upload all changed files under the /vendors directory via SFTP from local, you simply make your extensions updates on local, move the _composer.lock_ to production, and run the Composer update on production via the command-line (an SSH connection). This will automatically update the production install with all Flarum extensions and dependencies defined in the _.lock_ file. Sweet! 

Specifically, the steps are:

1. ssh tunnel into WebFaction remote server
1. get into the _webapp/discussion_ directory
1. run the following Composer command:
`composer install --no-dev --prefer-dist`

That command will read the .lock file and use it to auto-update all extensions on production that were installed on local dev.

## 8) Unfreeze the production site 

The two actions in step 4 need undone now so people can use the boards again.
 