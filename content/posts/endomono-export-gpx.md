---
title: "Export trainings from Endomondo as GPX files"
date: 2020-06-01T11:30:03+00:00
tags:
    - endonomdo
    - api
author: "Heiner"
aliases:
    - /2020/06/trainings-gpx-datei-endomondo-exportieren/
---

Seit etlichen Jahren zeichne ich meine Trainings mit Endomondo auf. Doch seit ca. einem Jahr gibt es mit Website und App immer mehr Probleme: Mal funktioniert das Einloggen nicht, mal werden die Trainings nicht synchronisiert. Bei mir war es Zeit für eine neue App – ich habe mich für Strava entschieden. Mit ein wenig Code oder meinem fertigem Programm kannst Du alle Deine Trainings als GPX aus Endomondo exportieren.

Auf der [Strava-Website gibt es einen Artikel](https://support.strava.com/hc/en-us/articles/216917747-Moving-your-activity-history-from-Endomondo-to-Strava) zur Frage, wie man von Endomondo zu Strava umzieht. Doch die Antwort ist erstmal nicht so toll: Man kann über die Endomondo-Website die Trainings jeweils einzeln als GXP-Datei exportieren.

Gut: GPX (GPS Exchange Format) ist ein Standard-Datei-Format zum Austausch von GPS-Koordinaten. Aus den Wegpunkten zusammen mit weiteren Metadaten (z.B. Datum, Sportart) kann jedes Deiner Trainings rekonstruiert werden.

Weniger gut: Ich habe mehr als 1.000 Trainings aus den letzten Jahren und wenig Motivation, mich einzeln durch diese hindurch zu klicken.

Im Modul-Repository für Node.JS, npmjs.com, gibt es jedoch das [Modul endomondo-api-handler](https://www.npmjs.com/package/endomondo-api-handler). Mit diesem ist mit wenig Aufwand das Suchen, Auswählen und Herunterladen von Trainings möglich:

```javascript
await api.processWorkouts(filter, async (workout) => {
  if (workout.hasGPSData()) {
    let filename = getFilename(workout);
    let gpx = await api.getWorkoutGpx(workout.getId());
    fs.writeFileSync(filename, gpx, 'utf8');
  }
});
```

Ich habe das Ganze mit ein bisschen “Drumherum” in ein [kleines Node.JS Programm gepackt](https://github.com/virtualzone/endomondo-exporter), das Du in meinem GitHub-Account findest. Mit diesem kannst Du ganz einfach Deine Trainings als GPX aus Endomondo exportieren:

```bash
./index.js --username=... --password=... --year=2019 --month=11 --dir=/home/john/trainings
```

Die Voraussetzung ist, dass [Node.JS](https://nodejs.org/) auf Deinem Computer installiert ist. Danach kannst Du mittels Git den Code aus meinem GitHub-Repository auschecken und den oben genannten Befehl zum Speichern Deiner Trainings ausführen:

```bash
git clone https://github.com/virtualzone/endomondo-exporter.git
cd endomondo-exporter
npm install
```

Der Import der GPX-Dateien bei Strava ist relativ einfach, da man bis zu 25 Dateien gleichzeitig hochladen kann. Scheinbar gibt es aber ein Rate-Limit – nach einigen Imports wurde mit einem Server Fehler geantwortet. Nach kurzer Wartezeit ging es dann aber jeweils wieder.

![](/img/strava-import.png)