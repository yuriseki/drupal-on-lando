# Setting up a local Drupal environment with Lando 
*by Yuri Seki*

Developing or maintaining a Drupal 10 website requires a reliable and efficient local development environment.  so developers can test and debug code without affecting the live website.

Lando is a free, open-source tool that provides an easy-to-use local development environment for Drupal and other websites. It’s flexible and compatible with other tools like a PHP version independent of your operating system, Composer, and Xdebug. It simplifies the process of setting up and maintaining a local environment, making it an excellent tool for both novice and experienced developers.

This article is intended for anyone who is involved in building and maintaining a Drupal website. Whether you are a developer, site builder, or a project manager, this guide will help you set up a local development environment with Lando, Composer, and Xdebug.

We’ll cover the following:
1. Installing Lando
2. Creating a Lando configuration file
3. Configuring Lando
4. Using Composer and Drush to install our Drupal website
5. Configuring Xdebug

By following these steps, you should have a fully functional local environment for Drupal 10 development, allowing you to focus on building and maintaining your website with confidence.

## How to set up your local environment
### Step 1: Install Lando
Lando is a free and open-source tool that provides an easy-to-use local development environment for Drupal and other websites. To install Lando, you need to have Docker installed on your computer. You can follow the Lando installation from the official documentation at [https://docs.lando.dev/getting-started/installation.html](https://docs.lando.dev/getting-started/installation.html)

### Step 2:  Create a Lando configuration file
Once you have Lando installed on your computer, you can use the command “lando init” to start the guided process of setting up your Drupal 10 environment:

```bash
mkdir drupal10 && cd drupal10

lando init
? From where should we get your app's codebase? 
❯ current working directory 
```
Select the option “current working directory” in order to use the current directory you are in.

```bash
? What recipe do you want to use? 
❯ drupal10
```
Select the option “Drupal10.” 

```bash
? Where is your webroot relative to the init destination? (.) web
```
As a good practice, I like to use the folder “web” to the Drupal installation, so then, type “web.”

```bash
? What do you want to call this app? my-drupal10
```
Provide a name for your application. In this example I’m using “my-drupal10.”

Lando will now generate a file named .lando.yml with the following content:
```yaml
name: my-drupal10
recipe: drupal10
config:
  webroot: web
```


### Step 3: Configure Lando

Though not required, additional configuration is possible to fine-tune your Drupal 10 environment. Update your .lando.yml file to add the PHP version, and the site URL for Drush.

```YAML
name: my-drupal10
recipe: drupal10
config:
  webroot: web
  php: 8.1 # Pin a specific version of PHP
  via: nginx # Use Nginx instead of Apache
  database: mariadb # Use Mariadb instead of MySQL

tooling:
  # Override the default "lando drush" command so that the
  # local website URL is passed to it. This is useful when
  # generating login links.
  drush:
    service: appserver
    env:
      DRUSH_OPTIONS_URI: "https://my-drupal10.lndo.site"
```

Now, with your customized configuration in place, start your Drupal 10 environment by typing "lando start" on your terminal.
```bash
lando start
```

### Step 4: Use Composer and Drush to install our Drupal website

The "drupal10" Lando recipe that we're using already has Composer 2 installed, so let’s use the built-in Composer command to create our Drupal 10 installation.

```bash
# Creating a drupal 10 project using composer.
lando composer create-project drupal/recommended-project:^10 drupal10 --stability dev --no-interaction

# Move the installation to the current folder, including the hidden files.
rsync -r --remove-source-files drupal10/ ./
rm drupal10 -rf

# Install a site local drush
lando composer require drush/drush

# Install drupal using the default database credentials
# provided by the Lando drupal10 recipe
lando drush site:install --db-url=mysql://drupal10:drupal10@database/drupal10 -y
```

Type “lando info” in order to check the configuration:
```bash
lando info
```

You should see something like:
```json
[ { service: 'appserver',
    urls:
     [ 'https://localhost:32873',
       'http://localhost:32874',
       'http://my-drupal10.lndo.site/',
       'https://my-drupal10.lndo.site/' ],
.
.
.
```

Which means that our local Drupal installation can be accessed by any of those URLs.

Open your browser and go to “https://my-drupal10.lndo.site.” 

As we’d installed Drupal via command line, a random password has been generated, so in order to login as administrator, use the Drush command:
```bash
lando drush uli
```

You’ll receive as a result something like:
```bash
https://my-drupal10.lndo.site/user/reset/1/1675734810/f8JVoS9kgCQt2zVGFWDBf5_zegk2ytCfKcPE8m8_P5s/login
```

Copy this URL, and paste it on the browser URL. You should see the admin user edit page.
![](assets/admin-user-edit-form.png)
Set a new password for the admin user.

### Step 5: Configure Xdebug
Xdebug is a PHP extension that provides debugging and profiling capabilities for PHP scripts. It helps developers find and fix errors in their code quickly and efficiently. Xdebug provides a range of features, including stack traces, variable watching, and code coverage analysis, that allow developers to gain a deeper understanding of their code and improve the quality of their work.

The use of Xdebug during local development is essential for several reasons. First, it helps developers save time and reduce frustration by enabling them to find and fix problems without having to go through the entire codebase. Second, Xdebug allows developers to gain a deeper understanding of their code and its behavior, allowing them to optimize it and make it more efficient. Finally, Xdebug provides a code coverage analysis, allowing developers to see which parts of their code are being executed and which parts are not. This information can help developers identify areas that need improvement and ensure that their code is working as expected.

In summary, Xdebug is a powerful tool for PHP developers, providing debugging and profiling capabilities that can greatly improve the quality of their work. The use of Xdebug during local development is essential for finding and fixing errors, optimizing code, and understanding code behavior. By using Xdebug, developers  can feel confident in their work and ensure that code is working as expected, making local development much smoother and more efficient.

Now, open the **.lando.yml** file and update it to appear like this:  
```YAML
name: my-drupal10
recipe: drupal10
config:
  webroot: web
  php: 8.1 # Pin a specific version of PHP
  via: nginx # Use Nginx instead of Apache
  database: mariadb # Use Mariadb instead of MySQL

services:
  appserver:
    overrides:
      environment:
        XDEBUG_MODE: 'debug' # Set the Xdebug mode option

tooling:
  # Override the default "lando drush" command so that the
  # local website URL is passed to it. This is useful when
  # generating login links.
  drush:
    service: appserver
    env:
      DRUSH_OPTIONS_URI: "https://my-drupal10.lndo.site"    
  # Add a "lando xdebug-on" command for enabling Xdebug
  xdebug-on:
    service: appserver
    description: Enable Xdebug.
    user: root
    cmd:
      - docker-php-ext-enable xdebug && kill -USR2 $(pgrep -o php-fpm) > /dev/null || /etc/init.d/apache2 reload
      - tput setaf 2 && echo "Xdebug On" && tput sgr 0 && echo
  # Add a "lando xdebug-off" command for disabling Xdebug
  xdebug-off:
    service: appserver
    description: Disable Xdebug.
    user: root
    cmd:
      - rm /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini && kill -USR2 $(pgrep -o php-fpm) > /dev/null || /etc/init.d/apache2 reload
      - tput setaf 1 && echo "Xdebug Off" && tput sgr 0 && echo
```

Save the file and rebuild your Lando environment by executing:
```bash
lando rebuild -y
```

By default,  Xdebug will be turned off. In order to switch, use **lando xdebug-on** or **lando xdebug-off**, as specified in the tooling section.

#### You’re Ready
You should now be able to start developing in Drupal because Lando has provided a reliable and efficient local development environment. With Lando, you’ll now be able to test and debug code without affecting a live website.
