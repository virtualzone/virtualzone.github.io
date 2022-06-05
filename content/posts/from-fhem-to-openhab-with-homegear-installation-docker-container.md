---
title: "From FHEM to OpenHAB with Homegear: Installation/Docker container"
date: 2016-08-28T11:30:03+00:00
tags:
    - fhem
    - openhab
    - homeautomation
    - docker
author: "Heiner"
aliases:
    - /2016/08/from-fhem-to-openhab-with-homegear-installation-docker-container/
---

For more than 2.5 years, I’ve now been running FHEM with several HomeMatic sensors and actors. Using the HM-CFG-LAN Configuration Tool as an I/O interface between [FHEM](http://fhem.de/) and the [HomeMatic](http://www.homematic.com/) devices, this setup has been running smoothly most of the time. The configuration was a bit tricky now and then, but it worked. However, [OpenHAB](http://www.openhab.org/) seems to become a really good choice. Version 2 is currently available as Beta 3. It features a modern web interface and an easy-to-use extension manager. More than a good reason to have a look at it. In this post, I’m going to show how to get started.

If you don’t know OpenHAB yet, here’s a short summary: OpenHAB is a vendor and technology agnostic open source automation software for smart homes. The software is developed in Java, has an extensible OSGI architecture and an actively growing community. It comes with a responsive web interface, allowing for being used on desktops and mobile devices equally. Last but not least, OpenHAB features a catchy programming syntax for rules, triggers, scripts and notifications.

OpenHAB has an integrated HomeMatic binding. If you’re using a CCU2, you can start with OpenHAB right out of the box. If you’re using another I/O interface like the HM-CFG-LAN Configuration Tool, you’ll need [Homegear](https://www.homegear.eu/) as an additional piece of software. Homegear communicates with your HomeMatic devices through the I/O interface. OpenHAB then connects to Homegear, which allows you to control all your HomeMatic sensors and actors using the OpenHAB software.

To get started, you should first choose if you’re going with Docker Containers (my preferred way of running server applications) or if you want to install OpenHAB and Homegear directly on your Linux System.

## Option 1: Using Docker Compose
There are official [Docker Images for OpenHAB](https://hub.docker.com/r/openhab/openhab/). However, there was no working image for Homegear. So I created my own: You can use this [Docker Image for Homegear](https://hub.docker.com/r/virtualzone/homegear/) if you want to.

1. Make sure that Docker is set up correctly and that the Docker Daemon is running. Read [Docker’s official guide](https://docs.docker.com/) for your operating system if you’re unsure.
1. Make sure that [Docker Compose](https://docs.docker.com/compose/overview/) is installed. I’m using Docker Compose instead of manually scoring the two containers because it’s much more convenient.
1. Create a directory for your OpenHAB setup, such as:

```bash
mkdir -p /docker/containers/openhab
```

4. Create a docker-compose.yml file in this directory with the following content:

```yaml
version: '2'
services:
  openhab:
    image: openhab/openhab:amd64-online
    volumes:
      - "/etc/localtime:/etc/localtime:ro"
      - "/etc/timezone:/etc/timezone:ro"
      - "/docker/storage/openhab/conf:/openhab/conf"
      - "/docker/storage/openhab/userdata:/openhab/userdata"
    ports:
      - "8080:8080"
    depends_on:
      - homegear
    links:
      - homegear
  homegear:
    image: virtualzone/homegear
    volumes:
      - "/etc/localtime:/etc/localtime:ro"
      - "/etc/timezone:/etc/timezone:ro"
      - "/docker/storage/homegear/homematicbidcos.conf:/etc/homegear/families/homematicbidcos.conf"
      - "/docker/storage/homegear/sql.db:/var/lib/homegear/db.sql"
```

This defines two containers: One for OpenHAB and one for Homegear. The OpenHAB container depends on Homegear (“depends_on”), so Docker Compose makes sure that Homegear is started before OpenHAB. Check the paths of the volumes. They’re probably different on your system.

5. Start up this composition by executing this command from the directory created above:

```bash
docker-compose up -d
```

The -d flag means “detached”, which makes the two docker containers run in the background. Skip this option if you want to see what’s going on.

6. Check if everything is fine:

```bash
docker-compose logs
```

## Option 2: Docker without Compose
This option is similar to option 1. However, you’ll have to start the two Docker Containers separately and manually, making sure that Homegear if started before OpenHAB.

1. Make sure that Docker is set up correctly and that the Docker Daemon is running. Read [Docker’s official guide](https://docs.docker.com/) for your operating system if you’re unsure.
1. Launch Homegear with the following command. You may want to copy the command to an executable shell file, so it’s handier to re-execute it later:

```bash
docker run \
        --name homegear \
        -v /etc/localtime:/etc/localtime:ro \
        -v /etc/timezone:/etc/timezone:ro \
        -v /docker/storage/homegear/homematicbidcos.conf:/etc/homegear/families/homematicbidcos.conf \
        -v /docker/storage/homegear/sql.db:/var/lib/homegear/db.sql \
        -d \
        --restart=always \
        virtualzone/homegear
```

3. Launch OpenHAB with the following command:

```bash
docker run \
        --name openhab \
        -v /etc/localtime:/etc/localtime:ro \
        -v /etc/timezone:/etc/timezone:ro \
        -v /docker/storage/openhab/conf:/openhab/conf \
        -v /docker/storage/openhab/userdata:/openhab/userdata \
        -p 8080:8080 \
        --link homegear:homegear \
        -d \
        --restart=always \
        openhab/openhab:amd64-online
```

4. Check if both containers are running:

```bash
docker ps
docker logs homegear
docker exec homegear tail -n 100 /var/log/homegear/homegear.err
docker exec homegear tail -n 100 /var/log/homegear/homegear.log
docker logs openhab
```

## Option 3: Installation without Docker
If you’re not comfortable with Docker, please refer to the [download page of Homegear](https://www.homegear.eu/index.php/Downloads) and the [install guides for OpenHAB](https://www.openhab.org/docs/).

## Configuring Homegear
Please note that if you’re running FHEM, you’ll have to stop it first. You can’t make two applications connect to the same HomeMatic I/O device (such as the HM-CFG-LAN). As of version 0.6, the HomeMatic configuration of Homegear is not in /etc/homegear/physicalinterfaces.conf anymore. Instead it’s in: /etc/homegear/families/homematicbidcos.conf If you’re using Docker, you’ll have to edit the file in the corresponding path of your host system (such as /docker/storage/homegear/homematicbidcos.conf). My homematicbidcos.conf looks like this:

```ini
[HomeMaticBidCoS]
id = KEQ....
## Options: cul, cc1100, coc, cuno, hmcfglan, hmlgw
deviceType = hmcfglan
host = 192.168.xxx.xxx
port = 1000
# lanKey = xxxxxxx
rfKey = xxxx
currentRFKeyIndex = 1
responseDelay = 60
```

Some explanations:

* id: The ID printed on the back side of your BidCoS I/O device.
* deviceType: The device type of your BidCoS device (cul, cc1100, coc, cuno, hmcfglan, hmlgw).
* host: The IP address of your I/O interface.
* port: Usually 1000, you probably don’t need to change this.
* lanKey: The AES key used for the communication between Homegear and your I/O interface (for securing the LAN connection). If you’ve been using FHEM before, you’ve probably disabled AES encryption using HomeMatic’s configuration utility, as FHEM doesn’t support encryption. You should add AES encryption later. For a quick start, comment out this line.
* rfKey: A random key used for securing the connection between Homegear and the HomeMatic devices (sensors, actors, etc.). You should note it down somewhere, because if you lose it, you’ll have to re-pair all your devices.

After saving the configuration file, you’ll have to restart the Homegear daemon or the Docker Container running Homegear. Take a look at the logs in /var/log/homegear/homegear.log to find out if Homegear successfully connects to the BidCoS device.

## Connecting OpenHAB to Homegear
* Browse to OpenHAB’s web interface at port 8080 (such as http://localhost:8080).
* Select the Paper UI (this one is new in OpenHAB 2).
* Go to “Extensions” and install “HomeMatic Binding”.
* Go to “Configuration” -> “Things”. Two new things should be detected automatically: “Homegear” and “GATEWAY-EXTRAS”. Add both of them. They should be indicated as “ONLINE” afterwards.

## That’s it – for now...
Congratulations: You’ve mastered the essential steps of setting up OpenHAB for your HomeMatic based smart home! Next time, I’ll write about adding HomeMatic devices to OpenHAB using Homegear.