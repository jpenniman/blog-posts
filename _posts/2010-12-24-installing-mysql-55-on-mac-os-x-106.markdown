---
layout: post
title: Installing MySQL 5.5 on Mac OS X 10.6
joomla_id: 34
joomla_url: installing-mysql-55-on-mac-os-x-106
date: 2010-12-24 08:25:00.000000000 -05:00
image: /blog/images/blank-tile.png
author: Jason M Penniman
excerpt: Or should I say "unable to install MySQL 5.5 on Mac OS X".  My first attempt at installing MySQL on Mac was with MySQL 5.1 on Snow Leopard.  That worked as expected and without any issues.  MySQL 5.5, however, was a bit tricky...
category: Database
---
Or should I say "unable to install MySQL 5.5 on Mac OS X".  My first attempt at installing MySQL on Mac was with MySQL 5.1 on Snow Leopard.  That worked as expected and without any issues.  MySQL 5.5, however, was a bit tricky.

After the install, I got this error on start up:

```
101224  9:50:00 [ERROR] Can't open the mysql.plugin table. Please run mysql_upgrade to create it.
```

MySQL is not started at this point, so running mysql_upgrade also fails (not that there is anything to upgrade, it's a fresh install)

Viva la Google!  After a little digging on Google (ok, a lot of digging), I found this article in the MySQL forums:

MySQL Forums :: Install :: Unable to install MySQL 5.5.8 on MacOS 10.6 <a href="http://forums.mysql.com/read.php?11,399397,399606#msg-399606">http://forums.mysql.com/read.php?11,399397,399606#msg-399606</a>

Worked like a charm!

So, the complete steps:

<ol>
<li>Download the dmg and install per the reference manual:  <a href="http://dev.mysql.com/doc/refman/5.5/en/macosx-installation-pkg.html">http://dev.mysql.com/doc/refman/5.5/en/macosx-installation-pkg.html</a></li>
<li>Edit: /usr/local/mysql/support-files/mysql.server<ol>
<li>$ sudo vi /usr/local/mysql/support-files/mysql.server</li>
<li>Locate the configuration defining the basedir and datadir and set the following : </li>
</ol>
<ul>
<li> 
<ul>
<li>basedir=/usr/local/mysql </li>
<li>datadir=/usr/local/mysql/data</li>
</ul>
</li>
</ul>
</li>
</ol>
<p>Hope this saves you some headache.  Good luck!</p>
<div></div>
