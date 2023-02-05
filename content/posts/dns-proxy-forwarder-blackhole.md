---
title: "Go-hole: A minimalistic DNS proxy and and blocker"
date: 2023-02-05T06:00:00+00:00
tags:
    - linux
    - docker
author: "Heiner"
---

You'll probaby know Pi-hole. It's a popular "DNS sinkhole" – a DNS proxy server which blocks certain requests, such a as those for well-known ad serving domains. The effect is a much less ad-cluttered web experience in your home network.

I've been using Pi-hole for several years as a Docker container on a Raspberry Pi. The Raspi is serving as a small home server on my home network.

However, as much as I like Pi-hole, I felt it got loaded with new features over the years and performed slower over the time. DNS queries took longer and longer until they were answered. With this experience in mind and out of pure interest (how complicated would it be to create a DNS proxy on my own?) I've created [Go-hole](https://github.com/virtualzone/go-hole).

## What is Go-hole?
[Go-hole](https://github.com/virtualzone/go-hole) is written in Go and very minimalistic with an eye to the primary requirements. However, it has all the features I personally need on my home network:

* Act as a network-wide central DNS server, handling all DNS queries from all queries
* Forward incoming queries to one or more upstream DNS servers
* Cache upstream query results for extremely fast recurring lookup handling
* Block queries for well-known ad-serving and malicious domains by using definable block list URLs
* Regularly update the black list source files
* Whitelist certain domains which would be blocked in view of the set up black lists
* Resolve local names

## How does it work?
Go-hole serves as DNS server on your (home) network. Instead of having your clients sending DNS queries directly to the internet or to your router, they are resolved by your local Go-hole instance. Go-hole sends these queries to one or more upstream DNS servers and caches the upstream query results for maximum performance.

Incoming queries from your clients are checked against a list of unwanted domain names ("blacklist"), such as well-known ad serving domains and trackers. If a requested name matches a name on the blacklist, Go-hole responds with error code NXDOMAIN (non-existing domain). This leads to clients not being able to load ads and tracker codes. In case you want to access a blacklisted domain, you can easily add it to a whitelist.

As an additional feature, you can set a list of custom hostnames/domain names to be resolved to specific IP addresses. This is useful for accessing services on your local network by name instead of their IP addresses.

## How to use Go-hole?
The simplest way of getting Go-hole up and running is by using the [pre-built Docker images](https://github.com/virtualzone/go-hole/pkgs/container/go-hole).

First, create a configuration file named ```config.yaml```. You can take a list at the [example config file](https://github.com/virtualzone/go-hole/blob/main/config.yaml) in the GitHub repository. On my home network, my ```config.yaml``` looks like this:

```yaml
listen: 0.0.0.0:53
upstream:
  - 8.8.8.8:53
  - 8.8.4.4:53
blacklist:
  - https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts
blacklistRenewal: 1440
whitelist:
  - raw.githubusercontent.com
  - www.googletagservices.com
local:
  - name: ha
    target:
    - address: 192.168.40.31
      type: A
    - address: 2a01:170:1172:40:40::31
      type: AAAA
```

This config sets the following:

* ```listen``` sets the listing address to 0.0.0.0 (any address) and the listing port to 53 (default DNS).
* ```upstream``` sets the upstream DNS servers to Google's DNS.
* ```blacklist``` sets the black list source URL.
* ```blacklistRenewal``` sets the automatic blacklist updating to a 1 day interval (1440 minutes).
* ```whitelist``` whitelists two domains which would be blacklisted otherwise.
* ```local``` sets a IPv4 (A record) and IPv6 (AAAA record) for the local name "ha".

After you've prepared your configuration file, you can start the Docker container like this:

```bash
docker run \
    --rm \
    --mount type=bind,source=${PWD}/config.yaml,target=/app/config.yaml \
    -p 53:53/udp \
    ghcr.io/virtualzone/go-hole:latest
```

If you don't want to run Go-hole with Docker (or [Podman, like I do](/posts/alpine-podman/)), you can use the [pre-built binaries](https://github.com/virtualzone/go-hole/releases) or build Go-hole from source.

## Conclusion
I'm using Go-hole for several weeks now as my home network's DNS server. It has completely replaced Pi-hole for my use cases. I've not observed any crashes or instabilities yet. My home network's DNS resolving times have greatly improved, making web browsing much faster than it has been before. Of course, Pi-hole has a lot more features than Go-hole. My implementation doesn't feature a web interface and for sure lacks other things you might like. However, none of these features are relevant to me.

I'd be happy to hear about your experience with this Pi-hole alternative.