---
layout: post
title: Installing Any Executable as a Windows Service
date: 2009-08-21 12:07:00.000000000 -04:00
author: Jason M Penniman
excerpt: I'll clean this up later, but in short...
category: General-IT
---
I'll clean this up later, but in short:

1. Download SrvAny.exe as part of the Windows server resource kit here: <a href="http://www.microsoft.com/downloads/details.aspx?familyid=9d467a69-57ff-4ae7-96ee-b18c4790cffd&amp;displaylang=en">http://www.microsoft.com/downloads/details.aspx?familyid=9d467a69-57ff-4ae7-96ee-b18c4790cffd&amp;displaylang=en</a>
2. Run: InstSrv.exe yourservicename "d:\pathtosrvany\svrany.exe"
3. Edit the registry entry for the newly created service and add a "Parameters" key called "Application".  Set the value to the full path of your executable.
