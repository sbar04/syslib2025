# Week 8: Installing MySQL
Final ingredient to the LAMP stack!

![Sandwich Stacker](../Images/sandwich_stacker.jpg)

 MySQL is an open-source relational database management system. <abbr title="Structured Query Language">SQL</abbr> (Structured Query Language) is a domain specific language that manages data. Relational databases handle tables of data that share a common field, and therefore are *related*.

## Installing MySQL
Update machine:
```
sudo apt update
sudo apt upgrade
sudo apt autoremove
sudo apt clean
```
I did have to `sudo reboot now` on this installation; it also took longer than normal.

Install MySQL Community Server package. This is a **metapackage** that installs the latest and most secure version of MySQL and its dependencies.

Per the [Ubuntu help page](https://help.ubuntu.com/community/MetaPackages), metapackages are links to existing package(s) and a script that installs them:
> These packages do not contain actual software, they simply depend on other packages to be installed. This setup allows entire sets of software to be installed by selecting only the appropriate metapackage.

See the `apt show mysql-server`:
>MySQL database server (metapackage depending on the latest version). 
>This is an empty package that depends on the current "best" version of mysql-server (currently mysql-server-8.0), as determined by the MySQL maintainers. Install this package if in doubt about which MySQL version you need. That will install the version recommended by the package maintainers.

**Confirm which versions are available** with the `apt` command:
```
apt policy mysql-server
``` 

**Install MySQL Community Server metapackage**:
```
sudo apt install mysql-server
```

**Check version**:
```
mysql --version
```

*which gave me:*
```
mysql  Ver 8.0.41-0ubuntu0.22.04.1 for Linux on x86_64 ((Ubuntu))
```

The install process should start and enable the database server. **Check with `systemctl`**, looking for `Loaded` and `Active`:
`systemctl status mysql`.

**Run the post-installation script:**
`sudo mysql_secure_installation`

This performs security checks and creates a secure baseline configuration for MySQL. This script prompted me for several things:
```
Validate passwords: Y
Password validation policy: 0 (zero) for LOW
Remove anonymous users: Y
Disallow root login remotely: Y
Remove test database and access to it: Y
Reload privilege tables now: Y
```
*I chose to validate passwords, but with a weak requirement just for this project. Could have said "no" to first query to use any kind of password.*

## Connecting to the Database
Login with:
```
sudo mysql -u root
```
*Logging in my the MySQL database as the root MySQL user.*

*MySQL promts start with `mysql>`, unlike the Linux  account in the `bash` shell which is `username@computer_name:path$`. Don't type `mysql>` into the prompts, it is already there.*

List available databases:
```
mysql> show databases;
```
*Note: commands in MySQL end in semicolon.*

See four databases:
1. information_schema
2. mysql
3. performance_schema
4. sys

These are administrative databases that I don't want to touch unless something goes wrong. 

Use `^l` to clear the screen.
Use `\q` to exit out.

## Set Up a Regular User Account
The root MySQL user should be used only for special administrative cases, like the `sudo` command in the `bash` shell. For regular use cases, I need a regular MySQL user.

Log in to MySQL server as root user:
```
sudo mysql -u root
```
Create a new user:
```
mysql> create user 'opacuser'@'localhost' identified by 'PASSWORD';
```
*Replace PASSWORD with whatever password I want. It just needs to be 8 characters with no special character requirements per the LOW validation policy.*

Should get a **Query OK** message which means the user has been created without issue.

## Create a Practice Database
Stay logged in as the root user to create a new database for the user account. 

Run the command in MySQL:
```
mysql> create database opacdb default character set utf8mb4 collate utf8mb4_unicode_ci;
```
This creates a database called `opacdb` and sets the character coding to UTF-8 to support international characters. I should get "Query OK, 1 row affected."

Run the `show` command to view this database:
```
mysql: show databases;
```
1. information_schema
2. mysql
3. opacdb
4. performance_schma
5. sys

For the `opacuser` to use this new `opacdb`, I granted `all privileges` on the `opacdb` to the user account `opacuser`:
```
mysql> grant all privileges on opacdb.* to 'opacuser'@'localhost' with grant option;
```
*This makes the `opacuser` the all-powerful user for this `opacdb` database only, not the others listed. The `opacdb.*` syntax means the `DatabaseName.AllTables`.*

Privileges in MySQL can be limited to specific privileges/operations/functions such as:
-   CREATE
-   DROP
-   DELETE
-   INSERT
-   SELECT
-   UPDATE
-   GRANT OPTION

*These are written in all caps, but these relational database keywords can be typed in lower case for easier typing.*

Could limit certain user accounts to only be able to SELECT, for example. If you used PHP to connect a MySQL database to a form on a website, you would only want random users to be able to select, but not delete/drop capabilities. Consider the purpose of the database and security risks for privileges.

Exit from the root MySQL user:
```
mysql> \q
```
## Log In as Regular User and Create Tables
As a default, the MySQL command line prompt is bare-bones. This can be changed in the `.bashrc` file. This is a hidden file in the ~home directory. See it using `ls -a`. This is the configuration for the `bash` shell
```
nano .bashrc
```
Add the following to the bottom of the file:
```
export MYSQL_PS1="[\d]> "
```
This changes the password for the MySQL prompt for our `opacuser` account.

 Then, save, exit, and source the file:
 ```
 source ~/.bashrc
 ```
 *Sourcing is executing the commands in the text file in the current instance of the shell.*

<hr>

### Logging In:
Log in as the opacuser:
```
mysql -u opacuser -p
```
*This logs us in as the user `opacuser` and the `-p` instructs MySQL to request the pass for the user as it is required. Without a `-p` I get:*
>ERROR 1045 (28000): Access denied for user 'opacuser'@'localhost' (using password: NO)

*If the password is typed in wrong, the following is returned:*
>ERROR 1045 (28000): Access denied for user 'opacuser'@'localhost' (using password: YES)

Enter the password. It will look like nothing is happening, but if entered correctly it will let you into MySQL. 

*If the password is typed in wrong, the following is returned:*
175 >ERROR 1045 (28000): Access denied for user 'opacuser'@'localhost' (using password: YES)
176 

Successful login takes me to the MySQL prompt line:
```
[(none)]> show databases;
```
*This new command prompt `[(none)]:` is from changing the `.bashrc` file.* 

See 3 databases:
1. information_schema
2. opacdb
3. performance_schema

Select the `opacdb` database:
```
[(none)]>: use opacdb
```
This will result in "**Database changed**", and the new prompt is `[opacdb]>` to indicate we are in that database. 
