---
title: "Analyze Traefik access log using InfluxDB and Grafana"
date: 2020-06-03T11:30:03+00:00
tags:
    - docker
author: "Heiner"
aliases:
    - /2020/06/traefik-access-log-influxdb-grafana-telegraf/
---

[Traefik](https://containo.us/traefik/) ist ein im Docker- und Kubernetes-Umfeld häufig eingesetzter Cloud Native Edge Router. Mit wenig Aufwand lassen sich die Zugriffslogs (Access Logs) von Traefik mittels Telegraf automatisch in eine InfluxDB überführen, um sie mit Hilfe von Grafana auszuwerten. In diesem Artikel zeige ich Dir, wie es geht.

In diesem Setup gibt es folgende wesentliche Elemente:

* Traefik v2 läuft als Docker Container auf einem Linux Host.
* Traefik schreibt die Accesslogs im JSON-Format nach STDOUT.
* Telegraf holt die JSON-Ausgaben vom Traefik-Container mit dem docker_log Input-Plugin ab.
* Um mit dem JSON-Output in der InfluxDB bzw. in Grafana etwas anfangen zu können, wandle ich sie mit dem Parser-Processor-Plugin von Telegraf in eigenständige Felder um. Das ist notwendig, da ansonsten nur die numerischen Werte als Metriken übernommen werden – die String-Werte werden standardmäßig verworfen.
* Mit dem Telegraf-Output-Plugin “influxdb” werden die Daten dann in die InfluxDB geschrieben.

## Traefik konfigurieren
Die traefik.yml beinhaltet dazu folgende Konfiguration:

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

Damit werden die Accesslogs im JSON-Format rausgeschrieben. JSON hat den Vorteil, dass es maschinell leichter weiterverarbeitet werden kann – wir müssen also nicht mit GROK-Patterns oder dergleichen hantieren. Außerdem werden die Request-Header zwar verworfen (“drop”), aber die Header “User-Agent” und “Content-Type” werden beibehalten.

## Telegraf konfigurieren
Die telegraf.conf habe ich folgendermaßen eingerichtet:

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

Wichtige Einstellungen sind hier:

* container_name_include gibt an, von welcher Container-Instanz die Logs eingesammelt werden sollen. In diesem Fall die Traefik-Instanz.
* parse_fields gibt an, welches Intput-Feld verarbeitet werden soll – in diesem Fall das Feld “message”.
* json_string_fields gibt an, welche Werte aus dem eingelesenen JSON-Objekt als String-Felder in die InfluxDB geschrieben werden sollen. Lässt man das weg, werden alle nicht-numerischen Felder verworfen.
* json_time_key und die anderen json_time-Einstellungen geben an, aus welchem JSON-Key und mit welchem Format die Logs den Zeitstempel für die Logs-Einträge beinhalten.
* Im Output-Plugin musst Du dann noch die Verbindung zur InfluxDB korrekt konfigurieren.

Das ganze soll als Beispiel dienen. Beachte bei der Speicherung, Auswertung und Verwendung der Access Logs bitte die geltenden Gesetze – in der EU bspw. die DSGVO und ggfls. weitere.