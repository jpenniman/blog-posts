---
layout: post
title: Using Linux Fulltime on my Surface Book
date: 2022-08-29
image: /blog/images/ubuntu-rider.png
author: Jason M Penniman
excerpt: As much as I do like Windows 11, I don't like how Windows and Office are now filled with ads and privacy really isn't front and center. Yes, you can turn a lot of data sharing off, but not all of it. A couple years ago I test-drove Linux as a development rig and was very impressed with the experience. So, I decided to take another look at Linux for daily use.
tags:
- linux
- ubuntu
- surface
---
As much as I do like Windows 11, I don't like how Windows and Office are now filled with ads and privacy really isn't front and center. Yes, you can turn a lot of data sharing off, but not all of it.

A couple years ago I test-drove Linux as a development rig and was very impressed with the experience. So, I decided to take another look at Linux for daily use. Linux was my primary OS for 5 years in the early 2000's.

## Distro

The last time I did this, I was using Fedora. I still like RedHat and derivatives, but this time I decided to go with Ubuntu 22.04 LTS. So far it works great on my Microsoft Surface Book 2 using the modifed linux-surface kernel. Even the pen works! So does detach. It plays nicer than Windows with my KVM. In Windows 11, if I undock and redock, I lose the external devices--e.g., monitor, mouse, and keyboard--and have to reboot the KVM. With Ubuntu, it's a seamless transition as expected.

## Dev Environment

Nearly all my development work is in dotnet and Java. I have a JetBrains Ultimate subscription, so firing up ToolBox and installing Rider, InteliJ, and DataGrip was a piece of cake.

Other tools I use that happen to be cross-platform:

- GitKraken
- Visual Studio Code
- Postman
- Draw.io
- Docker Desktop - yes, I know, docker-ce is available natively on Linux, but I like the developer experience of Docker Desktop.

With SQL Server supporting Linux and a Microsoft-official docker container, I can even still develop against SQL Server.

## Office/Productivity

The standard here has to be LibreOffice. Microsoft Office, though, is available on the web if absolutely needed. Some nice folks even created Electron apps for Office Web that integrate Outlook notifications with Gnome's notification center.

## Collaboration

Microsoft Teams on Linux wasn't bad. Yes, it was lacking, but allowed me to do everything I needed. Now that Teams on Linux is end-of-life, it's either the Web App, or some other product.

Zoom has a Linux client. Not that I'm a fan. By far the worst privacy policy out there for video conferencing.

Slack also has a Linux client.

## Cloud Storage

This one was a bit annoying. Since I have Office 365 subscriptions, one for personal, and one for business, all my documents are on OneDrive and OneDrive for Business. However, there is no official Linux Client.

I tried ExpanDrive, but it would crash when trying to add OneDrive--other providers seemed to work. I really liked what ExpanDrive had to offer, especially in that it behaves like OneDrive in its integration with File Manager and only downloading the off-line copy when you open the file. If they get it fixed, maybe I'll give them another go.

ODrive is another cross-platform, multi-cloud sync tool. It was just way too much work to get working properly. If I'm going to pay for a product, I want it to "just work".

Dropbox has a native Linux client with the same features as their Windows and Mac clients. If I stick with this Linux setup, I might just switch to Dropbox. It just seems like a waste of the 2TBs I have between my two OneDrive accounts (personal and business).

As long as the web apps are sufficient, I may just wait and see what the good folks at Proton do with ProtonDrive. If they end up with good OS support, I may just move all my email and cloud storage over to them.

## Sadly, still need Windows

Unfortunately, I still need Windows...

### Visual Studio

For all our customer projects that deploy to SQL Server, we use SQL Server Database projects and DACPACs.  Currently, Visual Studio for Windows is the only IDE that has full support for these projects. Even Visual Studio for Mac does not support them. I might be able to change up my workflows a bit. All our DACPACs are actually built and deployed by Azure Pipelines, never locally. Rider can open the SQL Project and I can add new scripts to it, I just need to remember to change the build property.  Another option would move to something like Fluent Migrator, but that's a lot or rework or a lot of projects just because I have a beef with Microsoft over their privacy practices.

### TurboTax

I use TurboTax for Business to do the business taxes. This is only available on Windows. I could just pay someone else to do my taxes.

## Virtualization

Since I still needed Windows for occasional tasks, I decided to just run a virtual machine.

VMware Workstation Pro was a no-go. It installed, but could not compile and install the kernel modules.

Good-ol` trusty Virtual Box to the rescue! So far, it "just works" and to my surprise is pretty snappy.

## Conclusion

I've been running this setup for a couple weeks now. Overall, I'm happy with this Ubuntu setup so far. I may try PopOS from System76. I'm hearing good things about the distro in terms of polish and stability beyond Ubuntu. I'm particularly interested in it's ability to switch between my Surface's integrated Intel GPU and dedicated NVidia GPU. I'm sure there is a utility to do the same with Ubuntu, but in PopOS it "just works" in order to support System76's own laptops.

I do some iOS development as well. For that, I use a Mac Mini. I may bite the bullet and consolodate back to one device and get a MacBook Pro, but I just can't justify the price and I really love my Surface Book. I use the pen all the time for note taking, drawing, and signing documents.

Cheers!
