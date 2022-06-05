---
title: "How to reduce PDF file size in Linux - Part 2"
date: 2015-08-15T11:30:03+00:00
tags:
    - linux
    - macos
    - tool
author: "Heiner"
aliases:
    - /2015/08/how-to-reduce-pdf-file-size-part-2/
---

Several months ago, I wrote a [blog post about reducing a PDF file’s size](/posts/reduce-pdf-file-size/). Since then, I’ve used that technique many times. However, you may want to control the DPI (dots per inch) even more specific. Here’s how to do it:

```bash
gs -dNOPAUSE -dBATCH -sDEVICE=pdfwrite -dCompatibilityLevel=1.7 \
-dDownsampleColorImages=true \
-dDownsampleGrayImages=true \
-dDownsampleMonoImages=true \
-dColorImageResolution=120 \
-dGrayImageResolution=120 \
-dMonoImageResolution=120 \
-sOutputFile=output.pdf input.pdf
```

Hint: This also works on MacOS. Just install GhostScript using [Homebrew](https://brew.sh/): 

```bash
brew install ghostscript
```