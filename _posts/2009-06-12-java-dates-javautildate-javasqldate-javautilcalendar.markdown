---
layout: post
title: 'Java Dates:  java.util.Date, java.sql.Date, java.util.Calendar'
date: 2009-06-12 07:55:00.000000000 -04:00
image: /blog/images/blank-tile.png
author: Jason M Penniman
excerpt: Working with dates in Java (though not limited to Java) has always been a
  nightmare.  There are lots of good articles out there in Google Land on the topic,
  but after some very good questions from my class, I thought I'd add my two-cents
  and code samples to the mix...
category: Development
---
Working with dates in Java (though not limited to Java) has always been a nightmare.  There are lots of good articles out there in Google Land on the topic, but after some very good questions from my class, I thought I'd add my two-cents and code samples to the mix.

Much of the confusion centered around conversion, so here's some basic how-to snippits:

## Convert java.util.Calendar to java.util.Date:

``` java
java.util.Calendar calendar = java.util.Calendar.getInstance();
java.util.Date utilDate = calendar.getTime();
//or by using the constructor
java.util.Date utilDate = new java.util.Date(calendar.getTimeInMillis());
```

## Convert java.util.Date to java.sql.Date:

``` java
java.util.Date utilDate = java.util.Calendar.getInstance().getTime();
java.sql.Date sqlDate = new java.sql.Date(utilDate.getTime());
```

## Convert java.util.Calendar to java.sql.Date:

``` java
java.util.Calendar calendar = java.util.Calendar.getInstance();
java.sql.Date utilDate = new java.sql.Date(calendar.getTimeInMillis());
```

Here we have to create a new instance of a java.sql.Date and pass the milliseconds into the constructor.  `Calendar.getTime()` returns a java.util.Date, and even though `java.sql.Date` inherits from `java.util.Date` (why my example of converting from `java.sql.Date` to `Calendar` works), remember, we cannot cast a base type to a sub type, unless the instance on the heap is already the subtype.

## Convert java.sql.Date to java.util.Calendar:

``` java
java.sql.Date sqlDate = resultSet.getDate(0);
java.util.Calendar calendar = java.util.Calendar.getInstance();
calendar.setTime(sqlDate);
```

## Parsing Strings:

Assigning a date value, for example, from a form to save into the database, gets a little tricky.  The easiest way I've found (there are many ways to get the job done), is to use the java.text.SimpleDateFormat class.  This class allows us to specify the input format, and supply a string to a parse method in that format to return a java.util.Date.  The valid format values can be found in the java docs for SimpleDateFormat.

``` java
try {
    java.text.SimpleDateFormat dateFormat =
      new java.text.SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss.SSSZ");
    
    java.util.Date parsedDate = dateFormat.parse("1976-06-17T14:55:30.987-0500");
    System.out.println(parsedDate.toString());
} catch (java.text.ParseException e) {
    e.printStackTrace();
}
```

A working sample of the code can be found <a href="http://sites.google.com/site/enterprisedeveloperproject/code/JavaDatesExample.zip?attredirects=0">here</a>.

Sun's documentation can be found here:

<a href="http://java.sun.com/j2se/1.5.0/docs/api/java/util/Date.html">java.util.Date</a>

<a href="http://java.sun.com/j2se/1.5.0/docs/api/java/sql/Date.html">java.sql.Date</a>

<a href="http://java.sun.com/j2se/1.5.0/docs/api/java/util/Calendar.html">java.util.Calendar</a>

<a href="http://java.sun.com/j2se/1.5.0/docs/api/java/text/SimpleDateFormat.html">java.text.SimpleDateFormat</a>
