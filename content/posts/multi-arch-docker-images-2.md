---
title: "Build Multi-Arch images on Docker Hub (Part 2)"
date: 2020-05-16T11:30:03+00:00
tags:
    - docker
author: "Heiner"
aliases:
    - /2020/05/multi-arch-images-mit-docker-hub-bauen-teil-2/
---

Im [ersten Teil dieses Artikels](/posts/multi-arch-docker-images-1/) habe ich Euch gezeigt, wie Ihr ein Multi-Arch-Docker-Projekt anlegt, das auf einer AMD64-Plattform auch für andere Zielarchitekturen wie bspw. ARM bauen kann. In diesem Teil zeige ich Euch, wie Ihr das Ganze im offiziellen Docker Hub zum Laufen bekommt.

Zunächst solltet Ihr ein Projekt im Docker Hub anlegen und dieses mit Eurem Quellcode-Repository verknüpfen. In meinem Fall nutze ich GitHub als Sourcecode-Repository und nutze die Build-Infrastruktur von Docker Hub. Die entsprechenden Einstellungen findet Ihr im Reiter “Builds”:

![](/img/multiarch-dockerhub-1.png)

Einen automatisierten Build im Docker Hub konfigurieren.
Dort könnt Ihr dann die Build-Konfiguration vornehmen. Zunächst muss angegeben werden, aus Source-Repository gebaut werden soll:

![](/img/multiarch-dockerhub-2.png)

Bei der Konfiguration muss zunächst das Sourcecode-Repository angegeben werden.
Anschließend legt Ihr fünf Build Rules an, nämlich eine ohne Angabe eines Architektur-Tags (in meinem Fall “latest”) und vier weitere je Zielarchitektur. Vier deshalb, weil wir in diesem Beispiel für AMD64, ARM32V6, ARM32V7 und ARM64V8 bauen. Solltet Ihr für andere Zielarchitekturen bauen wollen, benötigt Ihr natürlich mehr oder weniger Build Rules:

![](/img/multiarch-dockerhub-3.png)

Die passenden Build Rules für die vier Zielarchitekturen.
Der Trick ist, dass das “ungetaggte” Image alle anderen Architektur-Images zugeordnet bekommt. Dadurch kann ein Anwender, der “docker run” oder “docker pull” auf Euer Image durchführt, das für seine Architektur passende Image automatisch laden, ohne explizit die Plattform nennen zu müssen. Ein Mac zieht somit das AMD64-Image, während ein Raspbian das ARM32V7-Image lädt und ein Raspberry Pi 4 mit 64bit-Ubuntu das ARM64V8 Image. Alles ohne weiteres zutun.

Das war es dann auch schon mit der Konfiguration. Ein Klick auf “Save and Build” stellt die ausstehenden Builds (hier fünf an der Zahl) in die Warteschlange. Meiner Erfahrung nach kann es auf der Docker Hub Infrastruktur auch für einfache Images durchaus ein paar Stunden dauern, bis alle Images gebaut wurden. Was schon erledigt ist und was noch aussteht, könnt Ihr unter “Recent Builds” verfolgen.

![](/img/multiarch-dockerhub-4.png)

Die Recent Builds geben Auskunft über die noch ausstehenden und schon erfolgten Automated Builds.
Ihr werdet sehen, dass die ersten Builds als fehlgeschlagen markiert werden. Das ist völlig normal! Ein Blick in die Build Logs zeigt den nachvollziehbaren Grund: Nach jedem Build wird das multi-arch-manifest.yaml Docker-Manifest angewandt. Bevor das letzte Ziel-Architektur.Image aber nicht fertig gebaut wurde, können nicht alle Architektur-Images dem “ungetaggten” Image hinzugefügt werden und das Build schlägt augenscheinlich fehl.

![](/img/multiarch-dockerhub-5.png)

Kein Grund zur Sorge: Der Fehler “failed with error: manifest unknown: manifest unknown”.
Tatsächlich wurde das jeweilige Image aber (hoffentlich) erfolgreich gebaut und gepusht. Erst beim letzten Multi-Arch-Image kann das Manifest-Tool dann auch erfolgreich seine Arbeit verrichten und die Architekturen verknüpfen. Lasst Euch davon also nicht aus der Ruhe bringen und beobachtet die Build Logs aufmerksam.

Ich wünsche Euch viel Spaß mit den Multi-Arch-Images im Docker Hub!