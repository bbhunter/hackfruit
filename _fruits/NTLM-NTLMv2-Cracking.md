---
title: NTLM, NTLMv2 Cracking
desc: Windows New Technology LAN Manager (NTLM) is a suite of security protocols.
tags: [Crypto, Hash, John, NTLM, Password, Windows]
alts: []
render_with_liquid: false
---

## Decrypt

- **NTLM**

    ```sh
    john --format=nt --wordlist=wordlist.txt hash.txt
    # or
    hashcat -m 1000 -a 0 hash.txt wordlist.txt
    ```

- **NTLMv2**

    ```sh
    john --format=netntlmv2 --wordlist=wordlist.txt hash.txt
    ```
