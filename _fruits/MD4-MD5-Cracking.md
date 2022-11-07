---
title: MD4, MD5 Cracking
desc: 
tags: [Cipher, Crack, Crypto, Hash, John, MD, Password]
alts: [Cryptography]
render_with_liquid: false
---

## Online Tools

- **[MD5 Center](https://md5.gromweb.com/){:target="_blank"}**

<br />

## Decrypt

- **MD4**

    Put the md4 hash into the file.

    ```sh
    echo -n '<md4-hash>' > hash.txt
    ```

    Then crack it.

    ```sh
    john --format=raw-md4 --wordlist=wordlist.txt hash.txt
    # or
    hashcat -m 900 -a 0 hash.txt wordlist.txt
    ```

- **MD5**

    Put the md5 hash into the file.

    ```sh
    echo -n '<md5-hash>' > hash.txt
    ```

    Then crack it.

    ```sh
    john --format=raw-md5 --wordlist=wordlist.txt hash.txt
    # or
    hashcat -m 0 -a 0 hash.txt wordlist.txt
    ```

<br />

## Encrypt

- **MD4**

- **MD5**

    ```sh
    echo -n 'hello' | md5sum
    md5sum sample.txt
    ```
