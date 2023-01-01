---
layout: post
title: 'Building Microservices with Asp.Net WebAPI, Owin, Ninject, NHibernate, and Azure -- Part 2: Error Responses and Exception' 
date: 2018-03-04
author: Jason M Penniman
excerpt: In this installment, we'll look at exception handling and error responses.
category: Development
tags:
- Microservices
- WebAPI
- ASP.Net
- Owin
- Ninject
- NHibernate
- DDD
- Azure
---

[Last time](/blog/development/2017/12/01/microservices-with-aspnet-owin-ninject-nhibernate-azure.html) 
we looked at setting up a microservice based on WebAPI with dependency injection an persistance. In this article, we'll
look at global exception handling and error responses.

## The Error Response
We return standard error message formats for 500 errors. In WebAPI, Microsoft uses `HttpError` internally within the framework. For consistency, we do the same. That way, whether the error was generated in the pipeline, or by our business logic, the message is the same. Under the hood,
HttpError inherits `Dictionary<string,object>`.  You can add custom key/value pairs for anything you'd like to return.

The response the looks something like...

``` json
{
    "message":"An error has occurred",
    "exceptionMessage":"Object not set to an instance of an object."
}
```
The dictionary keys to look up standard error information are available on the 
[HttpErrorKeys](https://docs.microsoft.com/en-us/dotnet/api/system.web.http.httperrorkeys) type.


Using it, we can generate a `IHttpActionResult` to return...

``` cs
public class ExceptionResult : IHttpActionResult
{
    readonly ExceptionHandlerContext context;
    public ExceptionResult(ExceptionHandlerContext context)
    {
        this.context = context;
    }

    public Task<IHttpActionResult> ExecuteAsync(CancellationToken cancellationToken)
    {
        return Task.Run(()=>{
            HttpError error = new HttpError(context.Exception, includeDetail: true);
            return context.Request.CreateErrorResponse(HttpStatusCode.InternalServerError, error);
        },cancellationToken);
    }
}
```

That's the simplified version. For public facing APIs we disable sending the stacktrace (setting `includeDetail: false`)
in the constructor for `HttpError`. Setting `includeDetail` to `false` will not include any `Exception` related information in the dictionary. 

## Setting up a Global ExceptionHandler

Rather than putting a try/catch in every controller action, we use a custom implementation of `System.Web.Http.ExceptionHandling.ExceptionHandler`.

``` cs
public class GlobalExceptionHandler : ExceptionHandler
{
    public override void Handle(ExceptionHandlerContext context)
    {
        if (ShouldHandle(context))
            context.Result = new ExceptionResult(context);
    }
}
```

Then, it's just a matter of using our custom `ExceptionHandler` instead of the default one. In our Startup.cs...

``` cs
var config = new HttpConfiguration();
config.Services.Replace(typeof(IExceptionHandler), new GlobalExceptionHandler());
```

That's really all there is to it. As I mentioned the last article, we use Azure Application Insights for our monitoring
and logging. This gets us all the request tracing and logging of these error messages for free. Notice that our `Handle()` method doesn't call `HandleException()`. This is do the exception will continue to be processed by the rest of the pipeline and be automatically logged in Application Insights.

## Exception Logger

If you're not using and APM solution like Application Insights, NewRelic, or AWS X-Ray (and I highly recommend you do), you also implement a custom `ExceptionLogger` to log the exception.

``` cs
public class GlobalExceptionLogger : ExceptionLogger
{
    public override void Log(ExceptionLoggerContext context)
    {
        //TODO: Implement your custom logging here
    }
}
```

Then, add it to your Startup.cs...

``` cs
var config = new HttpConfiguration();
config.Services.Add(typeof(IExceptionLogger), new GlobalExceptionLogger());
```

## Recap

So far we've looked at:

1. Creating a clean, bare-bones WebAPI project
2. Bootstrapping Dependency Injection using Ninject
3. Configuring persistance using NHibernate
4. Configuring transient fault handling for the persistance layer
5. Setting up a global `ExceptionHandler` to deal with exceptions in a unified, consistent way
