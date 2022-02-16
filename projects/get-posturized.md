---
layout: page
title: Get Posturized
description: >
  How you install Hydejack depends on whether you start a new site,
  or change the theme of an existing site.
hide_description: true
sitemap: false
permalink: /get-posturized
---

**Get Posturized** is a prototype device that helps correct unhealthy posture and alleviates their negative effects. It uses a gyroscope to detect the optimal position relative to the body posture. The overall device was created using an Arduino Uno board and an MPU 6050 device.

|                                              |                                               |
|:--------------------------------------------:|:---------------------------------------------:|
| ![alt-text-1](/projects/assets/img/test.jpg) | ![alt-text-2](/projects/assets/img/test2.JPG) |

<!---
A caption for an image.
{:.figcaption}
-->

## How to Use
 * Using a t-shirt with pockets, preferably, put the Arduino side of the device into the pocket.
 * Tape the other end of the device over the shoulder onto the back of your shoulder blades.
 * Open up the Android App and connect your phone to the Bluetooth module on the device.
 * Stand straight and start caliberating your ideal posture on the app.
 * Afterwards, the device will vibrate whenever you lean to far forwards or backwards from the ideal posture and the app will notify you of the incorrect posture.
 
This project was created for the IEEE Winter Quarterly Project 2018 which is a competition that IEEE UCSD holds every quarter of the school year. Since the theme of this competition was "Health", we opted for creating a device that would help assist users in correcting back posture as the amount of people suffering from back pain due to working white-collar jobs or using mobile phones is rapidly increasing. Some of the challenges we faced included lack of experience with Arduinos and the lack of documentation on the MPU 6050 device that we were using. Another challenge included time constraints, having to balance working on this competition with working/studying for school.

## Filtering MPU (Motion Processing Unit) Data
Originally, we wanted to use a moving average filter (simple FIR filter) which operates by averaging a number of points from the input to produce a point in the output. This however, required a great deal of memory to store a few points in an array and took alot of time to calculate just one output point, so we went with a simple averaging filter that does the same thing as the moving average filter, but each input point to average is only used once.

Averaging filter seudocode:
~~~js
void loop()
{
  for loop for (number of points to average) {
    average += measurement()
    delay(1)
  }
  average = average / (number of points to average)
}
~~~

Moving Average filter seudocode:
~~~js
void loop()
{
  value = measurement()
  Store value in array
  Increment array counter
  if array counter > max, set counter back to 0
  for loop for (number of points to average) {
    running average += array[i]
  }
  running average = running average / (number of points to average)
  delay(100);
}
~~~

## Tools Used  
 * Arduino Uno
 * MPU 6050
 * Mini vibration motor disc
 * HM-10 Bluetooth module
 * C++
 * Soldering
 * Circuit building
 * Android app creating software