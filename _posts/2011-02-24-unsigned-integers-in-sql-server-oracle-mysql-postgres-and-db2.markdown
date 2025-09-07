---
layout: post
title: Unsigned Integers in SQL Server, Oracle, MySQL, Postgres, and DB2
date: 2011-02-24 15:33:00.000000000 -05:00
image: /blog/images/blank-tile.png
author: Jason M Penniman
excerpt: A recent project required the need to store an unsigned 64-bit integer in
  a SQL Server database table. BIGINT won't cut it because BIGINT has a max value
  of 9,223,372,036,854,775,807 (signed 64-bit integer), and the unsigned
  64-bit integer's max value is 18,446,744,073,709,551,615.  The solution...
category: Development
tags:
- SQL Server
---
A recent project required the need to store an unsigned 64-bit integer in a SQL Server database table. BIGINT won't cut it because BIGINT has a max value of 9,223,372,036,854,775,807 (signed 64-bit integer), and the unsigned 64-bit integer's max value is 18,446,744,073,709,551,615.  The solution is to use NUMERIC(20) instead.

I recalled that other DBMS's do support unsigned integers as integer types vs. coercion with numeric, so I decided to compare a few of them and document them here.

<table class="table table-striped table-responsive">
  <thead>
    <tr>
      <th>DBMS</th>
      <th>Unsigned 64Bit</th>
      <th>Unsigned 32Bit</th>
      <th>Unsigned 16Bit</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>SQL Server (as of "Denali")</td>
      <td>numeric(20)</td>
      <td>numeric(10)</td>
      <td>numeric(5)</td>
    </tr>
    <tr>
      <td>Oracle (as of 11g)</td>
      <td>number(20), numeric(20)</td>
      <td>number(10), numeric(10)</td>
      <td>number(5), numeric(5)</td>
    </tr>
    <tr>
      <td>MySQL (as of 5.x)</td>
      <td>bigint, numeric(20)</td>
      <td>int, numeric(10)</td>
      <td>smallint, numeric(5)</td>
    </tr>
    <tr>
      <td>Postgres (as of 9.x)</td>
      <td>numeric(20)</td>
      <td>numeric(10)</td>
      <td>numeric(5)</td>
    </tr>
    <tr>
      <td>DB2 UDB (as of 9.x)</td>
      <td>numeric(20)</td>
      <td>numeric(10)</td>
      <td>numeric(5)</td>
    </tr>
  </tbody>
</table>