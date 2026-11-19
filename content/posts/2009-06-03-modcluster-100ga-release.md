---
title: "mod_cluster 1.0.0.GA release"
slug: modcluster-100ga-release
date: 2009-06-03 10:27 +0200
year: "2009"
description: "The first GA of mod_cluster, a dynamic replacement for mod_proxy_balancer that takes context and load information straight from the cluster nodes."
---

mod_cluster brings a dynamic configuration to mod_proxy. In fact mod_cluster is a replacement of mod_proxy_balancer and it uses some of the new features of httpd-trunk like slotmem.

The improvements to mod_proxy/mod_jk are:

- Dynamic configuration of contexts (no more 404 or 400 when an application is being redeployed).
- Load information received from the node of the cluster.
- Asynchronous cping/cpong via a STATUS message from the cluster.
- Minimal configuration.

Paul also blogged on the topic, see http://pferraro.wordpress.com/

For more see http://www.jboss.org/mod_cluster/ , try it and join the growing community.
