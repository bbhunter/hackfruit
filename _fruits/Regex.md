---
title: Regular Expressions (Regex/RegExp)
desc: Patterns of text that specifies a search pattern in text.
tags: [Expression, Pattern, Regex, Regular, String, Text]
alts: []
render_with_liquid: true
---

{% raw %}

## Online Tools

- **[RegExp Playground](https://projects.verou.me/regexplained/){:target="_blank"}**

    You’re able to try various patterns in this site easily.

- **[RegExr](https://regexr.com/){:target="_blank"}**

    Learn, build, and test regex.

<br />

## Commands

1. **Pattern Matching**

    ```sh
    grep '^0\d{9,10}$' example.txt
    ```

<br />

## Example Patterns

1. **Basic**

    ```sh
    [a-z]
    # a, b, c, ..., y, z
    [a-z]+
    # abc, aaabbbcc, ghslkdja, ...
    [a-zA-Z]
    # a, A, b, B, ..., z, Z
    [a-z]zz
    # azz, bzz, czz, ...
    [a-zA-Z]+zz
    # azz, AaBbCdefzz, hkIkWEzz, ...
    [a-z]?zz
    # zz, azz, bzz, ...

    [0-9]
    # 0, 1, 2, ..., 9
    [0-9]+
    # 0, 00, 123, 77777, ...

    [^a]pple
    # bpple, cpple, zpple, 2pple, ...
    [^a-c]pple
    # dpple, epple, 2pple, ...

    a.c
    # aac, abc, a"c, a;c, a c, ...
    a\.c
    # a.c

    abc?
    # abc, ab

    \d
    # 0, 1, ..., 9
    \D
    # a, b, A, Z, ;, +, ?, ....
    \w
    # a, A, b, B, ..., z, Z, 0, 1, 3, ...9
    \W
    # !, ", #, ...
    \s
    # \r, \n
    \S
    # a, A, b, B, ..., 0, 1, ..., !, ", #, ...

    a{3}
    # aaa
    abc{3}
    # abccc
    \d{3}
    # 123, 444, 987, ...

    a{2,5}
    # aa, aaa, aaaa, aaaaa

    a{3,}
    # aaa, aaaa, aaaaa, aaaaaa, ...

    a*
    # <NONE>, a, aa, aaa, aaaa, ...
    a+
    # a, aa, aaa, aaaa, ...

    ^abc
    # start with "abc"
    xyz$
    # end with "xyz"
    EOF\$$
    # end with "EOF$"

    hello (john|jane)
    # hello john, hello jane

    (abc){3}
    # abcabcabc
    ```

2. **In Real**

    - **IPv4 Address**

        ```sh
        (\d{1,3}\.){3}\d{1,3}
        # 127.0.0.1, 8.8.8.8, 255.255.255.255, ...
        ```

    - **Email Address**

        ```sh
        (\w+)@(\w+)\.\w+(\.\w+)?
        # user@examle.com, info@example.jp, ...
        ```

    - **TEL**

        ```sh
        ^0\d{9,10}$
        # 0000000000, ...

        ^0\d{1,3}-\d{1,4}-\d{4}
        # 000-000-0000, ...
        ```

{% endraw %}