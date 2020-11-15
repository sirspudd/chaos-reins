---
title:  "Building a toolchain using crosstool-ng"
date:   2020-11-28 12:10:59 -0700
published: true
tags: [embedded, linux, oss]
---

crosstool-ng is one hell of a tool; there is an AUR recipe:

crosstool-ng, yay -S crosstool-ng suffices or whatever manual steps you chose to subject yourself to.

I am attempting to build a toolchain for the Jetson Nano; this board has the dubios destinction of being the board which required the most experimentation with different toolchains in order to find one which worked. I am now curious to see if I can build a nice modern gcc and simply have it work on a modern jetson development image.

Functional requirements:

* multilib support (the base image is Ubuntu, aarch64)
