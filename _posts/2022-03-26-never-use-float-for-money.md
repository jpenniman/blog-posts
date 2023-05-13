---
layout: post
title: Never use float for money
date: 2022-03-26
author: Jason M Penniman
image: /blog/images/money.jpg
excerpt: The web is riddled with articles on this very topic, yet it bares repeating--never use float for money. I repeat--never never never never never use a floating point data type to represent monetary values or do financial math. So how do we handle money?
tags:
- dotnet
- C#
- currency
- money
---

The web is riddled with articles on this very topic, yet it bares repeating: never use float for money. I repeat: never never never never never use a floating point data type to represent monetary values or do financial math.

## Why?

Single and Double precision floating point numbers are not accurate. Ironically, it's by design. They are optimized for performance where absolute accuracy is not a concern.

Microsoft talks about it briefly here: https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/floating-point-numeric-types

> [...]there's no double or float instance that exactly represents 0.1. Because of this difference in numeric types, unexpected rounding errors can occur in arithmetic calculations when you use double or float for decimal data.

Baeldung has an excellent writeup on the science behind why floats behave the way they do: https://www.baeldung.com/cs/floating-point-numbers-inaccuracy

## So what should we use for money and financial calculations?

Decimal. Unless your language has a specific data type for money.  VBA for example has a Currency datatype.  T-SQL also has a Money and SmallMoney datatypes. Under the hood they are all highly accurate base10 (rather than base2) representations of the numeric data.

Here's some code that exploits the inaccuracy of float and double vs decimal. Consider the value 123.453123. If we multiply that by 100, we would expect the result to be 12,345.3123. However, as you can see, with float and double, that is not the case. Only decimal represents the numbers and computation correctly:

``` csharp
float   f1 = 123.453123f;
double  d1 = 123.453123;
decimal m1 = 123.453123m;

Console.WriteLine(f1 * 100.0f);
Console.WriteLine(d1 * 100.0);
Console.WriteLine(m1 * 100.0m);

//12345.3125
//12345.312300000001
//12345.3123000
```

## Summary

As you can see, if mathamatical accuraccy is required, use decimal.

A note on handling currency. Depending on the system, it may be beneficial to create a Money type in your domain. The design pattern for this is the Money Pattern. Maybe I'll write up an article on how I have implemented this in the past.

Cheers!
