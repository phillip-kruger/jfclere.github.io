---
title: "Playing with openssl 3.5.x and PQC"
slug: playing-with-openssl-35x-and-pqc
date: 2025-09-11 12:01 +0200
year: "2025"
description: "First steps with post quantum cryptography in openssl 3.5: an ML-DSA-65 certificate, s_server, and what curl and Firefox make of it."
---

PQC = Post Quantum Cryptography.

PQC can be used in 2 places: the key exchange and the key/cert themselves. For the moment browsers like Firefox and Google Chrome only support the key exchange part.

Create a PQC key/cert using openssl:

```bash
openssl req \
    -x509 \
    -newkey mldsa65 \
    -keyout localhost-mldsa.key \
    -subj /CN=localhost \
    -addext subjectAltName=DNS:localhost \
    -days 30 \
    -nodes \
    -out localhost-mldsa.crt
```

Start openssl s_server:

```bash
openssl s_server \
    -cert localhost-mldsa.crt -key localhost-mldsa.key \
    -trace -port 4433
```

Use curl to test:

```bash
curl -k -v https://localhost:4433/
```

you will get:

```text
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384 / X25519MLKEM768 / id-ml-dsa-65
```

If you try Firefox you will get an error:

```text
Error code: SSL_ERROR_NO_CYPHER_OVERLAP
```

that is expected as Firefox doesn't support the key/cert openssl is using.

Create a RSA key/cert using openssl:

```bash
openssl req \
    -x509 \
    -keyout localhost.key \
    -subj /CN=localhost \
    -addext subjectAltName=DNS:localhost \
    -days 30 \
    -nodes \
    -out localhost.crt
```

Start openssl s_server using the 2 keys and 2 certificates:

```bash
openssl s_server \
    -cert localhost-mldsa.crt -key localhost-mldsa.key \
    -dcert localhost.crt -dkey localhost.key \
    -trace -port 4433
```

Check that curl is working and using the PQC key/cert pair:

```bash
curl -k -v https://localhost:4433/
```

```text
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384 / X25519MLKEM768 / id-ml-dsa-65
```

Try Firefox now, the certificate is self-signed but now Firefox accepts it.
