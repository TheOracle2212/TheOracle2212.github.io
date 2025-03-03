---
layout: post
title: "A Vacuum fluorescent display clock #01"
description: "My desk is getting worse"
date: 2024-12-02
feature_image: images/apple-watch.jpg
tags: [vfd, clock]
---

## What happened?
I was given a Vacuum fluorescent display (VFD) for the primary purpose of making a clock. To do so, I decided to add a GPS for accurate time (and date) and better still, latitude and longitude. With these, I could technically turn the clock on anywhere in the world, give it a couple of seconds and let it show me the time accurate to the timezone. Just like that! Better still, with a built in battery, it should run for a couple of days, providing Real Time Clock (RTC) level timekeeping, waiting to recieve accurate time again. However, the one Waveshare NEO M6N I had on hand didn't have a battery. It'll still work fine, but powering the GPS everytime would be a 'cold start', taking time to identify and establish a connection with several satellites. Moreover, there'd be no RTC time output from the clock. 
<!--more-->

(Keyboard update coming soon. Appling for NS at the moment and learning Rust - already out!!)
## Dealing with the GPS
**put pictures of both GPS modules here**

As it turns out, a drone gps, such as generic one I had, had a built in compass (?) and a newer version of the GPS module in it - a Neo M7. The on-board antenna looked slightly better. And to top it all off, it had a battery! Using the included wire, I hacked to it a series of jumper wires and put my ear up on the Serial monitor. It still output the same NMEA sentences, which was great. To parse those sentences and get the relevant infomation, I used Mikal Hart's [TinyGPS+] (https://github.com/mikalhart/TinyGPSPlus) library. Quite simply, you've got to send over comes over serial to the library through the `encode()` function. Then, call whatever you need, however you need. I did this quite simply: `gps.encode(ss.read());`. I could have written my own library or a handful of functions to parse whatever I needed, but that seemed like a waste of perfectly good development time.

I read [this] (https://www.sparkfun.com/datasheets/GPS/NMEA%20Reference%20Manual-Rev2.1-Dec07.pdf) to get an idea of the NMEA sentences and what they did in particular (waste of perfectly good development time?).

Once I got a simple clock piping the values from the GPS, adjusting it by a timezone offset, and Serial printing it to the VFD, I decided to add a series of features to improve its usability. 

**put pictures of basic timekeeping**

## Automatic dimming
I plan to put this on my bedside table, and I really didn't want the bright screen disturbing my sleep, so I wanted it to dim automatically. There were two ways of going about this, adjusting the brightness based on the time of day, but my days aren't rigid and in place. So I used a ambient light sensor and read its analog value every 300 loops or so:
```
{
Serial.write(0x1B);
  if (countcycle == 300) {
    if (analogRead(A0) < 500) {
      brightness = 0xF9;
    } else if (analogRead(A0) < 800) {
      brightness = 0xFC;
    } else {
      brightness = 0xFF;
    }
    countcycle = 0;
  }
  Serial.write(brightness);
  countcycle += 1;
}
``` 
Interstingly, this takes a couple of minutes. This gives me a decent refresh time for the dimming, but doesn't slow anything down too much.
From here, I'm just waiting for a better microcontroller to arrive (yes!!) to use. The full program (out in my next post) is like already ~80% of my progam memory.
