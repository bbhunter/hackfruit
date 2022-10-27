---
title: PEM Cracking
desc: Privacy Enhanced Mail (PEM). It uses RSA encryption.
tags: [Crypto, Hash, John, Password, PEM, RSA]
alts: []
render_with_liquid: false
---

## Decrypt

First of all, you need to format the PEM file to make the John to recognize it.

```sh
pem2john example.pem > hash.txt
```

Crack the hash.

```sh
john --wordlist=wordlist.txt hash.txt
```