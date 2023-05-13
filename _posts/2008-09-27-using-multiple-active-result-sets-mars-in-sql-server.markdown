---
layout: post
title: Using Multiple Active Result Sets (MARS) in SQL Server
joomla_id: 16
joomla_url: using-multiple-active-result-sets-mars-in-sql-server
image: /blog/images/blank-tile.png
date: 2008-09-27 14:47:00.000000000 -04:00
author: Jason M Penniman
excerpt: A lot of ground was covered in connecting our TellerUI to the database.
  We implemented FinderMethods in our Customer class that return an IList<customer>
  back to the UI to be bound to the results grid. We also implemented an App.config
  file to store our connection string...
category: Development
---
A lot of ground was covered in connecting our TellerUI to the database. We implemented FinderMethods in our Customer class that return an IList<customer> back to the UI to be bound to the results grid. We also implemented an App.config file to store our connection string.

While we did get the code working, it was not as expected. We originally implemented it to use the same connection for all operations throughout the find and load process, so we we're not opening and closing connections over and over; a very costly operation. However, this caused a runtime error. The connection would not allow multiple data readers to be open at a time. Our stop gap solution to get the app working was to simple use a new connection for every call to the database.

I spent some time debugging the original design. The solution was a setting in the connection string -- `MultipleActiveResultSets=True` -- which allows us to have multiple resultsets open at a time in SQL Server 2005 and 2008.

The new connection string in the App.config looks like this...

``` xml
<add name="acme" connectionstring="server=localhost;database=acmebank;integrated security=SSPI;MultipleActiveResultSets=True" />
```

## Multiple Active Result Sets (MARS)

Microsoft's documentation on this is found here: <a href="http://msdn.microsoft.com/en-us/library/ms131686.aspx">http://msdn.microsoft.com/en-us/library/ms131686.aspx</a>

Multiple Active Result Sets is a Microsoft SQL thing. What's interesting, is there is no reason to have it turned off. Under the hood, all of the MARS header information is being generated anyway. So, there is absolutely no performance difference between having it on or off. Microsoft may be seeing this as more of a security feature, and there for turning it off to "restrict" what can be done across a connection.

The Oracle .Net Client supports this functionality by default for all versions of Oracle.

I haven't found any documentation on MySQL or Postgres. I'll have to some testing and let everyone know.
