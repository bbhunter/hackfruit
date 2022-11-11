---
title: Pivoting
desc: Accessing obtained over one machine to exploit another machine deeper in the network.
tags: [Network, Pivot]
alts: []
render_with_liquid: false
---

## Enumerate Network

After entering the target server, enumerate and search other networks.

1. **Check the ARP Cache in Target Machine**

    ```sh
    arp -a
    ```

2. **Check IP Addresses**

    In Linux

    ```sh
    cat /etc/hosts
    cat /etc/resolv.conf
    nmcli dev show
    ```

    In Windows

    ```sh
    Get-Content c:\Windows\System32\drivers\etc\hosts
    ipconfig /all
    ```

3. **Search Other Network Ranges**

    ```sh
    nmap 10.0.0.1-255
    ```

    Or manual searching.

    ```sh
    for i in {1..255}; do (ping -c 1 10.0.0.${i} | grep "bytes from" &); done
    ```

4. **Port Scan**

    ```sh
    nmap 10.0.0.1
    for i in {1..65535}; do (echo > /dev/tcp/10.0.0.1/$i) >/dev/null 2>&1 && echo $i is open; done
    ```

## Use Proxychains

**[Proxychains](https://github.com/haad/proxychains){:target="_blank"}{:rel="noopener"}** forces any TCP connection made by any given application to follow through proxy like TOR or any other SOCKS4, SOCKS5 or HTTP(S) proxy.

1. **Examples**

```sh
proxychains telnet <target-ip>
proxychains nc <target-ip> 23
```

2. **Configure**

```sh
# Copy the original config file
cp /etc/proxychains.conf .

# If performing nmap for port scan through proxychains, comment out the following. Otherwise it will hang and crash.
proxy_dns
```

3. **If You Lost Config File**

```sh
wget https://raw.githubusercontent.com/haad/proxychains/master/src/proxychains.conf -O /etc/proxychains.conf
```