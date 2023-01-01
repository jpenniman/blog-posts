---
layout: post
title: Reflecting on Reflection
date: 2009-12-10 19:21:00.000000000 -05:00
author: Jason M Penniman
excerpt: I was approached by a developer suggesting a reflection routine for taking
  values from a data transfer object and populating a business entity and vice versa.  He argued that writing a TransferObjectAssembler for each DTO was too time consuming, and would be better suited with a reflection routine. My response...
category: Blog
---

I was approached by a developer suggesting a reflection routine for taking values from a data transfer object and populating a business entity and vice versa.  He argued that writing a TransferObjectAssembler for each DTO was too time consuming, and would be better suited with a reflection routine.

My response was no, not a good practice.  But why?...

I was approached by a developer suggesting a reflection routine for taking values from a data transfer object and populating a business entity and vice versa.  He argued that writing a TransferObjectAssembler for each DTO was too time consuming, and would be better suited with a reflection routine.

My response was no, not a good practice.  But why?  It seemed clean enough.  It was fairly straight forward code for the particular implementation scenario he was looking at.  So, why not?  It would certainly save a few million lines of code.

But what are we really saving?  And at what cost?

It's no secret--reflection is slooooowwwww.  At least in comparison to explicit design-time coding.  Case in point: I helped out with a code review of a similar implementation where reflection was being used to create a generic routine to "convert" DTOs to business entities and vice-versa.  While the DTOs were quite large, they were not ridiculously complex, and very typical of the particular size system.  "Save" times, by simply getting rid of the reflection routines and using explicit code for the "conversions" was reduced to 1~2ms from 30 seconds.

So, when should we reflection?

Reflection is a cool tool, and certainly has it's place--don't get me wrong.  I'm a fan, and have used it many times.  My golden rule for reflection use is: "Reflection is to only be used when what's being reflected is unknown at design time."  Which, if you think about it, can be quite a bit in a large enterprise application...  driver loading in DAO Factories, for example, or plug-in loading.  But, if you know what the code needs to be, such as how to convert a DTO to a Business Entity, using reflection to save development time will simply result in a poor performing application.

My suggestion for this developer?  Split the difference... write a code generation routine that uses reflection to generate the explicit code.  Now the production code is fast and explicit, and it didn't take days/weeks to write.

Moral of the story... laziness breeds bad code.
