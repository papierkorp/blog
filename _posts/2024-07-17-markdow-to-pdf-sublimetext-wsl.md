---
layout: post
title: "Build a pdf from markdown in sublimetext with WSL"
date: 2024-07-17
tags: webdev howto sublimetext wsl
subtitle: "How to build a pdf File from a markdown file using WSL in Sublimetext"
---

# Intro

Today I wanted to build a pdf File from a markdown File without having to litter my Windows. So I wanted to use WSL.

Since I use Sublime Text as my Editor of Choice it should be used as well.

# Install Preqrequisites

**Pandoc**

First of we need a tool to convert Files. For this the most common would be Pandoc:

Install Instructions:

```bash
wget https://github.com/jgm/pandoc/releases/download/3.2.1/pandoc-3.2.1-linux-amd64.tar.gz
sudo tar -xvzf pandoc-3.2.1-linux-amd64.tar.gz
sudo mv -v pandoc-3.2.1 /usr/local
echo 'export PATH=$PATH:/usr/local/pandoc-3.2.1/bin' >> ~/.bashrc
```

**PDF-Engine**

Now we need an PDF-Engine, there are quite a few to choose from but the first one I found after googling was xelatex.
Straight forward: `sudo apt-get install texlive-xetex`


**Sublime WSLBuild Plugin**

Everything is already premade for us, we just have to Install this Plugin: https://packagecontrol.io/packages/WslBuild

# Create a Build

The final step is to create a Sublime Build, go to: `Tools - Build System - New Build System…`

```json
{
    "target": "wsl_exec",
    "cancel": {"kill": true},
    // "wsl_cmd": "whoami && pwd && cat /etc/os-release", //to debug if everything is working
    "wsl_cmd": "/usr/local/pandoc-3.2.1/bin/pandoc --verbose --pdf-engine=xelatex --citeproc -o '$file_base_name.pdf' '$file_name' && explorer.exe $(wslpath -w $unix_file_path); explorer.exe $(wslpath -w $unix_file_path/$file_base_name.pdf)", //if you dont want to open the folder/file after finishing, just leave everything behind the && out
    "wsl_working_dir": "$unix_file_path"
}
```

Finnaly just use this build on the markdown File with `CTRL + b`.


... and voila you got yourself a pdf. =)