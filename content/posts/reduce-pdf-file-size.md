---
title: "How to reduce PDF file size in Linux"
date: 2012-11-21T11:30:03+00:00
tags:
    - linux
    - macos
    - tool
author: "Heiner"
aliases:
    - /2012/11/how-to-reduce-pdf-file-size-in-linux/
---

Using a single line of GhostScript command on my Ubuntu’s terminal, I was able to reduce the size of a PDF file from 6 MB to approximately 1 MB:

```bash
gs -dNOPAUSE -dBATCH -sDEVICE=pdfwrite -dCompatibilityLevel=1.4 -dPDFSETTINGS=/screen -sOutputFile=output.pdf input.pdf
```

You can also use the following parameters for -dPDFSETTINGS instead of /screen:

* /screen – Lowest quality, lowest size
* /ebook – Moderate quality
* /printer – Good quality
* /prepress – Best quality, highest size

**Update:** Read [Part 2 of this blog post](/posts/reduce-pdf-file-size-2/) for more detailled file size reduction settings.

**Hint:** This also works on MacOS. Just install GhostScript using [Homebrew](https://brew.sh/): 

```bash
brew install ghostscript
```