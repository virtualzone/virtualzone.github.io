---
title: "Setting up Alpine Linux with Rootless Docker"
date: 2022-06-19T15:00:00+00:00
tags:
    - linux
    - docker
author: "Heiner"
---

As of Docker Engine v20.10, it's possible to run the Docker daemon as a non-root user (Rooless mode). This is especially valuable in view of security aspects. Rootless mode mitigates potential vulnerabilities in the Docker daemon.

However, at the time of writing, setting up Docker in rootless mode is not straightforward if you're using Alpine Linux as your host system. This is why I summarized the steps to get Docket Rootless up and running on Alpine Linux.

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

## Install Docker Rootless
1. Install `newuidmap`, `newgidmap`, `fuse-overlayfs` and `iproute2` tools, all required by Rootless Docker:
    ```
    # apk add shadow-uidmap fuse-overlayfs iproute2
    ````
1. Enable cgroups v2 by editing `/etc/rc.conf` and setting `rc_cgroup_mode` to `unified`.
1. Enable the cgroups service:
    ```
    # rc-update add cgroups && rc-service cgroups start
    ````
1. Allow your user to access Podman in rootless mode:
    ```
    # modprobe tun
    # echo tun >>/etc/modules
    # echo <USER>:100000:65536 >/etc/subuid
    # echo <USER>:100000:65536 >/etc/subgid
    ```
1. Install Docker and Docker Compose v2:
    ```
    # apk add docker docker-cli-compose
    ```
1. Allow Docker access for your user:
    ```
    # addgroup <USER> docker
    ```
1. Enable the iptables module:
    ```
    # echo "ip_tables" >> /etc/modules
    # modprobe ip_tables
    ```
1. Install Docker rootless:
    ```
    $ curl -fsSL https://get.docker.com/rootless | sh
    ```
1. Create an init script in `/etc/init.d/docker-rootless`:
    ```
    #!/sbin/openrc-run

    name=$RC_SVCNAME
    description="Docker Application Container Engine (Rootless)"
    supervisor="supervise-daemon"
    command="/home/<USER>/bin/dockerd-rootless.sh"
    command_args=""
    command_user="<USER>"
    supervise_daemon_args=" -e PATH=\"/home/<USER>/bin:/sbin:/usr/sbin:$PATH\" -e HOME=\"/home/<USER>\" -e XDG_RUNTIME_DIR=\"/home/<USER>/.docker/run\""

    reload() {
        ebegin "Reloading $RC_SVCNAME"
        /bin/kill -s HUP \$MAINPID
        eend $?
    }

    ```
1. Make the created init script executable, add it to the default runlevel and start it:
    ```
    # chmod +x /etc/init.d/docker-rootless
    # rc-update add docker-rootless
    # rc-service docker-rootless start
    ````
1. Create a `.profile` file in your home directory with the following contents:
    ```
    export XDG_RUNTIME_DIR="$HOME/.docker/run"
    export DOCKER_HOST=unix://$XDG_RUNTIME_DIR/docker.sock
    export PATH="/home/<USER>/bin:/sbin:/usr/sbin:$PATH"
    ```
1. Log out and log in again.
1. Check if Docker Rootless works:
    ```
    $ docker ps
    $ docker run --rm hello-world
    ```