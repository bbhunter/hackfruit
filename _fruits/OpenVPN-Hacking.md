---
title: OpenVPN Hacking
desc: 
tags: [MSS, MTU, Network, OpenVPN, VPN]
alts: []
render_with_liquid: false
---

## Set Correct MTU (Maximum Transmission Unit)

1. **Get correct MTU**

    Start Ping at the 1500 mtu and decrease the 1500 value by 10 each time.  
    On Linux,

    ```sh
    # -M: mtu discovery
    # -s: data size
    ping -M do -s 1500 -c 1 <target-ip>

    # if the packet loss, 
    ping -M do -s 1490 -c 1 <target-ip>

    # if the packet loss yet,
    ping -M do -s 1480 -c 1 <target-ip>

    # if packet loss yet,
    ping -M do -s 1470 -c 1 <target-ip>

    # continue until packet is received...
    ```

2. **Get correct MSS (Maximum Segment Size)**

    After you find the correct MTU, now you need to get the MSS from the MTU.  
    To get the correct one, subtract 40 from the value of the MTU.

    ```
    mss = mtu - 40
    ```

    For example, if you get 1470 value of the MTU, the MSS is 1430.

3. **Set correct MSS into the config file of OpenVPN**

    Set **mssfix** in the configuration file (.ovpn) of the OpenVPN.

    ```
    mssfix = 1430
    ```

    Replace the 1430 value with the value you found.