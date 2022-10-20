---
title: "Connecting multiple networks to a Podman container"
date: 2022-10-16T17:00:00+00:00
tags:
    - linux
    - docker
author: "Heiner"
---

I'm running my containers with [Podman in Rootless Mode](/posts/alpine-podman/) on Alpine for about four months now. However, an annoying problem has haunted me ever since:

When a container was connected to more than one network, outgoing connections were not working correctly.

Consider a container connected to two bridge networks:

```
$ podman run --rm -it \
      --network net1 \
      --network net2 \
      alpine /bin/ash
```

Inside the container, the two networks are connected correctly:

```
# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth1@if17: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP qlen 1000
    inet 10.89.0.7/24 brd 10.89.0.255 scope global eth1
4: eth0@if18: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP qlen 1000
    inet 10.89.2.6/24 brd 10.89.2.255 scope global eth0
```

However, pinging a host on the internet only works using one of the two network interfaces:

```
# ping -I eth0 8.8.8.8
PING 8.8.8.8 (8.8.8.8): 56 data bytes
64 bytes from 8.8.8.8: seq=0 ttl=42 time=4.075 ms
```

```
# ping -I eth1 8.8.8.8
PING 8.8.8.8 (8.8.8.8): 56 data bytes
...
2 packets transmitted, 0 packets received, 100% packet loss
```

## The solution
The solution is quite simple: You will need to set ```net.ipv4.conf.all.rp_filter``` to 2.

On my Alpine system, rp_filter was set to 1 by default. The setting controls the source path validation within the kernel's IPv4 network stack. 1 means "strict", whereas 2 means "loose".

You can try the solution temporarily by running:

```
# sysctl -w net.ipv4.conf.all.rp_filter=2
```

To survive the next reboot, persist the setting by adding it to ```/etc/sysctl.conf```:

```
# echo "net.ipv4.conf.all.rp_filter=2" >> /etc/sysctl.conf
```

For more information, you can take a look at [this article](https://sysctl-explorer.net/net/ipv4/rp_filter/).