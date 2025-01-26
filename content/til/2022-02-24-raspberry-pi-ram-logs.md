---
title: 'Reducing Raspberry Pi SD card writes'
tags: [til]

comments: false
---

Recently my Raspberry Pi's SD Card stopped working, most likely due to the high volume of read/write operations on it. I run a `pi-hole` which is constantly running in the background, and generates a lot of logs. Since these are written to disk, the integrity of the SD Card is known to wear down significatly.

I was running Raspbian on it, which does not optimize any I/O (that I'm aware of). I did a little research to see what can be done and found a really cool alternative: [DietPi](https://dietpi.com).

The unique thing about DietPi which applies here is that it can [hold all logs in RAM](https://dietpi.com/docs/software/log_system/#dietpi-ramlog), and then sync them once an hour to the SD Card.

So far it seems to work well, and in worst-case scenario I might lose an hour of logs if the Pi restarts or something.
