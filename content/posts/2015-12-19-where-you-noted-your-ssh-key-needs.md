---
title: "Where you noted your ssh key needs renew"
slug: where-you-noted-your-ssh-key-needs
date: 2015-12-19 19:34 +0100
year: "2015"
description: "After upgrading to Fedora 23 ssh stopped working with a 'Roaming not allowed' message, which turned out to be ten-year-old DSA keys that need replacing."
---

After updating to Fedora 23 I noted I can't use ssh from my laptop to servers I use to connect to. I have got the weird message with `ssh -vvv`:

```text
debug1: Roaming not allowed by server
```

Obviously it isn't a server problem. While trying to understand I have found: https://www.gentoo.org/support/news-items/2015-08-13-openssh-weak-keys.html

Of course adding

```text
PubkeyAcceptedKeyTypes=+ssh-dss
```

in .ssh/config helps.

Checking the key shows that I need new key pairs soon:

```text
-rw-------. 1 jfclere jfclere   736 Nov 22  2006 id_dsa
-rw-r--r--. 1 jfclere jfclere   619 Nov 22  2006 id_dsa.pub
```

I have been using the same keys for ~10 years in fact!!!

Well, mostly only inside the Red Hat vpn. Time to change keys and spot the servers I am using only from time to time...
