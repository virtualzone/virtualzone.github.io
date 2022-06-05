---
title: "Using Let’s Encrypt / EFF’s CertBot with NGINX in Docker"
date: 2017-02-11T11:30:03+00:00
tags:
    - docker
    - letsencrypt
    - nginx
author: "Heiner"
aliases:
    - /2017/02/using-lets-encrypt-effs-certbot-with-nginx-in-docker/
---

I’m using [NGINX](https://nginx.org/) in a [Docker](https://www.docker.com/) Container as a front-end HTTP(s) Webserver, performing SSL termination and proxying incoming requests to various other Docker Containers and VMs. Now that I’ve switched my certificates to [Let’s Encrypt](https://letsencrypt.org/), I wondered how to integrate [EFF’s CertBot](https://certbot.eff.org/) (which is recommended by Let’s Encrypt) with my setup. Here’s how I did it.

First, I’ve added two new volumes to my web-front-end’s Docker Compose File:

```yaml
version: '2'
services:
  webfrontend:
    container_name: webfrontend
    [...]
    volumes:
      - "/etc/localtime:/etc/localtime:ro"
      - "/etc/timezone:/etc/timezone:ro"
      - "/docker/storage/webfrontend/letsencrypt/www:/var/www/letsencrypt"
      - "/docker/storage/webfrontend/letsencrypt/etc:/etc/letsencrypt"
```

Next, I’ve added the following location block to each of my virtual hosts:

```
location /.well-known/ {
    alias /var/www/letsencrypt/;
}
```

I’m using the [palobo/certbot Docker Image](https://hub.docker.com/r/palobo/certbot/) to create the certificates, using this shell script:

```sh
#!/bin/sh

docker pull palobo/certbot

GetCert() {
        docker run -it \
                --rm \
                -v /docker/storage/webfrontend/letsencrypt/etc:/etc/letsencrypt \
                -v /docker/storage/webfrontend/letsencrypt/lib:/var/lib/letsencrypt \
                -v /docker/storage/webfrontend/letsencrypt/www:/var/www/.well-known \
                palobo/certbot -t certonly --webroot -w /var/www \
                --keep-until-expiring \
                $@
}

echo "Getting certificates..."
GetCert -d www.mydomain.com -d mydomain.com
GetCert -d somedomain.net

echo "Restarting Web Frontend..."
cd /docker/containers/webfrontend
docker-compose down
docker-compose up -d
cd -

echo "Done"
```

The script starts CertBot in a Docker Container for each requested certificate. Because the /etc/letsencrypt and the /var/www/.well-known directory is also used by my NGINX front-end Container (see above), these steps can be performed by the script:

1. Using the [webroot plugin](https://certbot.eff.org/docs/using.html#webroot), a random file is created under the /.well-known/acme-challenge/ directory.
1. Let’s Encrypt can access and verify this file as the folder is aliased using the Location blocks in the NGINX config.
1. The generated private key and public certificate is placed in /etc/letsencrypt/, which is in turn a volume for the NGINX web-frontend.

You can use the generated certificates by adding these two lines to your NGINX vhost config:

```
ssl_certificate     /etc/letsencrypt/live/www.mydomain.com/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/www.mydomain.com/privkey.pem;
```