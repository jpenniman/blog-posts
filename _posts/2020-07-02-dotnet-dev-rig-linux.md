---
layout: post
title: A .Net Development Rig in Linux
date: 2020-07-02
image: /blog/images/ubuntu-rider.png
author: Jason M Penniman
excerpt: After playing with Rider for a while, I decided to play around with Linux again to see what .Net development is like on Linux rig in 2020. So I fired up an Ubuntu 20.04 LTS VM and started installing.
tags:
- dotnet
- linux
- ubuntu
---

After playing with Rider for a while, I decided to play around with Linux again to see what .Net development is like on Linux rig in 2020. So I fired up an Ubuntu 20.04 LTS VM and started installing.

## .Net Core

After updating the OS with all the latest patches, I installed both .Net Core 2.1 LTS and 3.1 LTS following Microsoft's instructions.

``` bash
wget https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb

sudo apt-get update; \

sudo apt-get install -y apt-transport-https && \
sudo apt-get update && \
sudo apt-get install -y dotnet-sdk-3.1 dotnet-sdk-2.1
```

## JetBrains IDEs

I have a JetBrains Ultimate subscription, so my next setup was installing JetBrains Toolbox. There is zero documentation on doing so, but it's pretty simple. Just download the tar ball from JetBrains, extract it to a directory of your choosing, and run it. Using the ToolBox, I installed my IDEs--Rider, IntelliJ, and DataGrip. It's worth noting that JetBrains also provides snapcraft packages for it's products (but not the Toolbox) so they are available from the Software utility in Ubuntu.

## Mono, Because Why Not?

For the halibut, I decided to install the latest Mono SDK to play with some of my multi-target projects.

``` bash
sudo apt install gnupg ca-certificates 
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF 
echo "deb https://download.mono-project.com/repo/ubuntu stable-focal main" | sudo tee /etc/apt/sources.list.d/mono-official-stable.list 
sudo apt update
sudo apt install mono-complete
```

## Git and GitKraken

Let's not forget git:

``` bash
sudo apt install git
```

I like a gui client for git, so I headed over to the App Store install GitKraken.

## Visual Studio Code

And of course Visual Studio Code, also available in the App Store.

## Slack and Microsoft Teams

For collaboration, back in the App Store, I installed Slack. Because I can, I also installed Microsoft Teams from Microsoft's preview channel.

## Conclusion

Voila!

![rider-ubuntu](/blog/images/ubuntu-rider.png)

Much easier than it used to be since Canonical created their SnapCraft App Store. It actually took me less time than it usually does to build a Windows machine.
I haven't worked in this environment a lot yet, but from what I have done, the experience is snappy and clean. All the tools behave and are as smooth as they are on my Windows laptop, except for Teams. Teams in still in preview, isn't quite at feature parity yet, and isn't 100% stable, but hey, it's an official Microsoft Office App on Linux! So that's something.

Cheers!
