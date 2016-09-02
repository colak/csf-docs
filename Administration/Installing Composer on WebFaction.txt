# Installing Composer on the WebFaction

Different web hosts might have different ticks and bugaboos to consider, but for WebFaction, our host, installation was super easy.

First, tunnel into the server via ssh. Then ensure you're in the $HOME location (i.e. account root: home/[username]). Then run the following six commands (each line is a separate command), one at a time:

1. `git clone https://github.com/composer/composer.git`
1. `git clone https://github.com/composer/composer.git`
1. `wget http://getcomposer.org/composer.phar`
1. `cd composer`
1. `php70 ../composer.phar install`
1. `find ./bin -type f -executable | xargs sed -i 's/env php$/env php70/g'`

Note we had to target which php version we wanted, which in our case was php 7.0, thus "php70" in the last two commands.

For WebFaction, this installs Composer at _/home/username/composer/bin/composer_, and anything that needs to run Composer on the server (regardless of _/webapp_ directory) can reference Composer as:

`/home/username/composer/bin/composer whatever-rapp
