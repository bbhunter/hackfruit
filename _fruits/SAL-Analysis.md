---
title: SAL Analysis
desc: .sal is a Saleae's capture file format. To analyze the file, use Saleae's Logic Analyzer.
tags: [Device, Hardware, Logic, SAL, Saleae]
alts: []
render_with_liquid: false
---

## Analysis

**[Saleae's Logic Analyzer](https://www.saleae.com/){:target="_blank"}{:rel="noopener"}** is a tool for hardware analysis.  
Download the analyzer and run it.

```sh
chmod +x ./Logic-x.x.x-master.AppImage
./Logic-x.x.x-master.AppImage
```

In the analyzer, click "Open a capture" and select the target ".sal" file.  
Open the "Analyzer" tab on the right of the windows and click on the "Async Serial".  
The dialog opens and click save button.

<br />

## Calculate Bit Rate from Intervals

```
Bit rate (bit/s) = 1 second / (interval(microseconds) x 10^(-6)) seconds
```