---
title: "Native USB boot for Raspberry Pi 4"
date: 2020-05-28T11:30:03+00:00
tags:
    - raspberrypi
author: "Heiner"
aliases:
    - /2020/05/nativer-usb-boot-raspberry-pi-4/
---

Here's something that's probably been eagerly-awaited not only by me: Finally, Raspberry Pi 4 can boot directly from USB devices. Without any of the widespread workarounds which require an SD card a primrary boot medium. This is made possible by a new firmware, the so-called EEPROM. Furthermore, a new 64 bit beta version of Raspberry OS is available, too (formerly known as Raspbian).

To get started, boot your Raspberry Pi with a Raspbian or Raspberry OS installation. This is required to upgrade the new beta firmware.

## Download Raspberry OS 64 bit
You can find the new 64 bit beta version of Raspberry OS [in a forum post](https://www.raspberrypi.org/forums/viewtopic.php?t=275370). Download the ZIP file. Install [Raspberry Pi Imager](https://www.raspberrypi.org/downloads/). I've installed the imager using Homebrew:

```bash
brew cask install raspberry-pi-imager
```

## Prepare an SD card with Raspberry OS
**Note:** This step is only required if your Raspberry Pi is now running Raspbian or Raspberry OS! We need Raspberry OS to flash the new firmware.

Open Raspberry Pi Imager and flash the downloaded image to an SD card.

![](/img/raspberry-usb.png)

Afterwards, boot your Pi from this new SD card.

## Flash EEPROM
EEPROM (electrically erasable programmable read-only memory) is your Raspberry Pi's firmware – sort of a basic system.

You can find the [changelog for the Raspberry Pi EEPROM on GitHub](https://github.com/raspberrypi/rpi-eeprom/blob/master/firmware/release-notes.md). The beta versions as of May 15th 2020 contain the required functionalities to boot from a USB drive – i.e. an SSD.

Install the required update tool on your Pi:

```bash
sudo apt update
sudo apt upgrade
sudo apt install rpi-eeprom
```

To flash the beta firmware (at your own risk!), switch to the beta channel by modifying the following file:

```bash
sudo nano /etc/default/rpi-eeprom-update
```

Change the line FIRMWARE_RELEASE_STATUS=”critical” to:

```bash
FIRMWARE_RELEASE_STATUS="beta"
```

Upgrade the firmware and reboot:

```bash
sudo rpi-eeprom-update -a
```

After the reboot, the following command should state that the new beta firmware has been installed:

```bash
sudo rpi-eeprom-update
```

Alternatively, you can flash the new EEPROM version by downloading it from [the GitHub repository](https://github.com/raspberrypi/rpi-eeprom/tree/master/firmware/beta) and run the following command:

```bash
sudo rpi-eeprom-update -d -f /tmp/pieeprom-2020-05-27.bin
```

## Prepare an SSD for USB boot
To make your Raspberry Pi boot from an USB drive (such as an SSD, an external hard drive or an USB thumb drive), use the Raspberry Pi Imager to write Raspberry Pi OS to your USB drive.

Finally, connect the USB drive to your Raspberry Pi 4, remove the SD card, and connect the power cord. Watch your Pi boot from USB - without any SD Card workaround.