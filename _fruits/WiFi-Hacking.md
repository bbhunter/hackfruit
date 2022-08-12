---
title: WiFi Hacking
desc: 
tags: [Aircrack, ARP, BSSID, IP, Macchanger, Network, WiFi]
alts: []
render_with_liquid: false
---

## 1. Investigation

1. **Online Tools**

    - **[WiGLE](https://wigle.net/){:target="_blank"}**

        Wireless Network Mapping. If you have the BSSID, you can get the location.

2. **Check Your Status**

    1. **Retrieve the Device IP Address**

        ```sh
        # IP address
        ip addr
        # IP address - Show the specific interface only
        ip addr show eth0
        ip addr show eth1
        ip addr show tun0

        # IPv4 only
        ip -4 addr
        # IPv6 only
        ip -6 addr

        # Static route
        ip route
        ```

    2. **Delete Network Interfaces From Your Devices**

        ```sh  
        ip link delete docker0
        ```

3. **Find Another Computer's IP Address/MAC Address on Network**

    ```sh
    arp -a
    arp -v

    # Specify interface
    arp -i eth0
    arp -i tun0
    ```

<br />

## 2. Crack using Default Router Credentials

```
admin:Admin
admin:admin
admin:password
admin:Michelangelo
root:admin
root:alpine
sitecom:Admin
telco:telco
```

<br />

## 2. MAC Address Spoofing

First of all, you need to use network adapter which has monitor mode on your machine.  
**[Aircrack-ng](https://github.com/aircrack-ng/aircrack-ng){:target="_blank"}** is a complete suite of tools to assess WiFi network security.

1. **Preparation**

    ```sh
    # Show available interfaces
    airmon-ng

    # Put an interface into monitor mode
    airmon-ng start wlan0
    airmon-ng start eth0
    # or
    iwconfig wlan0 mode monitor
    iwconfig eth0 mode monitor

    # Choose the access point (monitor mode)
    airodump-ng wlan0mon
    ```

2. **Retrieve Client's MAC Addresses**

    ```sh
    # Retrieve client's MAC address from the chosen access point
    # -c 9: channel 9
    # --bssid: target router MAC address
    # -w psk: the dump file prefix
    # eth0: interface name
    airodump-ng -c 6 --bssid XX:XX:XX:XX:XX:XX -i wlan0mon
    airodump-ng -c 9 --bssid 00:14:6C:7E:40:80 -w psk eth0
    ```

3. **Spoof MAC Address using the Retrieved Address**

    ```sh
    # Take down the network at first
    ip link set wlan0 down

    # Set MAC address which you got by airodump-ng in the previous section
    macchanger -m XX:XX:XX:XX:XX:XX wlan0

    # Bring up the network
    ip link set wlan0 up
    ```

4. **Confirmation**

    ```sh
    # Check the current MAC address
    macchanger -s wlan0
    ```

5. Reset to the Original MAC Address

    ```sh
    # Reset to the original (permanent) MAC address
    macchanger -p wlan0
    ```

<br />

## 3. WiFi Password Recovery

1. **Windows**

    Open Command Prompt as root privilege like Administrator

    ```sh
    # Show all network names you've accessed and saved
    netsh wlan show profile
    
    # Show the details of the specific network including password
    netsh wlan show profile name="network-name" key=clear
    ```

<br />

## 4. Other Useful Tools

- **[Bettercap](https://www.bettercap.org/){:target="_blank"}**

    The Swiss Army knife for 802.11, BLE, IPv4 and IPv6 networks reconnaissance and MITM attacks.

- **[OUI Standards](https://standards-oui.ieee.org/oui/oui.txt){:target="_blank}"**

    List of MAC OUI (Organizationally Unique Identifier). You can get the information from the BSSID.

    1. **Access to the OUI Standards**
    
        If the target BSSID is "B4:5D:50:AA:86:41", search text by inputting "B4-5D-50" on the string search.  
        Then check the information.
