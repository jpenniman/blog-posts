---
layout: post
title: Installing TimeTTracker MX2 on a Smartphone
date: 2008-02-08 12:52:00.000000000 -05:00
author: Jason M Penniman
excerpt: Installing TimeTTracker MX2 on a UT Starcom X5800 smartphone... Here's some background...
category: General-IT
---

ISSUE:  Installing TimeTTracker MX2 on a UT Starcom X5800 smartphone

Operating systems:

* Windows Vista Business 64-bit
* Windows Mobile 6.0

Here's some background...

I've been using TimeTTracker MX2 by R&amp;F Consulting, Inc. ( <a href="http://www.rfcons.com">http://www.rfcons.com</a> ) for quite some time now, and love it.  If you're a consultant, you have to take a look at their product.

If you are in a profession that requires you keep accurate record of your time and expenses, especially while on the go, then you also know the pain when such a tool suddenly stops working.

I'm using Windows Vista Business 64-bit on my laptop, and at one point had upgraded to WMDC 6.1.

My last mobile device was the UT Starcom XV6700 (Verizon Wireless version) running Windows Mobile 5.0.  TimeTTracker MX2 installed and sync'd flawlessly in this configuration.

Having some performance issues with the XV6700 itself, and unhappy with it's bulky size, I downsized to the UT Starcom X5800 (Verizon Wireless SMT5800), running Windows Mobile 6.0.  That caused two key incompatibility issues.

The issues:

1. There is an sync issue with WMDC 6.1 and CE 6.0.  Microsoft... well, at least support techs and developers at Microsoft, have acknowledged the issue (would you believe many users can't get outlook 2007 to sync with that configuration? ).  TimeTTracker MX2 specifically get an error on sync.  Specifics can be found in R&amp;F Consulting's Knowledge base at <a href="http://www.rfcons.com/index.php?id=70&amp;tx_knowledgebase_pi1%5Bcmd%5D=kb_viewOneArticle&amp;tx_knowledgebase_pi1%5Buid%5D=42&amp;cHash=830c68fd72">http://www.rfcons.com/index.php?id=70&amp;tx_knowledgebase_pi1[cmd]=kb_viewOneArticle&amp;tx_knowledgebase_pi1[uid]=42&amp;cHash=830c68fd72</a>
2. Windows Smartphones are security-locked, also termed application-locked.  This prevents unsigned software (more specifically, software not signed with an installed trusted certificate authority) to be installed on the device.  So, TimeTTracker MX2 mobile install fails with the following errors:

On the desktop side of the mobile install...

<img style="margin: 0pt 10px 10px 0pt; float: left; cursor: pointer;" src="/blog/images/tttmx2_error.jpg" alt="" id="BLOGGER_PHOTO_ID_5164705985899290402" border="0" />

On the device side...

“Installation was unsuccessful.  The program or setting cannot be installed because it does not have sufficient system permissions.”

The solution (well, _A_ solution)...

After chatting with R&amp;F Consulting, Verizon, UT Starcom, and Microsoft, here's what I found…

NOTE: **** This will void the warranty on your phone. ****

If you google for “unlocking windows mobile”, you find lots of info on unlocking phones, here's the one  that worked for me...

<a href="http://robertpeloschek.blogspot.com/2006/03/howto-application-unlock-your-windows.html">http://robertpeloschek.blogspot.com/2006/03/howto-application-unlock-your-windows.html

I followed Robert's instructions exactly.  The only hiccup I ran into was, after the reboot, the home screen was undefined.  I simply selected a home screen in settings and all is well.

After the unlocking, the install of TimeTTracker MX2 worked flawlessly.  The next step was downgrading WMDC to 6.0.  The issue here, is that the uninstaller leaves the executables, so they never get overwritten on a reinstall of 6.0, leaving you with 6.1.  Here's an article on how to remove WMDC 6.1 manually...

<a href="http://forum.soft32.com/pda/Sync-problem-WMDC-fixed-rolling-back-WMDC-ftopict79588.html">http://forum.soft32.com/pda/Sync-problem-WMDC-fixed-rolling-back-WMDC-ftopict79588.html</a>

Synchronization works perfectly!

I hope this helps.
