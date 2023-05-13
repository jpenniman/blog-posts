---
layout: post
title: My Experience with JetBrains Rider
date: 2020-07-02
author: Jason M Penniman
image: /blog/images/rider.png
excerpt: I've been using JetBrains Rider for a bit. Here's my impressions on the popular IDE for dotnet.
tags:
- dotnet
- C#
- Rider
---

_Disclaimer: This is an OpEd and mostly anecdotal. Performance wasn't measured, just perceived based on whether it left like the IDE was hanging a lot or productivity was impacted._

## Update

After using Rider for a few years now, I have to admit it is now my daily driver. I still like Visual Studio, but I even find myself using Rider for all my personal projects. I even use DataGrip more than SQL Server Management Studio these days.



---
I'll start off by saying: there is no replacement for Visual Studio. None. That's my opinion anyway.

That said, JetBrains Rider is a decent .Net IDE, especially if you are a ReShaper fan (I am not) or IntelliJ fan (I am). What made me decide to give it a go? Well, I started work at Linedata as a Cloud Solution Architect and Application Architect. They are heavy users of ReSharper, and use it to enforce code formatting, quality, standards, and test coverage (dotCover) as part of their CI builds. It only took a few hours of me getting fed up with ReSharper crippling Visual Studio for me to install Rider and give it a go. I've been using it at work since.

## Some take-aways

After using Rider as my primary C# IDE for a few months, here's my impression...

### Good Impressions

**Performance:** Visual Studio on its own is faster than Rider. At least that is the perception. However Rider feels significantly faster than Visual Studio + ReSharper. 

**IntelliJ/ReSharper Goodness:** I do like some of the code snippet generation and refactoring capabilities that still aren't in Visual Studio. 

**It is cross platform.** So of course I built a Linux rig to mess around with it. :) It feels even faster on Linux, then again, so does nearly everything else.

**Consistency:** If you spend time across different languages--for example, Java, C#, Python, Go--all of JetBrains IDEs are based on the same IntelliJ shell and all have the same feel and experience. So, while it may not be one IDE to rule them all, muscle memory transfers and they are all pretty much setup the same visually.

### Bad Impressions

**Language Support:** It's a C# IDE. It supports F# with an add-in, and, as of this writing, it's support for VB is 2 versions behind. ~~It doesn't support C++ at all. So if you are in a mixed environment like I am, you still need Visual Studio if your solutions are mixed (or Visal Studio Code). If C# and C++ projects are is separate solutions, you could use JetBrains CLion for the C++ bits.~~ _2022.3+ supports C++ and C# projects in the same solution._

**Ecosystem:** if you are heavily in Microsoft ecosystem--Azure, SQL Server, .Net, etc.--while there are plugins, the tooling just isn't as complete or as seamlessly integrated as with Visual Studio, but that is to be expected.

## Conclusion

All in all I'm happy with it and use it every day at Linedata for my regular development. Visual Studio is till my go-to IDE for all my shenanigans outside of Linedata.
