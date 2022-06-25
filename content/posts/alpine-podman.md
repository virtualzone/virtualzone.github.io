---
title: "Setting up Alpine Linux with Podman"
date: 2022-06-25T18:00:00+00:00
tags:
    - linux
    - docker
author: "Heiner"
---

Recently, I've written a blog post on [how to set up Rootless Docker on Alpine Linux](/posts/alpine-docker-rootless/). Today I'm showing you how to set up Podman. Podman has a rootless architecture built in. It's an alternative to Docker, providing an almost identical command line interface. Thus, if you're used to Docker CLI, you won't have any issues working with Podman.

Podman was initially developed by RedHat and is available as an open source project. You can run your well known Docker images from Docker Hub and other registries without any changes. This is due to the fact that both Docker and Podman are compatible with Open Container Initiative (OCI) images.

In my tests, Podman had a signicantly smaller memory footprint. From my point of view, it seems perfectly suitable for low power machines. However, it comes without a daemon, so you'll have to set up some init scripts in order to restart your containers when your system reboots. I'll cover this at the end of this article.

## Download and install Alpine
First, we'll download the Alpine Linux ISO image and install the OS. We'll then enable the community repository as it contains packages we'll need to set up Docker in non-root mode.

1. Get Alpine Linux ISO from: https://www.alpinelinux.org/downloads/
1. Boot system from ISO and run:
    ```
    # setup-alpine
    ```
1. Reboot and install the nano edit:
    ```
    # apk add nano
    ```
1. Enable community repository in the following file:
    ```
    # nano /etc/apk/repositories
    ```
1. Update the index of available package:
    ```
    # apk update
    ```

## Add a user and allow her to use doas
If you did not create a regular user account during the installation, it's time to do it now:

1. Install doas:
    ```
    # apk add doas
    ```
1. Create user and add it to the `wheel` group in order to use root privileges:
    ```
    # adduser <USER> wheel
    ```
1. Allow users in group `wheel` to use doas by editing the file `/etc/doas.d/doas.conf` and adding the following line:
    ```
    permit persist :wheel
    ```
1. Log out and log in to the new account.

## Install Podman
Now comes the important part: Setting up Podman.

1. Enable cgroups v2 by editing `/etc/rc.conf` and setting `rc_cgroup_mode` to `unified`.
1. Enable the cgroups service:
    ```
    # rc-update add cgroups && rc-service cgroups start
    ````
1. Install podman:
    ```
    # apk add podman
    ```
1. Allow your user to access Podman in rootless mode:
    ```
    # modprobe tun
    # echo tun >>/etc/modules
    # echo <USER>:100000:65536 >/etc/subuid
    # echo <USER>:100000:65536 >/etc/subgid
    ```
1. Enable the iptables module:
    ```
    # echo "ip_tables" >> /etc/modules
    # modprobe ip_tables
    ```
1. Check if Podman works by running a Hello World container using your user account:
    ```
    $ podman run --rm hello-world
    ```

## Allow ports < 1024 (optional)
By default, only ports >= 1024 can be exposed by non-root users. To change this, change the minimum unprivileged port in `/etc/sysctl.conf`:
```
$ sudo echo "net.ipv4.ip_unprivileged_port_start=80" >> /etc/sysctl.conf
```

## Using Podman and Pods
If you are used to Docker, you can use Podman just the way to used to control Docker. One difference is that Podman can group multiple containers into Pods (that's where the name comes from: Pod Manager). You may know Pods from Kubernetes. Containers in a Pod share a namespace, a network and a security context.

* List running containers:
    ```
    podman ps
    ```
* List existing pods:
    ```
    podman pod ps
    ```
* Create a new pod:
    ```
    podman pod create pod-web
    ```
* Create a container inside the previously created Pod:
    ```
    podman run --rm -d \
        --pod pod-web \
        docker.io/library/nginx:alpine
    ```

## Starting containers on system start
Because Podman follows a daemonless concept, containers are not started along with the non-existing Daemon on system boot. Instead, Podman recommends using systemd to start, stop and restart containers when the system starts.

On Alpine, we're using OpenRC instead of systemd by default. I'm using Podman's built-in functionity for exporting and importing Kubernetes YAML definitions together with a small OpenRC init script.

1. Install runuser so your init script can create Pods in the name of your rootless user:
    ```
    # apk add runuser
    ```
1. Create a folder to store your init scripts, such as `/home/<user>/pods/init.d/`.
1. Generate a Kubernetes YAML for an existing Pod by issuing the following command and saving the YAML file in your previously created directory:
    ```
    podman generate kube <pod-name>
    ```
    Alternatively, you can write the YAML file manually. Please refer to [Podman's documention](https://docs.podman.io/en/latest/markdown/podman-generate-kube.1.html) for more information on supported (and unsupported) Kubernetes YAML syntax.
1. Create a file named `pod` in this folder with the following contents and make it executable (`chmod +x pod`):
    ```bash
    #!/sbin/openrc-run

    depend() {
        after network-online 
        use net 
    }

    cleanup() {
        /sbin/runuser -u ${command_user} ${command} pod exists ${pod_name}
        result=$?
        if [ $result -eq 0 ]; then
                /sbin/runuser -u ${command_user} ${command} pod stop ${pod_name}
                /sbin/runuser -u ${command_user} ${command} pod rm ${pod_name}
        fi
    }

    start_pre() {
        cleanup
    }

    stop() {
        ebegin "Stopping $RC_SVCNAME"
        cleanup
        eend $?
    }
    ```
1. Create one init script per Pod you want to control with the following contents (adjust as needed). Name it appropriately and make it executable (i.e. `chmod +x pod-traefik`):
    ```bash
    #!/sbin/openrc-run

    name=$RC_SVCNAME
    pod_name=traefik
    command_user="<user>"
    command="/usr/bin/podman"
    command_args="play kube --network traefik /home/${command_user}/pods/${pod_name}/pod.yaml"

    source "/home/${command_user}/pods/init.d/pod"
    ```
1. Create a symlink in `/etc/init.d/`:
    ```
    # cd /etc/init.d && ln -s /home/<user>/pods/pod-traefik
    ```
1. Use rc-update to the add your OpenRC Pod init script to the default runlevel:
    ```
    # rc-update add pod-traefik
    ```