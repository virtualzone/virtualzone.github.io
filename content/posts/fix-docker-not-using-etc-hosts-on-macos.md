---
title: "Fix Docker not using /etc/hosts on MacOS"
date: 2016-08-28T11:30:03+00:00
tags:
    - macos
    - docker
author: "Heiner"
aliases:
    - /2016/08/fix-docker-not-using-etc-hosts-on-macos/
---

On my MacBook with Mac OS X 10.11 (El Capitan) and Docker 1.12.0, Docker did not read manually set DNS entries from the /etc/hosts file.

When I executed “docker push” for example, this resulted in “no such hosts” errors:

```log
Put http://shuttle:5000/v1/repositories/webfrontend/: dial tcp: lookup shuttle on 192.168.65.1:53: no such host
```

On Mac OS, Docker is running in a host container itself. Thus, you’ll have to add DNS entries to the container’s /etc/hosts file. To fix it, get into the running Docker Host:

```bash
screen ~/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux/tty
```

This took a while on my machine, I needed to press Ctrl+C for the login prompt to show up. Log in with “root” (no password required).

Edit the /etc/hosts file in the Docker Host using vi:

```bash
vi /etc/hosts
```

Note: Insert after pressing “i”, save by pressing Escape and then type “:wq” <Return>.

Restart the Docker Daemon with:

```bash
service docker restart
```

Detach from the screen session by pressing Ctrl+A, then press D.

Docker should now use the correct /etc/hosts entries.