---
layout: post
title: Using NHibernate in Asp.Net Core
date: 2019-07-05
author: Jason M Penniman
image: /blog/images/blank-tile.png
excerpt: In this article, I'll set up a basic Asp.Net Core microservice with NHibernate.
category: Development
tags:
- NHibernate
- AspNetCore
---

Starting with version 5.1, NHibernate now supports .Net Core and .Netstandard as well as the full framework. Let's take a quick look at how to set it up in an ASP.Net Core 2.x/3.x application.

## Configuration

As of this writing, NHibernate doesn't support the configuration system within Asp.Net Core based on Microsoft.Extensions.Configuration. The good news is, you don't have to use a `hibernate.cfg.xml` file or the `<hibernate-configuration>` section in app.config. You can add the properies manually, and pull the values from Asp.Net Core's configuration, taking advantage of all the goodies that come with it. This opens up a world of configuration deployment possibilities.

``` cs
var config = new Configuration();
IDictionary<string, string> properties = new Dictionary<string, string> {
    { "connection.connection_string", Configuration.GetConnectionString("Default") },
    { "dialect", "NHibernate.Dialect.SQLiteDialect" },
    { "hbm2ddl.keywords", "auto-quote"},
    { "show_sql", "true"},
    { "generate_statistics", "true"}
};

    config.AddProperties(properties);
    config.AddXmlFile("hibernate.hbm.xml");
});
```

## Setting up  ISessionFactory and ISession with Dependency Injection

### Registering NHiberate

We can use Asp.Net Core's built-in dependency injection to manage the Configuration, ISessionFactory, and ISession lifecycles. We only want one instance of the configuration and ISessionFactory, so we'll register those as singletons. We want an ISession per Http request, so we'll register ISession in scope. Remember, with Asp.Net core, the container's default scope is per request.

``` cs
services.AddSingleton<Configuration>((provider) => {
    Configuration config = new Configuration();
           
    IDictionary<string, string> properties = new Dictionary<string, string> {
        { "connection.connection_string", this.Configuration.GetConnectionString("Default") },
        { "connection.driver_class", "MilestoneTG.NHibernate.TransientFaultHandling.SqlServer.SqlAzureClientDriver, MilestoneTG.NHibernate.TransientFaultHandling.SqlServer" },
        { "dialect", "NHibernate.Dialect.MsSql2012Dialect" },
        { "hbm2ddl.keywords", "auto-quote"},
        { "show_sql", "true"},
        { "generate_statistics", "true"}
    };

    config.AddProperties(properties);
    config.AddXmlFile("hibernate.hbm.xml");
});

services.AddSingleton<ISessionFactory>((provider) => {
    return provider.GetService<Configuration>().BuildSessionFactory();
});

services.AddScoped<ISession>((provider) => {
    return provider.GetService<ISessionFactory>().OpenSession();
});
```

### Injecting ISession Into Your Services

A common mistake I see among developers new to dependency injection (and even some veterans) is registering the services as singtons. This causes undesired behavior because you've effectively made ISession a singleton as well, since it will never be instantiated again for furture requests. Remember the rule of scoping, you can only inject the same scope or higher, not lower. Singletons and only have singletons as dependencies. Scoped can have scoped or singletons as dependencies. Transient registrations can have singletons, scoped, or other transient instances as dependencies.

Knowing this, we'll make our services scoped:

``` cs
services.AddScoped<IOrderService, OrderService>();
```

Your service would look something like this:

``` cs
public class OrderService : IOrderService
{
    ISession session;
    public OrderService(ISession session)
    {
        this.session = session;
    }
}
```

And the controller would look something like this:

``` cs
public class OrdersController : ControllerBase
{
    IOrderService service;
    public OrdersController(IOrderService)
    {
        this.service = service;
    }
}
```

When the container is managing the lifecycle for ISession, make sure you're not closing or disposing of ISession manually. Let the container do it.

### What If My Service NEEDS to be a Singleton?

If you do have a service the depends on NHibernate that should be a singleton, for example an in-memory cache or other service with an expensive instansiation, inject `ISessionFactory` instead, and use it to create a new session within the service itself.

``` cs
public class StateProvinceService : IStateProvinceService
{
    ISessionFactory factory;
    IList<StateProvince> cache;
    public StateProvinceService(ISessionFactory factory)
    {
        this.factory = factory;
    }

    public async Task Init()
    {
        using(ISession session = factory.OpenSession())
        {
            cache = await session.QueryOverAsync<StateProvince>().List();
        }
    }
}
```


## Initializing the SessionFactory

It is a good idea to initialize the configuration and session factory on startup so the first request doesn't take the major performance hit.

``` cs
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.ApplicationServices.GetService<ISessionFactory>(); 

    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }
    else
    {
        app.UseHsts();
    }

    app.UseHttpsRedirection();
    app.UseMvc();
}
```

## A Nuget Package to Make It Even Easier!

[My company](https://milestonetg.com) maintains a nuget package that handles all this boiler-plate for you and provides an NHibernate options section for appsettings.json.

[MilestoneTG.NHibernate.Extensions.AspNetCore](https://www.nuget.org/packages/MilestoneTG.NHibernate.Extensions.AspNetCore/)

With this package, you configure NHibernate in your appsettings.json, or other Microsoft.Extensions.Configuration provider.

``` js
{
  "Logging": {
    "LogLevel": {
      "Default": "Warning"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "Default": "server=(localdb)\\mssqlserver;"
  },
  "NHibernate": {
    "ConnectionStringName":  "Default",
    "Properties": {
      "connection.driver_class": "MilestoneTG.NHibernate.TransientFaultHandling.SqlServer.SqlAzureClientDriver, MilestoneTG.NHibernate.TransientFaultHandling.SqlServer",
      "dialect": "NHibernate.Dialect.MsSql2012Dialect",
      "hbm2ddl.keywords": "auto-quote",
      "show_sql": "true",
      "generate_statistics": "true"
    }
  }
}
```

Then use the extension methods for adding NHibernate to the container and initializing the session factory:

``` cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);

    services.AddNHibernate();
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseNHibernate();

    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }
    else
    {
        app.UseHsts();
    }

    app.UseHttpsRedirection();
    app.UseMvc();
}
```

More information on using these extensions as well as the source code can be found on github: [https://github.com/milestonetg/nhibernate-extensions](https://github.com/milestonetg/nhibernate-extensions).

## A Note on WebSessionContext

Don't use it. As of this writing, it's not actually supported in .Net Core. It's a better practice to dependency injection and manage the lifecycle. Even in my full framework applications, I use DI to manage the lifecycle rather then WebSession.
