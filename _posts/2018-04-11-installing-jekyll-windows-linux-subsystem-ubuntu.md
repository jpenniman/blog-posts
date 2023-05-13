---
layout: post
title: Installing Jekyll on Windows Subsystem for Linux - Ubuntu 16.04 
date: 2018-04-11
image: /blog/images/blank-tile.png
author: Jason M Penniman
excerpt: A quick guide on installing Jekyll on Windows Subsystem for Linux (formally Bash on Windows) Ubuntu 16.04 on Windows 10.
category: Development
tags:
- Jekyll
- Ubuntu
- Windows 10
- WSL
---

I prefer to run ruby stuff under Linux rather than Windows. I run Windows 10, so will be using the Ubuntu 16.04 image for the Windows Subsystem for Linux. Though these steps should work in a full installation of Ubuntu as well.

Install Ruby
```
sudo apt install ruby ruby-dev ruby-bundler
```

Install dev dependencies
```
sudo apt install gcc make build-essentials
```

Install Jekyll
```
sudo gem install jekyll
```

Install bundler dependencies
```
bundler install
```

I have the best luck running jekyll with `bundler exec`
```
bundler exec jekyll build
bundler exec jekyll serve
```

Happy blogging!