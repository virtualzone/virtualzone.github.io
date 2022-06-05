---
title: "Raspberry Pi OS: Remove unnecessary packages"
date: 2020-06-07T11:30:03+00:00
tags:
    - raspberrypi
author: "Heiner"
aliases:
    - /2020/06/raspberry-pi-os-64-bit-lite-desktop-pakete-entfernen/
---

Kürzlich [schrieb ich darüber](/posts/usb-boot-raspberry-pi/), dass es eine erste 64 bit Beta-Version von Raspberry Pi OS (ehemals Raspbian) gibt. Diese gibt es bislang leider nur in der Desktop-Variante und noch nicht als Lite-Version. Mit zwei Befehlen kannst Du jedoch ganz leicht die – sofern Du sie nicht benötigst – überflüssigen Desktop-Pakete deinstallieren.

Die Beta von Raspberry Pi OS 64 bit kannst Du im [Download-Verzeichnis der Raspberry Pi Seite](https://downloads.raspberrypi.org/raspios_arm64/images/) herunterladen. Auf eine SD-Karte oder SSD bekommst Du das heruntergeladene Image am einfachsten mit dem [Raspberry Pi Imager](https://www.raspberrypi.org/downloads/).

![](/img/raspberry-usb.png)

Nach dem Start habe ich mit folgenden beiden Befehlen die für mich überflüssigen Desktop-Pakete deinstalliert:

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