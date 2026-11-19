---
title: "Playing with Tomcat and PQC"
slug: play-with-tomcat-and-pqc
date: 2026-03-03 14:55 +0100
year: "2026"
description: "Configure a Tomcat connector with an ML-DSA and an RSA certificate on OpenSSL 3.5, through tc-native or FFM, and test both with curl."
---

To use PQC in Tomcat you need to use OpenSSL at least version 3.5, either with the APR connector, with the OpenSSLImplementation or FFM. The current versions of the JVM are not supporting TLS and PQC so we have to use OpenSSL.

Based on my previous posts, create a RSA key/cert pair and an MLDSA-65 one for PQC.

Configure a connector with 2 key/cert pairs:

```xml
<Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
           maxThreads="150" SSLEnabled="true">
    <SSLHostConfig>
        <Certificate certificateFile="conf/localhost-mldsa.crt"
                     certificateKeyFile="conf/localhost-mldsa.key"
                     type="MLDSA" />
        <Certificate certificateFile="conf/localhost/localhost.crt"
                     certificateKeyFile="conf/localhost/localhost.key"
                     type="RSA" />
    </SSLHostConfig>
</Connector>
```

Make sure to have tc-native or FFM configured.

## tc-native

You need a libtcnative*.so linked with OpenSSL 3.5+ in the path of LD_LIBRARY_PATH or in the bin directory of your Tomcat installation, and the listener in server.xml:

```xml
<Listener className="org.apache.catalina.core.AprLifecycleListener" />
```

## FFM

You need a "recent" version of the JVM (at least 22), OpenSSL 3.5+ libraries in the LD_LIBRARY_PATH and the following in server.xml:

```xml
<Listener className="org.apache.catalina.core.OpenSSLLifecycleListener" />
```

## To test

Start Tomcat and test with curl for example. You can tell a PQC enabled curl to use x25519 and ask for a classical RSA key/cert:

```bash
curl -k -v --curves X25519 --sigalgs RSA-PSS+SHA256 https://localhost:8443/
```

Or X25519MLKEM768 and ask for an ML-DSA-65 key/cert for PQC:

```bash
curl -k -v --curves X25519MLKEM768 --sigalgs ML-DSA-65 https://localhost:8443/
```
