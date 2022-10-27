---
title: "OpenRC Script for 'podman kube play'"
date: 2022-10-26T15:00:00+00:00
tags:
    - linux
    - docker
author: "Heiner"
---

In June, I've [written about](/posts/alpine-podman/) my approach to starting and stopping Podman Pods using OpenRC scripts on Alpine Linux. However, that approach had two major drawbacks: First, the pods were started in the foreground, causing OpenRC to wait for all pod initialization tasks to complete. If an image needed to be pulled first, this could lead to longer delays, significantly increasing system startup times. Secondly, requesting the status of a previously started pod always stated "crashed". This is due to the fact that OpenRC is not able to identify the exact process spawned by Podman.

I've therefore improved my OpenRC startup script to be used with ```podman kube play``` YAML files. In this post, I'm presenting my results. If you have further improvements, please let me know.

## What does *not* work
The ```podman pod create``` command features the ```--infra-conmon-pidfile=file``` option. This option writes the PID of the infra container's conmon process to a file. 

Using this option, it was easy to enable OpenRC identifying the status of a Pod and start the Pod in background:

```ini
pidfile="/run/${RC_SVCNAME}.pid"
command_background=true
```

Unfortunately, the ```--infra-conmon-pidfile=file``` option is not (yet?) available when using the ```podman kube play``` command.

I've tried to discover the infra container's PID file using the ```podman inspect``` command and using this value dynamically in my OpenRC scripts:

```bash
podman inspect --format '{{ .PidFile }}' somecontainer-infra
```

However, OpenRC doesn't seem happy with PID files appearing and disapperaring dynamically.

## What *does* work

I've created a ```pod``` script which is sourced by multiple ```pod-*``` scripts.

The ```pod``` script includes functions for getting the status of a Pod and stopping a Pod. The script assumes that your Pod's Kubernetes YAML is located at ```/home/${command_user}/pods/${pod_name}/pod.yaml```.

**/home/your-user/pods/init.d/pod**
```bash
#!/sbin/openrc-run

name=$RC_SVCNAME
command="/usr/bin/podman"
networks_=''
for n in ${pod_networks}; do
	networks_="${networks_} --network $n";
done
command_args="play kube ${networks_} /home/${command_user}/pods/${pod_name}/pod.yaml >/dev/null 2>&1 &"

depend() {
	after network-online 
	use net 
}

cleanup() {
	/sbin/runuser -u ${command_user} -- ${command} pod exists ${pod_name}
	result=$?
	if [ $result -eq 0 ]; then
	        /sbin/runuser -u ${command_user} -- ${command} pod stop ${pod_name} > /dev/null
        	/sbin/runuser -u ${command_user} -- ${command} pod rm ${pod_name} > /dev/null
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

status() {
	/sbin/runuser -u ${command_user} -- ${command} pod exists ${pod_name} 2> /dev/null
	result=$?
	if [ $result -eq 0 ]; then
		einfo "status: started"
		return 0
	else
		einfo "status: stopped"
		return 3
	fi
}
```

The script for controlling a Pod "xyz" can look like this.

* ```command_user``` specifies the user running the Pod
* ```pod_name``` sets the Pod's name
* ```pod_networks``` sets a space-separated list of networks the Pod should be connected to

**/home/your-user/pods/init.d/pod-xyz**
```bash
#!/sbin/openrc-run

command_user="your-user"
pod_name=xyz
pod_networks='network1 network2 ...'

source "/home/${command_user}/pods/init.d/pod"
```

Using root (i.e. using ```doas``` or ```sudo```), you can then create a symlink in ```/etc/init.d``` and add the pod to the default run level at boot time:

```bash
cd /etc/init.d
ln -s /home/<user>/pods/pod-xyz
rc-update add pod-xyz
```

Use ```rc-service``` to start and stop your Pod:

```bash
doas rc-service pod-xyz start
```