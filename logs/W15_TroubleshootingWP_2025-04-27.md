
# Week 15: Disaster Strikes (and Troubleshooting Ensues)

I'd be lying if I said this was fun under time pressure! 

In the final week of this semester, I somehow lost the ability to access my main Ubuntu server that was home to my WordPress and Omeka site content. 

## What happened?

After I had successfully added records into my Omeka digital library, and I was back on the WordPress admin site making changes to the homepage, adding plugins, and creating other pages. 

I had no issues saving these changes, and I could see my changes reflected on the webpage outside of the admin interface.

I had downloaded some different themes and a couple of plugins, but all was running smoothly.

The next morning, I couldn't get my site to load. I couldn't get anything from my web server to load, including the `index.php` or `index.html` pages from week 6, or the OPAC and cataloging modules from weeks 10 and 11. 

I tried logging in to my VM instance via SSH in browser, as I have been all semester, and I never could get in. I'd "open in browser window," get the white loading screen that says **"transferring SSH keys to the VM,"** and then it would go to **"Establishing connection to SSH server..."**

I never got past that screen. If I waited long enough, I'd get an error:
**Connection via Cloud Identity-Aware Proxy Failed. Code: 4003**

If I tried to connect without the Cloud Identity-Aware Proxy, it would still fail. 

### Troubleshooting 
I made sure my firewall allowing traffic was allowing TCP ingress traffic from all IP addresses on port 22. 

In my GCloud logs, I could see I was getting a number of errors, though. 

I clicked on my server, **main-ubuntu**, looked at "Logging" and filtered it to show me anything at the **Error** level or higher. 

I had 50+ alerts starting March 26th with "**invalid SSH key entry - expired key**." However, the date of expiration keeps updating with each error, so I wasn't sure what to do with that.

I thought this may have something to do with the VM loading, as sometimes it doesn't work for me in browser, but seeing the frequency and the dates makes me think this is a larger issue. I want to try fixes from [this forum](https://stackoverflow.com/questions/73860848/how-do-i-resolve-invalid-ssh-key-entry-error-when-starting-app-with-gce) after the semester. 

The answer that made the most sense to me at the time was [here](https://www.googlecloudcommunity.com/gc/Infrastructure-Compute-Storage/VPN-usage-resulted-in-quot-invalid-ssh-key-entry-expired-key/m-p/733601), where someone had the same error after accidentally filling their disk.

I tried to run the Dev Ops agent to see my disk utilization, but it always said "no data is available" no matter the time period I put in. I'm not sure if this was an issue with not installing Dev Ops before the issue started, or if Dev Ops doesn't work with the selected zone, operating system and architecture of my server. 

### Sandbox Testing Snapshot Backups
I didn't want to mess up anything in my **main-ubuntu** server or syslib2025 project, so I created a whole new gcloud project to test in. 

I started a new VM instance following my notes from week 3. I skipped installing git, but I did download some software before making a snapshot so I could see if the snapshot saved that information. 

I then installed Apache2, MySQL, and the PHP modules from week 7. I followed the instructions for making the OPAC and cataloging modules. I made a second snapshot here. 

I then installed WordPress and made some edits to my site before creating a third snapshot. 

#### New Instance from Snapshot
I then created a new instance from this latest snapshot, making it 20GB this time and an e2-medium type machine. 

Everything loaded as expected from that new IP address. There was a weird font difference between my original WordPress site and the one from the third snapshot, and I have no clue why. Both sites were functional, though. 

I did test changing my Omeka directory name, since I didn't really like `DigitalLibrary`. That went smoothly, so I changed it later in my the instance I am using to finish this class. 

I also wasn't afraid of messing this instance up, so I played around as the root user in MySQL to see the types of tables that were being made for Omeka and Wordpress. 

### Back to Main SysLib2025 Project
Back in my main syslib2025 project in gcloud, I confirmed that my server hosting Koha, **koha-main**, was still fine and I could access the OPAC and staff interfaces. 

My **main-ubuntu** server was still inaccessible through the CLI, and I still couldn't get any page from my IP address to load, though the status was still "running" according the gcloud. 

I made a snapshot of it in this state, and loaded a new instance of this snapshot (**main-try1**) on a 20GB e2-medium disk. I thought if the disk had filled, this would be easy and my site would run fine. 

I walked through all of the WordPress installation steps to check I still had everything installed (using `apt list --installed` to check for `php-curl` and the like) and that my configuration files had the correct information. 

In the GUI, I could get to the site but it would load VERY slowly with no plugin functionality and wonky HTML displays. If I tried to login to the admin page (which was also displaying in a broken format), I got an error that the connection timed out. 

Good news, though! My Omeka site was perfectly fine. All my content was saved, and my appearance changes were saved too. 

So I'm thinking I had to have messed something up in the WordPress configurations, or I have a bad theme or plugin making things break. 

*As a side note, I also tested that my git configurations still worked, so I pushed a test file, and then removed it. This messed me up later, though, on a different instance because the files looked to be synced, but my commits were not.*

#### Disk Full?
Just to check, I created a new VM instance (**main-try2-micro**) from the latest **main-ubuntu** snapshot with only 10GB and the e2-micro machine type. This is identical to the main-ubuntu server.

This machine loaded fine, and I was using 85.4% of 9.51GB, so I'm thinking I didn't go over on disk space on that original server. Something else is wrong. 

#### Bad Theme? Bad Plugin? 
Back in the **main-try-1** instance, I decided to delete my plugins from the CLI. I was able to see my plugins from the plugin directory `/var/www/html/library/wp-content/plugins` (I had renamed the `wordpress/` directory to `library/`). 

I simply deleted the plugins in that folder using `sudo rm -r plugin-directory`. This seemed to work, but I then tried the WordPress CLI tool, [WP-CLI](https://wp-cli.org/). 

```
cd /usr/local/bin
sudo curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
sudo mv wp-cli.phar wp
sudo chmod u+x wp
sudo wp theme list --allow-root --path='/var/www/html/wordpress'
sudo wp theme activate twentytwentyfour --allow-root --path='/var/www/html/wordpress'
``` 
Here, I downloaded the tool, renamed it to `wp` for easier use, looked at the full list of themes I had downloaded, and in the last command activated one of the default themes. 

However, this did nothing to fix my loading problems in the GUI. 

I checked my plugins:
```
sudo wp plugin list --allow-root --path='/var/www/html/wordpress'
```
And deactivated the only one I hadn't deleted earlier:
```
sudo wp plugin deactivate <name_of_plugin> --allow-root --path='/var/www/html/wordpress'
```
This did not solve my loading issues and inability to access the admin side of the site.

### Starting Fresh with WordPress
Exhausting all other options, I decided to start a completely fresh VM instance, **fresh-ubuntu-main**, from the latest main-ubuntu snapshot (one where I hadn't played with WP-CLI, out of an abundance of caution). I was hoping not to lose the pages, posts, and events I had created, but it was inevitable. 

I knew this snapshot would give me a functional Omeka digital library with content already in it, and I could restart with WordPress by deleting the directory, and removing the user and database from MySQL.

I also made it 20GB and an e2-medium machine type to give it more power and space than I need, just in case. 

#### First, Some Cleanup 

First, I ran my normal updates, and a new kernel was available so I ran the reboot as well:
```
sudo apt update
sudo apt upgrade -y
sudo apt autoremove
sudo apt clean
sudo apt reboot
```
I then made sure to do a `git pull` to sync this instance's local repository with GitHub. This cleared issues I was having with it. 

I had tried to upload the Week 14 log from the instance **main-try-1**, and I got a note that "Your branch and origin/main' have diverged and have 1 and 2 different commits each, respectively."

If I had stuck with this instance, I would have needed to resolve this:
```
git fetch origin
git pull origin main
git add <conflicting files>
git commit -m "resolved merge conflicts"
git push origin main
```

#### 2 Word 2 Press: Trying WordPress AGAIN, But For Real This Time
I ran through the WordPress installation instructions again, downloading the `latest.zip` file, renaming it, and walking through the installation on a browser.  

In MySQL, I deleted the `wordpress` database:
```
	sudo mysql -u root
	show databases;
	drop database wordpress;
```

I deleted the `wordpress` user:
```
	select user from mysql.user;
	drop user 'wordpress'@'localhost';
```

And then I went to my root directory and removed the WordPress directory:
```
	cd /var/www/html
	sudo rm -r library
```

All clear. 

*I also changed my Omeka directory from `DigitalLibrary/` to `digital-collections` for my own happiness.*

I reinstalled WordPress according to my own documentation. I checked my PHP and MySql versions, and I checked my server version. I used `apt list --installed` to check that I had all the necessary things installed. 

I used `wget` to download the `latest.zip` file, unzipped it, deleted the zip file
```
sudo wget https://wordpress.org/latest.zip
sudo unzip latest.zip
sudo rm latest.zip
```

I renamed my directory:
`sudo mv wordpress/ library`

I made all the necessary changes to the  `wp-config.php` file, and changed the file ownership of the directory to be owned and group owned by **www-data**.  

I installed the plugins I had used before, and the theme.

All is well. 

I'll be looking in to the invalid SSH key entry when I get a chance. I also didn't try adding a disk to my **main-ubuntu** instance, which also could have fixed this if it were in fact a disk usage issue (which it is pretty much confirmed not to be). I may try that just on one of the instances just to see what that process looks like.

## What I Learned
There was a lot of freedom in troubleshooting this because I literally couldn't break it more than it was broken. I enjoyed the freedom of playing in an entirely new project, and learning how to delete instances and projects. 

I looked at MySQL databases out of curiosity because I wasn't afraid to accidentally drop data or mess anything up.

This makes me feel excited for the end of the semester when I can just get in there and play with no pressure or potential repercussions for my website.
