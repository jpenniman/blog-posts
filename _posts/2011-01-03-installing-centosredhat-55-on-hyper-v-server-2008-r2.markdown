---
layout: post
title: Installing CentOS/RedHat 5.5 on Hyper-V Server 2008 R2
date: 2011-01-03 08:30:00.000000000 -05:00
image: /blog/images/blank-tile.png
author: Jason M Penniman
excerpt: CentOS/Red Hat 5.5 on Hyper-V R2 seems to work quite well.  The trick is to download the Linux Integration Services from Microsoft and follow the install guide before creating the VM.  The guide provides specific installation guidelines for creating/configuring the VM and installing the OS itself...
category: General-IT
---
CentOS/Red Hat 5.5 on Hyper-V R2 seems to work quite well.  The trick is to download the Linux Integration Services from Microsoft and follow the install guide before creating the VM.  The guide provides specific installation guidelines for creating/configuring the VM and installing the OS itself.

At the time of this writing, the latest version can be found here:<br /><br />Linux Integration Services v2.1 for Windows Server 2008 Hyper-V R2

<a href="http://www.microsoft.com/downloads/en/details.aspx?FamilyID=eee39325-898b-4522-9b4c-f4b5b9b64551&amp;displaylang=en">http://www.microsoft.com/downloads/en/details.aspx?FamilyID=eee39325-898b-4522-9b4c-f4b5b9b64551&amp;displaylang=en</a>

...but please be sure you download the latest version at the time of your reading this.

## Creating The Virtual Machine

When creating the virtual machine, **use the "Legacy Network Adapter"**.  To reduce any issues, I recommend making this the only network adapter configured.  CentOS, actually, any non-windows server OS or Windows versions prior to 2003, will not see the virtual "Network Adapter" until integration services are installed.  Once integration services are installed, we can reconfigure the machine.  I'll walk through that later in this post.

## Installing CentOS/Red Hat

When installing the OS, make sure you choose to install the "Development Libraries" and "Development Tools".  The Linux Integration Services need to be compiled for your kernel, so the kernel-devel, kernel-headers, and gcc need to be installed.  I also installed VNC Server and SSH for remote management.

Another caveat... the mouse doesn't work for CentOS in the Hyper-V console window even after integration services are installed.  During the firewall setup, I ensured SSH was enabled and added the VNC Server port -- 5901:tcp

Installation Summary:

* Development Libraries
* Development Tools
* VNC Server
* SSH

Since I had no mouse, and my host is headless anyway, I used SSH for the remaining steps.

After the installation is complete, verify that required packages are installed:

```
# yum list kernel*

kernel.x86_64                         2.6.18-194.26.1.el5              installed
kernel-devel.x86_64                   2.6.18-194.26.1.el5              installed
kernel-headers.x86_64                 2.6.18-194.26.1.el5              installed
```
You should see your kernel and the devel and header packages for that kernel.  The versions should match.

For warm and fuzzies, you can verify the running kernel.  Version numbers should match:

```
# uname -a<
```

If the devel and headers packages are not installed, you can install them now:

```
# yum install kernel-devel kernel-headers
```

Then reverify.

If you want the latest updates, and it is always recommended you do so, now is the time to do it:

```
# yum update
```

Then reboot:

```
# reboot
```

## Installing Linux Integration Services

Attach the ISO in the Hyper-V manager.

Create a mount point for the cd and mount it:

```
# mkdir /mnt/cdrom
# mount /dev/cdrom /mnt/cdrom
```

Create a directory to store the installation files and copy the contents of the cd to it:

```
# mkdir /opt/linux_ic_v21_rtm
# cp –R /mnt/cdrom/* /opt/linux_ic_v21_rtm
```

Make and Install the drivers and reboot:

```
# cd /opt/linux_ic_v21_rtm/
# make
# make install
```

If you are running 64-bit CentOS, also install the time extentions:

```
# yum install adjtimex
```

After Integration Services are installed (and time extentions are installed on 64-bit), reboot:

```
# reboot
```

I like to do the reboot to verify that I'm going to get a clean boot with no failures and not panics.  When CentOS is loading, show the details of the boot and watch the log.  All should go well.

## Configure The Synthetic Network Adapter

Once CentOS is done booting, shutdown so we can reconfigure the virtual networking:

```
# shutdown now
```

Open the virtual machine settings in Hyper-V Manager.  Remove the "Legacy Network Adapter" and add the synthetic "Network Adapter".

Power up the virtual machine.  Again, watch the boot details.  The ethernet interfaces should come up normally.

At this point you should have a clean running CentOS VM on Hyper-V.  Let's verify the networking interface:

```
# ifconfig
 seth0     Link encap:Ethernet  HWaddr 00:15:5D:01:0A:04
 inet addr:192.168.1.158  Bcast:192.168.1.255  Mask:255.255.255.0
 inet6 addr: fe80::215:5dff:fe01:a04/64 Scope:Link
 UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
 RX packets:219 errors:0 dropped:0 overruns:0 frame:0
 TX packets:115 errors:0 dropped:0 overruns:0 carrier:0
 collisions:0 txqueuelen:1000
 RX bytes:43009 (42.0 KiB)  TX bytes:15612 (15.2 KiB)
```

You should see an entry to "seth0".  This is the synthetic network adapter provided by hyper-v.  It should have a valid MAC address and IP address.  A simple nslookup, ping, or yum query will verify network connectivity.

```
# ping google.com
```

This is the point where I recommend configuring the network in CentOS.  In my case, I want static IP addresses for all my servers, so by running setup and going into the "Network Configuration", I can set the dns name and all IP information.

```
# setup
```

Changes can be verified:

```
# ifconfig
```

It also can't hurt to reboot.

## Conclusion

If you follow the instructions provided by Microsoft, it's actually quite painless.  Stray from the steps and/or prerequisites, and you're in for a day or two of headaches.

As mentioned, the mouse does not work in the Hyper-V console window.  The good news is this limitation seems to be isolated to the Hyper-V Manager only.  VNC connections to the server work well and have full GUI support, including the mouse.
