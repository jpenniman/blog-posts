---
layout: post
title: Configuration Reload in .Net
date: 2022-05-12
author: Jason M Penniman
excerpt: The 12 Factor App guidelines suggest separating configuration from code as a best practice. More specifically, it should be deployed separately as well. We shouldn't have to redeploy the entire application in order to make a configuration change. Ideally, we don't want to have to restart our app either.  This eliminates downtime when making configuration changes that don't otherwise require downtime such as timeout values, retry configuration, logging level, and distributed trace sampling rate.
tags:
- dotnet
- C#
- configuration
- 12factor
---
The [12 Factor App guidelines](https://12factor.net/) suggest separating configuration from code as a best practice. More specifically, it should be deployed separately as well. We shouldn't have to redeploy the entire application in order to make a configuration change. Ideally, we don't want to have to restart our app either.  This eliminates downtime when making configuration changes that don't otherwise require downtime such as timeout values, retry configuration, logging level, and distributed trace sampling rate.

For providers that support it, [Microsoft.Extensions.Configuration](https://docs.microsoft.com/en-us/dotnet/core/extensions/configuration) can reload the configuration when it changes. Let's take a look. We'll be using the JSON file provider for these examples with the following configuration:

``` json
{
    "Foo": {
        "Bar": 1
    }
}
```

## IConfigurationRoot

When we change a configuration value, the value in the IConfiguration instance also changes. The key to making it all work is the reloadOnChange parameter. Given a simple console app, we can see the behavior as we modify the appsettings.json file as the program is running.

``` csharp
var config = new ConfigurationBuilder()
    .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
    .Build();

while (true)
{
    Console.WriteLine(config["Foo:Bar"]);
    Thread.Sleep(1000);
}
```

## Binding Strongly Typed Configuration Classes

If we bind our configuration to POCOs, the bound classes don't get updated unless you're re-binding every time you need to access the config.

``` csharp
var config = new ConfigurationBuilder()
    .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
    .Build();

var foo = new Foo();
config.Bind("Foo", foo);

while (true)
{
    Console.WriteLine(foo.Bar);
    Thread.Sleep(1000);
}

class Foo
{
    public int Bar { get; set; }
}
```

This means, our settings classes can't be registered as Singletons, nor can they be injected into Singletons. For example, if we are building an AspNet Core Rest API and all our services and repositories requiring configuration settings are scoped to the request, we can setup our dependency injection like so:

``` csharp
services.AddScoped<Foo>(container =>
{
    var settings = new Foo();
    var configuration = container.GetRequiredService<IConfigurationRoot>();
    configuration.Bind("Foo", settings);
    return settings;
});
```

There is a performance concern with this approach, however, since we are instantiating and binding the POCO on every request rather than once at app startup.

If we have Singlton services and want settings to reload, we need to either use the Service Location Pattern (not recommended) or use the Options Pattern and a monitor.

## IOptions, IOptionsSnapshot, and IOptionsMonitor

When using the Options pattern there are some caveats. When binding using `IOptions<T>`, instances are registered as singletons. Just as with our strongly-typed settings classes, these do not get refreshed.

``` csharp
var config = new ConfigurationBuilder()
    .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
    .Build();

var container = new ServiceCollection()
    .AddOptions()
    .Configure<Foo>(config.GetSection("Foo"))
    .BuildServiceProvider();
    
var options = container.GetRequiredService<IOptions<Foo>>();
while (true)
{
    Console.WriteLine(options.Value.Bar);
    Thread.Sleep(1000);
}
```

If we want to get the new configuration, we need to use either `IOptionsSnapshot<T>` or `IOptionsMonitor<T>`.

`IOptionsSnapsnot<T>` is registered with a scoped lifetime. So, if we have no singletons, and all our services are registered with scoped or transient lifetimes, `IOptionsSnapshot<T>` will do the trick.

``` csharp
var config = new ConfigurationBuilder()
    .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
    .Build();

var container = new ServiceCollection()
    .AddOptions()
    .Configure<Foo>(config.GetSection("Foo"))
    .BuildServiceProvider();
    
while (true)
{
    var snapshot = container.CreateScope().ServiceProvider.GetRequiredService<IOptionsSnapshot<Foo>>();
    Console.WriteLine(snapshot.Value.Bar);
    Thread.Sleep(1000);
}
```

Much like our POCO approach above, however, the configuration is bound every new scope, so every request to our AspNet Core Rest API is fetching and binding the new config.

If we do have singletons and need refreshed options, or we just want to be as efficient as possible, we can use `IOptionsMonitor<T>`.  This is registered as a singleton, but, as its name suggests, monitors the configuration for changes and updates the values in the cached options class.

``` csharp
var config = new ConfigurationBuilder()
    .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
    .Build();

var container = new ServiceCollection()
    .AddOptions()
    .Configure<Foo>(config.GetSection("Foo"))
    .BuildServiceProvider();
    
var options = container.GetRequiredService<IOptionsMonitor<Foo>>();
while (true)
{
    Console.WriteLine(options.CurrentValue.Bar);
    Thread.Sleep(1000);
}
```

There is also an OnChange event we can subscribe to and perform custom logic on change of a configuration.

Since the initial binding happens at startup, is cached, and is rehydrated on a background thread and only when there is a change, request performance is not impacted. This gives us the same performance as if we created a POCO and registered it as a singleton.

But I Don't Want My Component to Take a Dependency on Microsoft Platform Extensions!
For those out there that don't like to explicitly reference third party code in their components, `IOptionsMonitor<T>` is easy enough to wrap. Don't get me wrong, I'm not against guarding against third parties. I've been burnt myself by frameworks disappearing. Remember Enterprise Library?

``` csharp
public interface ISettings<T> where T : class
{
    event EventHandler<T> OnChange;
    T Value { get; }
}

sealed class ConfigurationOptionsSettings<T> : ISettings<T> where T : class
{
    readonly IOptionsMonitor<T> _optionsMonitor;

    public ConfigurationOptionsSettings(IOptionsMonitor<T> optionsMonitor)
    {
        _optionsMonitor = optionsMonitor;
        _optionsMonitor.OnChange(options => OnChange?.Invoke(this, options));
    }

    public event EventHandler<T>? OnChange;
    public T Value => _optionsMonitor.CurrentValue;
}

public static class ServiceCollectionExtensions
{
    public static IServiceCollection ConfigureSettings<T>(this IServiceCollection services, IConfiguration configuration) where T : class
    {
        services.Configure<T>(configuration);
        services.AddSingleton<ISettings<T>, ConfigurationOptionsSettings<T>>();
        return services;
    }
}
```

## Providers that Support Reload

I'm sure this list is not complete, but here's a handful of providers that support an auto reload of the configuration:

- JSON, XML, INI Providers
- NetEscapades.Configuration.Yaml
- Azure App Configuration
- AWS App Config/System Parameter Store
- MilestoneTG.Extensions.Configuration.S3.Json
- MilestoneTG.Extensions.Configuration.S3.Yaml

## A Note on Serverless

If you're deploying to serverless platforms like AWS Lambda or Azure Functions, then `IOptionsSnapshot<T>` and `IOptionsMonitor<T>` offer no value and with the background threads, may actually cause more problems then they would solve. If you want the flexibility to deploy anywhere, of just want to keep it "Clean" (See Clean Architecture by Robert "Uncle Bob" Martin), the wrapping things up in your own abstraction such as the `ISettings<T>` example above is definitely a way to go. It would allow the clean separation of the infrastructure concern of how your settings are fetched. Persistent container, virutal machine, or bare metal deployments can take advantage of a reloadable `IOptionMonitor<T>` under the hood as we did above, while a serverless deployment can use a simple newed up instance of T.

## A Note on .Net Framework

Microsoft.Extensions.Configuration targets .NetStandard2.0, so it can be used in .Net Framework 4.6.1 and higher in addition to .Net Core and .Net 5+.  In fact, I use it in all my legacy framework apps!

## Conclusion

It is a good practice to separate configuration from code. In doing so, we also want the ability to reload the configuration when it changes without having to restart our application. .Net provides this capability out of the box. Keep it clean and further abstract away `IOptionMonitor<T>` so we can deploy anywhere.

Cheers!
