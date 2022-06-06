---
title: "Analyze Traefik access log using InfluxDB and Grafana"
date: 2020-06-03T11:30:03+00:00
tags:
    - docker
author: "Heiner"
aliases:
    - /2020/06/traefik-access-log-influxdb-grafana-telegraf/
---

[Traefik](https://containo.us/traefik/) is a Cloud Native Edge Router, often deployed in Docker and Kubernetes environments. With little effort, you can use Telegraf to transport Traefik's access logs to an InfluxDB, where it can be analyzed using Grafana.

This setup contains the following elements:

* Traefik v2 runs as a Docker container on a Linux host.
* Traefik outputs access logs in JSON format to STDOUT.
* Telegraf fetched the Traefik container's JSON output using the docker_log input plugin.
* To work with the JSON output in InfluxDB and Grafana, we need to convert them using Telegraf's parser preprocessor plugin into distinct fields. Otherwise, only numeric fields are kept as metric values. String values are discarded by default.
* We're using Telegraf's output plugin "influxdb" to write them to InfluxDB.

## Configure Traefik
traefik.yml contains the following settings:

```yaml
accessLog:
  format: json
  fields:
    headers:
      defaultMode: drop
      names:
          User-Agent: keep
          Content-Type: keep
```

This makes Traefik output access logs in JSON format. JSON can easily be processed by machines – so we don't have to deal with GROK patterns or such workarounds. Furthermore, request headers get dropped, but the "User-Agent" und "Content-Type" are kept.

## Configure Telegraf
My telegraf.conf looks like this:

```ini
[[inputs.docker_log]]
    endpoint = "unix:///var/run/docker.sock"
    from_beginning = false
    container_name_include = ["traefik_traefik_1"]


[[processors.parser]]
    namepass = ["docker_log"]
    parse_fields = ["message"]
    merge = "override"
    data_format = "json"
    json_string_fields = [
        "ClientHost",
        "RequestAddr",
        "RequestCount",
        "RequestHost",
        "RequestMethod",
        "RequestPath",
        "RequestProtocol",
        "RequestScheme",
        "downstream_Content-Type",
        "request_User-Agent",
        "time"
    ]
    json_time_key = "time"
    json_time_format = "2006-01-02T15:04:05Z"
    json_timezone = "UTC"


[[outputs.influxdb]]
    urls = ["http://influxdb:8086"]
    database = "telegraf"
    username = "telegraf"
    password = "..."
```

Important settings are:

* container_name_include specifies from which container instance the logs are collected. It's our Traefik instance.
* parse_fields specifies which input field is to be processed. It's the field "message".
* json_string_fields specifies which values from the read JSON object are to be written to InfluxDB as string fields. If not specified, all non-numeric fields are dropped.
* json_time_key and the other json_time settings specify in which JSON keys and in which date-time format the timestamps for our log entries are contained.
* The output plugin needs to be configured so that Telegraf can connect to the InfluxDB.

This is just meant to be an example. Please mind applicable law when storing, processing and using the access logs – such as GDPR in the European Union.