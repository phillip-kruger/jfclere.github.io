---
title: "boinc with http-proxy"
slug: boinc-with-http-proxy
date: 2005-06-24 18:10 +0200
year: "2005"
description: "Getting the boinc client to attach to a project from behind an HTTP proxy by hand-editing the proxy_info block in client_state.xml."
---

First:

```bash
boinc_client -attach_project
```

It will fail.

Second, edit the file client_state.xml, add something like the following:

```xml
<proxy_info>
   <use_http_proxy/>
   <http_server_name>wexpert</http_server_name>
   <http_server_port>8888</http_server_port>
</proxy_info>
```

Third:

```bash
boinc_client -attach_project
```

That should work.
