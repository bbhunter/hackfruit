---
title: Windows Forensics
desc: 
tags: [Forensics, Malware, Windows]
alts: []
render_with_liquid: false
---

## 1. Registry Hives

1. **Use Registry Editor**

    You can find registry key in the Registry Editor.

    1. **Open Registry Editor**

        1. Push **Windows key + R key** to open a prompt and type **"regedit"** to open Registry Editor.

        2. In Registry Editor, right-click on the value of the left pane and click.
    
    2. **Default Location of Registry Hives**

        They are located in **C:\Windows\System32\config**.  
        These hives are:

        - **DEFAULT (mounted on HKEY_USERS\DEFAULT)**

        - **SAM (mounted on HKEY_LOCAL_MACHINE\SAM)**

        - **SECURITY (mounted on HKEY_LOCAL_MACHINE\Security)**

        - **SOFTWARE (mounted on HKEY_LOCAL_MACHINE\Software)**

        - **SYSTEM (mounted on HKEY_LOCAL_MACHINE\System)**

    3. **The Other Hives**

        The other registry hives are located in **C:\Users\<username>**.  
        These hives are:

        - **NTUSER.DAT (mounted on HKEY_CURRENT_USER) - located in C:\Users\<username>\\**

        - **USRCLASS.DAT (mounted on HKEY_CURRENT_USER\Software\CLASSES) - located in C:\Users\<username>\AppData\Local\Microsoft\Windows**

        Note that these hives are hidden files.

    4. **Amcache Hive**

        Amcache Hive is located in **C:\Windows\appcompat\Programs\Amcache.hve**.  
        This hive stores the information on programs that were recently run on the system.

<br />

## 2. Acquire Registry Data

1. **Use KAPE**

2. **Use [Autopsy](https://www.autopsy.com/){:target="_blank"}**

3. **Use [FTK Imager](https://www.exterro.com/ftk-imager){:target="_blank"}**

<br />

## 3. Explore Registry Data

1. **Use [Registry Viewer](https://accessdata.com/product-download-page){:target="_blank"}**

<br />

## 4. System Information

1. **OS Version**

    The location is **HKEY_CURRENT_USER\Software\Microsoft\Windows NT\CurrentVersion**.

<br />

## 5. Memory Forensics

1. **Use Volatility**

    **[Volatility](https://github.com/volatilityfoundation/volatility3){:target="_blank"}** extracts digital artifacts from volatile memory (RAM) samples. memory capture file is like .bin, .mem, .raw, .sav, .vmem.

    1. **Commands**

        ```sh
        # OS information
        python vol.py -f example.vmem windows.info

        # Process information
        python vol.py -f example.vmem windows.pslist
        python vol.py -f example.vmem windows.psscan
        python vol.py -f example.vmem windows.pstree

        # Network connections
        python vol.py -f example.vmem windows.netscan

        # Hidden processes
        python vol.py -f example.vmem windows.ldrmodules

        # Detect malware
        python vol.py -f example.vmem windows.malfind

        # DLL files
        python vol.py -f example.vmem windows.dlllist
        ```
