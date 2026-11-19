---
title: "Playing with Apache httpd and PQC"
slug: playing-with-apache-httpd-and-pqc
date: 2025-09-11 16:34 +0200
year: "2025"
description: "Give Apache httpd an ML-DSA-65 certificate next to the RSA one, then watch Firefox and PQC and non-PQC builds of curl pick different key exchanges."
---

At the time of writing, browsers like Firefox and Google Chrome only support PQC for key exchange. curl, when compiled with openssl 3.5.x, supports the exchange and the cert/key.

Based on my previous post, create a RSA key/cert pair for the browsers.

Create an MLDSA-65 for curl tests:

```bash
openssl req \
    -x509 \
    -newkey mldsa65 \
    -keyout localhost-mldsa-65.key \
    -subj /CN=localhost \
    -addext subjectAltName=DNS:localhost \
    -days 30 \
    -nodes \
    -out localhost-mldsa-65.crt
```

Configure httpd.conf:

```text
Listen 4433
<VirtualHost *:4433>
SSLEngine on
SSLCertificateFile localhost.crt
SSLCertificateKeyFile localhost.key
# PQC cert/key
SSLCertificateFile localhost-mldsa-65.crt
SSLCertificateKeyFile localhost-mldsa-65.key
</VirtualHost>
```

Start Apache httpd.

Test with Firefox, accept the self-signed certificate and you will get the "It works!" page.

Test with curl:

```bash
curl -k -v https://localhost:4433/
```

you get:

```text
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384 / X25519MLKEM768 / id-ml-dsa-65
```

Use a curl that doesn't support PQC (compiled with a 3.2.x openssl for example):

```bash
curl -k -v https://localhost:4433/
```

you get:

```text
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384 / x25519 / RSASSA-PSS
```

You can also tell a PQC curl to use x25519 and ask for an RSA key/cert:

```bash
curl -k -v --curves X25519 --sigalgs RSA-PSS+SHA256 https://localhost:4433/
```

Or X25519MLKEM768 and ask for an RSA key/cert:

```bash
curl -k -v --curves X25519MLKEM768 --sigalgs RSA-PSS+SHA256 https://localhost:4433/
```
