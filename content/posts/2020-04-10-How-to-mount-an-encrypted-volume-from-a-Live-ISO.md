---
title: "How to mount an encrypted volume from a live ISO"
description: "if you desstroyed the boot section on your Linux computer and you want to access your data, you will need to encrypt the volume from a live ISO"
date: 2020-04-10
---

If you destroyed somehow your Linux system, which has an encrypted volume, and you want to log into this, but booting is not working, boot from a live ISO, and type:
<pre>$ lsblk -f</pre>
you will see sda1 oder sdb1 as your encrypted system
<img class="alignnone size-full wp-image-499" src="https://joergi77.files.wordpress.com/2020/04/encrypted.png" alt="encrypted" width="806" height="383" />

this decrypt it and mount it (for example sda1
<pre>$ sudo cryptsetup luksOpen /dev/sda1 crypted_sda1
Enter passphrase for /dev/sda1:</pre>
after entering the passphrase you can see that this is now decrypted<img class="alignnone size-full wp-image-500" src="https://joergi77.files.wordpress.com/2020/04/decrypted.png" alt="decrypted" width="944" height="94" />

you are now able to mount the decrypted volume:
<pre>$ sudo mount /dev/mapper/crypted_sda1 /mnt</pre>
<img class="alignnone size-full wp-image-503" src="https://joergi77.files.wordpress.com/2020/04/mounted.png" alt="mounted" width="938" height="93" />
As you see here, the volume is now mounted to /mnt

You can then log into your encrytped volume by manjaro-chroot (example for Manjaro system)
<pre>$ sudo manjaro-chroot /mnt</pre>
(i read that non Manjaro users use `mhwd-chroot` but I haven't tried it)

As you can see, you are now in your own filesystem.
<img class="alignnone size-full wp-image-504" src="https://joergi77.files.wordpress.com/2020/04/chroot.png" alt="chroot" width="552" height="52" />
You are now able to install new packages or delete old ones with `apt install xxx` or `pacman -S xxx`
