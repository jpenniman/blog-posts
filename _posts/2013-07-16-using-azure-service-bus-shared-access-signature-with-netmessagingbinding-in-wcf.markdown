---
layout: post
title: Using Azure Service Bus Shared Access Signature with NetMessagingBinding in  WCF
date: 2013-07-16 13:35:00.000000000 -04:00
author: Jason M Penniman
excerpt: I Googled, Binged, and scoured internet and various documentation and could
  not find this setting. &nbsp;I was able to reverse engineer the declarative markup
  based on the properties in the classes and code samples.<br />If you want to use
  the shared access signature authentication with the netMessagingBinding in WCF declaratively
  in your web.config or...
category: Blog
---
I Googled, Binged, and scoured internet and various documentation and could not find this setting. I was able to reverse engineer the declarative markup based on the properties in the classes and code samples.

If you want to use the shared access signature authentication with the netMessagingBinding in WCF declaratively in your web.config or app.config, here's how....

Given the service bus connection string....

```
Endpoint=sb://acme.servicebus.windows.net/;SharedAccessKeyName=efudd;SharedAccessKey=f9JFgRYdLdnPTQv4/EC5ixt4iHVUSTCnhOg1uep9lsW=
```

The configuration for your transportClientEndpointBehavior looks like this...

``` xml
<transportClientEndpointBehavior>
  <tokenProvider>
    <sharedAccessSignature keyName="efudd" key="f9JFgRYdLdnPTQv4/EC5ixt4iHVUSTCnhOg1uep9lsW=" />
  </tokenProvider>
</transportClientEndpointBehavior>
```

Happy Coding!

