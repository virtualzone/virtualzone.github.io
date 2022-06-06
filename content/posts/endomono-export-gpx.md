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

I've been using Endomondo for years to track my trainings. However, I've been experiencing a lot of issues with Endomondo over the last months: Sometimes it's not possible to log in. Other times, my trainings won't get synced. So it's time a new app. I've decided to give Strava a try. With a few lines of code, I've managed to export all my training data as GPX files. These can be imported to Strava, so my training history won't get lost.

There's an [article on Strava's website](https://support.strava.com/hc/en-us/articles/216917747-Moving-your-activity-history-from-Endomondo-to-Strava) on how to move from Endomondo to Strava. But the answer is a bit too easy: Using Endomondo's website, you can only export a single training at a time in GPX file format.

The good: GPX (GPS Exchange Format) is an standard file format used to exchange GPS coordinates. Using the GPS waypoints and some meta data (i.e. date, type of training), each of your trainings is reconstructable.

The bad: I've done more than 1,000 trainings in Endonomdo and I'm not willing to export each of them one by one.

In Node.JS' module respository, npmjs.com, there's a [module named endomondo-api-handler](https://www.npmjs.com/package/endomondo-api-handler). Using this, it's easy to search, select and download trainings from Endomondo's servers:

```javascript
await api.processWorkouts(filter, async (workout) => {
  if (workout.hasGPSData()) {
    let filename = getFilename(workout);
    let gpx = await api.getWorkoutGpx(workout.getId());
    fs.writeFileSync(filename, gpx, 'utf8');
  }
});
```

I've used this module to create a [little Node.JS tool](https://github.com/virtualzone/endomondo-exporter) which can be found on my GitHub account. You can use it to export *all* of a year's trainings from Endonomdo:

```bash
./index.js --username=... --password=... --year=2019 --month=11 --dir=/home/john/trainings
```

In order to use this tool, [Node.JS](https://nodejs.org/) must be installed on your computer. You can then check out the tool's source code from my GitHub repository and run the following commands to make the tool ready to run:

```bash
git clone https://github.com/virtualzone/endomondo-exporter.git
cd endomondo-exporter
npm install
```

Importing GPX files to Strava is quite easy: You can upload 25 training files at once. There seems to be some rate limiting. I've received server errors after several imports. Waiting a few minutes solved that.

![](/img/strava-import.png)