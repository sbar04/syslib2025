# Managing Software
This week I learned about managaing software with `sudo` commands and apt installer

## sudo
So as a user we don’t control the root directory, because user root (superuser) does that, but `sudo` command gives us that power

## apt
Package manager that handles installation, upgrades, and uninstalls software

### Locate and Install Software
`apt search`
* Doesn't require sudo because I am not installing anything
* Use less to see more; `| less` pipes it through the less command
* `apt list` gives a full list of everything `apt` can install; put this through to a txt file for easier searching!

`apt show packageName`
* Shows more info about a package

`sudo apt install packageName`
* installs package

### Remove Software and Purge Related Files

`sudo apt --purge remove packageName`
* removes package
* `--purge` removes the config files in the /etc directory too

`sudo apt autoremove`
* removes any depedencies after removing a package

`sudo apt clean`
* removes extra files after installations

### REFRESHER: Keep System Up-To-Date
```
sudo apt update
sudo apt upgrade
sudo apt autoremove
sudo apt clean
```
---

## Fun Finds
* Enjoy having `tree` installed for easier file navigation
* `tldr` is making learning command options way easier; `man` is still useful but I find myself using `tldr` more now
* used `duf` for a bit to track my disk usage
* Everyone else seems to love `ninvaders`, might need to try it out! 
