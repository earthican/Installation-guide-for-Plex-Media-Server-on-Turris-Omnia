# Installation guide for Plex Media Server on Turris Omnia

> WIP version

## Overview

The installation consists of these steps:

1. Create Debian LXC container
2. Simple Debian configuration
3. Install Plex Media Server
4. Connect to CIFS

## 1. Create Debian LXC container

**[Official Manual](https://www.turris.cz/doc/en/howto/lxc)**

```
lxc-create -t download -n Debian
```

- Distribution: **Debian**
- Release: **Stretch**
- Architecture: **armv7l**

## 2. Simple Debian configuration

Connect to container:

```
lxc-start -n Debian
lxc-attach -n Debian
```

Change hostname (for eg. `Debian`):

```
apt-get update
apt-get upgrade
apt-get install nano
nano /etc/hostname
```

Set new hostname to localhost:

```
nano /etc/hosts
```

And add this line (for `Debian` as hostname):

```
127.0.1.1   Debian
```

Install packages:

```
apt-get install git-core openssh-server rsync sudo fakeroot cifs-utils -y
```

[Optional] Create your user (eg. `petr`):

```
adduser petr
```

Run `visudo` commnad and add:

```
petr ALL=(ALL:ALL) ALL
```

Log to your user using SSH or sudo:

```
sudo su petr
```

## 3. Install Plex Media Server

Add repository

```
echo "deb https://dev2day.de/pms/ stretch main" | sudo tee /etc/apt/sources.list.d/pms.list
sudo echo "deb https://downloads.plex.tv/repo/deb/ public main stretch main" >> /etc/apt/sources.list
```

More dependencies...

```
apt-get install apt-transport-https gnupg2 wget
```

Add dev2day GPG key

```
wget -O - https://dev2day.de/pms/dev2day-pms.gpg.key | sudo apt-key add -
```

Fix for automatic start:

```
sudo touch /usr/lib/plexmediaserver/start.sh
```

Install:

```
sudo apt-get update
sudo apt-get install -t stretch plexmediaserver-installer
```

## 4. Connect to CIFS

Write down `plex` user id (`uid`):

```
id -u plex
```

Create empty folder for mount:

```
mkdir -p /media/video
```

Add mount to `/etc/fstab` (set `uid` from `plex` user)

```
//192.168.1.10/video /media/video cifs uid=107,gid=1000,iocharset=utf8 0 0
```

[For more about CIFS mount](http://midactstech.blogspot.cz/2013/09/how-to-mount-windows-cifs-share-on_18.html)

## 5. Mount Disk From Host to LXC

```
# ls -la /dev/sda1 # or whatever your device path is
brw-r--r--    1 root     root        8,   1 Dec 28 22:09 /dev/sda1
```

Mount your drive

```
mount /dev/sda1 /media/video
```

Take note of the **8** and note whatever your number is.

Edit LXC config

```
nano /srv/lxc/Debian/config
```

Add the following lines:

```
# Mount drives
lxc.cgroup.devices.allow = c 8:* rwm
lxc.mount.entry = /media/video/ media/video none bind,optional,create=dir 0 0
```

Again, note the **8** in the first line and replace with yours

## Credits

[Plex Media Server ARM package for Debian/Ubuntu Linux](https://tproenca.github.io/pmsarm7/)
