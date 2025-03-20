---
layout: post
title: '2 ways to sync your Sublimetext Config with another Device'
date: 2024-07-19
tags: dev-environment howto sublimetext git
subtitle: 'Using GIT and Cloud to sync and backup your Sublimetext Configuration'
comments_id: 1
---

# What

So I was using Sublime Text at work with my Work Device, as well at home in with my Gaming PC setup.
I wanted to sync and backup my Sublime Setup.

In the beginning I just copied my User Folder on a USB Stick and transported this USB between Home and Work :D

> a better solution has to be found

And there was, in the following I will describe the 2 solutions I used

# GIT

At first i tried to sync everything with a Github Repository, which worked fine. The only problem I got, is that i regularly forgot to pull on one device which leads to a merge conflict with the `Preferences.sublime-settings` file. Its not that much of a problem but still anoying.

Here is how I did it:

**Configure Sublime Text**

Start Sublime Text and configure it

**Create a new GIT repo**

Create a new Repo at gitlab/github whichever provider you want to use and copy the origin link.

```bash
cd ~/AppData/Roaming/'Sublime Text'/Packages/User
git init
git add --all
git commit -m "Inital Commit"

git remote add origin https://github.com/papierkorp/sublimetext.git
git branch -M main
git push -u origin main -f
```

**Use the Repo on another Device**

```bash
rm -R ~/AppData/Roaming/'Sublime Text'/Packages/User
mkdir ~/AppData/Roaming/'Sublime Text'/Packages/User

git clone https://github.com/papierkorp/sublimetext.git ~/AppData/Roaming/'Sublime Text'/Packages/User
```

# Cloud

Since I was annoyed with the continous merge conflicts I tried another solution with a Cloud Storage. I still use GIT as a backup.
First you need some kind of Cloud Provider like OneDrive, GoogleCloud or Dropbox.

**Configure Cloud Provider**

Since I use Windows and OneDrive is already preinstalled I will use this.

Go to settings, than on the top right corner click on `OneDrive` and login to your Microsoft Account.

**Configure in Windows**

- start the cmd as Administrator
- close SublimeText

```bash
mkdir C:\Users\Markus\OneDrive\Sublime
# move the whole User Folder from the AppData folder to the Cloud Folder
move "C:\Users\Markus\AppData\Roaming\Sublime Text\Packages\User" "C:\Users\Markus\OneDrive\Sublime\User"
# create a symlink between the Cloud Folder to the AppData Folder
mklink /d "C:\Users\Markus\AppData\Roaming\Sublime Text\Packages\User" "C:\Users\Markus\OneDrive\Sublime\User"
```

**Use it on another Device**

- start the cmd as Administrator
- close SublimeText

```bash
rmdir -recurse "C:\Users\Markus\AppData\Roaming\Sublime Text\Packages\User"
mklink /d "C:\Users\Markus\AppData\Roaming\Sublime Text\Packages\User" "C:\Users\Markus\OneDrive\Sublime\User"
```

# Afterwords

Its as easy as this few steps and you got yourself a sync as well as a backup for you whole Sublime Text Setup :)
