# Creating a Bare Bones Cataloging Module

Now it's time to build a module where we can catalog from a web browser. 

## Create the Cataloging Page

First, create the **HTML** page containing the form to enter bibliographic data. 

This form needs to include the data fields from the **books** table: 
* author
* title
* publisher
* copyright

This page, **index.html** will go in a new directory for our cataloging module.
```
cd /var/www/html
sudo mkdir cataloging
```
In this new directory, create the **index.html** file:
```
cd cataloging
sudo nano index.html
```
Insert the following content:
```
<!DOCTYPE html>
<html>
<head>
    <title>Enter Records</title>
</head>
<body>
    <h1>OPAC Library Administration</h1>

    <p>This is the library administration page for entering records into the OPAC.</p>
    <p>Please do not use this page unless you are an authorized cataloger.</p>

    <form action="insert.php" method="post">
        <label for="author">Author:</label>
        <input type="text" name="author" id="author" required><br><br>

        <label for="title">Book Title:</label>
        <input type="text" name="title" id="title" required><br><br>

        <label for="publisher">Publisher:</label>
        <input type="text" name="publisher" id="publisher" required><br><br>

        <label for="copyright">Copyright:</label>
        <input type="number" name="copyright" id="copyright" min="1000" max="2300" required>

        <input type="submit" value="Submit">
    </form>
</body>
</html>
```
*Note: The __index.html__ file references the __insert.php__ in the form action line.*

## Create the PHP Script
Then, we need the PHP script to communicate between the data entered into the form from the cataloging HTML page and the MySQL `opacdb` database, and **books** table. 

This PHP script matches the form from the cataloging HTML page we just made, and the structure in the **books** table. 

In the `/var/etc/www/cataloging` directory, create the **insert.php** file:
```
sudo nano insert.php
```
and insert the following:
```
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cataloging: Data Entry</title>
</head>
<body>

<h1>Cataloging: Data Entry</h1>

<?php

// Load MySQL credentials
require_once '/var/www/login.php';

// Enable MySQL error reporting
mysqli_report(MYSQLI_REPORT_ERROR | MYSQLI_REPORT_STRICT);

// Establish connection
$conn = new mysqli($db_hostname, $db_username, $db_password, $db_database);
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}

// Prepare and bind SQL statement
$stmt = $conn->prepare("INSERT INTO books (author, title, publisher, copyright) VALUES (?, ?, ?, ?)");
$stmt->bind_param("ssss", $author, $title, $publisher, $copyright);

// Set parameters and execute statement
$author = $_POST["author"];
$title = $_POST["title"];
$publisher = $_POST["publisher"];
$copyright = $_POST["copyright"];

if ($stmt->execute() === TRUE) {
    echo "New record created successfully";
} else {
    echo "Error: " . $stmt->error;
}

// Close statement and connection
$stmt->close();
$conn->close();
?>

<p><a href='index.html'>Return to Cataloging Page</a></p>
<p><a href='../mylibrary.html'>Return to Library Home Page</a></p>
</body>
</html>
```
*Note: the `<a href>` elements show that clicking "Return to Cataloging Page" will take us back to the **index.html** page, which is the cataloging module we just created.*

*The "Return to Library Home Page" refers back to `../mylibrary.html`, with the  `..` going back one directory from the cataloging directory to the `/var/etc/html` directory.*

## Security
To limit access to the web interface cataloging module, we will use Apache2's authorization mechanism `htpasswd`. 

First, create an authentication file in the `/etc/apache2` directory. This is where Apache2 stores configuration files. This file will contain a **hashed** password and a the username we specify.

```
sudo htpasswd -c /etc/apache2/.htpasswd libcat
```
>*Note: Using `/etc/apache2/.htpasswd` keeps us from having to change directories before creating this file.*

>*__libcat__ is the username we are creating this password for.*

>*The `-c` option of the `htpasswd` command creates the passwdfile. It rewrites and truncates any existing passwdfile.

The command line will prompt you to create a password:
```
New password:
Re-type new password:
Adding password for user libcat
```

Then, we have to tell the Apache2 web server to use the `htpasswd` to control access to the cataloging module. 

Open the **apache2.conf** file:
```
sudo nano /etc/apache2/apache2.conf
```
Edit the following stanza (line 172; use the `^W` command in `nano` to search for "Directory /var/www" for easier searching):
```
<Directory /var/www/>
  Options Indexes FollowSymLinks
  AllowOverride None
  Require all granted
</Directory>
```
to 
```
<Directory /var/www/>
  Options Indexes FollowSymLinks
  AllowOverride All
  Require all granted
</Directory>
```
We need to **AllowOverride** for **All**.

Save and exit the file. 

Then, go back to the cataloging directory and create the **.htaccess** file.
```
cd /var/ww/html/cataloging
sudo nano .htaccess
```
And add the following:
```
AuthType Basic
AuthName "Authorization Required"
AuthUserFile /etc/apache2/.htpasswd
Require valid-user
```
*This tells Apache2 to look in the **.htpasswd** file for the login info.*

Check that the configuration file is okay:
```
apachectl configtest
```
**Syntax OK** means all is good, and I can restart Apache2 and check its status:
```
sudo systemctl restart apache2
systemctl status apache2
```
System should be __active__ and __running__.

### Permissions and Ownership
Apache2's user account on the Linux server is __www-data__. The account details are found in the __/etc/passwd__ file.

See all users on the system and their information:
```
cat /etc/passwd
```

To see just the information for the __www-data__ user, we can use `grep`:
```
grep "www-data: /etc/passwd"
```
This outputs:
```
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
```
*We can see the Apache2 user's home directory is __/var/www__ and that its default shell is __/usr/sbin/nologin__, which means this user cannot login to a shell.*

Because there is a user account for Apacher2, we can limit file permissions and ownership to this user. 

In general:
* static files (HTML, CSS, JS) should be readable by __www-data__. Other users can own these files. 
* Directories where Apache needs to __write__ data or applications that need write access should be owned by __www-data__. 
* Configuration files should be readable but not writable by __www-data__. 

To do this:
1. Change the group ownership of __/var/www/html__ to __www-data__:
```
sudo chown :www-data /var/www/html
```
2. Set the __setgid bit__ on __/var/www/html__. This will make it so any new files created within this directory inherit the group ownership of the parent directory (which we just changed). 
```
sudo chmod -R g+s /var/www/html
```
*Note: the `-R` makes this a recursive change. The `g` indicates that the other users in this file's group are being affected by this change, and the `s` sets the user or group ID on execution of the command.*

## Time to Catalog!
From here, going to the __externalIP/cataloging__ page should prompt us for the username and password we created in the security portion. 

Upon successful entry, the form allows us to enter information into the four fields of the HTML form. 

To check that these submissions are successful, I can go into MySQL from the CLI, and look in the __books__ table to confirm entries from the web interface were created successfully and appear in that table. 


