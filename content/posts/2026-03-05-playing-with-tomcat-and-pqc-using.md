---
title: "Playing with Tomcat and PQC (using keytool)"
slug: playing-with-tomcat-and-pqc-using
date: 2026-03-05 13:42 +0100
year: "2026"
description: "Create ML-DSA and RSA key pairs with keytool and serve both from one Tomcat SSLHostConfig, with curl traces to prove which certificate is used."
---

As explained in my previous post the JVM doesn't have the TLS support yet. But the keytool can be used to create the keystore and Tomcat will use OpenSSL to do the TLS crypto part.

Create the keystore (I have used java25):

```bash
# key/cert signer for certificates.
keytool -keystore keystore -storepass changeit \
        -genkeypair -alias ec -keyalg EC \
        -dname CN=CA_test -ext bc
# key/cert ML-DSA to use for test.
keytool -keystore keystore -storepass changeit -genkeypair -alias mldsa -keyalg ML-DSA -groupname ML-DSA-65 -dname CN=localhost -signer ec
# traditional key/cert to use for test.
keytool -keystore keystore -storepass changeit -genkeypair -alias mykey -keyalg RSA -dname CN=localhost -signer ec
```

In the Tomcat connector:

```xml
<SSLHostConfig>
    <Certificate certificateKeystoreFile="conf/keystore"
                 certificateKeyAlias="mykey"
                 certificateKeystorePassword="changeit" type="RSA" />
    <Certificate certificateKeystoreFile="conf//keystore"
                 certificateKeyAlias="mldsa"
                 certificateKeystorePassword="changeit" type="MLDSA" />
</SSLHostConfig>
```

To test use FFM and curl:

```bash
curl -ivk --curves X25519 https://localhost:8443 -o /dev/null
```

The certificate will be the ML-DSA one:

```text
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384 / x25519 / id-ml-dsa-65
```

and later in the trace:

```text
*   Certificate level 0: Public key type ML-DSA-65 (15616/192 Bits/secBits), signed using ecdsa-with-SHA384
*   Certificate level 1: Public key type EC/secp384r1 (384/192 Bits/secBits), signed using ecdsa-with-SHA384
```

```bash
curl -ivk --curves X25519 --sigalgs RSA-PSS+SHA256 https://localhost:8443 -o /dev/null
```

The certificate will be the other, traditional one:

```text
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384 / x25519 / RSASSA-PSS
```

and later in the trace:

```text
*   Certificate level 0: Public key type RSA (3072/128 Bits/secBits), signed using ecdsa-with-SHA384
*   Certificate level 1: Public key type EC/secp384r1 (384/192 Bits/secBits), signed using ecdsa-with-SHA384
```
