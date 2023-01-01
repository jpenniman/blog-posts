---
layout: post
title: Setting Up a Reporting Database
date: 2010-07-28 14:32:00.000000000 -04:00
author: Jason M Penniman
excerpt: 'Using a live OLTP database for complex reporting has a significant performance impact on the OLTP application.  A common architectural design is to have a separate reporting database from your application''s OLTP database...'
category: Database
---

Using a live OLTP database for complex reporting has a significant performance impact on the OLTP application.  A common architectural design is to have a separate reporting database from your application's OLTP database.

Before I get into reporting database options, a critical area to look at is tuning the queries for the reports.  You should start there before introducing a more complex and costly environment.  Just because a query is fast on a 10GB DB doesn't mean it will be on a 100GB DB.

Assuming you've fine-tuned the queries and database, let's take a look at ways to create a reporting database.

I've used several approaches in the past, each with great success.  Like anything else, there are different solutions that fit different scenarios.  I list a few here, but instead of breaking out pro's vs con's, which are subjective, I'll simply list the caveats to using each.

There are several things to consider when choosing a reporting database solution:

* Cost: Initial and TCO
* Hardware requirements
* How up-to-date the reporting data needs to be... does it have to be real-time?
* Are there transformations that need to happen?
* Load/Resource usage on the existing OLTP without reporting
* Load/Resource usage introduced by reporting

The four broad "solutions" typically used are replication, log shipping, database mirroring, and SSIS Jobs.  There are many different combinations and setups for each of these, and with their own caveats.

### Replication

* Realtime (as close to it a possible)
* DB can be used to load-balance reads (not writes unless you use merge replication) from the OLTP is some cases.
* Requires another server, some additional networking hardware(ideally), and an additional SQL Server license. Though, if you are in a virtual environment, this can be "bent" a little... Microsoft only requires license per physical socket... you can have as many VM's on a single machine with the same license.
* More DB hours: you've just increased the complexity of the sql server environment.

### Log Shipping

* Dependent on transaction log backups.
* Not realtime, but can be near realtime (Microsoft recommends log backups every 2-5 minutes, but beware of the performance implications of this).
* More DB hours: you've just increased the complexity of the sql server environment.
* It should be on it's own server
* More DB hours: you've just increased the complexity of the sql server environment.

### SSIS Jobs

* Doesn't require another server. (though, your situation may need separate hardware altogether).
* Not realtime
* Depending on the complexity of the scripts, you may need a C# developer to create and maintain them.

Database Mirroring is similar to log-shipping except that it's a little closer to real-time and has automatic failover should you also want your copy to double as a standby server (could help in justifying the need for additional hardware and licenses).

If you choose to do any of it on the same physical server, make sure the database files are on different physical disks; if they share the same spindles in any way, you've not only defeated the purpose, but made matters worse.  If you do consider the same server to reduce costs, understand first the CPU and memory utilization of the OLTP app and the reporting.  CPU contention, just as disk contention, will not yield you the results you are looking for.

What I typically do is setup replication.  I configure the replicated copy as read-only and use it for all real-time reporting.  I then setup SSIS jobs that read from the copy and denormalize and transform the data into a data warehouse for more complex decision reporting.  These jobs, depending on need, are schedule anywhere from every hour to once a day, week, or month.

<img border="0" src="http://4.bp.blogspot.com/_4F2sW8e1XyU/S_lQE6WcGiI/AAAAAAAAAOM/vinHQ49DH_s/s320/ReportingDB_1.png" width="320" />
Figure 1

This offers the greatest flexibility and performance across the OLTP and reporting systems.

As I said, there's no one solution, and you may find that each of your customers require a slightly different approach depending on their needs.
