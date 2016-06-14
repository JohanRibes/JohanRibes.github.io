---
layout: post
title:  "Raspberry Pi 3 initialisation"
date:   2016-05-22 12:03:17 +0100
categories: raspberry-pi
---
This tutorial is a short explanation of how to setup a RPi3 with wi-fi and ruby support for my experimentations.


## Raspberry Pie 3 basic minibian installation
First, you need to download last img of the minibian OS wich is a light version of the raspbian (no GUI, no GCC, etc..).
Go to [minibian website](https://minibianpi.wordpress.com/).

Insert a new sd card in your reader and open a new terminal. You have to be `root` or use `sudo`.

Then you will have to:

1. Locate the identifier of your SD card.
2. Unmount it .
3. data dump the img to your sd card.

```shell
df -h 
diskutil unmount /dev/disk1s1 
sudo dd bs=1m if=./Downloads/raspbian_wheezy_20140726.img of=/dev/rdisk1
```

Then you can boot your Pi on the SD card and plug the ethernet cable to your local network.
We will access it via SSH and donwload several packets from the inernet so you have to get the ip adress that the DHCP has assigned to the raspberry pi.

## Connection and update
Assuming that the Pi IP adress is 192.168.1.12 you wil acces it via ssh trough :

```shell
ssh root@192.168.1.12
root@192.168.1.22s password: raspberry 

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sat May 21 13:18:38 2016 from 192.168.1.11
root@minibian:~# 
```

As you read, default login is `root` and password `raspberry`.
Then, we will update packets database, retrieve the last bios, flash it and reload.

```shell
apt-get update
apt-get upgrade
rpi-update
reboot
```

After the boot, you can consider your install as fresh and updated.

## Basic configuration and user
These are some `dpkg`commands to set locales, timezone and keyboard.

```shell
dpkg-reconfigure tzdata 
dpkg-reconfigure locales
dpkg-reconfigure console-data 
apt-get install raspi-config
```

Use raspi-config to expand the minibian to match full size of your SD card.


```shell
adduser pi
usermod -G pi,adm,dialout,cdrom,sudo,audio,video,plugdev,games,users,netdev pi
```

Now you can use sudo ;).

## Some Packages
```shell
apt-get install build-essential ruby
```

## Wi-Fi setup
First, install the drivers and `wpa -supplicant`

```shell
apt-get install firmware-realtek wpasupplicant
```

Then, we will make a declaration of our wlan interface in `/etc/network/interfaces`

```conf
allow-hotplug wlan0
iface wlan0 inet manual
wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf
iface default inet dhcp
```

As you don't want to store your `12345678` paswword in wpa_spupplicant config file, you will have to encrypt it before.

```shell
root@minibian:/var/jekyll/blog# wpa_passphrase my-ssid "12345678"
network={
	ssid="my-ssid"
	#psk="12345678"
	psk=8616de5a4442378109b0822c9467995b9ed6583395b04f57756be5cab6ea176a
}
```
 
You can paste the result in a fresh new `/etc/wpa_supplicant/wpa_supplicant.conf` file, without the `#psk` line of course.
Last step wake up the wlan interface and check if DHCP Discovery has succeded.

```shell
ifup wlan0
ifconfig wlan0
```


## Backup
Using DD you can reverse the operation of dumping data to the sd. Using it reverse will backup your install.
You can do it on you raspberry if you decide to create a new partition on the SD and save only the used space or if you save the output on the network.
But here, I will do it via a card reader on my mac.

```shell
dd if=/dev/rdisk1 of=~/Desktop/pi.img bs=1m
```

If you have a 8Gb SD Card, - event if you do not use the whole filesystem - the output img will be 8Gb.
So if you want to save only the used space, use `df` and a calculator to match the exact bloc numbers and run the command below.

 For a 2Gb install
dd if=/dev/rdisk1 of=~/Desktop/pi.img bs=1m count=2048

If you want some gunzip to save disk space

```shell
dd if=/dev/rdisk1 bs=1m | gzip > ~/Desktop/pi.gz
```

So unziping and dumping should be 

```shell
gzip -dc ~/Desktop/pi.gz | sudo dd of=/dev/rdisk1 bs=1m
```

### Sources
* [minibian website](https://minibianpi.wordpress.com/).
* Official Raspberry Pi [Wi-Fi](https://www.raspberrypi.org/documentation/configuration/wireless/wireless-cli.md) configuration.
* Raspbery Pi forum [cloning topic](https://www.raspberrypi.org/forums/viewtopic.php?t=49503&p=385472)
* Smittytone [website](https://smittytone.wordpress.com/2013/09/06/back-up-a-raspberry-pi-sd-card-using-a-mac/).
* SuperUser.com [cloning topic](http://superuser.com/questions/568236/can-i-use-dd-to-clone-a-larger-sd-card-to-a-smaller-sd-card-if-the-actual-partit).

