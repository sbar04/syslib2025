# Week 6: Installing Apache
This week went super smoothly! 

Conceptualizing the internet as a world wide file system that retrieves and displays files from web servers makes this make a lot more sense to me. 

## Before
Read Apache's [Getting Started](https://httpd.apache.org/docs/2.4/getting-started.html) page to understand:
* __Clients, Servers, and URLs__
	* URL = Uniform Resource Locators specify protocols, servernames, and URL path, etc.
	* Clients (browser) connects to a server (Apache) with the protocol and requests the resource with the URL path
* __Hostnames and DNS__
	* DNS = Domain Name System
	* IP Addresses can be private or public
		* See gcloud instance for public address
		* See private IP with `ip a` command
		* 127.0.0.1 is a loopback address that a computer uses to refer back to itself. Can also use localhost.
	* A host file is an OS file that maps hostnames to IP addresses (000.0.0.0 --> www.example.com)
* __Configuration Files and Directives__
	* Apache is configured via text files located in `/etc/apache2`
	* Server is configured by putting configuration directives (keyword followed by arguments to set values) in configuration text files
* __Web Site Content__
	* Static vs Dynamic content
	* HTML files, CSS, image files that reside in the filesystem are static
	* Dynamic content is generated at the time of a request and can change from request to request. APIs, etc.
* __Log Files and Troubleshooting__
	* Apache saves error logs that tell you what went wrong, when, and how to fix it
	* ErrorLog directive tells you where these files are saved

## Installation
I updated my system before installing Apache.

Followed the software management process described in the [Week 5 log](https://github.com/sbar04/syslib2025/blob/main/logs/W5_manageSoftware_2025-02-09.md) to `search` and `show apache2` before `sudo apt install apache2`. 

__NEW COMMAND__: `systemctl`
* manages _systemd_ system and service manager in Linux
* `systemctl status servicename` lets us check the status of a service
* `systemctl status apache2` should tell us that it is __Active__ (running and live!) and __Loaded__
	* __Enabled__ means that the software starts automatically on reboot

## Browsing
### Text based Web Browser
Installed both `w3m` and `elinks` for my command line browsers. Preferring elinks for now.

See the _default site_ using our loopback IP address:

`w3m 127.0.0.1`

Or

`elinks localhost`

Can also see the default site using the system's private IP. Find this address with `ip a`, and run it through `elinks`.

### Graphical Browser
To see the default site using Firefox, Chrome, etc., find the system's external IP address in the Google Cloud Console.

If it does not open, double check that the URL is http, NOT https.

The Apache2 Ubuntu Default Page provides a configuration overview and tell us the default __document root__ is `/var/www`.

## Creating a Web page
Remember that the web is, at its simplest, a filesystem that has been made available to the wide world. The web server is what provides access to part of the filesystem. That point of access is called the **document root**.

The web server provides access to the part of the filesystem that has been made available on the web. The __document root__ is the point of access for a server to reach that filesystem. 

Most systems look for the `index.html` file as the default page. Currently, this file takes us to the Apache2 Ubuntu Default Page. To change it:
```
cd /var/www/html/
sudo mv index.html index.html.original
sudo nano index.html 
```
*Use `sudo` outside of the ~home directory to act as the root user.*

Enter HTML in this document. Save it. Reload the browser with this page, and it changes to whatever was put in the `index.html` file. `index.html` acts as sort of the "home page" for websites. 

### HTML Refreshers
I haven't used HTML to build a page from scratch before.

I watched some refresher videos on HTML for complete beginners on YouTube. I'm also trying to learn some CSS basics.

I'll be playing with [this page](http://34.173.172.135/index.html) more as I learn!
