---
title: "Building a raspberry pi streaming cam"
description: "All the steps, which are needed for using your raspberry pi for streaming"
categories: [raspberrypi, howto]
tags: [raspberrypi, streaming, cam, howto]
date: 2021-07-23
---
# the setup
This is how my setup looks like:<br>
<img src="https://raw.githubusercontent.com/joergi/blog/main/images/raspi-webcam-setup.jpg" alt="complete technical setup" width="300"> <br>

as webcam I use a Waveshare 10300 RPi Camera (E)  
(bought it <a href="https://www.welectron.com/Waveshare-10300-RPi-Camera-E">here</a>, unpaid advertisment)

# Install raspberry pi image
Download the images from the <a href="https://www.raspberrypi.org/software/">raspberry pi website</a>  
copy it to your sd card.  
on the webpage they recomment to install the rpi-manager via `sudo apt install rpi-imager`, but for me it wasn't working because of dependency problems.  
So I installed it the classic way:  
```shell
unzip -p YOUR_RASPBIAN_IMAGE.zip | sudo dd of=/dev/YOURDEVICE bs=4M conv=fsync
```
check with `lsblk -p` which is your SD card, so you are not deleting your harddrive.  
Maybe plug it in and out to be sure.   
For more information about this, go to the offical <a href="https://www.raspberrypi.org/documentation/installation/installing-images/linux.md">(Linux) installation page</a>.
# Activate your (already pluged in) Camera 
Go to settings -> interfaces -> cam -> and switch it on
Make now a restart, and it should be activated

# Install and set-up MOTION
I used for streaming the service called `motion`.  
Install it via 
```shell
sudo apt-get install motion -y
```
As I never used motion before, I followed this real great <a href="https://tutorials-raspberrypi.de/raspberry-pi-ueberwachungskamera-livestream-einrichten/">raspberry pi cam tutorial</a> (German). The only thing I changed was the `framerate` to 100 instead of 10.  

edit the motion.conf file:
```shell
sudo nano /etc/motion/motion.conf
```
turn the daemon on:  
`daemon on`
allow others to watch the stream in the network:  
`stream_localhost off`
set the target for your stream, I used the same as the blog above recommend me:  
`target_dir /home/pi/Monitor`

I set the `width 1280` and `height 720` and `framerate 100`

I uploaded my complete `motion.conf` <a href="https://github.com/joergi/tryouts/blob/main/raspberry-pi/streaming-cam/motion.conf">to my Github repo</a> 

Then, set the demon to yes:
```shell
sudo nano /etc/default/motion
```
set: `start_motion_daemon=yes`

now set the rights of the folder correct:
```shell
mkdir /home/pi/Monitor
sudo chgrp motion /home/pi/Monitor
chmod g+rwx /home/pi/Monitor
```
you can start it now with `sudo motion` 

# Use your local IP to stream to your browser:
find out over `ifconfig` what your local IP is.  
I can reach in my wifi then the stream under: http://192.186.1.175:8081   
Remember: the port 8081 was defined in the point above

![Alt text](https://raw.githubusercontent.com/joergi/blog/main/images/webcam-in-action.png "screenshot of the raspberrypi streaming to the my browser")

# start motion everytime you start your raspberry pi
to have it always running I used a cronjob:
```shell
$ crontab -e
```
and in the cronjob I save:
```shell
@reboot sudo motion
```
that made it working for me!

