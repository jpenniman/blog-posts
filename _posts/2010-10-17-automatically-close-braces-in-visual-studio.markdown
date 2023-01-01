---
layout: post
title: Automatically Close Braces in Visual Studio
date: 2010-10-17 17:41:00.000000000 -04:00
author: Jason M Penniman
excerpt: 'If you''re an Eclipse or Netbeans user, you''ll be very familiar with this
  topic.  These IDEs automatically close braces, parenthesis, quotes, and brackets
  -- ( {},[],(),"",'''' ) -- for you as you type in the code editors.  It may
  sound like laziness, but it can actually save you debug/troubleshooting time as
  it ensures (well, helps to ensure) that for every open you have a corresponding
  close.'
category: Development
---
If you're an Eclipse or Netbeans user, you'll be very familiar with this topic.  These IDEs automatically close braces, parenthesis, quotes, and brackets -- ( {},[],(),"",'' ) -- for you as you type in the code editors.  It may sound like laziness, but it can actually save you debug/troubleshooting time as it ensures (well, helps to ensure) that for every open you have a corresponding close.

One complaint I've always had, along with many other people based on my Google searches, is that Microsoft Visual Studio does not have this functionality.  If you're like me and spend a lot if time in both the Java IDE's and Visual Studio, you probably find it very annoying that the editors don't behave quite the same.  It can really mess with your muscle memory and make you less productive.

While there are a few very good commercial packages out there that provide this functionality (along with a host of other cool toys), I haven't been able to find any free/open ones, so I wrote one.  Milestone Visual Studio .Net Toys in born.

Milestone Visual Studio .Net Toys is an open source project currently available on sourceforge at: <a href="http://sourceforge.net/projects/vstoys">http://sourceforge.net/projects/vstoys</a> and will be available on the Milestone's web site (<a href="http://www.milestonetg.com/">http://www.milestonetg.com</a>) shortly.

Though the only feature currently available is the auto closing, the long term goal is to continue to provide useful productivity tools we've all become accustomed to in Eclipse that aren't available in Visual Studio--ie. Rename a namespace on move of a class to another folder or project :)

I hope you find the add-in useful and the source code inspiring.  If you have any requests on what you would like to see the toy box, let me know.
