---
title: "Creating an encrypted file container on macOS"
date: 2016-12-06T11:30:03+00:00
tags:
    - macos
author: "Heiner"
aliases:
    - /2016/12/creating-an-encrypted-file-container-on-macos/
---

Some years ago, I’ve used TrueCrypt to create encrypted containers for storing sensitive files. However, TrueCrypt is nowadays considered insecure and I’m on macOS Sierra 10.12 now – time for another solution. Luckily, macOS has integrated means for creating encrypted containers and saving sensitive information in it. You don’t need any additional software for this. As far as I know, this solution also works for previous versions of Mac OS X, like Mac OS X 10.11 (El Capitan) and Mac OS X 10.10 (Yosemite).

These containers are saved as DMG files. You probably know this file extension from installing downloaded software on your Mac. DMG files are Apple Disk Images, bundling a set of folders and files into a single file. Unlike installation images downloaded from the web, these DMG files can optionally be encrypted using an AES 128 bit or AES 256 bit encryption key.

To create an encrypted file container, open the Disk Utility using the Spotlight Search (press Cmd + Space).

Using the menu bar, navigate to “File” > “New Image” > “Blank Image…”.

Choose an appropriate name for your image and select the following settings:

* Save as: The filename of your encrypted DMG file.
* Name: A name shown when your DMG file is mounted.
* Size: The size of your container. The DMG file will take exactly the specified size and the amount of data you can store in the container is limited to this specified size. However, you can shrink and grow your DMG at a later time.
* Format: Choose “Mac OS Extended (Journaled)”.
* Encryption: Choose between 128 bit AES and 256 bit AES encryption (for sensitive information, I’d go for 256 bit, just in case…). You’ll be prompted to enter an encryption key. Be sure to remember this one really good. There will be no way to recover a lost encryption key!
* Partitions: Choose “Single Partition – Apple Partition Map”.
* Image Format: Choose “read/write disk image”.

Next, click “Create” to create your image. This may take a few minutes, depending on the size of your DMG and the speed of the device you’re creating the container on (i.e. a network share).