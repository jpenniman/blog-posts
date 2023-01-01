---
layout: post
title: Hypervisor and Host Upgrade at Milestone
date: 2011-01-03 08:53:00.000000000 -05:00
author: Jason M Penniman
excerpt: <p>It was time to upgrade the virtual machine host at Milestone.  Since most of our critical systems are hosted, the server(s) here didn't need to be anything to extreme. The servers in house are development and test lab related. The goals for the upgrade were to:</p><div><ul><li>Upgrade the hardware</li><li>Go Green</li><li>Experiment with various hypervisors to see which ones can be used in SOHO environments with low investment, low TCO, high ROI, and little or no headache when it comes to install and maintenance.</li></ul></div>
category: General-IT
---
<p>It was time to upgrade the virtual machine host at Milestone.  Since most of our critical systems are hosted, the server(s) here didn't need to be anything to extreme.  The servers in house are development and test lab related.  The goals for the upgrade were to:</p>
<div>
<ul>
<li>Upgrade the hardware</li>
<li>Go Green</li>
<li>Experiment with various hypervisors to see which ones can be used in SOHO environments with low investment, low TCO, high ROI, and little or no headache when it comes to install and maintenance.</li>
</ul>
</div>

<h2>Hardware Selection</h2>

<div>I first took a look at Apple's Mac Mini and Mac Mini Server.  These machines are certainly an upgrade from what I have and meet my green requirement with a mier 85W max power consumption, but they only have a dual core processor.  In most cases, I would say they are perfect for the SOHO environment.  Because we would be spinning up several virtual machines, I wanted to kick it up a notch.  I was also concerned about the availability of drivers with the Mac Mini, since I'd be wiping OS X in favor of a bare-metal hypervisor.</div>
<div></div>
<div>After looking at several systems, I settled on something a bit non-traditional--a Dell Inspiron Zino HD, Dell's answer to the Mac Mini.  I know, I know... not "server grade", but as I said, theses systems aren't critical... mostly development and testing servers, and I was able to get quad core, 8GB RAM, and only 90W max power consumption compared to the 400W of the current dinosaur at nearly half the cost--a very nice happy medium for my needs.<br />
<div></div>
<div>Under normal circumstances, hardware would have been scrutinized a little better.  Anytime you're looking for server hardware (or any other for that matter), you should always consider compatibility, stability, fault tolerance (in the case of servers), upgradeability, and serviceability.</div>
<div></div>
<div>

<h2>Selecting the Hypervisor</h2>

</div>
<div>The next, I wanted to choose a new hypervisor.  We had been using VMware ESXi in the past (and VMware Server before that), but decided to explore other options.  So I looked at:</div>
<div>
<ul>
<li>VMware ESXi</li>
<li>Citrix XenServer 5.6</li>
<li>Microsoft Hyper-V 2008 R2</li>
</ul>
</div>

<h3>VMware ESXi</h3>

<div>The free version is attractive and sufficient for our needs at Milestone.  But, if I was going to continue to use it, I had to find a GUI based manager for it if it was going to meet my requirements.  I found an interesting open source tool called OpenQRM (<a href="http://www.openqrm.com/">http://www.openqrm.com/</a>).  It's a web-based tool written in php that can manage VMware ESXi, Citrix XenServer, Xen, and various other platforms.</div>
<div></div>

<h3>Citrix XenServer</h3>

<div>The free version has a bit more features than VMware ESXi's free version.  It has a GUI (windows based) and live migrations.  Even the enterprise licensing is less expensive than VMware.  With an add-on pack, XenCenter can also be used to manage your Hyper-V hosts.</div>
<div></div>

<h3>Microsoft Hyper-V</h3>

<div>Free, so long has you have Windows Server Standard, Enterprise, or Datacenter.  Microsoft's licensing can be a bit confusing from there as to what license requirements are needed for the VM's, but your Microsoft rep can help you through that.  One advantage to Hyper-V is driver availability.  While linux and BSD have broad driver availability, it's still not as broad or readily available as Windows drivers.  From a cost stand point, Hyper-V is more attractive for growth, should you needs grow to the point of needing the enterprise solutions/licenses from VMware or Citrix.</div>
<div></div>
<div>The biggest difference between the three is the virtualization approach.  VMware is full virtualization.  The virtual machines are completely abstracted from the hardware, and the hardware is vitualized.  In other words, your virtual machines see the "VMware SATA controller" and the "VMware Gigabit NIC", not the actual, say Intel, hardware for your server.  The big advantage is it makes it easier to move a VM from host to host, and there is less issues with drivers.</div>
<div></div>
<div>In contrast, XenServer and Hyper-V use paravirtualization.  Virtual machine guests use a "modified" kernel/driver set that allows the guest to communicate with the hypervisor for resource management, but they sit on the raw hardware.  This is similar to the "cell" concept IBM and Sun have used for years when virtualizing mainframes and mini-computers.  The advantage to paravirtualization is speed; there is no abstraction layer, so in theory, it's faster.  A disadvantage is the modified kernels/drivers.  Windows handles it pretty well, since it's not a direct mod to OS, just drivers and services.  Linux, however, is not as forgiving.  Modifying the kernel is simple and srtaight forward, the first time.  The headache comes when you want/need to update the kernel.  This conflicts with the mods injected by the hypervisor tools and causes a kernel panic on reboot.  Not terrible to work around, just something to be aware of when working with XenServer or Hyper-V.  Because your sitting on hardware, you need to be careful when moving a VM from host to host.</div>
<div></div>


<h2>The Chosen One</h2>

<div>Initially, I settled on Citrix XenServer, but when I couldn't even install it because it didn't see the network adapter on the Dell Zino, nor was I wanting the hastle of finding a linux driver that would work with the their custom kernel.  One of my goals for the experiment was to find something "easy"... something simple to get going and maintain for SOHO and small business customers.  So, Microsoft Hyper-V 2008 R2 it is.</div>
<div></div>
<div>There are two main options for installing Hyper-V:</div>
<div><ol>
<li>Hyper-V with Core installation (Microsoft has a DVD specifically for this to simplify installation)</li>
<li>Full Windows Server installation with the Hyper-V role enabled</li>
</ol></div>
<div>I tried both.  Starting with a Core install.  It kept the host side lighter, as the only OS components installed were those required to run and manage Hyper-V.  The downside for me was remote management.  I have a Mac running Windows 7 in a VM on Parallels.  My virtual network setup for it is NAT, for many reasons.  It turned out that because of this, the asynchronous communication required by the tools didn't play nice.  Though, any Windows 7 machines on the network configured easily and worked perfectly.</div>
<div></div>
<div>Because I needed the ability to manage this from my Mac (one reason I really liked OpenQRM), I finalized on a full installation of Windows Server with the Hyper-V role enabled.  With this approach, I can RDP to the server to manage it.</div>
<div><br />
<h2>Further Lab Work</h2>
I did like the other options and was disapointed that the hardware didn't play nice with XenServer.  So, I am going to put together a lab machine in the near future and try out both ESXi and XenServer with OpenQRM.  Reading the literature, these look like very viable low-cost, low TCO, and high ROI solutions for virtualizing.<br /><br /></div>
<div>

<h2>Related Posts</h2>

<a href="http://jpenniman.blogspot.com/2011/01/installing-centosredhat-55-on-hyper-v.html">Installing CentOS/Red Hat on Hyper-V</a><br /><br /><br /></div>
</div>
<div><img src="https://blogger.googleusercontent.com/tracker/7838145634981365715-4856087659581718264?l=jpenniman.blogspot.com" border="0" width="1" height="1" /></div>
