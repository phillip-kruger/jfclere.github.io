---
title: "Using Tomcat with mod_heartmonitor"
slug: using-tomcat-with-modheartmonitor
date: 2009-04-09 09:31 +0200
year: "2009"
description: "Use the heartbeat load balancing scheduler in httpd-trunk with mod_proxy_balancer and a Tomcat listener to get a partially dynamic load balancing setup."
---

httpd-trunk contains a logic to receive heart beats from backend servers. Associated with mod_proxy_balancer and the heartbeat load balancing scheduler algorithm you can have a partially dynamic load balancing behaviour.

Configure httpd.conf with something like the following (using httpd-trunk):

```text
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_ajp_module modules/mod_proxy_ajp.so
LoadModule proxy_balancer_module modules/mod_proxy_balancer.so
LoadModule heartmonitor_module modules/mod_heartmonitor.so
LoadModule lbmethod_heartbeat_module modules/mod_lbmethod_heartbeat.so
LoadModule lbmethod_byrequests_module modules/mod_lbmethod_byrequests.so

ProxyPass /myapp balancer://mycluster/myapp lbmethod=heartbeat
HeartbeatStorage logs/hb.dat
HeartbeatListen 224.0.1.105:23364

<Proxy balancer://mycluster>
   BalancerMember ajp://10.33.144.3:8009 route=jfpc ping=10
   BalancerMember ajp://10.33.144.4:8009 route=neo3 ping=10
</Proxy>
```

Add the listener in server.xml of each tomcat (using tc_trunk):

```xml
<Listener className="org.apache.catalina.ha.backend.HeartbeatListener" Port="8009"/>
```

Port is the Port of the AJP Connector where httpd is going to proxy the requests.

Start httpd and the Tomcats and have fun with the "dynamic" cluster.
