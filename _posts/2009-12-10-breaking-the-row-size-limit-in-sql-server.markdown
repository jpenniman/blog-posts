---
layout: post
title: Breaking the Row Size Limit in SQL Server
date: 2009-12-10 19:39:00.000000000 -05:00
author: Jason M Penniman
excerpt: Microsoft SQL Server, as of version 9.0 (2005), allows you to cheat the max row size limit of 8K. If we have a row of variable length data types (ie. varchar), the total byte count can be more than 8K. SQL Server will magically spill the data over to the next page. Cool right? Well...
category: Blog
---

Microsoft SQL Server, as of version 9.0 (2005), allows you to cheat the max row size limit of 8K. If we have a row of variable length data types (ie. varchar), the total byte count can be more than 8K. SQL Server will magically spill the data over to the next page.

Cool right? Well... yes and no. Cool if your hands are tied and it gets you out of a jam. Not cool if you care about database performance and scalability.

Having a row span more than one page (in Oracle we call them blocks), results in page (or block) chaining. The overhead involved in block chaining can cause some significant performance hits depending on how often it happens, size of the table, fragmentation, etc.

This goes for most any popular RDBMS. Let's look at Oracle (no point in just picking on SQL Server). Oracle allows a 64K max row size (and that's a hard limit... no loosy goosy there). However, block size is determined by the value of the db_block_size init parameter set during database creation. Oracle recommends an 8K db_block_size for a general purpose (OLTP with some reporting) database. If your row is wider than 8K, then, you guessed it, block chaining.

Same goes for MySQL (sort of). The InnoDB engine, like Oracle, has a 64K limit. The default, uncompressed, block size is 16K. However, anything over 8K will be written to the next page (block) and will result in block chaining.

In contrast, IBM DB2 does NOT allow block chaining. By default, DB2 page size is 4K, but is configurable to 4, 8, 16, or 32K, and unlike Oracle, this is on a per tablespace basis rather than at the database level.

So why is block chaining bad? Well, it's not horrible, but the resulting performance does depend on a lot of factors. One major factor in particular is fragmentation. If the blocks aren't contiguous, it can result in some pretty significant database performance degradation. I'll leave database fragmentation and how to manage it for another post... or maybe three... seems appropriate to discuss this tuning knob in a few database systems.

While were on the topic of Max Row Size in SQL Server, tables exceeding the 8K limit (per their column definitions, not data content) cannot participate in SS replication. Tables containing more than 246 columns cannot participate in merge replication and tables with more than 1000 columns cannot participate in transactional replication.

There's a lot more to blocks/pages, row size, how it all relates, and how it impacts performance and scalability, but is out of scope for this post. Each engine handles it a little differently. In general, however, if your row exceeds the block size, it will result in block chaining, and *could* result in poor performance.

I left PostgresSQL out of the post intentionally. With a 1.6TB (yes, terabyte) max row size, it's a whole discussion in and of itself (and yes, just because you can, doesn't mean you should).
