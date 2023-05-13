---
layout: post
title: Simulating Latency and Bandwidth Restrictions in WCF
date: 2010-10-17 19:06:00.000000000 -04:00
image: /blog/images/blank-tile.png
author: Jason M Penniman
excerpt: If you're developing WCF applications that will run over a WAN, VPN, or
  the public internet, then network bandwidth and latency is a concern. Messages take
  longer to transmit over these networks than over your local area network (LAN).
  As such, part of your development and testing needs to include testing the application over these typed of "slower" connection...
category: Development
---
If you're developing WCF applications that will run over a WAN, VPN, or the public internet, then network bandwidth and latency is a concern. Messages take longer to transmit over these networks than over your local area network (LAN). As such, part of your development and testing needs to include testing the application over these typed of "slower" connection.

To perform this testing there are two main options:  have a connection at your disposal, or simulate one.  Most of us will probably have to simulate one.  There are a few projects out there that are basically bridges, typically linux based, that you put between your client and server that allow you to simulate these slower connections. If you are a Java developer, you're probably familiar with JMeter. JMeter's approach doesn't involve a bridge, but rather simulates the latency at the client.

I was chatting with Peter Lin, a friend, colleague, and JMeter committer, about it one night. After he mentioned how JMeter simulated latency and low bandwidth, I decided to create a simulator based on JMeter's thread sleep concept at the client leveraging WCF's extensibility though behaviors and inspectors. The result is an open source project aptly named "WCF Latency Simulator".

<img border="0" src="http://3.bp.blogspot.com/_4F2sW8e1XyU/TLucqgnUbeI/AAAAAAAAAOs/ui8yTXPcIM0/s1600/Screen+shot+2010-10-17+at+9.01.59+PM.png" />

As you can see, the UI is very simplistic. It allows you to set the latency in milliseconds and the bandwidth in bits per second. More importantly, it's on-the-fly. You can turn these knobs while your application is running and the new settings will be picked up without having to restart your application.

Side note: It's a .Net application, I promise. The windowing is a skin provided by Parallels so my Windows apps are fit in more seamless with my Mac :)

## Usage

It's fairly easy to plug it into your application. You need to reference the `MilestonTG.WcfLatencySimulator.Wcf.dll` library in your client application. When you create the instance of your client proxy (I always recommend this be done with a factory), simply add the `MilestoneTG.WcfLatencySimulator.Wcf.SimulatorClientBehavior` end point behavior to the endpoint:

``` cs
client.Endpoint.Behaviors.Add(new SimulatorClientBehavior());
```

Start the simulator UI, dial in your settings, and fire up your application. The behavior communicates with the simulator using named pipes, so it's important that the simulator be running before you launch your application. As a future enhancement, I'll fix it so the order doesn't matter, but for now, simulator first, your application second.

## Custom Providers

One of the components of this tool is the settings provider. This is where the simulator gets it's settings from. You can use the simulator through the API by referencing the singleton: `MilestoneTG.WcfLatencySimulator.Library.Simulator` found in the `MilestoneTG.WcfLatencySimulator.Library.dll` assembly. The Simulator Class has the method `SetProvider()` that accepts an `ISettingsProvider` as a parameter. This interface contains only one method, `GetSettings()` that returns a `MilestoneTG.WcfLatencySimulator.Wcf.LatencySettings` object.

``` cs
namespace MilestoneTG.WcfLatencySimulator.Library
{
    ///<summary>
    /// This interface must be implemented by all settings providers.  The simulator uses the this interface to retreive the
    /// settings from the provider.
    /// </summary>
    public interface ISettingsProvider
    {
        ///<summary>
        /// When implemented, returns the LegacySettings supplied by the implementing class.
        ///</summary>
        LatencySettings GetSettings();
    }
}
```

A provider then, might look like:

``` cs
public class SettingsProvider : ISettingsProvider
{
    public LatencySettings GetSettings()
    {
        return new LatencySettings() 
        { 
            Latency = (<span>int</span>)_latencySpinner.Value, 
            Bandwidth = (<span>long</span>)_bandwidthSpinner.Value 
        };
    }
}
```

To use your custom provider:

``` cs
SettingsProvider _settingsProvider = new SettingsProvider();
Simulator.Instance.SetProvider(_settingsProvider);
Simulator.Instance.Start();
```

## The Project

You can find the project on sourceforge at <a href="http://sourceforge.net/projects/wcflatencysim">http://sourceforge.net/projects/wcflatencysim</a>.  Both the binaries and the source code is available and distributed under the Apache2 license.

I hope you find this tool useful.  Please feel free to drop a comment; I'd like to hear your ideas for additional features/functionality.
