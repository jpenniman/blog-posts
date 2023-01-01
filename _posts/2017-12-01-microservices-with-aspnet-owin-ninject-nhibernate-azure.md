---
layout: post
title: Building Microservices with Asp.Net WebAPI, Owin, Ninject, NHibernate, and Azure
date: 2017-12-01
author: Jason M Penniman
excerpt: We've been building microservices here for a while, so I thought I'd share how we do it.
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
We've been building microservices here for a while, so I thought I'd share how we do it. I'll try to cover the basics of 
how we implement all our services without getting too complex. I'll provide a high-level overview of the basic components 
that lay the foundation for our services and share some code snip-its. The goal is to provide you with a simple foundation
to build your own microservices framework strategy upon.

## What's a Microservice?
There are a lot of opinions on the matter. When I use the term microservice in this article, I'm referring to a service
that is independently deployable and encapsulates a single bounded context. Though the techniques in this article could be
applied to any service, even large monoliths.

## Core Technology Stack
Even though the benefit of microservices is "use what every you want", we found that having a standard set of tools in the tool
box keep our lives simple and allow us to produce services more quickly. If your shop does give developers freedom of choice,
it's a good idea to standardize each choice and build reusable packages and utilities for those choices. 

Our go-to tools of choice...

### Asp.Net
Almost all our services are in either Asp.Net WebAPI or Asp.Net Core. This article will cover Asp.Net WebAPI, but the general 
approach is the same, just slightly different implementation details. We do have some mircoservices built on Azure Web Jobs and
Azure Functions, but that's for another topic.

### Owin
We use Owin for the middleware. We find it much easier to configure and extend. We can insert various custom middleware 
at different points in the pipeline for doing things like request logging, global error handling, security validation, etc.
That's not to say you couldn't do all that with HttpModules, we just prefer Owin. Plus, Microsoft provides lots of Owin Middleware
goodies for things like caching, CORS configuration, authentication with Active Directory, Facebook, Google, etc.

### Ninject
We use dependency injection with almost all our projects. In addition to aiding testability, it also allows us to
nicely manage the NHibernate.ISession life cycle. When working with Asp.Net WebAPI, our container of choice is Ninject. 
When working with Asp.Net Core, we use the built-in DI system. 

### NHibernate
We use NHibernate for persistance if the service is working with a relational database. We've used both NHibernate and 
EntityFramework. There are pros and cons to both, we just prefer NHibernate. When it comes to persistance in our services,
we always try to select the most appropriate storage medium for the data the service is dealing with. We do have many services
using Azure Table Storage and Redis. For this article, we'll look at relational persistance using NHibernate.

### Application Insights
We use Application Insights for all our logging, tracing, and service monitoring. If you're on AWS, take a look at
Application X-Ray. Newrelic and Splunk are also good tools for this.

### Azure App Services
All of our Asp.Net based services are hosted in Azure App Services. We have also built services for clients running on 
Amazon Web Services--those are primarily on EC2 instances managed by Elastic Bean Stalk, though we're helping them get
services moved over to Docker running on Amazon ECS.

## Setting Up A New Project
So let's setup a basic service...

### Create a New ASP.Net project
We always start with an Empty Asp.Net project. The WebAPI project template sets it up the non-owin way, and adds a lot
of bloat that's not needed, for us at least. We use swagger, a custom swagger.ui template, and docfx for documentation, 
so the MVC support and documentation starter that the project template builds out is unnecessary. Select "Empty" and leave
WebForms, MVC, and WebAPI **unchecked**.

![Empty WebProject Template]({{ "/assets/images/new-empty-webapi.png" | absolute_url }})

### Setup Ninject, Owin, and WebAPI
Once we have our blank project, install the `Ninject.Web.WebApi.OwinHost` NuGet package, and if you're hosting on IIS, 
also add the Owin Host for IIS, `Microsoft.Owin.Host.SystemWeb`.

```
Install-Package Microsoft.Owin.Host.SystemWeb
Install-Package Ninject.Web.WebApi.OwinHost
```

Between those two packages, all the necessary dependencies will be installed to launch an Asp.Net WebAPI service on IIS.

Then we just need to add a `Startup` class. You can use the template in Visual Studio. We remove the app.UseWebApi() and
add the ninject middleware instead...

```cs
app.UseNinjectMiddleware(CreateKernel);
app.UseNinjectWebApi(config);
```

`CreateKernel` is a method we create in `Startup` to configure the Ninject kernel (container)...

```cs
IKernel CreateKernel()
{
    IKernel kernel = new StandardKernel();

    //TODO: Add you bindings here

    return kernel;
}
```

We map our routes by attributes. It's a little more verbose, but very explicit. Sometimes relying on convention magic 
can lead to unintended behavior. It also puts the config for the route right on the action the route executes, eliminating
any confusion. So, `config.MapHttpAttributeRoutes()` also needs to be added to the `Configure` method. The complete 
`Startup` class skeleton looks like this...

```cs
[assembly: OwinStartup(typeof(Startup))]
public class Startup
{
    public void Configure(IAppBuilder app)
    {
        var config = new HttpConfiguration();
        config.MapHttpAttributeRoutes();

        app.UseNinjectMiddleware(CreateKernel);
        app.UseNinjectWebApi(config);
    }

    IKernel CreateKernel()
    {
        IKernel kernel = new StandardKernel();

        //TODO: Add you bindings here

        return kernel;
    }
}
```

That's all that is needed to fire up a WebAPI service. A lot less code than the template. :) If you feel so inclined, this
is a good point to templify your project. Then you can use your custom template on future projects.

### Configuring NHibernate for Injection
Now, the NHibernate bits. Install NHibernate...

```
Install-Package NHibernate
```

We also use mapping attributes with NHibernate. So all we install...
```
Install-Package NHibernate.Mapping.Attributes
```

If we require an audit history of changes, we also add Envers...
```
Install-Package NHibernate.Envers
```
> If you're not familiar with Envers, it's worth a look. It's an NHibernate plugin that automatically creates history tables
> for any entities marked as Audited. It also provides the ability to query that history from within the ORM. 

Now we can add the `ISessionFactory` and `ISession` to our container...

```cs
IKernel CreateKernel()
{
    IKernel kernel = new StandardKernel();

    kernel.Bind<ISessionFactory>().ToMethod(
        (context)=>{
            Configuration config = new Configuration();
            //Serialize mapping attributes to XML
            using (MemoryStream stream = new MemoryStream())
            {
                Assembly mappedAssembly = this.GetType().Assembly;
                HbmSerializer.Default.HbmNamespace = mappedAssembly.FullName;
                HbmSerializer.Default.HbmAssembly = mappedAssembly.FullName;
                HbmSerializer.Default.Serialize(stream, mappedAssembly);
                stream.Position = 0;

                //Add the xml mapping to the configuration
                config.AddInputStream(stream);
            }
            config.Configure();
            return config.BuildSessionFactory();
        }
    ).InSingletonScope();

    kernel.Bind<ISession>().ToMethod(
        (context)=>{
            var factory = context.Kernel.Get<ISessionFactory>();
            return factory.OpenSession();
        }
    ).InRequestScope()
     .OnDeactivation(
        (session)=> {
            if (session.Transaction != null 
                && session.Transaction.IsActive 
                && !session.Transaction.WasCommitted 
                && !session.Transaction.WasRolledBack)
            {
                session.Transaction.Rollback();
            }
        }
    );

    return kernel;
}
```

A few critical key points... notice the ISessionFactory is in Singleton scope. This is because we only want one instance of
an ISessionFactory. ISession is in Request scope. This is because we want a new ISession for each HTTP request to our
microservice. OnDeactivation of our ISession instance (at the end of the request in our case), we want to rollback any
uncommitted transactions. We'll come back to this again later as it plays a role in our exception/error handling.

If you don't want to clutter up the Startup class, you can put the NHibernate stuff in a custom `NinjectModule` and just add 
the module to the kernel. We use modules since our setup is always the same, so we can just import our NuGet package and add
the module. This ensures NHibernate is always configured correctly for every service.

> **Transient Fault Handling**
>
> Distributed systems are volatile. Logic that would normally make in-process calls in a monolith are no being made across
> the network, or even the internet. This is a HUGE topic I'll have to address in a separate post, but for our purposes here,
> let's look at the NHibernate configuration. We use the `NHibernate.SqlAzure` package when connecting to SQL Server or Azure SQL Database 
> and configure the following properties:
> ```xml
> <property name="transaction.factory_class">NHibernate.SqlAzure.ReliableAdoNetWithDistributedTransactionFactory, NHibernate.SqlAzure</property>
> <property name="connection.driver_class">NHibernate.SqlAzure.SqlAzureClientDriverWithTimeoutRetries, NHibernate.SqlAzure</property>
> ```
> This ensures we handle transient faults (hiccups in network connectivity) when talking to SQL Server across the network.

## Recap
So far we have a basic service project with dependency injection and persistance. At this point you could start implementing your API,
but there's more to microservices than just a useful API.

## What's Next?
### Global Exception Handling
In Part 2, we'll take a look at global exception handling and error responses.

### Monitoring and Logging
Also in Part 2, we'll take a look at monitoring our microservices and implementing a logging strategy.
