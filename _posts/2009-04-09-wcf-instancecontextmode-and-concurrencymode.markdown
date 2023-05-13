---
layout: post
title: WCF InstanceContextMode and ConcurrencyMode
date: 2009-04-09 14:28:00.000000000 -04:00
image: /blog/images/blank-tile.png
author: Jason M Penniman
excerpt: I recently ran into a concurrency issue with WCF on a customer's system.  I'll get into the details later (though not too much detail... NDA ;) ), but it involved some testing of the various combinations of InstanceContextMode and ConcurrencyMode settings for a WCF service...
category: Development
---

I recently ran into a concurrency issue with WCF on a customer's system.  I'll get into the details later (though not too much detail... NDA ;) ), but it involved some testing of the various combinations of InstanceContextMode and ConcurrencyMode settings for a WCF service.  The table below is an attempt to clarify the results...


<table border="1" cellpadding="3" cellspacing="0" width="100%"><tbody><tr><td width="33%"><span>InstanceContextMode<br /></span></td><td width="33%"><span>ConcurrencyMode<br /></span></td><td width="33%"><span>Resulting Behaviour<br /></span></td></tr><tr><td width="33%"><span>PerCall<br /></span></td><td width="33%"><span>Single<br /></span></td><td width="33%"><ul><li><span>New InstanceContext created per call</span></li><li><span>Calls processed concurrently, because each call gets it's own instance<br /></span></li></ul></td></tr><tr><td width="33%"><span><br /></span></td><td width="33%"><span>Multiple<br /></span></td><td width="33%"><span>Same as with ConcurrencyMode.Single<br /></span></td></tr><tr><td width="33%"><span>PerSession<br /></span></td><td width="33%"><span>Single<br /></span></td><td width="33%"><ul><li><span>New InstanceContext is created for each client proxy connection</span></li><li><span>Each call is pooled and processed serially in a FIFO manner</span></li></ul></td></tr><tr><td width="33%"><br /></td><td width="33%"><span>Multiple<br /></span></td><td width="33%"><ul><li><span>New InstanceContext is created for each client proxy connection<br /></span></li><li><span>Each call is processed as soon as it arrives<br /></span></li><li><span>A single instance of the proxy is required for concurrent processing from a single client<br /></span></li></ul></td></tr><tr><td width="33%"><span>Singlton<br /></span></td><td width="33%"><span>Single<br /></span></td><td width="33%"><ul><li><span>Single instance of an InstanceContext<br /></span></li><li><span>Each call is pooled and processed serially in a FIFO manner</span></li></ul></td></tr><tr><td width="33%"><span><br /></span></td><td width="33%"><span>Multiple<br /></span></td><td width="33%"><ul><li><span>Single instance of an InstanceContext<br /></span></li><li><span>Each call is processed as soon as it arrives<br /></span></li><li><span>The developer must manage thread-safety<br /></span></li></ul></td></tr></tbody></table>


It's also worth noting that if you are using ServiceHost in an executable assembly, that the Main() method not be marked with the single threaded appartment attribute [System.STAThreadAttribute()].  This causes the application to be single threaded, and the additional threads will not be created regardless of the ConcurrencyMode.Multiple setting.  Rather, it should be decorated with the multi-threaded appartment attribute [System.MTAThreadAttribute()].
