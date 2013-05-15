# ilr-drupal-core

The Drupal code behind ILR's public web site.

## Acquia Cloud Hosting

The directory structure of this repo follows that required by Acquia Cloud Hosting. Acquia files and directories are in the root, including the directory docroot, which is the root directory of a standard Drupal install.

### Setting up this repo in your working environment.

#### Cloning the repo

Say you have a local web server running at http://33.33.33.40 and you want to make this Drupal distro available there. Follow these steps.

First, clone the repo as you usually would into a working directory within your web server's document root. For example, to clone to a directory named ilrdrupal, open a terminal and type:

    $ cd /var/www
    $ git clone git@github.com:ilrWebServices/ilr-drupal-core.git ilrdrupal

This places Drupal's root directory at %webroot%/ilrdrupal/docroot.

> **Note:** These instructions assume that you're running the terminal-based commands 
> while logged into your virtual server via an SSH session. Since Vagrant mirrors the /var/www/ directory 
> in a directory on your local machine - e.g., c:\vagrant-files\www, you could also do these commands there from a command prompt.

Before running Drupal's install script, you'll have to add one more piece to account for the way Acquia manages the database.

#### Creating the setting-php.inc file

Acquia Cloud Hosting has come up with a cutomized strategy for managing database conenction information in the Drupal settings.php file so that the same code base can be deployed to three servers: dev, staging, and production. 

Instead of including the database information within Drupal's settings.php file itself, it adds code to the settings.php file so that the correct database connection information is loaded dynamically based on include files that sit outside of the Drupal root. In a standard install of Drupal, there is a settings.php file for each site and that settings file contains sensitive information such as database credentials. Usually the line, _sites/*/settings*.php_ is added to a Drupal project's _.gitignore_ file so that those settings.php files won't be copied into the public repo. In the _ilr-drupal-core_ repo, _docroot/sites/defualt/settings.php_ is not in the .gitignore file because it **has to be** included as part of the repo and pushed to our Acquia git remote to make Acquia Hosting happy.

So, for every copy of the _ilr-drupal-core_ repo that you want running in your local environment, you will need a corresponding include file from which settings.php gets its database connection information. These include files need to be stored in a specific location in a directory named _site-php_, which also contains the code required by _settings.php_ to find your included file.

The quickest way to get up and running is to clone the _site-php_ repo from our GitHub account and save the clone in your web root (i.e., /var/www/). To do so, type this in your terminal:

    $ cd /var/www
    $ git clone git@github.com:ilrWebServices/site-php.git site-php

Once you've done that, follow these steps to finish the installation of ilrdrupal:

1. On your web server, create a new mySql database and note the database name and the 
   username and password of a user with full rights to that database.

2.  Create a new settings-php.inc file for your new site in a subdirectroy of /var/www/site-php/ilr that has 
    the same path (minus the trailing /docroot) as your Drupal root directory. For example, in our case this would be /var/www/site-php/ilr/ilrdrupal/settings-php.inc. There is a blank default version of the settings-php.inc file at /var/www/site-php/ilr/default-settings-php.inc. Start with a copy of that.

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

4.  Before saving and exiting settings-php.inc, add the following line:

        $base_url = 'http://33.33.33.40/ilrdrupal/docroot';

    This may be required by certain modules that would otherwise fail to determine the Drupal root.

5. Go  to http://33.33.33.40/ilrdrupal/docroot/install.php in your browser. You should now be able to complete the Drupal install. As with any new install of Drupal where the database connection information has been added to settings.php, you will receive a database error complaining about a missing semaphore table if you try to go to any other address on your site.

    If you receive the error:

        There was a problem initializing Drupal. Drupal can't load because database settings have not been provided. 
        Please create a database settings file at /var/www/site-php/ilr/ilrdrupal/settings-php.inc and try again.

    Then you made a typing mistake in your directory name or file name so Drupal can't find the include file at /var/www/site-php/ilr/ilrdrupal/settings-php.inc.Go fix the mistake and try again.

#### Shorter URLs

If http://33.33.33.40/ilrdrupal/docroot seems like a lot to type and look at, you can always create an alias in Apache and this will all still work. For example, to get your site running at http://33.33.33.40/mydrupal, add the following line: 

    Alias /mydrupal "/var/www/ilrdrupal/docroot"

to the \<VirtualHost> block of /etc/apache2/apache2.conf. If you're on our default Vagrant virtual boxes, then don't edit apache2.conf directly. Edit the file that apache2.conf includes: /etc/apache2/sites-enabled/000-default.

Finally, restart Apache

    $ sudo /etc/init.d/apache2 restart

Go back to http://33.33.33.40/mydrupal and everything should still work.
