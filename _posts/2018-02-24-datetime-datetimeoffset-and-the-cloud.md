---
layout: post
title: C# DateTime, DateTimeOffset and the Cloud
date: 2018-02-23
author: Jason M Penniman
excerpt: In this article, we'll take a look at the challenges that arise when deploying applications that use System.DateTime to cloud providers like Azure and AWS.
category: Development
tags:
- C#
- .net
- dotnet
- DateTime
- DateTimeOffset
- Cloud
- Azure
- AWS
---

One thing to remember, `System.DateTime` has no knowledge of timezone. It's only aware that the date and time represents 
UTC or the local timezone of the machine on which the code is running.

In the cloud, by default (and it's best to keep it that way), the "server's" timezone is set to UTC. So what? Well...

The serializers do some magic that can yield undesired results depending on your situation. Take the following...

```cs
DateTime today = DateTime.Today;
```

Seems harmless. If you inspect this instance further, you'll see that the Kind is set to `DateTimeKind.Local`. So, when 
working with the instance, it will behave in the timezone of the machine. If you serialize it on your laptop, and your 
laptop's timezone is set to "Easter Standard Time" (America/New York for you Linux/Mac folks)...

```cs
string json = JsonConvert.SerializeObject(today);
Console.WriteLine(json);
```
...you get...
```
2018-02-23T00:00:00-05:00
```

Notice the timezone offset. If `DateTime` doesn't store timezone information, where did it come from? The machine. The Kind
is Local, so the serializer looks gets the local timezone from the local machine and uses that offset.

Here's the gotchya. If you send this to an API running in Azure or AWS or any server running in UTC, deserialize it back 
into a DateTime and look at the ISO8606 representation, you get...

```
2018-02-23T05:00:00+00:00
```

ARG!!!! Whaaaaaat!!! See, `DateTime` doesn't know about timezone. The serializer, by default, looks sees the timezone and 
says "Oh, you want a Local Kind. Ok. I'll create a DateTime object that is the same time in my local timezone, in this case
UTC.  Had we sent this to a server with it's timezone set to PST, we would see the pacific time at the server...

```
2018-02-22T21:00:00-08:00
```

So how do I get the value to stay exactly what I send to the server? A few ways.

**Option 1**. Use ensure DateTimeKind is Unspecified.

```cs
DateTime today = new DateTime(DateTime.Today.Ticks, DateTimeKind.Unspecified);
```

Now your local serialized value is:

```
2018-02-23T00:00:00
```

And, on the server, because the serializer doesn't know if you mean UTC or Local, it just keeps it as is, and also
creates a DateTime with the Kind set to Unspecified...

```
2018-02-23T00:00:00
```

The drawback? Kind is a readonly property. The only way to set it is in the constructor. Also, different serialization options
and different serializers, and ORMs could affect this. The only way to be sure would be to always check for Kind and convert
it if necessary.  That equals more code, more overhead, and way more potential for bugs.

**Option 2**. Convert all your times to UTC client side before sending them to the server, then convert them to local time
when you receive data from the server. Always working in UTC at the server.

A lot of APIs take this approach. It doesn't fit every scenario. One that comes to mind is dealing with just the date and 
needing to work with days. We have an API for one of our customers that checks for where they day falls within the week
and calculates the beginning of the week and special logic if a date/time range spans midnight. These types of scenarios, at
least for our customer, require the calculations be done in the local timezone.  Another is reporting. If you're storing the
datetime in UTC, reporting tools potentially need to convert that time to the local time of the user.

```cs
DateTime todayUtc = DateTime.Today.ToUniversalTime();

DateTime todayLocal = todayUtc.ToLocalTime();
```

**Option 3**. Use `System.DateTimeOffset` instead. DateTimeOffset _**is**_ timezone offset aware and will deserialize at the server
exactly as you sent it up, without trying to convert it to it's own local timezone. 

```cs
DateTimeOffset today = DateTimeOffset.Now.Date
```
Gives you...
```
2018-02-23T00:00:00-05:00
```
...on both sides of the wire. 

MS SQL Server also supports DateTimeOffset.

Note, the struct simply stores the offset, not the actual timezone id. There are many timezones that can map to a single
offset.

## Conclusion
Use `System.DateTimeOffset` for all your dates/date-times as a general rule. Every scenario is different, but DateTimeOffset
will give you the most flexibility.

## Have some fun
If you want to play around with how different DateTime values serialize and deserialize with default settings and ASP.Net 
APIs running in Azure, you can post the following payload to `http://datetimefun.azurewebsites.net/api/dates`

```json
{
    "localDate": "2018-02-23T00:00:00-05:00",
    "utcDate": "2018-02-23T00:00:00Z",
    "localDateTime": "2018-02-23T10:31:44.7555783-05:00",
    "utcDateTime": "2018-02-23T15:31:44.760707Z",
    "localDateOffset": "2018-02-23T00:00:00-05:00",
    "utcDateOffset": "2018-02-23T00:00:00-05:00",
    "localDateTimeOffset": "2018-02-23T10:31:44.7560504-05:00",
    "utcDateTimeOffset": "2018-02-23T15:31:44.7613326+00:00"
}
```
The result shows you what the server deserialized to and what it echoed back to the client.

## Further reading
The [Microsoft docs](https://docs.microsoft.com/en-us/dotnet/standard/datetime/choosing-between-datetime) provide great 
guidance on when to use which type and how to avoid timezone related issues.
