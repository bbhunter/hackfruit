---
title: Recon
desc: Basic reconnaisance flows.
tags: [DB, DNS, Exploit, Network, Nmap, OSINT, Recon, Subnet, Web]
alts: [OSINT]
render_with_liquid: false
---

## Network Investigation

1. **Scan Using Nmap**

    - **Subnet Scan**

        You need only the **ping scan (skip port scan)** by adding the option "-sP".

        ```sh
        # /24 - 255.255.255.0
        nmap -sP <target-ip>/24

        # /16 - 255.255.0.0
        nmap -sP <target-ip>/16

        # /8 - 255.0.0.0
        nmap -sP <target-ip>/8
        ```

<br />

## Port Scan

1. **Use Nmap**

    - **Basic Scan**

        It's recommened to do as stealth scan (SYN scan) by adding the option "-sS".

        ```sh
        sudo nmap -sS <target-ip>

        # OS version detection (-V)
        # Default NSE (-C)
        sudo nmap -sSVC <target-ip>
        # All detection
        sudo nmap -sS -A <target-ip>
        ```

        Skipping the host discovery

        ```sh
        sudo nmap -sS -Pn <target-ip>
        ```

        Scanning all ports.

        ```sh
        sudo nmap -sS -p- <target-ip> --min-rate 1000
        sudo nmap -sS -p 1-65535 <target-ip> --min-rate 1000
        ```

        Scanning the specific range ports.

        ```sh
        sudo nmap -sS -p 1000-1500 <target-ip>
        ```

        Scannning top ports.

        ```sh
        sudo nmap -sS --top-ports 100 <target-ip>
        ```

        First 1000 ports.

        ```sh
        sudo nmap -sS -p-1000 <target-ip>
        ```

    - **UDP Scan**

        Sometimes you need the UDP scan.

        ```sh
        nmap -sU --top-ports 25 <target-ip>
        ```

    - **Other Scan Techniques**

        FIN scan

        ```sh
        nmap -sF <target-ip>
        ```

        Xmas scan

        ```sh
        nmap -sX <target-ip>
        ```

    - **Firewall Bypass**

        ```sh
        # Fragmented packets
        nmap -f <target-ip>

        # Specify MTU (Maximum Transmission Unit)
        nmap --mtu 16 <target-ip>
        nmap --mtu 24 <target-ip>

        # Decoy
        nmap -D RND:3 <target-ip>
        ```

    - **Using Nmap Scripting Engine (NSE)**

        ```sh
        nmap -sC <target-ip>
        nmap --script vuln <target-ip>
        ```

2. **Use Massscan**

    **[Masscan](https://github.com/robertdavidgraham/masscan){:target="_blank"}** is a TCP port scanner.

    - **Basic Scan**

        ```sh
        masscan <target-ip>/16
        masscan <target-ip>/24
        ```

        Specify ports

        ```sh
        masscan <target-ip>/16 -p 80,443
        ```

        Specify port range / All ports

        ```sh
        masscan <target-ip>/16 -p 22-80
        masscan <target-ip>/16 -p 0-65535
        ```

        Specify top ports

        ```sh
        masscan <target-ip>/16 --top-ports 100
        ```

<br />

## Find Vulnerabilites

1. **Automation**

    - **Use Nuclei**

        **[Nuclei](https://github.com/projectdiscovery/nuclei){:target="_blank"}** is a vulnerability scanner based on simple YAML based DSL. 

        ```sh
        nuclei -h
        ```

1. **Use Exploit DB**

    - **Searchsploit**

        You can search vulnerabilites of Exploit-DB in command-line.

        ```sh
        searchsploit <keyword>
        ```

        If you found vulnerabilities of target, copy them to current directory.  
        For example,

        ```sh
        searchsploit -m windows/remote/42031.py
        # or
        searchsploit -m 42031
        ```

    **[Exploit-DB](https://www.exploit-db.com/){:target="_blank"}** is a database of exploits.  
    Find the exploit and download it.  
    For example:

    ```sh
    wget https://www.exploit-db.com/raw/42966 -O exploit.py
    ```

    Format the exploit code for UNIX.

    ```sh
    dos2unix exploit.py

    # Manual converting
    sed -i 's/\r//' example.py
    ```
