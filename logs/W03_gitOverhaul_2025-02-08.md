# Overhaul of GitHub repo!!

Major Changes:
* Made an images repository on GitHub (remotely) by using a placeholder.txt
* Added in a file from my personal computer (didn't know how to do this from my VM without a link to said image; since learned I can upload file to VM in menu bar)
* Pulled those changes to my local system

## Week 3
* Generally learned more CLI commands including `stat` command
* Used that to get info on an old entry_one.md file
* Used that info to rename the file when I moved it to my new logs folder

## General Git processes this week...
I had to do multiple git pushes to sync these changes!
Edited README.md in nano, but I needed the preview feature while practicing with markdown

git add README.me; commited it; and pushed to main

THEN

git commit -a; this deleted my entry_one file, and placeholder.txt in Image directory

Still could not see the renamed entry_one file in logs repository 
git add logs; commit and pushed to origin main

All seemed to be caught up and synched!

Edited README in GitHub remotely for ease with markdown.
Commited changes to links, formatting of timeline list, and added photo + quote

---
# Notes for Future
* Git configuration following textbook was easy
* Generated SSH key instead of HTTPS which is another option on Git
* Cloned repo; seemed to work

## Pushing to Git
* `git status` will show how many files I have untracked
* `git add fileName` or `git add .` (which adds everything that has had changes since last push)
* `git commit -m "commit message here"` the -m lets you write the commit message in the command line; this is also why I configured a text editor when doing git configs
* A commit provides a snapshot of my files
* `git push` actually pushes it to GitHub 
* IF I accidentally stage something with `git add fileName`, undo it with `git restore --staged fileName`

## Pulling from GitHub
* If I edit in GitHub rather than the CLI, I need to pull those changes to my machine
* `git pull` will do that
* need to pull before I can push again; keep it clean. Work in the CLI.

