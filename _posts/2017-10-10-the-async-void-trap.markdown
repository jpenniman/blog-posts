---
layout: post
title: The Async/Void Trap
date: 2017-10-10 21:31:39.000000000 -04:00
image: /blog/images/blank-tile.png
author: Jason M Penniman
excerpt: <p>I'll start by saying, you should ALWAYS avoid async/void. Now, I'm sure there are edge cases, but the reality is you can get into a lot of trouble with async/void.</p>
category: Development
---

<a href="https://github.com/milestonetg/dotnet-examples/tree/master/AsyncVoid" target="_blank">Example Code</a>


I'll start by saying, you should ALWAYS avoid async/void.  Now, I'm sure there are edge cases, but the reality is you can get into a lot of trouble with async/void.

A good <a href="https://msdn.microsoft.com/en-us/magazine/jj991977.aspx" target="_blank">article in MSDN Magazine</a> goes over some best practices when using async/await and asynchronous programming patterns.

As the article points out, avoid async/void. However, it also states that an exception to that rule is event handlers where the delegate returns void. Even with those event handlers, you need to be extremely careful. Here's why--await is actually not blocking.  I know if feels like it is, but remember, async/await is just syntax sugar for ContinueWith() and context management.

Let's look at some code.

Take some method Foo() that returns void and calls an async method Bar()

``` csharp
async void Foo()
{
    await Bar();
}

Task Bar()
{
    return Task.Delay(10000);
}
```

Now, if we call Foo(), it will return immediately, and Bar() will be executed in background on another thread.

``` csharp
static void Main()
{
    Foo();
    Console.WriteLine("Foo Returned");
}
```

Not what you expected was it?

The danger?  Consecutive event handlers.

One of our developers was working on a Xamarin based mobile app. Android activities have several events in the lifecyle that fire in succession--OnCreate() followed by OnResume().  There was logic in the OnCreate that was using async apis and logic in onResume that was using async apis. Because the async/void returns immediately, the Android system went ahead and called OnResume before OnCreate was actually finished.  If there were no dependency order to the logic, it wouldn't have mattered, but in our case there was.

The solution? Actually wait for the result.

``` csharp
void Foo()
{
    Bar().ConfigureAwait(false).GetAwaiter().GetResult();
}
```

ConfigureAwait(false) is used to not switch back to the original context so we don't get a dead lock. Then we get the awaiter and get the result of the task execution.  GetResult() blocks and waits for the task to complete so it can return the value of the Task.Result property.

Disclaimer: This isn't a perfect solution either. Depending on the context, this can still dead lock.

## Conslusion

Asynchronous programming is hard in a language/framework that isn't 100% async. Scenarios like this are why Microsoft recommends "async all the way"--because as soon as you have to sprinkle some sync code in there, you can get into trouble if you're not careful.

## Related video segment
<iframe src="//www.youtube.com/embed/8GMlwIeB0tw" width="560" height="314" allowfullscreen="allowfullscreen"></iframe>
