---
title: PowerShell
desc: 
tags: [PowerShell, Windows]
alts: [PowerView, Windows-Privilege-Escalation]
render_with_liquid: false
---

## Start PowerShell

In Linux

```sh
pwsh

# Execute PS commands without entering shell.
pwsh -Command <cmdlet>
```

In Windows

```sh
powershell
```

<br />

## Basic Commands

- **OS Information**

    ```powershell
    $PSVersionInfo
    ```

- **Change Directory**

    **'cd'** in Linux.

    ```powershell
    Set-Location -Path c:\Users\Administrator\Desktop
    ```

- **List Files**

    **'ls'** in Linux.

    ```powershell
    Get-ChildIteme -File -Hidden
    Get-ChildItem -File -Hidden -ErrorAction SilentlyContinue
    Get-ChildItem -Directory -Hidden
    Get-ChildItem -Path .\Desktop
    Get-ChildItem -Recurse
    ```

- **View the Content of Files**

    **'cat'** in Linux

    ```powershell
    Get-Content -Path example.txt
    # 'cat | wc -l' in Linux
    Get-Content -Path example.txt | Measure-Object -Word
    # 
    (Get-Content -Path example.txt)[318]
    ```

- **Find Files**

    **'find'** in Linux

    ```powershell
    Select-String -Path 'c:\Users\Administrator\Desktop' -Pattern '*.txt'
    ```

- **Set Content to a File**

    **'echo hello > example.txt'** in Linux

    ```powershell
    Set-Content -Path .\example.txt -Value hello
    ```

- **Download Web Content**

    **'wget'** in Linux

    ```powershell
    Invoke-WebRequest -Uri http://10.0.0.1:8000/example.exe -OutFile .\example.exe
    ```

- **Cryptography**

    **'md5sum'** in Linux

    ```powershell
    Get-FileHash -Algorithm MD5 example.txt
    ```

- **Print Text Strings**

    **'strings'** in Linux

    ```powershell
    .\Strings.exe -accepteula example.exe
    ```

- **Add New User**

    **'useradd'** in Linux

    ```powershell
    New-LocalUser -Name "username" -Description "My first account" -NoPassword

    # with password
    $Password = Read-Host -AsSecureString
    New-LocalUser -Name "username" -Password $Password -FullName "New User" -Description "My first account"
    ```

- **Show the Manual of Command**

    **'man'** or **'--help'** in Linux

    ```powershell
    Get-Help Get-ChildItem
    Get-Help Invoke-WebRequest
    ```

- **Create New File**

    **'touch'** in Linux

    ```powershell
    New-Item example.txt
    $null > example.txt
    ```

- **Create New Folder**

    **'mkdir'** in Linux

    ```powershell
    mkdir example_folder
    ```

- **Remove Files**

    **'rm'** in Linux

    ```powershell
    rm exxample.txt
    rm -r example_folder
    ```

- **Reboot Computer**

    **'reboot'** in Linux

    ```powershell
    Restart-Computer
    ```

- **View NTFS Files Streams**

    ```powershell
    Get-Item -Path file.exe -Stream *
    ```

<br />

## Active Directory

```sh
# List all domain objects in AD
Get-DomainObject -Identity "dc=example,dc=com" -Domain example.com

# List all domain controllers in AD
Get-DomainController

# List all computers in the newtork
Get-NetComputer <hostname> | Select-Object -Property name, msds-allowedtoactonbehalfofotheridentity

# Get the machine which participates the Active Directory
Get-ADComputer <PC-NAME> -properties dnshostname,serviceprincipalname

# Remove the current SPN attribute
Set-ADComputer <PC-NAME> -ServicePrincipalName @{}

# Set new DNS hostname to that of the DC
Set-ADComputer <PC-NAME> -DnsHostName VULNDC.vuln.local
```

