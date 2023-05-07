---
layout: post
title: Permission Denied error saving in Draw.io on Linux
date: 2022-09-10
author: Jason M Penniman
excerpt:  I installed the desktop version of Draw.io on my Ubuntu system using the Ubuntu Store (snap).  When trying to save files to my Windows partition (or mounted device other than / (root), a received the following error... "Error opening directory. Permission denied."
tags:
- linux
- drawio
- snap
---
I installed the desktop version of Draw.io on my Ubuntu system using the Ubuntu Store (snap).  When trying to save files to my Windows partition (or mounted device other than / (root), a received the following error:

![Error opening directory: Permission denied.](../images/Screenshot%20from%202022-09-10%2012-20-23.png)

After some digging, I found that the permission for the snap defaulted external devices as disallowed.

![Permission disabled](../images/Screenshot%20from%202022-09-10%2012-20-50.png)

Had to simply enable "Read/write files on removable storage devices", and viola!

![fixed](../images/Screenshot%20from%202022-09-10%2012-20-56.png)

Hope this helps other seeing the same behave from this or other snaps.

Cheers!
