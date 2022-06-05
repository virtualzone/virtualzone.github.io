---
title: "How to set up HTTPS/SSL in WordPress behind Proxy (nginx, HAProxy, Apache, lighttpd)"
date: 2016-08-27T11:30:03+00:00
tags:
    - wordpress
    - proxy
author: "Heiner"
aliases:
    - /2016/08/how-to-set-up-https-ssl-in-wordpress-behind-proxy-nginx-haproxy-apache-lighttpd/
---

Today I changed the accessibility of my blog from HTTP (unencrypted) to HTTPS/SSL. My blog is running WordPress behind an nginx proxy server. However, while the pages themselves loaded successfully from HTTPS, the embedded static resources like JavaScripts, Images, CSS files etc. did not. Here’s how I fixed it.

The cause of this issue is that WordPress doesn’t seem to detect the original protocol scheme (HTTPS) correctly when running behind a proxy. Thus, if the connection between your user’s browser and your proxy/loadbalancer is HTTPS, but the connection between your proxy server and WordPress is HTTP only, WordPress thinks that it’s running on HTTP instead of HTTPS. Therefore it places sets the absolute URLs incorrectly to HTTP.

This results in mixed content warnings. Modern browsers prevent loading resources from HTTP when the embedding page had been loaded from HTTPS. To fix this, taking the following steps worked for me:

Make sure that your proxy or load balancer adds the “X-Forwarded-*” HTTP request headers when proxying incoming requests to your WordPress backend server. My nginx configuration contains these lines:

```
proxy_set_header X-Forwarded-Host $host;
proxy_set_header X-Forwarded-Server $host;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; proxy_set_header X-Forwarded-Proto $scheme;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header Host $host;
```

* Install and activate the [SSL Insecure Content Fixer](https://de.wordpress.org/plugins/ssl-insecure-content-fixer/) plugin in your WordPress installation’s admin panel.
* Navigate to Settings -> SSL Insecure Content.
* Set “HTTPS detection” to “HTTP_X_FORWARDED_PROTO (e.g. load balancer, reverse proxy, NginX)”.
* Navigate to Settings -> General.
* Set the “WordPress Address (URL)” and “Site Address (URL)” to your new HTTPS address.
* Check if everything is working as expected.