---
title: "Deutsche Bahn,Docker and Wifi in an ICE and VPN problems"
description: "As a developer with installed Docker you could have a problem connecting to the WiFi in an ICE, the so called WiFi on ICE, because they use the same IP Range."
date: 2019-09-06
---

The Deutsche Bahn is providing free wifi for some years now.
It works for most of the people pretty well, but if you are a developer you could have some issues. I was suffering a lot with this, this is why I'm posting it, to help some of you

As you can see, the WiFi is running on the IP 172.18.xxx

<img class="alignnone size-full wp-image-453" src="https://joergi77.files.wordpress.com/2019/09/wp_wifi_on_ice.png" alt="wp_wifi_on_ice" width="557" height="470" />

If you are running docker it can happen, that you have in your ifconfig a bridge already on this IP:

<pre>$ ifconig

br-bcc4b1218247: flags=4099<UP,BROADCAST,MULTICAST> mtu 1500
inet 172.18.0.1 netmask 255.255.0.0 broadcast 172.30.255.255
ether 02:42:55:fb:67:74 txqueuelen 0 (Ethernet)
RX packets 0 bytes 0 (0.0 B)
RX errors 0 dropped 0 overruns 0 frame 0
TX packets 0 bytes 0 (0.0 B)
TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0
</pre>


you can delete this bridge by using the following commands:
<pre>sudo ip link set br-bcc4b1218247 down
sudo brctl delbr br-bcc4b1218247</pre>
This should work, that you can connect then to the WiFi on Ice

UPDATE: 12.09.2023
### Loginpage is not opening
Normally the login page should update automatically, but soemtimes (on Linux) it isn't.  <br/>
Go to <a href="https://login.wifionice.de" target="_blank">https://login.wifionice.de</a> for opening it in your browser 

UPDATE:
you can also delete all docker networks easily with:
<pre>$ docker network prune</pre>

I had the problem, that I could not reach some websitse in our company network,when I'm using a VPN.
The ping to our internal subdomain was working, but I could not reach it in my browser:
<pre>$ ping subdomain.intern.ourdomain.net
PING something.elb.amazonaws.com (10.x.x.x) 56(84) bytes of data.
64 bytes from 10.x.x.x (10.x.x.x): icmp_seq=1 ttl=254 time=108 ms
64 bytes from 10.x.x.x (10.x.x.x): icmp_seq=2 ttl=254 time=192 ms
64 bytes from 10.x.x.x (10.x.x.x): icmp_seq=3 ttl=254 time=697 ms
64 bytes from 10.x.x.x (10.x.x.x): icmp_seq=4 ttl=254 time=344 ms
64 bytes from 10.x.x.x (10.x.x.x): icmp_seq=5 ttl=254 time=52.2 ms
64 bytes from 10.x.x.x (10.x.x.x): icmp_seq=6 ttl=254 time=54.3 ms
64 bytes from 10.x.x.x (10.x.x.x): icmp_seq=7 ttl=254 time=267 ms
64 bytes from 10.x.x.x (10.x.x.x): icmp_seq=8 ttl=254 time=181 ms
64 bytes from 10.x.x.x (10.x.x.x): icmp_seq=9 ttl=254 time=141 ms
--- something.elb.amazonaws.com ping statistics ---
9 packets transmitted, 9 received, 0% packet loss, time 13011ms</pre>
the ifconfig for the vpn tunnel and the wifi were showing the following:
<pre>....
tun0: flags=4305<UP,POINTOPOINT,RUNNING,NOARP,MULTICAST> mtu 1500
....
wlp4s0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST> mtu 1440
.....</pre>
My colleague, who is traveling every day by the ICE of Deutsche Bahn told me to set down the MTU, then everything was working:
<div>
<div class="">
<div id="post_ehiojfyp4jbbxqbf9nubrwtt8r" class="a11y__section post same--user same--root  " role="listitem" aria-label="marc.bruessel at 8:21 AM Friday, September 6 wrote, ip link set wlp4s0 mtu 500">
<div id="postContent" class="post__content " role="application" aria-hidden="true">
<div>
<div>
<div id="ehiojfyp4jbbxqbf9nubrwtt8r_message" class="post__body   ">
<div class="post-message post-message--collapsed">
<div class="post-message__text-container">
<div id="postMessageText_ehiojfyp4jbbxqbf9nubrwtt8r" class="post-message__text">
<pre>$ sudo ip link set wlp4s0 mtu 500</pre>
<pre>$ sudo ip link set tun0 mtu 500</pre>
Updated 24.09.2019:
It seems that, if you have the network-manager running, it can happen, that you have to rerun this process, every time you try to log into the wifi @ ice
