---
title: "Install patch for Thinkpad Trackpoint Keyboard II"
description: "When using the new Thinkpad Trackpoint Keyboard II there is a bug with the middle mouse button"
categories: [linux, howto]
tags: [linux, kernel, thinkpad, trackpoint, keyboard, howto]
date: 2022-10-14
---

## New Thinkpad Trackpoint Keyboard II
<img src="https://raw.githubusercontent.com/joergi/blog/main/images/trackpoint-keyboard.jpg" width="400"><br>
(i maybe should have cleaned it, before the picture...)


I got my first Thinkpad Trackpoint keyboard as a farwell gift from a company I worked for (thanks <3 ).  
I don't use any mouse, I just use the red "joystick" (called trackpoint) to navigate. For users which aren't used to it, it's a horror if they have to use my keyboard. But if you are used to it, you don't wanna work without it anymore.  
As you can see, the old one is not in a good shape anymore. And as working-from-home is the new normal, the company I work for, offers us the equipment we need, so I ordered a new one.  
This is a newer version, which is running without a cable but wirelesse with an USB dongle. So far so good.

## The middle button
When I tried it out, everything seems to work as expected - but the middle button was showing strange behavior.  
Normally you can scroll in a browser with the middle button pressed and moving the trackpoint up and down (or left and right).  
The scrolling dikernel-d work, but it always opened the content while pressing the button in a new tab. So, while scrolling on Twitter, it means a lot of new tabs were opened.  
After searching a little bit, it seems my keyboard is not broken, but there is a known error in the driver.  

## Patching the kernel
After searching more, I found a [patch on Gitlab.freedesktop.org](https://gitlab.freedesktop.org/libinput/libinput/-/issues/547#note_1325369).  
Downloading the patch and run `./make.sh` was leading to an error that `Missing ../crypto/bio/bss_file.c` was needed.  
So, searching some more, it seems that I did not had the signing keys for the kernel. So I made the signing keys, [as described here](https://superuser.com/a/1322832).
After that I had to copy the signing key:e old one is not in a good shape anymore. And as working-from-home is the new normal, the company I work for, offers us the equipment we need, so I ordered a new one.
```bash
cp signing_key.pem /usr/src/linux-headers-5.15.0-50-generic/certs
```
I'm pretty sure, that I will need to do this step again, whenever I'm on a new linux header and I need to sign something else...  
Ae old one is not in a good shape anymore. And as working-from-home is the new normal, the company I work for, offers us the equipment we need, so I ordered a new one.fter this was done, I could continue with the patch, as described in the link above:  
```bash
sudo rmmod hid-lenovo; sudo insmod /lib/modules/$(uname -r)/extra/hid-lenovo.ko
```
Somehow this was still failing: `ERROR: could not insert module Operation not permitted`
As I broke my laptop screen a few days ago, and we just put my harddisc into a newer laptop, the secure boot was still on. After turning in the BIOS the secure boot off, the command above was working.
Then a restart, and the keyboard is more or less working as expected.  
(Somehow it still does open sometimes the links in new tabs, but only sometimes, not every time)

The patch should be in the Linux 5.19.x kernel.


