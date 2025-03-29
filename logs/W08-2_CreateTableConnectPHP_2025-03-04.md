# Week 8: Create MySQL Table and Connect with PHP

Follows Week 8: Installing MySQL.

## Creating MySQL Tables
Log in as a regular user:
```
mysql -u opacuser -p
```
*This logs me in as the regular user `opacuser` created in the last log, and uses the `-p` option to prompt the password set when creating the account. Enter the password when prompted; the command line will not indicate typing but the correct password will grant access.*

**Look at the databases available to the `opacuser`:**
```
[(none)]> show databases;
```
**Switch to the `opacdb` database:**
```
[(none)]> use opacdb;
```
**Create a table**
Relational databases contain tables. A database in MySQL is similar to an overall Excel workbook file, and the tables are like the specific sheets within the file. 

Tables need to be composed well because the composition dictates how well data is described, and how tables connect and interact with (**relate to**) each other.  

**To create a table:**
**Don't copy the [opacdb] part, that is the line prompt and indicates I'm in the `opacdb` database within MySQL.*
```
[opacdb]> create table books (
        id int unsigned not null auto_increment,
        author varchar(150) not null,
        title varchar(150) not null,
        copyright year(4) not null,
        primary key(id)
);
```
This is a new table titled `books`. 
It has four fields that will be the columns:
1. id
2. author
3. title
4. copyright year

The `id` field will act as the **primary key** as indicated by the second to last line. This key is used as a unique identifier for that record. This field is specified as being an `int` integer, that is a positive number `unsigned`, and that should not be empty (`not null`). With each record, it should increment by a single integer `auto_increment`. 

The `author` and `title` fields can have a maximum length of `150` characters and not be empty (`not null`). 

The `copyright` field is limited to the `year` data type, which must adhere to a specific syntax of `YYYY` (`4`), and not be empty (`not null`). 

Should get a "***Query OK***". Check the tables with:
```
[opacdb]> show tables;
```
To see the structure of the table:
```
[opacdb]> describe books;
```
### Add Records to Table
```
[opacdb]> insert into books (author, title, copyright) values
('Jennifer Egan', 'The Candy House', '2022'),
('Imbolo Mbue', 'How Beautiful We Were', '2021'),
('Lydia Millet', 'A Children\'s Bible', '2020'),
('Julia Phillips', 'Disappearing Earth', '2019');
```
With the MySQL `insert` command, I specified the three required fields (author, title, copyright) of the `books` table.

The `id` field is created and will increment automatically.

View this with the `select` command:
```
[opacdb]> select * from books;
```

### MySQL Commands

Followed along with the textbook, but see the following example commands for MySQL syntax hints.

**Retrieving records or parts of records:**
```
[opacdb]> select author from books;
[opacdb]> select author, title from books;
[opacdb]> select title from books where author like '%mbue%';
[opacdb]> select author, title from books where title not like '%e';
```
*Note the `select` command, the fields, the single quotations around what you're asking it to look for, and the `%` telling it to look for any characters around that sequence.*

**Alter the table structure to hold more data:**
```
[opacdb]> alter table books add publisher varchar(75) after title;
```
*This alters a publisher field that can be up to 75 characters, and places this field after the title field.*

```
[opacdb]> describe books;
```
*The `describe` command shows the structure of the table, now with the publisher field showing.*

Add in the publisher field data:
```
[opacdb]> update books set publisher='Simon \& Schuster' where id='1';
[opacdb]> update books set publisher='Penguin Random House' where id='2';
[opacdb]> update books set publisher='W. W. Norton \& Company' where id='3';
[opacdb]> update books set publisher='Knopf' where id='4';
```

See changes:
```
[opacdb]> select * from books;
```

**Deleting Records:**
```
[opacdb]> delete from books where author='Julia Phillips';
```
*This deletes the entire row/record that has Julia Phillips in the author field. Note that the `id` is not reused.*

**Adding Records:**
```
[opacdb]> insert into books
       (author, title, publisher, copyright) values
       ('Emma Donoghue', 'Room', 'Little, Brown \& Company', '2010'),
       ('Zadie Smith', 'White Teeth', 'Hamish Hamilton', '2000');
```
*We first describe which values we want to `insert` into the `books` table with the 2nd line of this command. Then, we add those values in subsequent lines.*

Check work:
```
[opacdb]> select * from books;
[opacdb]> select author, publisher from books where copyright < '2011';
[opacdb]> select author from books order by copyright;
```
## Install PHP and MySQL Support

I need to connect PHP and MySQL to use both on my website. 
```
sudo apt install php-mysql php-mysqli
```
*This installs the basic support and some extra modules.*

Restart Apache2 and MySQL so they recognize they are connected:
```
sudo systemctl restart apache2
sudo systemctl restart mysql
```
### Creating PHP Scripts
PHP needs to authenticate itself before it can connect to MySQL. Creating a  `login.php` file in the **document root's** *parent* directory `/var/www` will do this. 

*Reminder: The document root (`/var/www/html`) is the directory that holds my website files. The server serves files in this directory to the web. Files in the document root `/var/www/html` have __read__ access for __other/world__.*

```
cd /var/www
sudo touch login.php
```

#### Changing File Ownership and Permissions

The file is created, but I need to **change the group ownership** of the file and **its permissions**. This will let Apache2 web server read it, but not the world, and prevents the password information from being accessible to random web users.

See the current permissions with `ls -l`:
```
-rw-r--r-- 1 root root    0 Mar  4 18:45 login.php
```
*Currently, the **file owner** is `root` and the **group owner** is `root`.*

```
sudo chmod 640 login.php
```
*The `chmod` command changes the file permissions.  `chmod 640` changes the file permissions to:*
* _user owner: read, write_
* _group owner: read only_
* _other/word: no permissions_

```
sudo chown :www-data login.php
```
*The`chown` command changes the file ownership. Using the `sudo` command to create the file made `root` the owner. The `chown` command run above changes the group ownership to `www-data`, which is the Apache user.*

Now the ownership is:
```
-rw-r----- 1 root www-data    0 Mar  4 18:45 login.php
```
*The file owner is still `root`, but the group `www-data` is the group owner of this file `login.php`.*

By modifying the file ownership and permission, AND placing it in the `/var/www` directory,  the `login.php` file is not accessible to the world. This file has login information for the MySQL user and should not be public.

#### PHP Script
Open the `login.php` file and add the following with appropriate substitutions:
```
<?php // login.php
$db_hostname = "localhost";
$db_database = "opacdb";
$db_username = "opacuser";
$db_password = "PASSWORD";
?>
```
#### Create PHP File for Website
The PHP file will display HTML but involve PHP interacting with the MySQL `opacdb` database.
```
cd /var/www/html
sudo nano opac.php
```
Added the following:
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MySQL Server Example</title>
</head>
<body>

    <h1>A Basic OPAC</h1>
    <p>We can retrieve all the data from our database and book table using a couple of different queries.</p>

    <?php
    // Load MySQL credentials securely
    require_once '/var/www/login.php';

    // Enable detailed MySQL error reporting
    mysqli_report(MYSQLI_REPORT_ERROR | MYSQLI_REPORT_STRICT);

    // Establish database connection
    $conn = new mysqli($db_hostname, $db_username, $db_password, $db_database);

    if ($conn->connect_error) {
        die("Connection failed: " . $conn->connect_error);
    }

    echo "<h2>Query 1: Retrieving Publisher and Author Data</h2>";

    // Query using prepared statement
    $stmt = $conn->prepare("SELECT publisher, author FROM books");
    $stmt->execute();
    $result = $stmt->get_result();

    while ($row = $result->fetch_assoc()) {
        echo "<p>Publisher " . htmlspecialchars($row["publisher"]) .
             " published a book by " . htmlspecialchars($row["author"]) . ".</p>";
    }

    $stmt->close();

    echo "<h2>Query 2: Retrieving Author, Title, and Date Published Data</h2>";

    $stmt2 = $conn->prepare("SELECT author, title, copyright FROM books");
    $stmt2->execute();
    $result2 = $stmt2->get_result();

    while ($row = $result2->fetch_assoc()) {
        echo "<p>A book by " . htmlspecialchars($row["author"]) .
             " titled <em>" . htmlspecialchars($row["title"]) .
             "</em> was released in " . htmlspecialchars($row["copyright"]) . ".</p>";
    }

    $stmt2->close();
    $conn->close();
    ?>

</body>
</html>
```
Save and exit.

#### Testing Syntax
Use the `sudo php -f` command to test. the `-f` option parses and executes the indicated file.
```
sudo php -f /var/www/login.php
```
*This should output nothing.*

```
sudo php -f /var/www/html/opac.php
```
This should output HTML.

#### Check It!
Go to `ExternalIP/opac.php` in a web browser to see the results of this PHP script!
