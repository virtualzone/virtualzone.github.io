---
title: "Build Multi-Arch images on Docker Hub (Part 1)"
date: 2020-05-15T11:30:03+00:00
tags:
    - docker
author: "Heiner"
aliases:
    - /2020/05/multi-arch-images-mit-docker-hub-bauen-teil-1/
---

Multi-Arch Docker Images sind eine tolle Sache: Benutzer Eurer Images ziehen automatisch die für Ihre Architektur passende Version Eures Image – ob AMD64, ARM64 oder ARM32. Normalerweise muss man Docker Images auf der Architektur bauen, auf der sie später auch verwendet werden. Durch die Verwendung des Emulators QEMU ist es jedoch möglich, auf einer AMD64-Architektur für alle anderen Zielplattformen mitzubauen. Kombiniert mit der Auto-Build-Funktion des Docker Hub ist das eine prima Arbeitserleichterung. Ich möchte Euch in diesem Beitrag zeigen, wie es geht.

Zunächst legt Ihr wie gewohnt ein Dockerfile für die AMD64-Architektur an – hier am Beispiel eines Alpine-Basis-Image:

```dockerfile
FROM amd64/alpine:3.11
...
```

Es folgt jeweils ein Dockerfile pro Zielarchitektur. In diesen wird zunächst die passende QEMU-Binary heruntergeladen und dann in das Ziel-Image hinein kopiert.

Dockerfile.arm32v6 für ARM32V6:

```dockerfile
FROM alpine:3.11 AS qemu
RUN apk --update add --no-cache curl
RUN cd /tmp && \
curl -L https://github.com/balena-io/qemu/releases/download/v3.0.0%2Bresin/qemu-3.0.0+resin-arm.tar.gz | tar zxvf - -C . && mv qemu-3.0.0+resin-arm/qemu-arm-static .

FROM arm32v6/alpine:3.11
COPY --from=qemu /tmp/qemu-arm-static /usr/bin/
...
```

Dockerfile.arm36v7 für ARM32V7:

```dockerfile
FROM alpine:3.11 AS qemu
RUN apk --update add --no-cache curl
RUN cd /tmp && \
curl -L https://github.com/balena-io/qemu/releases/download/v3.0.0%2Bresin/qemu-3.0.0+resin-arm.tar.gz | tar zxvf - -C . && mv qemu-3.0.0+resin-arm/qemu-arm-static .

FROM arm32v7/alpine:3.11
COPY --from=qemu /tmp/qemu-arm-static /usr/bin/
...
```

Dockerfile.arm64v8 für ARM64V8:

```dockerfile
FROM alpine:3.11 AS qemu
RUN apk --update add --no-cache curl
RUN cd /tmp && \
curl -L https://github.com/balena-io/qemu/releases/download/v3.0.0%2Bresin/qemu-3.0.0+resin-aarch64.tar.gz | tar zxvf - -C . && mv qemu-3.0.0+resin-aarch64/qemu-aarch64-static .

FROM arm64v8/alpine:3.11
COPY --from=qemu /tmp/qemu-aarch64-static /usr/bin/
...
```

Zusätzlich wird legt Ihr eine Datei Namens “multi-arch-manifest.yaml” an. In dieser wird angegeben, welches Image welcher Architektur zugeordnet wird. Die nach den obigem Schema mit QEMU gebauten Image sind nämlich zunächst als AMD64-Architektur gelistet, was natürlich nicht stimmt. Durch das Docker-Manifest kann das angepasst werden. Hier am Beispiel meines virtualzone/compose-updater Image, den Namen müsst Ihr natürlich anpassen:

```yaml
image: virtualzone/compose-updater:latest
manifests:
  - image: virtualzone/compose-updater:amd64
    platform:
      architecture: amd64
      os: linux
  - image: virtualzone/compose-updater:arm32v6
    platform:
      architecture: arm
      os: linux
      variant: v6
  - image: virtualzone/compose-updater:arm32v7
    platform:
      architecture: arm
      os: linux
      variant: v7
  - image: virtualzone/compose-updater:arm64v8
     platform:
       architecture: arm64
       os: linux
       variant: v8
```

Nun fehlen nun noch die Hooks. Diese werden von der Docker Registry vor bzw. nach den entsprechenden Build-Schritten aufgerufen. Wir benötigen Post-Push- und Pre-Build-Hook.

Der Pre-Build-Hook wird von der Registry vor dem Bauen eines Image aufgerufen. Hier müssen wir QEMU laden und ausführen. Der Dateiname muss “pre_build” lauten und chmod 755 haben:

```bash
#!/bin/bash

BUILD_ARCH=$(echo "${DOCKERFILE_PATH}" | cut -d '.' -f 2)

[ "${BUILD_ARCH}" == "Dockerfile" ] && \
{ echo 'qemu-user-static: Registration not required for current arch'; exit 0; }

docker run --rm --privileged multiarch/qemu-user-static:register --reset
```

Der Post-Push-Hook wird von der Registry aufgerufen, sobald ein Image fertig gebaut ist und ins Repository gepusht wurde. Hier muss das Manifest-Tool von Docker installiert und anschließend ausgeführt werden. Der Dateiname muss “post_push” lauten und chmod 755 haben:

```bash
#!/bin/bash
curl -Lo manifest-tool https://github.com/estesp/manifest-tool/releases/download/v1.0.0/manifest-tool-linux-amd64
chmod +x manifest-tool
./manifest-tool push from-spec multi-arch-manifest.yaml
```

Damit ist Euer Projekt vorbereitet und bereit für Multi-Arch-Builds.

Im [nächsten Teil zeige ich Euch](/posts/multi-arch-docker-images-2/), wie Ihr die “Automated Builds” im Docker Hub konfiguriert, um den Multi-Arch-Build auch tatsächlich durchzuführen.