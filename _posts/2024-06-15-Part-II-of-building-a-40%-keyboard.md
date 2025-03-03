---
title: "Keyboard - Ortholinear 40% Part II"
layout: post
mathjax: false
categories: Ergonomics, Keyboard, Ortholinear, 40%, build, Part II, 2
---
Rather than a how-to guide (there are sooo many for interested parties), this is more of my experience in building my very unique keyboard.

## Parts
60 (5 rows of 12 columns) keyswitches and diodes
Microcontroller -  a Raspberry Pi Pico W I had on my desk while I was developing a drivier for the LIS3DH accelrometer.
A decent amount of 22 awg wire.

## The process.
Keyboards are built in a matrix format:
!(Matrix...)[https://trauring.org/wp-content/uploads/2015/05/4x4-Keyboard-Matrix-253x300.jpg]

The keys are wired like so, but there are diodes in the way to prevent ghosting (see !(Wired keyboard build)[https://www.crackedthecode.co/a-complete-guide-to-building-a-hand-wired-keyboard/#key-switch-diode-installation]).
Ghosting and phantom keys are eliminated by the use of a diode

**picture of keyboard so far**

It then becomes 
