---
layout: post
title: "2 ways to sync your Sublimetext Config with another Device"
date: 2023-09-18
tags: development-environment howto sublimetext git
subtitle: "Using GIT and Cloud to sync and backup your Sublimetext Configuration"
---


**per GIT**

__Einrichten__

aus dem kopf raus:

Sublime Text starten + konfigurieren dann:

<code bash>
cd ~/AppData/Roaming/'Sublime Text'/Packages/User
git init
git status
git add --all
git commit -m "Inital Commit"

# repo auf github oder wo auch immer erstellen

git remote add origin https://github.com/papierkorp/sublimetext.git
git branch -M main
git push -u origin main -f
</code>

__Verwenden__

  * Neue Sublime-Text Version installieren.
  * Strg + Shift + p - Install Package Control
  * zu "%appdata%\Sublime Text 3\Packages" navigieren 
    * Manuell: "User" Ordner speichern
    * per Gitbash: ''cd ~/AppData/Roaming/'Sublime Text'/Packages/User'' + ''git clone https://github.com/papierkorp/sublimetext.git .''


**per Cloud**

__Einrichten__

Sublime Text starten + konfigurieren (erstellt automatisch den User Ordner)
cmd als admin starten

<code bash>
mkdir C:\Users\Markus\OneDrive\Sublime
move "C:\Users\Markus\AppData\Roaming\Sublime Text\Packages\User" "C:\Users\Markus\OneDrive\Sublime\User" 
mklink /d "C:\Users\Markus\AppData\Roaming\Sublime Text\Packages\User" "C:\Users\Markus\OneDrive\Sublime\User"
</code>

__Verwenden__

cmd als admin starten

<code bash>
rmdir -recurse "C:\Users\Markus\AppData\Roaming\Sublime Text\Packages\User"
mklink /d "C:\Users\Markus\AppData\Roaming\Sublime Text\Packages\User" "C:\Users\Markus\OneDrive\Sublime\User"
</code>