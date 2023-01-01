---
layout: post
title: Adding New Microsoft Extensions to Legacy WCF and ASMX Web Services
date: 2018-02-17
author: Jason M Penniman
excerpt: In this article, We'll look at how to use new Microsoft.Extensions.* in a legacy WCF and ASP.Net Web Services (ASMX) applications.
category: Development
tags:
- WCF
- ASMX
- Ninject
- Configuration
- Logging
- Caching
- DI
- IoC
- core
- dotnet
---
I was asked by several people recently:

1. Can I use dependency injection with my legacy WCF and ASMX web services?
2. Microsoft has some new stuff they released with core--specifically, a new configuration system, caching, and logging 
providers. Can I use them in my legacy WCF  and ASMX web services?

The answer to both is yes you can! In fact, I recommend it!

To illustrate this, we'll take a look at taking a sample application that has a WCF service and a ASMX service and we'll:

1. Add dependency injection using Ninject.
2. Add the new Configuration provider from the new `Microsoft.Extensions.Configuration` packages.
3. Add caching functionality using Microsoft's new `Microsoft.Extensions.Caching` packages.
4. Use the new logging providers from `Microsoft.Extensions.Logging` packages.

## Dependency Injection
We use Ninject, but you can just as easily use AutoFac or SimpleInjector as they also have packaged integrations for WCF.
You can use other containers, but may need to hand-roll your own `ServiceFactory`.

Both WCF and ASMX service applications (any IIS Hosted application, except OWIN, for that matter) require 
`Ninject.Web.Common.WebHost` and `Ninject.Web.Common`. Installing `Ninject.Web.Common.WebHost` will install both since 
`Ninject.Web.Common.WebHost` depends on `Ninject.Web.Common`.

```
Install-Package Ninject.Web.Common.WebHost
```

For WCF, we also need to install the WCF integration package for Ninject:

```
Install-Package Ninject.Extensions.Wcf
```

Change Global.asax to inherit from NinjectHttpApplication and implement CreateKernel(). This is required for all IIS based
applications (except OWIN).

``` cs
public class Global : NinjectHttpApplication
{
    protected override IKernel CreateKernel()
    {
        IKernel kernel = new StandardKernel();

        return kernel;
    }
}
```

There. DI bootstrapped and ready to go. Now we're ready to register the new Configuration, Caching, and Logging 
providers with our container.

## Configuration
For more on using IConfiguration with legacy projects, check out 
[Ben Foster's blog article on 'Using .NET Core Configuration with legacy projects'](http://benfoster.io/blog/net-core-configuration-legacy-projects)

For more information on the new configuration api, take a look at the 
[Microsoft docs on Configuration](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?tabs=basicconfiguration).

```
Install-Package Microsoft.Extensions.Configuration
Install-Package Microsoft.Extensions.Configuration.Json
Install-Package Microsoft.Extensions.Options
```

With the packages installed, we just need to register the types with Ninject...

``` cs
kernel.Bind<IConfigurationRoot>().ToMethod(context => {
    return new ConfigurationBuilder()
        .AddJsonFile("appsettings.json.config", optional: true)
        .Build();
});

kernel.Bind(typeof(IOptions<>)).To(typeof(OptionsManager<>)).InSingletonScope();
kernel.Bind(typeof(IOptionsSnapshot<>)).To(typeof(OptionsManager<>)).InRequestScope();
kernel.Bind(typeof(IOptionsMonitor<>)).To(typeof(OptionsMonitor<>)).InSingletonScope();
kernel.Bind(typeof(IOptionsFactory<>)).To(typeof(OptionsFactory<>));
kernel.Bind(typeof(IOptionsMonitorCache<>)).To(typeof(OptionsCache<>)).InSingletonScope();
```

## Caching
For demo purposes, we'll be using the in-memory cache. In a clustered production environment, we recommend Redis. 
This can be accomplished with the redis provider `Microsoft.Extensions.Caching.Redis`.

For more information on the caching options available out of the box, take a look at the 
[Microsoft docs on Caching](https://docs.microsoft.com/en-us/aspnet/core/performance/caching/distributed).

```
Install-Package Microsoft.Extensions.Caching.Memory
```

With the packages installed, we just need to register the provider with Ninject...

``` cs
kernel.Bind<IDistributedCache>().To<MemoryDistributedCache>().InSingletonScope();
```

## Logging
For demo purposes, we'll just add the Debug provider. This provider simply writes to the debug output window in 
Visual Studio. As with the caching, you can use any provider you'd like or even write your own.

For more information on the new logging api, take a look at the 
[Microsoft docs on Logging](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/logging/?tabs=aspnetcore2x).

``` cs
kernel.Bind<ILoggerFactory>().ToMethod(context => {
    return new LoggerFactory().AddDebug(minLevel: LogLevel.Debug);
}).InSingletonScope();

//Logger<T> takes ILoggerFactory into it's constructor
kernel.Bind(typeof(ILogger<>)).To(typeof(Logger<>)).InSingletonScope();
```

## Completed Global.asax

The completed Global.asax would look something like this...

``` cs
public class Global : NinjectHttpApplication
{
    protected override IKernel CreateKernel()
    {
        IKernel kernel = new StandardKernel();

        RegisterConfiguration(kernel);
        RegisterCache(kernel);
        RegisterLogging(kernel);

        return kernel;
    }

    void RegisterConfiguration(IKernel kernel)
    {
        // Register IConfigurationRoot and build the configuration.
        kernel.Bind<IConfigurationRoot>().ToMethod(context => {
            return new ConfigurationBuilder()
                // using the .config extension so IIS doesn't try to serve the file as static content.
                .AddJsonFile("appsettings.json.config", optional: true)
                .Build();
        });

        // Register Options. This allows us to use the IOptions binding feature
        // and is required by the distributed caching implementations.
        kernel.Bind(typeof(IOptions<>)).To(typeof(OptionsManager<>)).InSingletonScope();
        kernel.Bind(typeof(IOptionsSnapshot<>)).To(typeof(OptionsManager<>)).InRequestScope();
        kernel.Bind(typeof(IOptionsMonitor<>)).To(typeof(OptionsMonitor<>)).InSingletonScope();
        kernel.Bind(typeof(IOptionsFactory<>)).To(typeof(OptionsFactory<>));
        kernel.Bind(typeof(IOptionsMonitorCache<>)).To(typeof(OptionsCache<>)).InSingletonScope();
    }

    void RegisterCache(IKernel kernel)
    {
        // For demo purposes, we'll just us the memory cache.
        // You could also use the Redis or SQL Server implementations just as easily.
        kernel.Bind<IDistributedCache>().To<MemoryDistributedCache>().InSingletonScope();
    }

    void RegisterLogging(IKernel kernel)
    {
        // Register and add providers to the factory
        kernel.Bind<ILoggerFactory>().ToMethod(context => {
            return new LoggerFactory().AddDebug(minLevel: LogLevel.Debug);
        }).InSingletonScope();

        // Logger<T> takes ILoggerFactory into it's constructor
        kernel.Bind(typeof(ILogger<>)).To(typeof(Logger<>)).InSingletonScope();
    }
}
```

With our Global.asax setup and our components registered, we can setup our WCF services and ASMX services to use our
distributed cache and our logging provider via dependency injection.

## WCF
Use the Ninject service factory in your WCF services. This allows us to create our instances of our WCF services
using the container and have all the service's dependencies injected:

``` html
<%@ ServiceHost 
        Language="C#" 
        Debug="true" 
        Service="OldMeetsNew_WCF.Service1" 
        CodeBehind="Service1.svc.cs" 
        Factory="Ninject.Extensions.Wcf.NinjectServiceHostFactory" %>
```

Then we can use standard constructor injection for our dependencies:

``` cs
public class Service1 : IService1
{
    IDistributedCache cache;
    ILogger<Service1> logger;

    public Service1(IDistributedCache cache, ILogger<Service1> logger)
    {
        this.cache = cache;
        this.logger = logger;
    }
    ...
}
```

## ASMX

ASMX (ASP.Net) Web Services aren't quite as "clean". Unfortunately, they do not support constructor injection without 
seriously hacking the pipeline--even then it's not perfect. These services require a default constructor. As a workaround,
we can use service location and constructor chaining. It's not perfect, but at least we can define the dependencies for 
our service with a constructor.

``` cs
public class WebService1 : System.Web.Services.WebService
{
    IDistributedCache cache;
    ILogger<WebService1> logger;

    ///<summary>
    /// Used by the ASP.Net pipeline to create an instance of the service.
    /// Dependencies will be resolved from the container using service location.
    ///</summary>
    public WebService1() 
        : this(((NinjectHttpApplication)HttpContext.Current.ApplicationInstance).Kernel.Get<IDistributedCache>(),
               ((NinjectHttpApplication)HttpContext.Current.ApplicationInstance).Kernel.Get<ILogger<WebService1>>())
    {
        //Leave this constructor empty. It's for ASP.Net's activation only.
        //Put all constructor logic in the overload.
    }

    ///<summary>
    /// Instantiates the service with the provided dependencies.
    /// Called from the default constructor.
    ///</summary>
    public WebService1(IDistributedCache cache, ILogger<WebService1> logger)
    {
        // Use this constructor for all initialization logic.
        this.cache = cache;
        this.logger = logger;
    }
    ...
}
```

## Summary

As you can see, the new stuff works just fine in legacy applications. There are some caveats. 
The latest packages (2.0 as of this writing) target netstandard2.0, so your project will need to be targeting 
net461 or higher.  You could use the 1.1 versions and target 4.5.2 if you can't target 4.6.1 or higher.