---
title: "Setting up a Kubernetes cluster with K3S, GlusterFS and Load Balancing"
date: 2021-09-03T11:30:03+00:00
tags:
    - kubernetes
author: "Heiner"
aliases:
    - /2021/09/setting-up-a-kubernetes-cluster-with-k3s-glusterfs-and-load-balancing/
---

I’ve recently written a tutorial which will guide you through setting up a Kubernetes cluster using K3S with virtual machines hosted at [Hetzner](https://www.hetzner.com/cloud), a German (Cloud) hosting provider. The tutorial uses [K3S](https://k3s.io/), a lightweight Kubernetes distribution which is perfectly suited for small VMs like Hetzner’s CX11. Additionally, the tutorial will show you how to set up Hetzner’s cloud load balancer which performs SSL offloading and forwards traffic to your Kubernetes system. Optionally, you will learn how to set up a distributed, replicated file system using [Kadalu](https://kadalu.io/), an opinionated storage system based on [GlusterFS](https://www.gluster.org/). This allows you to move pods between the nodes while still having access to the pods’ persistent data.

[Read the tutorial in Hetzner’s Online Community.](https://community.hetzner.com/tutorials/k3s-glusterfs-loadbalancer)