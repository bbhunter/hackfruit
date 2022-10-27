---
title: GPG Cracking
desc: GNU Privacy Guard (GPG) is a free-software replacement for Symantec's PGP cryptographic software suite.
tags: [Crypto, GPG, Hash, John, Password]
alts: []
render_with_liquid: false
---

## Decrypt

 1. **Crack the Passphrase from the Private Key**

    First of all, you need to format the private key to make the John to recognize it.

    ```sh
    gpg2john private_key.asc > key.txt
    gpg2john private_key.sig > key.txt
    ```

    Crack the passphrase using the formatted text.

    ```sh
    john --wordlist=/path/to/wordlist key.txt
    ```

2. **Import the Private Key**

    ```sh
    gpg --import private_key.asc
    gpg --import private_key.sig
    ```

3. **Decrypt GPG (PGP) using the Passphrase**

    At that time, you'll be asked for the passphrase, so enter the passphrase you gotten in the previous section.

    ```sh
    # -d: decrypt
    gpg -d example.gpg
    gpg -d example.pgp
    ```

<br />

## Encrypt

```sh
# -e: encrypt
gpg -e example.txt

# -c: encrypt only with symmetric cipher
gpg -c example.txt
```