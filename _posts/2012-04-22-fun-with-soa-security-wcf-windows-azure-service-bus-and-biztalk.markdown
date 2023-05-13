---
layout: post
title: Fun with SOA Security, WCF, Windows Azure Service Bus, and BizTalk
joomla_id: 57
joomla_url: fun-with-soa-security-wcf-windows-azure-service-bus-and-biztalk
date: 2012-04-22 10:02:00.000000000 -04:00
image: /blog/images/blank-tile.png
author: Jason M Penniman
excerpt: I've been having some fun lately on an integration project involving and
  on-premises BizTalk 2010 implementation and the Windows Azure Service Bus. We've
  run across some interesting challenges that aren't well documented, if at all, so
  I wanted to share with you all. I'll take a tutorial approach through real
  world scenarios as well as...
category: Blog
---
I've been having some fun lately on an integration project involving and on-premises BizTalk 2010 implementation and the Windows Azure Service Bus. We've run across some interesting challenges that aren't well documented, if at all, so I wanted to share with you all. I'll take a tutorial approach through real world scenarios as well as talk a little bit about the academics.

The business requirement: Publish a BizTalk Orchestration to the Windows Azure Service Bus and secure the service with client/service certificates.

Seems simple enough, but the implementation had me banging my head against a wall. Rather than throw it all at you at once, let's start small...

## A Simple WCF Service

Let's start with a simple WCF service and client that we'll use throughout the rest of the discussion/lab. I won't go to too much into the detail of creating WCF services and consuming them. I 'm assuming if you're researching Azure and/or BizTalk integration, you know how to create a simple WCF application.

I say simple, but I did want somewhat real world rather than just a "HelloWorld", so if you want to follow along, you'll need the AdventureWorks sample database.

Let's look at a service that an AdventureWorks Cycle customer could use to programmatically check the status of an order from their own application.

``` csharp
[ServiceContract]
public interface IOrderService
{
  [OperationContract]
  SalesOrder GetOrder(String orderNumber);
}
```

Nothing fancy, just a service contract with a single operation that takes in an order number and returns a SalesOrder data contract. SalesOrder is defined as:

``` csharp
[DataContract]
public class SalesOrder
{
  [DataMember]
  public int OrderId { get; set; }
  
  [DataMember]
  public String SONumber { get; set; }
  
  [DataMember] 
  public int Status { get; set; }
  
  [DataMember] 
  public DateTime? ShipDate { get; set; }
  
  [DataMember]
  public string CustomerNumber { get; set; }
}
```

The implementation is just a simple database query using Entity Framework:

``` csharp
public class OrderService : IOrderService
{ 
  public SalesOrder GetOrder(String orderNumber)
  {             
    SalesOrder dto = null; 
    using (var context = new AdventureWorksEntities())
    {
      SalesOrderHeader order =
            (from so in context.SalesOrderHeaders
            where so.SalesOrderNumber == orderNumber                       
            select so).FirstOrDefault(); 
      
      if (order != null)
      {
        dto = new SalesOrder();
        dto.CustomerNumber = order.Customer.AccountNumber;
        dto.OrderId = order.SalesOrderID;
        dto.ShipDate = order.ShipDate;
        dto.SONumber = order.SalesOrderNumber;
        dto.Status = order.Status;
      }
    } 
    return dto;
  }
}
```
You can host it in IIS if you want, but I like to create a stand-alone service host. That way I can run the demo code on machines that don't have IIS. A quick and dirty server:

``` csharp
class Server     
{ 
  static void Main(string[] args)
  {
    StartLocal();
  }
  
  static void StartLocal()
  {
    String listeningOnHttp = "http://localhost:9000/OrderService/";
    ServiceHost host = new ServiceHost(typeof(Services.OrderService), new Uri(listeningOnHttp));
    host.Open();
    Console.WriteLine("Server Started.  Listening at: " + listeningOnHttp);
    Console.ReadLine();
  }
}
```

And the cleaned up App.config:
``` xml
<?xml version="1.0" encoding="utf-8"?> 
<configuration>
  <system.serviceModel>
    <behaviors>
      <serviceBehaviors>
        <behavior name="">
          <serviceMetadata httpGetEnabled="true"/>
            <serviceDebug includeExceptionDetailInFaults="true"/>
        </behavior>
      </serviceBehaviors>
    </behaviors>
    <services>
      <service name="WcfServer.Services.OrderService">
        <endpoint address="" binding="wsHttpBinding" contract="WcfServer.Services.IOrderService"/> 
        <endpoint address="mex" binding="mexHttpBinding" contract="IMetadataExchange"/> 
      </service>
    </services>
  </system.serviceModel>
  <connectionStrings>
    <add name="AdventureWorksEntities" connectionString="metadata=res://*/Entities.OrderDataModel.csdl|res://*/Entities.OrderDataModel.ssdl|res://*/Entities.OrderDataModel.msl;provider=System.Data.SqlClient;provider connection string=&amp;quot;data source=.\sql2008r2;initial catalog=AdventureWorks;integrated security=True;multipleactiveresultsets=True;App=EntityFramework&amp;quot;" providerName="System.Data.EntityClient"/>
  </connectionStrings>
</configuration>
```

Ok, server complete. A build and run should get you a running service host that you can point your browser to and view the WSDL. 

And now the client: 

Add a service reference to the service we just created, then create a simple client app: 

``` csharp
static void Main(string[] args)
{
  OrderServiceTest();
} 

private static void OrderServiceTest()
{
  OrderServiceClient proxy = new OrderServiceClient();
  SalesOrder order = proxy.GetOrder(<span class="str">"SO43661");
  proxy.Close(); 
  if (order != null)
  {
    Console.WriteLine(String.Format("Order Number: {0}\nStatus: {1}\nDate Shipped: {2}", order.SONumber, order.Status, order.ShipDate.Value.ToString("MM/dd/yyyy")));
  } 
  else
  {
    Console.WriteLine(<span class="str">"Order not found.");
  }
  Console.ReadLine();
  }
} 
```

And the client's cleaned up app.config: 
``` xml
<? xml version="1.0" encoding="utf-8" ?>
<configuration>
  <system.serviceModel>
    <bindings>
      <wsHttpBinding>
        <binding name="WSHttpBinding_IOrderService">
          <security>
            <transport realm=""/>
          </security>
          </binding>
      </wsHttpBinding>
    </bindings>
    <client>
      <endpoint address="http://localhost:9000/OrderService" 
        binding="wsHttpBinding"
        bindingConfiguration="WSHttpBinding_IOrderService"
        contract="AdventureWorks.IOrderService"
        name="WSHttpBinding_IOrderService">
      </endpoint>
    </client>
  </system.serviceModel>
</configuration>
```

Good to go. Fire up the Server and the Client. You should get a successful test.

## Let's Talk Security

So now we have a service we want to expose to customers. A public facing service means we have to secure it some how. There are several ways to do that, but since this proof of concept is about certificates, we'll stick to our options there.

The WsHttpBinding offers three security modes:

* Transport
* Message
* TransportWithMessageCredential

**Transport** is where we can implement SSL/TLS. We can also implement client certificates at the transport level. The advantage to using transport is it's faster. Another advantage is it's interoperable with older frameworks. The major drawback is it's only good for the first hop.

**Message** security allows us to implement client/server certificates. In addition to authenticating the service and consumer, the message itself is also encrypted. This mode falls under the "end-to-end" security models. Since the message is encrypted, communication stays secure from end to end. The drawbacks are performance and interoperability. It requires WS-I* compatible frameworks that will actually play nice with WCF, so officially, that's WCF and Oracle's Metro stack.

**TransportWithMessageCredential** combines the two, giving us transport level security (SSL/TLS) and end-to-end message level security.

### Creating Certificates for Development

We'll need a total of three (3) certificates for these labs. 

* Root Certificate Authority
* Server Certificate
* Client Certificate

We'll be using the `makecert` utility to create the certificates. You can find more information on creating the certificates here: 

<a rel="nofollow" href="http://msdn.microsoft.com/en-us/library/ff647171.aspx">http://msdn.microsoft.com/en-us/library/ff647171.aspx</a> <a rel="nofollow" href="http://msdn.microsoft.com/en-us/library/ff648498.aspx">http://msdn.microsoft.com/en-us/library/ff648498.aspx</a> <a rel="nofollow" href="http://msdn.microsoft.com/en-us/library/ff650751.aspx">http://msdn.microsoft.com/en-us/library/ff650751.aspx</a> 

Some details that have hung me up in the past... make sure the keys are exportable (-pe) so you can transfer the certificates to other machines, and make sure your server and client certificates are configured for key exchange (-sky exchange). Without key exchange, a handshake isn't possible.

**Root Authority**

The root authority certificate is self-signed and installed into the Trusted Root store. 

``` sh
makecert -pe -n"CN=MilestoneDevCA" -ss Root -sr LocalMachine -a sha1 -sky signiture -r -sv MilestoneDevCA.pvk MilestoneDevCA.cer
```

We'll use the root authority to sign the server and client certs. 

**Server Certificate** 

``` shell
makecert -pe -sky exchange -sk MilestoneDevServerCert -iv MilestoneDevCA.pvk -n "CN=localhost" -ic MilestoneDevCA.cer MilestoneDevServerCert.cer -sr LocalMachine -ss My
```

**Client Certificate** 

``` shell
makecert -pe -sky exchange -sk MilestoneDevClientCert -iv MilestoneDevCA.pvk -n "CN=MilestoneDevClientCert" -ic MilestoneDevCA.cer MilestoneDevClientCert.cer -sr LocalMachine -ss My
```

Add the certificate to HTTP.SYS.  The certhash is the thumbprint of the server certificate: 

``` shell
netsh http add sslcert ipport=0.0.0.0:9443 certhash=a8b0fcd7082848f81b13355833b238fe037b33db appid={00112233-4455-6677-8899-AABBCCDDEEFF} clientcertnegotiation=enable
```

If you're using IIS rather than the stand-alone service host in this lab, then configure the SSL certificate in the IIS management console rather than using netsh.

### Implementing Transport Level Security

With the certificates created and installed, we're ready to configure our little WCF app to use transport security and client certificates. The new server app.config: 

1. Add a binding configuration.
2. Set the security mode to "Transport"
3. Set the transport clientCredentialType to "Certificate"
4. Configure the endpoint to use the new binding configuration

``` xml
<? xml version="1.0" encoding="utf-8"?>
<!-- Server Config -->
<configuration>
  <system.serviceModel>
    <protocolMapping>
      <add scheme="https" binding="wsHttpBinding"/>
    </protocolMapping>
    <bindings>
      <wsHttpBinding>
        <binding name="certificateBinding">
          <security mode="Transport">
            <transport clientCredentialType="Certificate"/>
          </security>
        </binding>
      </wsHttpBinding>
    </bindings>
    <behaviors>
      <serviceBehaviors>
        <behavior name="serviceBehaviors">
          <serviceMetadata httpGetEnabled="true"/>
          <serviceDebug includeExceptionDetailInFaults="true"/>
        </behavior>
      </serviceBehaviors>
    </behaviors>
    <services>
      <service name="WcfServer.Services.OrderService" behaviorConfiguration="serviceBehaviors">
        <endpoint address=""  binding="wsHttpBinding" bindingConfiguration="certificateBinding"
          contract="WcfServer.Services.IOrderService"/>
        <endpoint address="mex" binding="mexHttpBinding" contract="IMetadataExchange"/>
      </service>
    </services>
  </system.serviceModel>
  <connectionStrings>
    <add name="AdventureWorksEntities" connectionString="metadata=res://*/Entities.OrderDataModel.csdl|res://*/Entities.OrderDataModel.ssdl|res://*/Entities.OrderDataModel.msl;provider=System.Data.SqlClient;provider connection string=&amp;quot;data source=.\sql2008r2;initial catalog=AdventureWorks;integrated security=True;multipleactiveresultsets=True;App=EntityFramework&amp;quot;" providerName="System.Data.EntityClient"/>
  </connectionStrings>
</configuration>
```
The new client app.config: 

1. Add the transport clientCredentialType to the binding
2. Add an endpoint behavior for the clientCredentials and specify the clientCertificate to use
3. Configure the endpoint to use the new behavior configuration

``` xml
<? xml version="1.0" encoding="utf-8" ?>
<!-- Client Config -->
<configuration>
  <system.serviceModel>
    <bindings>
      <wsHttpBinding>
        <binding name="WSHttpBinding_IOrderService">
          <security mode="Transport">
            <transport clientCredentialType="Certificate"/>
          </security>
        </binding>
      </wsHttpBinding>
    </bindings>
    <behaviors>
      <endpointBehaviors>
        <behavior name="clientBehaviors">
          <clientCredentials>
            <clientCertificate findValue="MilestoneDevCert" storeLocation="LocalMachine" storeName="My" x509FindType="FindBySubjectName"/>
          </clientCredentials>
        </behavior>
      </endpointBehaviors>
    </behaviors>
    <client>
      <endpoint address="https://localhost:9443/OrderService" binding="wsHttpBinding"  bindingConfiguration="WSHttpBinding_IOrderService" contract="AdventureWorks.IOrderService"  name="WSHttpBinding_IOrderService" behaviorConfiguration="clientBehaviors" />
    </client>
  </system.serviceModel>
</configuration>
```

That's all there is to it. Let's test it. Fire up the server and client and the test should be successful. If you get errors validating the certificates, double check the certificate installation in the Certificates MMC snap in. 

### Implementing Message Level Security

Changes to the server app.config: 

1. Set the binding security mode to "Message"
2. Set the transport clientCredentialType to "None"
3. Add the message clientCredentialType of "Certificate"
4. Add the serviceCredentials service behavior and provide the serviceCertificate

You'll notice that I've set the certificateValidationMode to "None". This is a work around for issues with verifying that the certificate is trusted and valid (not that one was provided), and is for development purposes only. In production, we would leave the default "ChainTrust".

``` xml
<? xml version="1.0" encoding="utf-8"?>
<!-- Server Config -->
<configuration>
  <system.serviceModel>
    <protocolMapping>
      <add scheme="https" binding="wsHttpBinding"/>
    </protocolMapping>
    <bindings>
      <wsHttpBinding>
        <binding name="certificateBinding">
          <security mode="Message">
            <transport clientCredentialType="None"/>
            <message clientCredentialType="Certificate"/>
          </security>
        </binding>
      </wsHttpBinding>
    </bindings>
    <behaviors>
      <serviceBehaviors>
        <behavior name="serviceBehaviors">
          <serviceMetadata httpGetEnabled="true"/>
          <serviceDebug includeExceptionDetailInFaults="true"/>
          <serviceCredentials>
            <serviceCertificate findValue="localhost" storeLocation="LocalMachine" storeName="My" x509FindType="FindBySubjectName"/>
            <clientCertificate>
              <authentication certificateValidationMode="None"/>
            </clientCertificate>
          </serviceCredentials>
        </behavior>
      </serviceBehaviors>
    </behaviors>
    <services>
      <service name="WcfServer.Services.OrderService" behaviorConfiguration="serviceBehaviors">
        <endpoint address="" binding="wsHttpBinding" bindingConfiguration="certificateBinding" contract="WcfServer.Services.IOrderService" />
        <endpoint address="mex" binding="mexHttpBinding" contract="IMetadataExchange"/>
      </service>
    </services>
  </system.serviceModel>
  <connectionStrings>
    <add name="AdventureWorksEntities" connectionString="metadata=res://*/Entities.OrderDataModel.csdl|res://*/Entities.OrderDataModel.ssdl|res://*/Entities.OrderDataModel.msl;provider=System.Data.SqlClient;provider connection string=&amp;quot;data source=.\sql2008r2;initial catalog=AdventureWorks;integrated security=True;multipleactiveresultsets=True;App=EntityFramework&amp;quot;" providerName="System.Data.EntityClient"/>
  </connectionStrings>
</configuration>
```

Changes to the client app.config: 

1. Set the binding security mode to "Message"
2. Set the transport clientCredentialType to "None"
3. Add the message clientCredentialType of "Certificate"
4. Add the serviceCertificate to the clientCredentials endpointBehavior

``` xml
<? xml version="1.0" encoding="utf-8" ?>
<!-- Client Config -->
<configuration>
  <system.serviceModel>
    <bindings>
      <wsHttpBinding>
        <binding name="WSHttpBinding_IOrderService">
          <security mode="Message">
            <transport clientCredentialType="None"/>
            <message clientCredentialType="Certificate"/>
          </security>
        </binding>
      </wsHttpBinding>
    </bindings>
    <behaviors>
      <endpointBehaviors>
        <behavior name="clientBehaviors">
          <clientCredentials>
            <clientCertificate findValue="MilestoneDevCert" storeLocation="LocalMachine" storeName="My" x509FindType="FindBySubjectName"/>
            <serviceCertificate>
              <authentication certificateValidationMode="None"/>
            </serviceCertificate>
          </clientCredentials>
        </behavior>
      </endpointBehaviors>
    </behaviors>
    <client>
      <endpoint address="http://localhost:9000/OrderService" binding="wsHttpBinding"  bindingConfiguration="WSHttpBinding_IOrderService" contract="AdventureWorks.IOrderService"  name="WSHttpBinding_IOrderService"  behaviorConfiguration="clientBehaviors" />
    </client>
  </system.serviceModel>
</configuration>
```

That's all there is to it. Let's test it. Fire up the server and client and the test should be successful. If you get errors validating the certificates, double check the certificate installation in the Certificates MMC snap in. 

## That's enough to digest of one afternoon. 

In my next post, we'll take a look at how we can use the Windows Azure Service Bus to further protect our application and on-premises network.