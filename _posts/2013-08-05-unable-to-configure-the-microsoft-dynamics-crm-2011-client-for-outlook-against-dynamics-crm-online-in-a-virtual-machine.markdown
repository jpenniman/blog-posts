---
layout: post
title: Unable to Configure the Microsoft Dynamics CRM 2011 Client for Outlook against
  Dynamics CRM Online in a Virtual Machine
date: 2013-08-05 06:41:00.000000000 -04:00
image: /blog/images/blank-tile.png
author: Jason M Penniman
excerpt: 'I came across the error in this article when trying to configure the CRM
  client for Outlook...<br /><span style="background-color: white; color: #333333;
  font-family: ''Segoe UI'', Arial, Verdana, Tahoma, sans-serif; font-size: 13px;">Error
  connecting to URL: https://myogranization.crm.dynamics.com/XRMServices/2011/Discovery.svc
  Exception: System.ArgumentNullException: Value cannot be null.</span><br /><span
  style="background-color: white; color: #333333; font-family: ''Segoe UI'', Arial,
  Verdana, Tahoma, sans-serif; font-size: 13px;"><br /></span>See this kb article...
  <a rel="nofollow" href="http://support.microsoft.com/kb/2498892">http://support.microsoft.com/kb/2498892</a><br
  />For me, the installation was fine....'
category: General-IT
---
I came across the error in this article when trying to configure the CRM client for Outlook...<br /><span style="background-color: white; color: #333333; font-family: 'Segoe UI', Arial, Verdana, Tahoma, sans-serif; font-size: 13px;">Error connecting to URL: https://myogranization.crm.dynamics.com/XRMServices/2011/Discovery.svc Exception: System.ArgumentNullException: Value cannot be null.</span><br /><span style="background-color: white; color: #333333; font-family: 'Segoe UI', Arial, Verdana, Tahoma, sans-serif; font-size: 13px;"><br /></span>See this kb article... <a rel="nofollow" href="http://support.microsoft.com/kb/2498892">http://support.microsoft.com/kb/2498892</a><br />For me, the installation was fine. &nbsp;The issue was my environment. &nbsp;I use a Mac and run Parallels to host my Windows 8 VM. &nbsp;By default, the user home directory when using Parallels is the home directory on the Mac, not in the Windows VM. &nbsp;So, to Windows and your Windows applications (in this case, the CRM Client and Outlook), this appears, and behaves, as a remote share. &nbsp;CRM Client for Outlook stores the off-line data files in the user's AppData folder in their profile.<br />The CRM Client Plug-In uses SQL Server CE 3.5. &nbsp;This version of SQL Server CE does not support access on remote shares, so the sync process fails when it tries to create and write to the data files.<br />The solution is to use SQL Server CE 4.0 instead. &nbsp;This knowledge base article goes through the steps of installing SQL Server CE 4.0 and configuring the CRM Client plug-in to use version 4...<br /><a rel="nofollow" href="http://support.microsoft.com/kb/2616319">http://support.microsoft.com/kb/2616319</a><br />Hope this saves some headaches. Cheers!<p><strong>Read more</strong>&nbsp;<a class="rssreadon" rel="external" title="Unable to Configure the Microsoft Dynamics CRM 201" href="http://jpenniman.blogspot.com/2013/08/unable-to-configure-microsoft-dynamics.html" >http://jpenniman.blogspot.com/2013/08/unable-to-configure-microsoft-dynamics.html</a></p>
