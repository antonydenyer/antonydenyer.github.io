---
layout: post
title: "Samsung Chromebox with XBMC"
date: 2012-07-09 18:07
comments: true
redirect_from: '/blog/2012/07/09/samsung-chromebox-with-xbmc/'
categories: [Chrombox,Google I/O]
---

At google IO 2012 every delegate got a free [Samsung Chromebox][1]. Personally I already have a laptop, desktop, tablet and smart phone. Why would I need a cut down desktop? It would probably be great for your Nan who has no idea what they're doing and just wants to email the grandkids.
So what should I do with this piece of hardware? Well my xbox classic is struggling to play some high def media files, it is over 10 years old, how about I use my free chromebox.

Enable Developer Mode Essentially flip a switch and erase all user data. 

[Here's how.][2]
Install ChrUbuntu I only had one problem following these 

[instructions][3], me, I didn't follow them. You must leave your box in developer mode, meaning that boot time takes almost a minute because of the developer mode warning.

Install XBMC

[XBMC has been accepted into debian.][4]  
```
sudo apt-get install xbmc
```
 Install VNC and SSH Invaluable, otherwise you'll be plugging in your keyboard and mouse every time you get a problem.&nbsp; 

[Install SSH][5]  
[Enable Remote Desktop Connections][6]

Run xbmc on startup I had a slight issue with xbmc starting just before the window manager had fully started. This was causing xbmc to start in a windowed state. Instead I wrote a small script to pause for 2 seconds then start xbmc.
```
#!/bin/sh  
sleep 5  
xbmc 
```
chmod +x and then add the file to [startup programs][7].
## Summary 
In total it took me about 90 minutes. My main annoyance is the beep you get on startup. I connected it to my TV through the dvi port which only supports 720p. I tried using display port to hdmi with audio but couldn't get any audio coming into my TV. No doubt this wouldn't be a problem with external audio. 

Would I recommend buying one explicitly for this purpose, no, unless the price tag is brought down significantly.

 [1]: http://www.samsung.com/uk/consumer/pc-peripherals/chrome-devices/chrome-devices/XE300M22-A1VUK
 [2]: http://dev.chromium.org/chromium-os/developer-information-for-chrome-os-devices/samsung-sandy-bridge
 [3]: http://chromeos-cr48.blogspot.co.uk/2012/04/chrubuntu-1204-now-with-double-bits.html
 [4]: http://xbmc.org/theuni/2012/04/10/xbmc-accepted-into-debian/
 [5]: https://help.ubuntu.com/10.04/serverguide/openssh-server.html
 [6]: http://www.makeuseof.com/tag/ubuntu-remote-desktop-builtin-vnc-compatible-dead-easy/
 [7]: https://help.ubuntu.com/community/AddingProgramToSessionStartup  
