---
title: Memory Forensics
desc: 
tags: [Forensics, Memory, Volatility]
alts: []
render_with_liquid: false
---

## Volatility

**[Volatility](https://github.com/volatilityfoundation/volatility3){:target="_blank"}** is an useful tool for memory forensics.

- **Windows**

    ```sh
    # Scan processes.
    python3 vol.py -f sample.elf windows.psscan.PsScan

    # List processes in a tree based on their parent process ID.
    python3 vol.py -f sample.elf windows.pstree.PsTree

    # Scan for file objects present in a windows memory image.
    python3 vol.py -f sample.elf windows.filescan.FileScan
    python3 vol.py -f sample.elf windows.filescan.FileScan | grep <keyword>

    # Dump files
    mkdir dumped_files
    python3 vol.py -f sample.elf -o dumped_files windows.dumpfiles.DumpFiles --physaddr <address-of-target-file>
    ```