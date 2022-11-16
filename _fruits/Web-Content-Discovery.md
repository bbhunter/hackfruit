---
title: Web Content Discovery
desc: 
tags: [API, Content, Dirb, Directory, Discovery, Ffuf, Gobuster, Web]
alts: [Web-Basic-Pentesting, Web-Login-Bypass]
render_with_liquid: false
---

## Manual Discovery

```sh
# Files
/robots.txt
/security.txt
/.well-known/security.txt
/sitemap.xml
/sitemaps.xml
/main.js
/script.js
/js/jquery.min.js
/js/main.js
/js/script.js

# Directories
/admin/
/blog/

# Sensitive information
/.env

# GitHub
/READ.md
/.git
/.github
/.gitignore

# Apache Tomcat
/manager

# If you know the users manage the website, try the usernames
/admin
/administrator
/john
/michael
```

<br />

## Tools

**[SecLists](https://github.com/danielmiessler/SecLists){:target="_blank"}{:rel="noopener"}** is useful wordlists.  
They are usually located in */usr/share/seclsits/* in Linux.

**rockyou** is also amazing wordlist. It is located in */usr/share/wordlists/rockyou.txt*.  

- **Ffuf**

    For bug bounty, set the rate-limiting by adding the **“-rate 1”** flag for avoiding to disrupt the website.

    ```sh
    ffuf -u https://vulnerable.com/FUZZ -w wordlist.txt 
    ffuf -u http://vulnerable.com/FFUF -w wordlist.txt -rate 1

    # Custom header (-H)
    ffuf -H "Cookie: key=value" -u https://vulnerable.com/FUZZ -w wordlist.txt 

    # -mc: Match HTTP statuc code
    ffuf -u http://vulnerable.com/FUZZ -w wordlist.txt -mc 200
    # -ms: Match HTTP response size
    ffuf -u http://vulnerable.com/FUZZ -w wordlist.txt -ms 1234
    ffuf -u http://vulnerable.com/FUZZ -w wordlist.txt -ms 50-300

    # -fc: Filter HTTP statuc code
    ffuf -u http://vulnerable.com/FUZZ -w wordlist.txt -fc 302
    # -fs: Filter HTTP response size
    ffuf -u http://vulnerable.com/FUZZ -w wordlist.txt -fs 1234
    ffuf -u http://vulnerable.com/FUZZ -w wordlist.txt -fs 50-300
    ```

- **Gobuster**

    ```sh
    gobuster dir -u https://vulnerable.com -w wordlist.txt
    ```

- **Dirb**

    ```sh
    dirb https://vulnerable.com/
    dirb https://vulnerable.com/ wordlist.txt

    # Custom header (-H)
    dirb https://vulnerable.com/ -H "Authorization: Basic {token}" wordlist.txt
    # File Extensions (-X)
    dirb https://vulnerable.com/ -X .txt
    ```

- **FeroxBuster**

    **[FeroxBuster](https://github.com/epi052/feroxbuster){:target="_blank"}{:rel="noopener"}** is a recursive content discovery.

    ```sh
    feroxbuster -u https://vulnerable.com

    # Specify extensions (-x)
    feroxbuster -u https://vulnerable.com -x html,js,php
    # No recursion (-n)
    feroxbuster -u https://vulnerable.com -n
    # Custom header (-H)
    feroxbuster -u https://vulnerable.com -H "Authorization: Bearer {token}"
    ```

- **Hakrawler**

    **[Hakrawler](https://github.com/hakluke/hakrawler){:target="_blank"}{:rel="noopener"}** is a simple web crawler designed for quick discovery of endpoints and assets within a web application.

    ```sh
    echo https://vulnerable.com | hakrawler
    ```

<br />

## Framework Detection from Favicon

Get the information of the used framework from favicon.

```sh
curl https://vulnerable.com/images/favicon.ico | md5sum
```

Then check what is the framework used in the website with the [OWASP Favicon Database](https://wiki.owasp.org/index.php/OWASP_favicon_database){:target="_blank"}{:rel="noopener"}.