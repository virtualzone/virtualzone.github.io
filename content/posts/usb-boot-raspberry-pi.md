---
title: "Native USB boot for Raspberry Pi 4"
date: 2020-05-28T11:30:03+00:00
tags:
    - raspberrypi
author: "Heiner"
aliases:
    - /2020/05/nativer-usb-boot-raspberry-pi-4/
---

Darauf habe sicher nicht nur ich gewartet: Endlich kann der Raspberry Pi 4 von USB-Laufwerken starten! Und das ganz ohne die weit verbreiteten Workarounds, bei denen noch eine SD-Karte als primäres Boot-Medium benötigt wurde. Möglich macht es eine neuen Firmware, ein sogenanntes EEPROM. Und nebenbei gibt es auch eine 64 bit Beta-Version von Raspberry OS, ehemals Raspbian.

Zunächst musst Du dazu Deinen Raspberry Pi mit einem Raspbian bzw. Raspberry OS booten. Nur mit diesem lässt sich die notwendige Beta-Firmware auf den Pi flashen.

## Raspberry OS 64 bit herunterladen
Die 64 bit Beta-Version von Raspberry OS findest Du derzeit verlinkt [in einem Beitrag im offiziellen Forum](https://www.raspberrypi.org/forums/viewtopic.php?t=275370). Lade die ZIP-Datei herunter. Installiere Dir außerdem den [Raspberry Pi Imager](https://www.raspberrypi.org/downloads/). Ich habe diesen unter macOS mit Homebrew installiert:

```bash
brew cask install raspberry-pi-imager
```

## SD-Karte mit Raspberry OS vorbereiten
**Hinweis:** Dieser Schritt ist nur notwendig, wenn Dein Raspberry Pi 4 noch nicht mit Raspbian oder Raspberry OS läuft! Wir brauchen Raspberry OS, um die Firmware des Raspberry Pi zu aktualisieren.

Öffne dazu den Raspberry Pi Imager und flashe das heruntergeladene Image auf eine ausreichend große SD-Karte.

![](/img/raspberry-usb.png)

Starte danach Deinen Pi von der SD-Karte mit dem Raspberry OS.

## EEPROM flashen
Das EEPROM (electrically erasable programmable read-only memory) ist die Firmware Deines Raspberry Pi – sozusagen das Basis-System.

Das [Changelog zum Raspberry Pi EEPROM findest Du auf GitHub](https://github.com/raspberrypi/rpi-eeprom/blob/master/firmware/release-notes.md). Die Beta-Versionen ab dem 15.05.2020 enthalten die notwendige Funktion, um Deinen Raspberry Pi 4 vollständig von einem USB-Laufwerk zu starten – beispielsweise von einer SSD.

Installiere zunächst in Raspberry OS das notwendige Update-Tool:

```bash
sudo apt update
sudo apt upgrade
sudo apt install rpi-eeprom
```

Um die Beta-Firmware (auf eigene Verantwortung!) auf Deinen Pi zu flashen, wechsle innerhalb des gestarteten Raspberry OS zunächst auf den Beta-Channel, indem Du die folgende Datei editierst:

```bash
sudo nano /etc/default/rpi-eeprom-update
```

Ändere die Zeile FIRMWARE_RELEASE_STATUS=”critical” auf:

```bash
FIRMWARE_RELEASE_STATUS="beta"
```

Nun aktualisiere die Firmware durch Ausführung des folgenden Kommandos, gefolgt von einem Reboot:

```bash
sudo rpi-eeprom-update -a
```

Nach dem Neustart sollte der folgende Befehl die Installation der aktuellen Beta-Firmware bestätigen:

```bash
sudo rpi-eeprom-update
```

Alternativ kann eine bestimmte EEPROM-Version auch direkt installiert werden, indem Du sie aus [GitHub herunterlädst](https://github.com/raspberrypi/rpi-eeprom/tree/master/firmware/beta) und dann folgenden Befehl ausführst:

```bash
sudo rpi-eeprom-update -d -f /tmp/pieeprom-2020-05-27.bin
```

## SSD für den USB-Boot vorbereiten
Um nun von einem USB-Laufwerk (bspw. SSD, externe Festplatte oder USB-Stick) booten zu können, verwende den oben erwähnten Raspberry Pi Imager, um das Raspberry OS auf Dein USB-Laufwerk zu schreiben.

Verbinde das USB-Laufwerk anschließend mit Deinem RPi 4, entferne die SD-Karte und starte ihn. Dein Raspberry Pi 4 sollte nun von USB starten – ohne SD-Karten-Workaround.