---
layout: post
title: "Allow non-root user to run command as root without password"
description: "How to allow non-root user to run one command without password"
date: 2023-05-22 20:00:00 +0200
category: misc
tags: misc linux
image: /assets/2023-05-23-allow-non-root/fly-d-zAhAUSdRLJ8-unsplash.jpg
---

![How to allow non-root user to run one command without password](/assets/2023-05-23-allow-non-root/fly-d-zAhAUSdRLJ8-unsplash.jpg)

Would you like to allow non-root user to run a specific command with root privileges, without adding them to sudoers and asking for password? Then you are in a right place! You can find the solution below.

## Introduction

While working on a python script which will gather and parse some data on debian machine i stumbled upon an issue.
Part of data i wanted to get was available from a program which required sudo privileges to run. It's my machine so i have root access, but having to type password for sudo every time i want to run the script was not ideal. Especially that eventually i might want to run it in automated manner.

So i needed a solution to allow an user to run only a specific command as root without the need to type in password. And i mean running a single command with specific params - command i need is a cli so i wanted to restrict parameters.

If you want to allow to run any executable you dont need to create a script - feel free to skip to the 2nd part.

I use this on debian, but it should also be applicable to other systems.

## How to allow non-root user to run a single script or command with params as root?

### Create script in /usr/local/sbin:

in `/usr/local/sbin/no_pw_sudo_script` file:

```bash
#!/bin/bash
# command which you want to run with sudo:
sudo /usr/sbin/hostapd_cli -i wlan0 all_sta
```

After that, apply correct permissions. We want the script to be executable, but only root should be able to edit it. If the regular user would have permissions to edit that file, it would allow them to run any command with root privileges:

```bash
sudo chown root:root /usr/local/sbin/no_pw_sudo_script
sudo chmod go-rwx /usr/local/sbin/no_pw_sudo_script
```

### 2. Allow to run it without sudo

We need to edit `/etc/sudoers` file to allow the user to run script created previously with root privileges, but without a need to type in password.

Remember to edit sudoers with `sudo visudo`, otherwise you may break something in your system :)

Add following lines, replacing `maciek` with username you want to allow to run the script.

```bash
maciek ALL=(ALL:ALL) ALL
maciek ALL=(ALL:ALL) NOPASSWD: /usr/local/sbin/no_pw_sudo_script
```

Replace "maciek" with the actual username of the regular user, and "/usr/local/sbin/no_pw_sudo_script" with the path to the command that you want the user to execute with sudo privileges.

You can now try running `/usr/local/sbin/no_pw_sudo_script` as the regular user and it should not require a password. However, it's essential to be cautious when granting sudo access to commands, as it can potentially compromise system security.

---

_Cover photo by [FLY:D]("https://unsplash.com/ja/@flyd2069?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)_
![](https://macap.ct8.pl/image.php?url={{site.url}}{{page.url}})
