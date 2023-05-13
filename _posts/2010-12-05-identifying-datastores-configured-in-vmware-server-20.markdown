---
layout: post
title: Identifying Datastores Configured in VMware Server 2.0
date: 2010-12-05 09:52:00.000000000 -05:00
image: /blog/images/blank-tile.png
author: Jason M Penniman
excerpt: I ran into the need to start a VMware Server 2.0 virtual machine from the command line.  The vmrun command uses the data store name to locate the vmx file. Unfortunately, the system documentation wasn't readily available, so I needed to determine what stores were configured...
category: General-IT
---
I ran into the need to start a VMware Server 2.0 virtual machine from the command line.  The vmrun command uses the data store name to locate the vmx file. Unfortunately, the system documentation wasn't readily available, so I needed to determine what stores were configured.

There are a few ways to do this:

At the command line as root:

```
# vmware-vim-cmd hostsvc/datastore/listsummary
```

This will list all datastores in inventory on the host.  And...

```
# vmware-vim-cmd vmsvc/getallvms
```

... will list all vms in inventory on the host, noting the datastore name and path to the vmx file.

The caveat to that is you need to be running the command as root.  If you're security conscious like I am, then ssh (how I was connecting the the headless host) doesn't allow root access, even doing an "su" won't work.  So unless you've given permissions to a non-root user to access the certificates and configs, the command line utility won't work.  So now what?  Well, thankfully, the configuration is stored in an xml file:

```
/etc/vmware/hostd/datastores.xml
```

...as long as you know the physical path to the vm you want to start, you can look for that path in the datastores.xml file and get the name.
