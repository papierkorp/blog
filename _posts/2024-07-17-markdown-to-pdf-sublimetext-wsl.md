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

Take a look at the Pandoc Documentation regarding the [pdf-engine](https://pandoc.org/MANUAL.html#option--pdf-engine).


**Sublime WSLBuild Plugin**

Everything is already premade for us, we just have to Install this Plugin: https://packagecontrol.io/packages/WslBuild

# Create a Build

The final step is to create a Sublime Build, go to: `Tools - Build System - New Build System…`

```json
{
    "target": "wsl_exec",
    "cancel": {"kill": true},
    "wsl_cmd": "/usr/local/pandoc-3.2.1/bin/pandoc --verbose --pdf-engine=xelatex --citeproc -o '$file_base_name.pdf' '$file_name' && explorer.exe $(wslpath -w $unix_file_path); explorer.exe $(wslpath -w $unix_file_path/$file_base_name.pdf)",
    "wsl_working_dir": "$unix_file_path"
}
```

Finally just use this build on the markdown File with `CTRL + b`.

# Outsource the config

To outsource common Config settings you can adjust the build, by creating a includes file and add it in front of the input: `$unix_packages/User/builds/incl/header-includes.yaml`. In my User Folder I created a build/incl/header-includes.yaml file. In Sublime Build you can access your Packages path with the `$packages` variable and the WSLBuild Plugin includes the WSL equivalent.

```json
{
    "target": "wsl_exec",
    "cancel": {"kill": true},
    "wsl_cmd": "/usr/local/pandoc-3.2.1/bin/pandoc -o '$file_base_name.pdf' $unix_packages/User/builds/incl/header-includes.yaml '$file_name' --verbose --pdf-engine=xelatex --citeproc -f markdown-implicit_figures && explorer.exe $(wslpath -w $unix_file_path/$file_base_name.pdf)",
    "wsl_working_dir": "$unix_file_path"
}
```

and a example config:
```yaml
---
linkcolor: "blue"
toc: true
geometry: 
  - top=2cm
  - bottom=1.5cm
  - left=2.5cm
  - right=2.5cm
header-includes: |
  \usepackage{enumitem}
  \setlistdepth{9}
  \setlist[itemize,1]{label=$\bullet$}
  \setlist[itemize,2]{label=$\bullet$}
  \setlist[itemize,3]{label=$\bullet$}
  \setlist[itemize,4]{label=$\bullet$}
  \setlist[itemize,5]{label=$\bullet$}
  \setlist[itemize,6]{label=$\bullet$}
  \setlist[itemize,7]{label=$\bullet$}
  \setlist[itemize,8]{label=$\bullet$}
  \setlist[itemize,9]{label=$\bullet$}
  \renewlist{itemize}{itemize}{9}
  \setlist[enumerate,1]{label=$\arabic*.$}
  \setlist[enumerate,2]{label=$\alph*.$}
  \setlist[enumerate,3]{label=$\roman*.$}
  \setlist[enumerate,4]{label=$\arabic*.$}
  \setlist[enumerate,5]{label=$\alpha*$}
  \setlist[enumerate,6]{label=$\roman*.$}
  \setlist[enumerate,7]{label=$\arabic*.$}
  \setlist[enumerate,8]{label=$\alph*.$}
  \setlist[enumerate,9]{label=$\roman*.$}
  \renewlist{enumerate}{enumerate}{9}
---
```


# Afterwords

... and voila you got yourself a pdf. =)