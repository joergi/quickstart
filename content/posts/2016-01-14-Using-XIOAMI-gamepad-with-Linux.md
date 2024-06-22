---
title: "Using XIOAMI gamepad with Linux"
description: "What you have to do on Linux to get the Xiaomi gamepad running - happy gaming"
date: 2016-01-14
---


<a href="http://xiaomi-mi.com/uploads/CatalogueImage/2_13621_1429630637.jpg"><img class=" alignright" src="http://xiaomi-mi.com/uploads/CatalogueImage/2_13621_1429630637.jpg" alt="XIAMOI Gamepad" width="100" /></a>I have the <a href="http://xiaomi-mi.com/more/mi-game-controller-bluetooth/" target="_blank">XIOAMI gamepad</a>, a cheap gamepad which looks like the classical XBOX gamepad
When I used it first with my Fire TV Stick, it worked like a charm. I wanted to try it with my Linux and with Steam.

When i used it in a game it was totally uncalibrated. Pressing the joystick to the top position had no effect at all on my game character.

<img class="  wp-image-196 alignright" src="https://joergi77.files.wordpress.com/2016/01/jscalib.png" alt="jscalib" width="248" height="108" />So, i had to calibrate with a small programm, which I install on the shell with:
```bash
sudo aptitude install jstest-gtk
```
start it with:
```bash
jstest-gtk
```

you should see the following pictures...
<ol>
	<li>press the <em>calibration</em> button</li>
	<li>press the <em>start calibration</em> button</li>
	<li>rotate the joystick (all joysticks on your gamepad in a circle</li>
	<li>move the joystick to<em> up - middle - down -middle - left -middle - right - middle</em></li>
	<li>press the OK button.</li>
	<li>save the profile</li>
</ol>
after than restarting the Steam game, it works perfect.
