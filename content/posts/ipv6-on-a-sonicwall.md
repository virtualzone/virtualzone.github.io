---
title: "How to enable IPv6 on a SonicWall (SonicOS 5.9) using NAT"
date: 2014-11-20T11:30:03+00:00
tags:
    - sonicwall
    - firewall
    - ipv6
author: "Heiner"
aliases:
    - /2014/11/how-to-enable-ipv6-on-a-sonicwall-sonicos-5-9-using-nat/
---

IPv6 aimed to make Network Address Translation (NAT) obselete as there are so many addresses available that every single device can have its own worldwide unique IPv6 address. However, even with IPv6, using NAT is a very simple way to get your devices behind a Dell SonicWall connected to IPv6 services on the internet. In contrast to going without NAT, all the devices behind your SonicWall will emerge under the SonicWall’s IPv6 address.

The following guide applies to Dell SonicWalls with SonicOS 5.9.0 (IPv6 is not supported in SonicOS 5.8 or below). A SonicWall TZ-215 is connected to an IPv6 capable router via the X1/WAN interface. There are devices connected to the SonicWall on the X0/LAN and W0/WLAN interfaces. There is also a virtual W0:V1 interface used for WLAN guests.

1. Log in to SonicWall’s administrative web interface (the default IP address on LAN is https://192.168.168.168).

2. Go to Network -> Interfaces and select to view IPv6.

* Determine SonicWall’s autonomous IPv6 address for the X1/WAN interface and note it down. You’ll need it later.
* Configure your X0/LAN interface: Check if it has a static IPv6 address starting with fd80::. Check “Enable Router Advertisement” and add a prefix fd80::, Lifetime = 1440 min.
* Configure your W0/WLAN interface: Check if it has a static IPv6 address starting with fd81::. Check “Enable Router Advertisement” and add a prefix fd81::, Lifetime = 1440 min.
* Do the same with other interfaces you want to enable for IPv6, such as W0:V1, X2, etc. Use fd82::, fd83::, etc. as prefixes.

3. Go to Network -> Address Objects and select to view IPv6.
Create/update the entry “WAN Primary IPv6” with the previously determined X1 IPv6 address. Set Zone = WAN, Type = Host.

4. Go to Network -> NAT Policies and select to view IPv6.
* Create a new NAT policy with the following settings: Original Source = Any Translated Source = WAN Primary IPv6 Original Destination = Any Translated Destination = Original Original Service = Any Translated Service = Original Inbound Interface = X0/LAN Outbound Interface = X1/WAN
* Create another new NAT policy with the same settings as before, but this time, select W0/WLAN as “Inbound Interface”.

5. On a client connected to the SonicWall, go to http://test-ipv6.com to check if your IPv6 configuration works.