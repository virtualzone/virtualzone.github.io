---
title: "Raspberry Pi OS: Remove unnecessary packages"
date: 2020-06-07T11:30:03+00:00
tags:
    - raspberrypi
author: "Heiner"
aliases:
    - /2020/06/raspberry-pi-os-64-bit-lite-desktop-pakete-entfernen/
---

Recently, [I wrote about](/posts/usb-boot-raspberry-pi/) the availability of the 64 bit beta version of Raspberry Pi OS (formerly known as Raspbian). Unfortunately, the new 64 bit beta is only available in the Desktop variant, containing lots of packages most lightweight server systems won't need. There's no lite variant of the 64 bit beta version available at the time of writing. However, you can easily remove the Desktop packages from a running installation with two easy commands.

You can download Raspberry Pi OS' 64 bit beta version [from the download directory on Raspberry Pi's website](https://downloads.raspberrypi.org/raspios_arm64/images/). The [Raspberry Pi Imager](https://www.raspberrypi.org/downloads/) makes it easy to burn the image to an SD card or external USB drive.

![](/img/raspberry-usb.png)

Enter the following commands (at your own risk!) to remove the Desktop packages after your Pi has started from the newly written card:

```bash
sudo apt-get remove --purge \
    x11-* \
    gnome-* \
    desktop-base \
    *-theme \
    dconf-gsettings-backend \
    gsettings-desktop-schemas \
    gtk- \
    gtk2-* \
    xdg-*
sudo apt-get autoremove --purge
```