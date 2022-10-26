---
title: Linux User & File Management
desc: 
tags: [Chmod, Chown, Deluser, Groupadd, Groupdel, Passwd, Linux, Useradd]
alts: []
render_with_liquid: false
---

## User Management

- **List All Users**

    ```sh
    cat /etc/passwd
    cat /etc/shadow
    ```

- **Add Users**
    
    ```sh
    useradd <new-user>
    useradd <new-user> -p <new-password>
    ```
    
- **Delete Users**
    
    ```sh
    deluser <user>
    ```
    
- **Set Passwords**
    
    The password, which is set, is stored into “/etc/shadow”.
    
    ```sh
    passwd <user>
    ```
    
- **Allow Users to Run Commands as Administrator**
    
    To do that, add users to the “sudo” group.
    
    ```sh
    usermod -a -G sudo <user>
    
    # Check the permissions
    su <user>
    sudo -l
    ```

<br />

## Group Management

- **List All Groups**
    
    ```bash
    cat /etc/group
    
    # List groups where the specific user in
    groups
    groups <user>
    ```
    
- **Create New Groups**
    
    ```bash
    groupadd <new-group>
    ```
    
- **Delete Groups**
    
    ```bash
    groupdel <group>
    ```
    
- **Add Users to Groups**
    
    ```bash
    # -a: append the user to the specific group
    # -G: group
    usermod -a -G <group> <user>
    ```
    
- **Change a User’s Primary Group**
    
    ```bash
    # -g: gid
    usermod -g <group> <user>
    ```

<br />

## File Management

- **Change Owners of Files/Directories**
    
    ```bash
    # User only
    chown <user> <file-name>
    chown <user> <dir-name>
    
    # Group only
    chown :<group> <file-name>
    chown :<group> <dir-name>
    
    # User and group
    chown <user>:<group> <file-name>
    chown <user>:<group> <dir-name>
    ```
    
- **Change Permissions of Files/Directories**
    
    ```bash
    # read+write+execute for an owner
    chmod 700 sample.txt
    chmod u+rwx sample.txt
    # read+write+execute for a group
    chmod 070 sample.txt
    chmod g+rwx sample.txt
    # read+write+execute for other users
    chmod 007 sample.txt
    chmod o+rwx sample.txt
    # read+write+execute for all users
    chmod 777 sample.txt
    chmod a+rwx sample.txt
    
    # read+write for an owner
    chmod 600 sample.txt
    ```

- **SUID/GUID of Files/Directories**
    
    ```bash
    # SUID for an owner
    chmod u+s sample.txt
    
    # GUID for a group
    chmod g+s sample.txt
    ```