---
title: "Being able to develop on an unknown phone on a Linux computer"
description: "Everything you have to set up to deploy an Android app to your phone via ADB"
date: 2015-11-14
---


When i tried to deploy a <em>Hello World</em> app from my Linux notebook to my THL 5000 notebook i got this result:

<a href="https://joergi77.files.wordpress.com/2015/11/android-dev.png"><img class="alignnone size-medium wp-image-47" src="https://joergi77.files.wordpress.com/2015/11/android-dev.png?w=300" alt="API Level 1 sucks" width="300" height="166" /></a>

So, my phone has API level 1, nice - NOT!

Googling my ass off, I could nowhere find the device id for this phone... So <a href="https://twitter.com/tbsprs" target="_blank">Tobi</a> told me: Brute-Force is the way to go and what should I say: it worked!

So: <a href="http://bfy.tw/2n2s" target="_blank">google</a> for the "51-android.rules" or just save <a href="https://raw.githubusercontent.com/snowdream/51-android/master/51-android.rules" target="_blank">this one</a>.

After that copy it in the right place
```bash
sudo cp /home/username/download/51-android.rules /etc/udev/rules.d/
```
and make it working with
```bash
sudo chmod a+r /etc/udev/rules.d/51-android.rules
```
When plugging now my THL 5000 to my notebook, everything works fine!
