# Installing Composer on the WebFaction

Different web hosts might have different requirements to consider, but for WebFaction, our host, installation was easy.

First, tunnel into the server via ssh. Then ensure you're in the $HOME location (i.e. account root: _home/username_). Then run the following six commands, one at a time:

1. `git clone https://github.com/composer/composer.git`
1. `git clone https://github.com/composer/composer.git`
1. `wget http://getcomposer.org/composer.phar`
1. `cd composer`
1. `php70 ../composer.phar install`
1. `find ./bin -type f -executable | xargs sed -i 's/env php$/env php70/g'`

Note the last two commands require targeting which PHP version you need, which in our case on WebFaction servers was PHP 7.0, thus "php70".

For WebFaction, this installs Composer at _/home/username/composer/bin/composer_, and anything that needs to run Composer on the server (regardless of which _/webapp_ directory) can reference Composer as:

`/home/username/composer/bin/composer whatever-app`
