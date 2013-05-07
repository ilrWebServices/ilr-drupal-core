# ilr-drupal-core

The Drupal code behind ILR's public web site.

## Acquia Cloud Hosting

The directory structure of this repo follows that required by Acquia Cloud Hosting. Acquia files and directories are in the root, including the directory docroot, which is the root directory of a standard Drupal install.

### Setting up this repo in your working environment.

#### Cloning the repo

Say you have a local web server running at http://192.168.1.1 and you want to make this Drupal distro available there. Follow these step.

First, clone the repo as you usually would into a working directory within your web server's document root. For example, to clone to a directory named ilrdrupal, open a terminal and type:

    $ cd /var/www
    $ git clone git@github.com:ilrWebServices/ilr-drupal-core.git ilrdrupal

This places Drupal's root directory at %webroot%/ilrdrupal/docroot

To see how things look so far, open a browser and go to http://192.168.1.1/ilrdrupal/docroot

Because you haven't set up a database yet, Drupal will attempt to load its install page, but will fail with the following message:

> There was a problem initializing Drupal. Drupal can't load because database settings have not been provided. 
> Please create a database settings file at /var/www/site-php/ilr/first/settings-php.inc and try again.

#### Creating the setting-php.inc file

Acquia Cloud Hosting has come up with a cutomized strategy for managing database conenction information in the Drupal setting.php file so that the same code base can be deployed to three servers: dev, stagin, and production. 

Instead of including the database information within Drupal's setting.php file itself, it adds code to the setting.php file so that the correct database connection information is loaded dynamically based on include files that sit outside of the web root. Usually %DRUPAL_ROOT%/sites/*/settings*.php is added to the .gitignore files so that those settings.php files won't be included in the repo, since by default they contain sensitive information. In the ilr-drupal-core repo, docroot/sites/defualt/settings.php is not in the .gitignore file because it *has to be* part of the repo.

So, for every copy of the repo that you have running in your local environment, you will need a corresponding include file from which settings.php gets its database connetion information. These need to be stored in a specific location in a directory named site-php, which also contains the code required by settings.php to find your included file.

The quickest way to get up and running is to clone the small repository that has a fresh copy of this site-php and save the clone in your web root (i.e., /var/www). To do all this, type this in your terminal:

    $ cd /var/www
    $ git clone git@github.com:ilrWebServices/site-php.git site-php

Once you've done that, follow these steps to finish the installation of ilrdrupal:

1. On your seb server, create a new mySql database and note the database name and the 
   username and password of a user with full rights to that database.

2.  Create the settings-php.inc file for your new site in a subdirectroy of /var/www/site-php/ilr that has 
    the same path (minus the trailing /docroot) as your Drupal root directory. For example, in our case this would be /var/www/site-php/ilr/ilrdrupal/settings-php.inc. There is a blank default version of the settings-php.inc file at /var/www/site-php/ilr/default-settings-php.inc.

    So, you could do all of that from a terminal with:

        $ cd /var/www/site-php/ilr
        $ mkdir ilrdrupal
        $ cp default-settings-php.inc ilrdrupal/settings-php.inc

3.  Edit /var/www/site-php/ilr/ilrdrupal/settings-php.inc and fill in your database information. 
    For example, if you made a database named 'drupal' with a user named 'drupal_user' and a password 
    of 'secret', then your settings-php.inc file should end up looking like this:

        <?php
        $databases = array (
          'default' => 
          array (
            'default' => 
            array (
              'database' => 'drupal',
              'username' => 'drupal_user',
              'password' => 'secret',
              'host' => 'localhost',
              'port' => '',
              'driver' => 'mysql',
              'prefix' => '',
            ),
          ),
        );

4.  Go back to http://192.168.1.1/ilrdrupal/docroot in your browser. You should now be able to complete the Drupal install.

#### Shorter URLs

If http://192.168.1.1/ilrdrupal/docroot seems like a lot to type and look at, you can always create an alias in Apache and this will all still work. For example, if to create an alias so that the new address was http://192.168.1.1/mydrupal, add the following within the <VirtualHost> block of /etc/apache2/apache2.conf (or to a file included by apache2.conf. On our default Vagrant virtual boxes this would be /etc/apache2/sites-enabled/000-default)

    Alias /mydrupal "/var/www/ilrdrupal/docroot"

Then restart Apache

    $ sudo /etc/init.d/apache2 restart

Go back to http://192.168.1.1/mydrupal and everything should work.
