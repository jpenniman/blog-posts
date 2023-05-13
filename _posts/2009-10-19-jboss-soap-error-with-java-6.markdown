---
layout: post
title: JBoss SOAP error with Java 6
date: 2009-10-19 16:55:00.000000000 -04:00
image: /blog/images/blank-tile.png
author: Jason M Penniman
excerpt: Error message "setProperty must be overridden by all subclasses
  of SOAPMessage". After reading this Jira issue...
category: Development
---
**Error message "setProperty must be overridden by all subclasses of SOAPMessage".**

After reading this Jira issue, I followed the steps mentioned in "JBoss5.0.0GA Installation_And_Getting_Started_Guide.pdf - section 5.4 Java6 Note" and copy below jar files from the JBOSS_HOME/client directory to the JBOSS_HOME/lib/endorsed 

* jbossws-native-saaj.jar
* jbossws-native-jaxrpc.jar
* jbossws-native-jaxws.jar
* jbossws-native-jaxws-ext.jar

and thats it !!

<a href="https://jira.jboss.org/jira/browse/JBWS-1439">https://jira.jboss.org/jira/browse/JBWS-1439</a>
