---
author: admin
date: "2011-11-20T11:48:32Z"
slug: google-music-cant-identify-your-computer-in-ubuntu
title: Google Music "could not identify your computer" in ubuntu
---

When I was trying to get Google Music working on my Ubuntu install, I was getting the following error:

![Ubuntu Error window stating "Login failed: Could not identify your computer, with a simple "Close" button](/images/bypost/google-music-cant-identify-your-computer-in-ubuntu/google-music-manager-error.png)

On my motherboard, I have two ethernet ports.  Doing an `ifconfig`, I saw something weird with my MAC addresses.

![](/images/bypost/google-music-cant-identify-your-computer-in-ubuntu/ifconfig-screenshot.png)

I'm only using `eth1`, but you can see that both of the ethernet ports are reporting the same MAC address of `aa:00:04:00:0a:04`.  I found [this post](http://www.fantaghost.com/2010/06/eth0-mac-address-fixed-on-aa0004000a04/) detailing that the DECNet tools is what causes the MAC address to be the same.  To remove them, run the following:

{{< highlight bash >}}
$ sudo apt-get remove dnet-common libdnet
{{< / highlight >}}

Reboot, and running `ifconfig` will show your ethernet ports mapped to their correct MAC address.  Now you should not have any problems running Google Music!