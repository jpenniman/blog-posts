---
layout: post
title: Handling Events on System.Diagnostics.Process
date: 2011-04-07 14:55:00.000000000 -04:00
author: Jason M Penniman
image: /blog/images/blank-tile.png
excerpt: If you want to handle events when working with System.Diagnostics.Process
  -- for example, the Exited event -- you have to first enable the raising of events
  using the EnableRaisingEvents property on Process...
category: Blog
---
If you want to handle events when working with System.Diagnostics.Process -- for example, the Exited event -- you have to first enable the raising of events using the EnableRaisingEvents property on Process.

``` cs
using System.Diagnostics;

class MyClass
{
    void RunProcess()
    {
        Process p = new Process();
        p.EnableRaisingEvents = true;
        p.Exited += new EventHander(Process_Exited);
    }
}
```
