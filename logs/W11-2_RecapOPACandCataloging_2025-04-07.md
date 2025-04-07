# OPAC and Cataloging Recap
Over the last few weeks, I have followed the lecture videos and text to set up an OPAC and a cataloging module. 

An OPAC, or Online Public Access Catalog, is a searchable database of the materials a library or library system holds. It is the online upgrade of bygone card catalogs and the primary way for users to find what is available at the library. 

The underlying database of an OPAC is often made up of many tables that store bibliographic information. Breaking data into multiple tables can reduce redundancy and improve consistency. This is an efficient way to manage and retrieve data.

These __relational databases__ take information from those tables, using unique identifiers, like record numbers, to __relate__ the information in records of one table to the records of another (and so on). More specifically, by using **foreign keys**, different tables in the same database can be linked together for better data retrieval.

An OPAC allows users to efficiently search for the information stored across these tables. 

## Step-by-Step Setup
### Creating an OPAC 
To create a searchable OPAC, I needed to set up an HTML page for it that will display in a browser, and a PHP script to pull data from the __opacdb__ database created with MySQL. See my [Week 8 documentation](https://github.com/sbar04/syslib2025/blob/main/logs/W08-2_CreateTableConnectPHP_2025-03-04.md) to see how that was set up. 

#### Create HTML Page for OPAC

The HTML page, __mylibrary.html__ that will display in a browser and allow users to search the `opacdb` database. 

I went to `/var/www/html` to put this in the Apache2 document root HTML directory and created the __mylibrary.html__ file:

```
cd /var/www/html
sudo nano mylibrary.hmtl
```
I typed in the script to create a basic HTML page explaining the OPAC and providing a form element to search the database: 
```
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>MySQL Server Example</title>
    </head>
<body>

    <h1>A Basic OPAC</h1>

    <p>In the form below, <b>optionally</b> enter text in the search field.
    Your search query will search by author, title, or publisher.
    Capitalization is not necessary.
    It's okay to enter partial information, like part of an author's, title's, or publisher's name.</p>

    <p>You can leave the search field empty and only enter dates.
    Regardless, both start and end dates are required for all searches.
    You can use the date fields to limit results, too.
    I added some extra records, which you can view to know what you can query:</p>

    <p><a href="opac.php">OPAC</a></p>

    <p>This is very much a toy, stripped down
    <a href="https://en.wikipedia.org/wiki/Online_public_access_catalog">OPAC</a>.
    The records are basic.
    Not only do they not conform to <a href="https://www.loc.gov/marc/">MARC</a>,
    they don't even conform to something as simple as <a href="https://www.dublincore.org/">Dublin Core</a>.

    <p>I also don't provide options to select different fields, like author, title, or publisher fields.
    Instead the search field below searches all the fields (author, title, publisher) in our <b>books</b> table.</p>

    <p>The key idea is to get a sense of how an OPAC works, though.</p>

    <h2>My Basic Library OPAC</h2>

    <form method="post" action="search.php">
        <label for="search">Search Terms (optional):</label>
        <input type="text" name="search" id="search">
        
        <br>
        
        <label for="start_date">Start Date:</label>
        <input type="date" name="start_date" id="start_date" required>
        
        <br>
        
        <label for="end_date">End Date:</label>
        <input type="date" name="end_date" id="end_date" required>
        
        <br>
        
        <input type="submit" value="Search">
    </form>

</body>
</html>
```
The HTML in this is fairly simply. Most importantly, it includes the `<form>` element. This element uses the POST `method`, and the `action` attribute refers to a file called __search.php__, which is where the data from this form will be sent.  

#### Create PHP Script for OPAC

I needed to create this __search.php__ file for the OPAC to function. In the `/var/www/html` directory:
```
sudo nano search.php
```
I entered the following PHP script:
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Search Results</title>
<style>
    table {
        border-collapse: collapse;
        width: 100%;
    }
    th, td {
        border: 1px solid black;
        padding: 8px;
        text-align: left;
    }
</style>
</head>
<body>

    <h1>Search Results</h1>

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

    if ($_SERVER["REQUEST_METHOD"] == "POST") {
        $search = trim($_POST['search']);
        $start_date = $_POST['start_date'];
        $end_date = $_POST['end_date'];

        // Prepared statement to prevent SQL injection
        $stmt = $conn->prepare("SELECT * FROM books 
                                WHERE (author LIKE ? OR title LIKE ? OR publisher LIKE ?) 
                                AND copyright BETWEEN ? AND ?");

        // Use wildcard search
        $search_param = "%$search%";
        $stmt->bind_param("sssss", $search_param, $search_param, $search_param, $start_date, $end_date);
        $stmt->execute();
        $result = $stmt->get_result();

        if ($result->num_rows > 0) {
            echo "<table>";
            echo "<tr><th>ID</th><th>Author</th><th>Title</th><th>Publisher</th><th>Copyright</th></tr>";

            while ($row = $result->fetch_assoc()) {
                echo "<tr>";
                echo "<td>" . htmlspecialchars($row["id"]) . "</td>";
                echo "<td>" . htmlspecialchars($row["author"]) . "</td>";
                echo "<td>" . htmlspecialchars($row["title"]) . "</td>";
                echo "<td>" . htmlspecialchars($row["publisher"]) . "</td>";
                echo "<td>" . htmlspecialchars($row["copyright"]) . "</td>";
                echo "</tr>";
            }

            echo "</table>";
        } else {
            echo "<p>No results found.</p>";
        }

        $stmt->close();
    }

    $conn->close();
    ?>

    <p><a href="mylibrary.html">Return to search page</a></p>

</body>
</html>
```
>This creates an HTML document. The header includes metadata about the document. There is a style section with basic CSS for formatting the results. 

>The PHP elements start with the opening PHP tag `<?php`, and each includes `//` comments that help explain each section.

>It refers to the `login.php` file made in Week 8.
```
    require_once '/var/www/login.php';
```
>It establishes a connection with `MySQL`, and uses the `POST` request method which is the same as the form method in the __mylibrary.html__ file. 

>There is a prepared statement to prevent SQL injection. This is where you can see the SQL commands that are to retrieve data from the __books__ table when a search is performed from the web form:
```
SELECT * from books
WHERE (author LIKE ?  or title LIKE ? or publisher LIKE ?
AND copyright BETWEEN ? and ?");
```

>The script echos out results in a table format. 

This HTML form and PHP script are connected with the __mylibrary.html__ file's reference to the __search.php__ file. This allows us to use the browser, displaying the form defined in the __mylibrary.html__ file, to search contents of the __books__ table using PHP. 

Changes to the data in the __books__ table are reflected in the website's thanks to the PHP script.

### Creating a Cataloging Module
Catalogers generally use their graphical user interface ILS or ISP to insert data into the catalog. 

To catalog new entries into the __books__ table from the web interface, rather than logging into MySQL, I had to set up a cataloging module function. 

This is very similar to setting up the OPAC search page.

#### Create HTML Page for Cataloging
First, I created a new directory for the cataloging module in the Apache2 document root:
```
cd /var/www/html
sudo mkdir cataloging
```
In this new directory, I created the __index.html__ file that acts as the main cataloging page. This will include the form to enter bibliographic data into the __books__ table:
```
cd cataloging
sudo nano index.html
```

This form needs to include the data fields from the **books** table: 
* author
* title
* publisher
* copyright

The following code creates the form that will display on the cataloging page and labels these fields:
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
This uses the `form` element to create a form, and creates labels for all of the fields used in the __books__ table.

This __index.html__ file references the __insert.php__ in the form action line. I need to create this PHP script in order for the data entered into the cataloging form to communicate with the **books** table in my `opacdb` database. 


#### Create PHP Script for Cataloging

In the `/var/etc/www/cataloging` directory, I created the **insert.php** file:
```
sudo nano insert.php
```
And inserted the following:
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
Now, I am able to catalog items into the __books__ table from a web browser. 

#### Adding Security
Because this is available on the web, I used Apache2's authorization mechanism `htpasswd` to prompt me for a password in order to use the web catalog page. 

First, I created an authentication file in the `/etc/apache2` directory. This is where Apache2 stores configuration files. This file will contain a **hashed** password and a the username we specify.
```
sudo htpasswd -c /etc/apache2/.htpasswd libcat
```
>*Note: Using `/etc/apache2/.htpasswd` keeps us from having to change directories before creating this file.*

>*__libcat__ is the username we are creating this password for.*

>*The `-c` option of the `htpasswd` command creates the passwdfile. It rewrites and truncates any existing passwdfile.

The command line will prompted me to create a password:
```
New password:
Re-type new password:
Adding password for user libcat
```

Then, I had to tell the Apache2 web server to use the `htpasswd` to control access to the cataloging module. 

I opened the **apache2.conf** file:
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

#### Permissions and Ownership
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

### Time to Catalog!
From here, going to the __externalIP/cataloging__ page prompted me for the username and password I created in the security portion. 

Upon successful entry, the form allows me to enter information into the four fields of the HTML form. 

To check that these submissions are successful, I can go into MySQL from the CLI, and look in the __books__ table to confirm entries from the web interface were created successfully and appear in that table. 

I can also go to my OPAC page and query for new records.

### Real-World OPACs and Cataloging Modules
To make this bare bones OPAC and Cataloging Module more similar to real-world ones, I would need to add and build upon several elements. 

First, I would need to add more tables to my database. My current table only includes five fields: an assigned ID, author, title, publisher, and publication year. A relational database would have many more tables with many more fields, where records in one table relate to records in another. 

Then, I would also need to make my OPAC and cataloging module more attractive visually and more usable with CSS and Javascript. 

Finally, I would need to add much more robust security elements. 

## Key Details

In order for everything to work properly, attention to detail is key. In each of the .html and .php files, precision is required for both the OPAC and the cataloging module to be functional. 

For example, both the __mylibrary.html__ file for the OPAC __index.html__ for the cataloging module reference .php files. 

Making sure that these reference the correct .php files, AND that those .php files contain the correct scripts to execute either a search (for the OPAC), or to insert records (for the cataloging module) is essential to the functionality of these web pages.

## Using Documentation
I followed along with the video and written lectures as I set up the OPAC and cataloging modules. 

While there were not any major gaps in the provided materials, I frequently found myself Googling unfamiliar terms (hashed password was new to me) or refreshing my memory on MySQL commands. I also used our Teams chat to troubleshoot. I didn't realize, for example, that in MySQL the data type `YEAR` only has a range of 1901-2155. If I were to re-do my __books__ table, I might consider using a different numerical data type to allow me to catalog books published prior to 1901. 

Typing up my logs for each week helped me better understand the coding in .html and .php files, security and permission settings, and how each of these elements interacts.
