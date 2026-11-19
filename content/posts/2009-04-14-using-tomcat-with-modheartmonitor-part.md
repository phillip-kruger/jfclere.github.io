---
title: "Using Tomcat with mod_heartmonitor (part 2)"
slug: using-tomcat-with-modheartmonitor-part
date: 2009-04-14 17:11 +0200
year: "2009"
description: "The second part on Tomcat with mod_heartmonitor, this time sending heartbeats over HTTP instead of multicast sockets, with the httpd and server.xml config."
---

This is the second blog on the topic of using Tomcat with mod_heartmonitor. Here it is using HTTP instead of multicast sockets... Well, no one likes multicast sockets but everyone likes the HTTP protocol, no?

In the httpd.conf file add something like the following (to the httpd.conf of the previous blog):

```xml
<Location /HeartbeatListener>
   SetHandler hearthbeat
   Order deny,allow
   Deny from all
   Allow from 127.0.0.1
   Allow from 10.33.144.4
</Location>
```

In the server.xml add ProxyList to the HeartbeatListener:

```xml
<Listener className="org.apache.catalina.ha.backend.HeartbeatListener" Port="8009" ProxyList="127.0.0.1:7779" />
```

Have fun!!!
