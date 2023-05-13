---
layout: post
title: NHibernate Releases 5.1 with support for .Net Core and .NetStandard 2.0 
date: 2018-04-08
image: /blog/images/blank-tile.png
author: Jason M Penniman
excerpt: The NHibernate Team has released 5.1 with support for .Net Core 2.0, ASP.Net Core 2.0 and .NetStandard 2.0.
category: Development
tags:
- NHibernate
- dotnet-core
- aspnet-core
- netstandard
---

The [NHibernate](http://nhibernate.info/) Team has done it! [Release 5.1](https://github.com/nhibernate/nhibernate-core/blob/5.1.0/releasenotes.txt) of the famous ORM for .Net supports both the full .Net Framework and .Net Core. Great work! The library is actually cross compiled for net461, netCoreApp2.0 and netStandard2.0 to account for some of the subtle nuances between net461 and core and to support older tooling that doesn't know how to work with netStandard libraries.

## Caveats
As noted in the release notes, there are a few caveats when running on Core:

> * Binary serialization is not supported - the user shall implement serialization surrogates for System.Type,
FieldInfo, PropertyInfo, MethodInfo, ConstructorInfo, Delegate, etc.
> * SqlClient, Odbc, Oledb drivers are converted to ReflectionBasedDriver to avoid the extra dependencies.
> * CallSessionContext uses a static AsyncLocal field to mimic the CallContext behavior.
> * System transactions (transaction scopes) are untested, due to the lack of data providers supporting them.

## WebSessionContext and ASP.Net Core

Also, [`WebSessionContext` is not compatible with ASP.Net Core](https://github.com/nhibernate/nhibernate-core/issues/1632) due to changes in how `HttpContext` is accessed and managed in ASP.Net Core. As an alternative, you can use the ASP.Net Core dependency injection to manage the life cycles of `Configuration`, `ISessionFactory`, and `ISession`. For example (over simplified for brevity):

``` cs
services.AddSingleton<NHibernate.Cfg.Configuration>((provider)=>{ 
    var cfg = new NHibernate.Cfg.Configuration();
    cfg.Configure("hibernate.cfg.xml");
    cfg.AddXmlFile("hibernate.hbm.xml");
    return cfg;
});

services.AddSingleton<ISessionFactory>((provider)=>{ 
    var cfg = provider.GetService<NHibernate.Cfg.Configuration>();
    return cfg.BuildSessionFactory();
});

services.AddScoped<ISession>((provider)=>{
    var factory = provider.GetService<ISessionFactory>();
    return factory.OpenSession();
});
```

In our opinion here at Milestone, this is the preferred way under ASP.Net Core to maintain `ISession`.

## New Second-Level Caching Libraries

If you are using ASP.Net Core, NHibernate.Caches also gets some new packages supporting [In-Memory](https://www.nuget.org/packages/NHibernate.Caches.CoreDistributedCache.Memory/) Cache, [Redis](https://www.nuget.org/packages/NHibernate.Caches.CoreDistributedCache.Redis/), [Memcached](https://www.nuget.org/packages/NHibernate.Caches.CoreDistributedCache.Memcached/), and [SQL Server](https://www.nuget.org/packages/NHibernate.Caches.CoreDistributedCache.SqlServer/) using the Microsoft.Extensions.DistributedCaching libraries. 

## More NetStandard/Core Support to Come!

Pull requests for [NHibernate.Mapping.Attributes](https://github.com/nhibernate/NHibernate.Mapping.Attributes/pull/12) and [NHibernate.Envers](https://bitbucket.org/RogerKratz/nhibernate.envers/pull-requests/28/support-netstandard-and-core-20/diff) are in the works for NetStandard support thanks to our team here at Milestone. Keep an eye out for them on NuGet.

At Milestone, we're working on some patterns and practices and new libraries for working with NHibernate in _**cloud-native**_ ASP.net Core microservices. Stay tuned!