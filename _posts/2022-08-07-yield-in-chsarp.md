---
layout: post
title: Yield in C#
date: 2022-08-07
author: Jason M Penniman
image: /blog/images/yield.png
excerpt: The yield keyword is used when the method or accessor is an enumerator. Using yield return returns one element at a time.
tags:
- dotnet
- C#
---
The yield keyword is used when the method or accessor is an enumerator. Using yield return returns one element at a time.

The thing to remember here is that `IEnumerable/IEnumerable<T>` is a stream, not a collection. This is where many developers misuse `IEnumerable` (myself included once upon a time). Consider the following code:

``` csharp
public IEnumerable<Person> GetPeople()
{
    var list = new List<Person>();
    // database connection/command logic removed for brevity
    while (reader.Read())
    {
        var person = MapPerson(reader);
        list.Add(person);
    }
    return list;   
}
```

This is probably what you are used to seeing, but is actually a misuse of `IEnumerable`. The code works just fine, but it is not leveraging the benefits of `IEnumerable` being a stream.

Let's refactor to use yield return:

``` csharp
public IEnumerable<Person> GetPeople()
{
    // database connection/command logic removed for brevity
    while (reader.Read())
    {
        var person = MapPerson(reader);
        yield return person;
    }
}
```

What we are doing now is streaming the elements one at a time. Each iteration by the consumer of this method comes back to the point of the yield and executes the next statement, in this case the while. The benefit? We eliminated a loop and an object allocation. Rather than looping through all the records, building up a list, then looping through the list, we just loop through the records once.
