---
layout: post
title: "How to add a second Account to the gitconfig"
date: 2023-09-18
tags: git howto
subtitle: "This shows how you can use a work and a private git Account in one Workspace based on SSH Keys on Windows and Linux."
comments_id: 2
---

# How it works

You have two options to log in to Git. The first is the HTTPS Protocol where you basically login with your username and password in a web form.
The second (and go to) option is via SSH Keys. So how do SSH Keys work?

SSH always consists of two elements:

- Public Key => `key_name.pub`
- Private Key => `key_name` or `key_name.ppk` (Windows Putty) or `key_name.pem` (Linux)
    + OPTIONAL - can be secured with a Passphrase for extra security

You can convert `.ppk` Files to `.pem` Files with [PuttyGen](https://www.puttygen.com/).

Now the public Key will be deposited on the server of the service you want to login to (e.g. github.com). So you have to copy/send/enter your public Key somehwere in the target system.

The Private Key on the other hand is used to authenticate to the public Key. It is like a password file for you while the Public Key is the username. If you used a passphrase you also have to enter the Passphrase before you can use the private Key.


# Generate SSH Keys

## Generate SSH Keys

open up a cmd/bash/gitbash/powershell

```bash
ssh-keygen -t rsa -b 4096 -C "work@email.com"  
id_rsa_work

ssh-keygen -t rsa -b 4096 -C "private@email.com"  
id_rsa_private
```

## Authenticate the SSH Keys

This command adds the private Key (including the passphrase) to the local SSH agent. It allows to enter the passphrase once per session rather than for each SSH connection.

Windows: open up powershell as admin

```powershell
Get-Service ssh-agent | Select-Object -Property Name, StartType, Status
Set-Service ssh-agent -StartupType Automatic
Start-Service ssh-agent
ssh-add C:\Users\Markus\.ssh\id_rsa_private
ssh-add C:\Users\Markus\.ssh\id_rsa_work
```

Linux:

```bash
eval `ssh-agent` #enable shh-agent
echo $SSH_AUTH_SOCK #make sure ssh-agent is running
ssh-add #add all private keys in ~/.ssh
ssh-add ~\.ssh\id_rsa_private #or add each key individually
ssh-add ~\.ssh\id_rsa_work

ssh-add -l #see all added keys
```

## Register Public Key with your git Repository (e.g. github.com)

```bash
tree ~/.ssh

├── id_rsa_private #private key, KEEP IT SAFE
├── id_rsa_private.pub #public key, will be deposited with Github
├── id_rsa_work
├── id_rsa_work.pub
└── known_hosts

cat ~/.ssh/id_rsa_work.pub #copy the content of the public Key (you can also go to the Explorer - Right Click - Edit)
xclip -sel clip < ~/.ssh/id_rsa_work.pub #another way to copy the content of the public key
```

As an example I will use github.com, but it will also work on any other git Server. (e.g. bitbucket, gitlab, space...)

- Open up [github.com](https://github.com/)
- Click on your Profile on the top right corner - Settings
- In the `Access` Header go to `SSH and GPG keys` 
- `SSH keys` Setting - `New SSH key`
    + Title: `Work PC`
    + Key type: `Authentication Key`
    + Key: Paste the key you copied one step earlier, should look something like this: `ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDT5+RFcly2O72Ajm/Vv+pfGH6UQ07SXAXS09CBDSX/5os6LOmFdX9ekIRXSssU9veW/6w1Nkjna+jWtEq/RhcTu4CDIYwX2DcYhjj66PdHD36+ND40c2VyVcEpHE58EDVZlN4pT0H+YhKK9J7rgLA/8MtRCA3j1fRx5vRtYmnapIFTxS2eVoxhW1cIjUiMgjSMp4mmTtVsmjr7DedDje58ROR7TWXkmNPY46PY2FJ/PjV+wgT/hNQY3KcFpJvB0as5aQptoH8U2NuIgTxBWuzVH/qhoaVciogzdQJOan2HFClg2AyYtmrttHeQESF+jaXV28RewjJzbsikuNu03KJtwWhrFvyDMUrq0T7INxhWS0nA6NPN86Y1sw8sABHGzuvWAavM3X3T6KxZJrQPyH0jZBOH8eJkX8k8r8A+421B36EkF+GRjN9fREWY+w6xLa6/+UttOKCGh3aGjMFteXy6kbt6NtqXmbtcUgHciKQ+cix1hNSDPXp74ubMQXeEP+/qeZbv7HmfTke+PWCF78zk5vyfdSUkOVpd8hbSTBqSIAcgCHTyuQVTBOd+UifY2W2qxCMLJiws8ur/rHBUbV8dUlhhtk66+Sw/wloEpPBsyEweuAlb1lOr3CA9LvRR63OUDT1Sz2CdyED0gebWsBYiBcJH31RXJG6R2c4KrYeiQQ== work@email.com`

Now our Public Key is registered with github and we cann access our account with the private Key, since both keys belong together and authenticate each other.




# SSH Config

This step is optional, I will give you the alternative to this in the [Git Config](#git-config) Section. But I certainly recommend using it.

Now we will create a ssh Config to manage our Keys locally. First create the Config File:

- Windows: `C:\Users\Markus\.ssh\config`
- Linux: `vi ~/.ssh/config`

Windows:

```bash
Host private
    HostName github.com
    User git
    IdentityFile C:/Users/Markus/.ssh/id_rsa_privat
    AddKeysToAgent yes

Host work
    HostName git.company.de
    User git
    IdentityFile C:/Users/Markus/.ssh/id_rsa_work
    AddKeysToAgent yes
    Port 12222
    StrictHostKeyChecking no
```

Linux:

```bash
Host private
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa_privat
    AddKeysToAgent yes
    UseKeychain yes

Host work
    HostName git.company.de
    User git
    IdentityFile ~/.ssh/id_rsa_work
    AddKeysToAgent yes
    Port 12222
    UseKeychain yes
    StrictHostKeyChecking no
```

You can use a lot of SSH [options](https://www.ssh.com/academy/ssh/config) in your config File. Take a look [here](https://www.ssh.com/academy/ssh/config) for all possibilies.


# Git Config

## .gitconfig

Finally we are going to create/edit our `.gitconfig` File.

- Windows: `C:\Users\Markus\.gitconfig`
- Linux: `vi ~/.gitconfig`

In our case we will create additional .gitconfig Files for each SHH Key.
So in the `.gitconfig` we will add:

Windows:

```bash
[includeIf "gitdir:C:/develop/private/"] #will include all .git Projects in this Folder/Subfolder
        path = ~/.gitconfig_private #we will create this file in the next step

[includeIf "gitdir:C:/develop/work/"]
        path = ~/.gitconfig_work

[core] #include gitconfigs which are valid everywhere
    excludesfile = ~/.gitignore
```

Linux:

```bash
[includeIf "gitdir:%(prefix)//mnt/c/develop/work/"] #will include all .git Projects in this Folder/Subfolder
        path = ~/.gitconfig_work #we will create this file in the next step

[includeIf "gitdir:%(prefix)//mnt/c/develop/private/"]
        path = ~/.gitconfig_privat

[core] #include gitconfigs which are valid everywhere
    excludesfile = ~/.gitignore
```

You can use all [gitconfig](https://git-scm.com/book/en/v2/Getting-Started-First-Time-Git-Setup) Settings from [here](https://git-scm.com/book/en/v2/Getting-Started-First-Time-Git-Setup).


## .gitconfig.work

You can create this file everywhere, you will just have to link it properly in the `.gitconfig`.
In the example above we are using the same folder as the .gitconfig file, so now its time to create our Custom Configs:

- Windows: `C:\Users\Markus\.gitconfig.work`
- Linux: `vi ~/.gitconfig.work`

```bash
[user]
    email = work@email.com
    name = Interesting Name
[credential "https://git.company.de"]
    provider = generic
[credential "https://git.company.de/specificProject/*"]
    helper = wincred
    username = in
    useHttpPath = true
[url "work:"]
   insteadOf = ssh://git@git.company.de:12222 #the default link provided by the company would be: ssh://git@git.company.de:12222/specificProject/project.git, you will have to cut the project out
   #the link above will be replaced with the ssh Config for "work"
   #and will look like this: "work:specificProject/project.git"
   #and finally "work" will be replaced with all of your settings in the SSH config File
```

If you dont want to use the SSH config and do it quick and dirty you can do it like this:

```bash
[user]
    email = work@email.com
    name = Interesting Name
[credential "https://git.company.de"]
    provider = generic
[credential "https://git.company.de/specificProject/*"]
    helper = wincred
    username = in
    useHttpPath = true
[core]
    sshCommand = "ssh -i C:/Users/Markus/.ssh/id_rsa_work" #Windows SSH command
    #sshCommand = "ssh -i ~/.ssh/id_rsa_work" #Linux SSH command
```

You can use all [gitconfig](https://git-scm.com/book/en/v2/Getting-Started-First-Time-Git-Setup) Settings from [here](https://git-scm.com/book/en/v2/Getting-Started-First-Time-Git-Setup).

## .gitconfig.private

You can create this file everywhere, you will just have to link it properly in the `.gitconfig`.
In the example above we are using the same folder as the .gitconfig file, so now its time to create our second Custom Config:

- Windows: `C:\Users\Markus\.gitconfig.private`
- Linux: `vi ~/.gitconfig.private`

```bash
[user]
    email = private@email.com
    name = Interesting Name
 
[github]
    user = papierkorp 

[url "privat:"]
    insteadOf = git@github.com:
```

For the details take a look in the section where we configured the [first](#.gitconfig.work) gitconfig file

You can use all [gitconfig](https://git-scm.com/book/en/v2/Getting-Started-First-Time-Git-Setup) Settings from [here](https://git-scm.com/book/en/v2/Getting-Started-First-Time-Git-Setup).

# Debugging

**Which Config is currently running?**

```bash
cd /c/develop/work/project
git config -l --show-scope --show-origin
git config user.email
```

**Change the git Link to the SSH link**

```bash
cd /c/develop/work/project
git remote set-url origin git@<repository.domain.com>:<username>/<repo_name>.git
```
