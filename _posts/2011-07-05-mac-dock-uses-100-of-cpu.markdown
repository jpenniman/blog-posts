---
layout: post
title: Mac Dock Uses 100% of CPU
date: 2011-07-05 07:20:00.000000000 -04:00
author: Jason M Penniman
image: /blog/images/blank-tile.png
excerpt: I noticed the fans in my Macbook Pro were running excessively.  When
  looking at Activity Monitor, I noted that the Dock was using 100% of the CPU.  A
  quick search on Google, and I found the culprit to be Parallels.  The root
  cause is an issue with the graphics API between OS X 10.6.8 and Parallels Desktop
  6.  I had indeed...
category: General-IT
---
I noticed the fans in my Macbook Pro were running excessively.  When looking at Activity Monitor, I noted that the Dock was using 100% of the CPU.  A quick search on Google, and I found the culprit to be Parallels.  The root cause is an issue with the graphics API between OS X 10.6.8 and Parallels Desktop 6.  I had indeed recently updated to 10.6.8 in preparation for Lion.

The details can be found in this Parallels KB article:

<a rel="nofollow" href="http://kb.parallels.com/en/111541">Mac Dock consuming 100% of CPU core after upgrade to Mac OS X 10.6.8

**Symptoms:** Dock consumes 100% of CPU

**Environment:** Mac OS X 10.6.8, Parallels 6.0.12090

**The Solution** Rather than downloading from the web site, I simply ran the "Check for Updates" on the Parallels application menu.