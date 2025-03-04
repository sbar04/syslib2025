# Week 8: LAMP Documentation Recap

Over the past few weeks, I have been installing and configuring the software that makes up a LAMP stack. A LAMP stack is a combination of:

* __L__ inux (operating system)
* __A__ pache2 (web server)
* __M__ ySQL (relational database management system)
* __P__ HP (scripting language that helps Apache2 and MySQL interact)

These components are all free to use and open-source.

This stack of software allows for the creation of a web server (via Apache2), with extra functionality through PHP and MySQL. 

Apache alone can serve static web pages from files in the **root directory**, and PHP and MySQL can add in dynamic web content.

All of these are essential for running a content management system (CMS) and integrated library system (ILS) which will come later this semester. 

## Installation of Apache2
Before installing anything, update the Linux machine:
```
sudo apt update
sudo apt upgrade
sudo apt autoremove
sudo apt clean
```

Searched for the correct software:
```
apt search apache2
apt show apache2
```
Confirmed this was correct and used the `sudo` command to install it:
```
sudo apt install apache2
```
And finally, checked the status with:
```
systemctl status apache2
```
This told me `apache2` is **Active** (running and live!), **Loaded**, and **Enabled** (meaning it starts automatically on reboot). All good.

The Apache2 Ubuntu Default Page provides a configuration overview and tell us the default **document root** is `/var/www`. This is where Apache2 will look to serve content to the web.

The `index.html` file acts as the default page of the website (`ExternalIP`). I made a backup of the original `index.html` file to preserve it as `index.html.orginal` which still shows the Apache2 Ubuntu Default Page which provides a configuration overview. I edited the new `index.html` file to show some basic HTML formatting for my first web page. Refreshing my browser tab with the `ExternalIP` address then showed that new HTML formatting as a web page. Success!

## Installation of PHP

PHP is a server side programming language that I had to install and configure to work with Apache2. 

I updated my machine, then used the following commands:
```
sudo apt install php libapache2-mod-php
```
*This installed the php and the libapache2-mod-php packages. The second one creates a connection between PHP and Apache2.*

Then I restarted Apache2:
```
sudo systemctl restart apache2
```
I confirmed the version with `php -v`:
```
PHP 8.1.2-1ubuntu2.20 (cli) (built: Dec  3 2024 20:14:35) (NTS)
Copyright (c) The PHP Group
Zend Engine v4.1.2, Copyright (c) Zend Technologies
    with Zend OPcache v8.1.2-1ubuntu2.20, Copyright (c), by Zend Technologies
```

Finally, I checked Apache2's status in case of any errors:
```
systemctl status apache2
```
*Still loaded and active. All good.*

### Checking PHP and Apache Connection
To check that PHP has been installed and that it is working with Apache 2, I created a PHP file in the web document root. 

```
cd /var/www/html/
sudo nano info.php
```
and then added the following:
```
<?php
phpinfo();
?>
```

Using Firefox, I checked this by visiting `http://MyExternaIP/info.php`.  It had the expected information about PHP, Apache2, the server, and their configurations. 

I removed this to not expose all of my server information:
```
sudo rm info.php
```

### Configuring Apache for PHP

I changed the default document that Apache looks for from `index.html` to `index.php` by editing the `dir.conf` file:

```
cd /etc/apache2/mods-enables
sudo cp dir.conf dir.conf.bak
sudo nano dir.conf
```
*This also made a backup file of the original configuration file.*

I edited the new `dir.conf` file to have `index.php` first in line as the default file Apache will search for to display:
```
DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
```
I tested this with:
```
apachectl configtest
```
Syntax Ok means it is all good! No typing mistakes that will break the application.

And then applied the changes. reloaded the Apache2 configuration, restarted the service, and checked Apache2's status:
```
sudo systemctl reload apache2
sudo systemctl restart apache2
sudo systemctl status apache2
```
### Created index.php File
I went back to the document root directory `/var/www/html/` to create an `index.php` file that now acts as the default page of my website (per the configuration above) when I go back to my IP address page. 


PHP is processed by the `php` program and Apache, and then converted into HTML (depending on some other conditions). 

I added the textbook supplied Browser & OS Detection HTML/PHP to the `index.php` file. 

After saving this file, when I reloaded my external IP in the address bar, the Browser & OS Detection was displayed as expected. 

Can visit my `externalIP/index.html` to see that HTML file displayed still, and `externalIP/index.html.original` to see the Apache2 Default Page.


## Installation of MySQL

Checked the version that would download:
```
apt policy mysql-server
```
Installed the metapackage:
```
sudo apt install mysql-server
```
Double checked the version I installed:
```
mysql --version
```
*which output:*
```
mysql  Ver 8.0.41-0ubuntu0.22.04.1 for Linux on x86_64 ((Ubuntu))
```
The installation should have started and enabled the database server:
```systemctl status mysql```
*Still Loaded and Active!*

Ran the post installation script to perform security checks and create the baseline configuration of MySQL:
```
sudo mysql_secure_installation
```
This script prompted me for several things:
```
Validate passwords: Y
Password validation policy: 0 (zero) for LOW
Remove anonymous users: Y
Disallow root login remotely: Y
Remove test database and access to it: Y
Reload privilege tables now: Y
```
*I chose to validate passwords, but with a weak requirement just for this project.*

I logged into MySQL with the `root` user:
```
sudo mysql -u root
```
And created a new user named `opacuser` with a PASSWORD:
```
mysql> create user 'opacuser'@'locaclhost' identified by 'PASSWORD';
```
And created a database named `opacdb` with coding set to UTF-8:
```
mysql> create database opacdb default character set utf8mb4 collate utf8mb4_unicode_ci;
```
And granted `opacuser` `all privileges` on the `opacdb` database:
```
mysql> grant all privileges on opacdb.* to 'opacuser'@'localhost' with grant option;
```
I logged out of the `root` MySQL account:
```
mysql> \q
```
Back in the `bash` shell in my home directory, I edited the `.bashrc` file to edit the appearance of the MySQL command line:
```
nano .bashrc
```
And added the following to the end of the file:
```
export MYSQL_PS1="[\d]> "
```
And then sourced this to actually execute the command:
```
source ~/.bashrc
```
This changes the prompt line in MySQL from the default `mysql>` to show the database I am working in.

Logging back in to MySQL as the `opacuser` with the `-p` prompt for password, I can `show databases` and successfully see the `opacdb`. As the `opacuser`, I am able to create a table in `opacdb`, add records to the table, retrieve those records or parts of those records, alter the table structure, add data to new fields, and delete and add records. Success!

## Connecting PHP and SQL
First I installed PHP and MySQL support:
```
sudo apt install php-mysql php-mysqli
```
I restarted Apache2 and MySQL so they recognize they are connected:
```
sudo systemctl restart apache2
sudo systemctl restart mysql
```
I then had to have PHP authenticate itself before connecting to MySQL. I did this by creating a `login.php` file in the document root's parent directory:
```
cd /var/www
sudo touch login.php
```
I changed the permissions for this `login.php` file:
```
sudo chmod 640 login.php
```
*This allows the user owner to read and write, the group owner to read only, and no permissions for other/world.*

I then changed the group ownership to `www-data`:
```
sudo chown :www-data login.php
```
Modifying the file ownership and permission, as well as placing in the `/var/www` directory, the `login.php` file is not accessible to web users which is necessary because I added the following login information for the `opacuser` in this file: 
``` 
<?php // login.php
$db_hostname = "localhost";
$db_database = "opacdb";
$db_username = "opacuser";
$db_password = "PASSWORD";
?>
```
### Creating a PHP File for Website
I added a file called `opac.php` to my Apache2 root directory. This PHP file has HTML formatting, but also involves PHP interacting with the MySQL `opacdb` database.
```
cd /var/www/html
sudo nano opac.php
```
In this file, I re-typed the text provided in the textbook, saved and exited the file.

I tested the syntax of the `login.php` with the `sudo php -f` command:
```
sudo php -f /var/www/login.php
```
*This gave me no output, which means no syntax errors since it is all PHP.*

I ran the same for the `opac.php` file:
```
sudo php -f /var/www/html/opac.php
```
Which output the HTML of the file.

I also tested this in my web browser `ExternalIP/opac.php` and saw the results of the script as expected!

## Takeaways
I didn't run into almost any issues with these installations and configurations. I had a few syntax errors while creating a table in MySQL that were easy to spot and fix. 

I also had some issues when I tried logging in to MySQL for the first time as the `opacuser` because the program does not indicate typing in the password field. I assumed I had set up the user incorrectly, and logged back in as the `root` user to see the permissions I had set for the `opacuser`. Nothing was wrong, I just needed to log out and trust that the password prompt was taking my input.
