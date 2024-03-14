---
title:  "Yocto Gotchas"
date:   2024-03-12 12:10:59 -0700
published: true
tags: [linux, yocto, qt, DISTRO_VERSION, SDK_VERSION, SDKPATH]
---

# Overview

I am responsible for a yocto distro which targets 2 custom devices. For the most part I have a high opinion of it, but I recently spent a fair amount of timing living with a shit situation which I eventually debugged to completion, so I am starting this page to centralize these issues.

# Qt rebuilds on each and every DISTRO_VERSION change

## TL&DR

SDKPATH is a pathological variable. It needs to be a static string. If you want to set your SDKPATH, you need to see SDKPATHINSTALL to something reasonable, which normally contains [DISTRO,SDK]_VERSION in the path. This changed upstream, and anyone setting this value is going to be ruing their life and the limits of their build server.

Yoe is an excellent reference distro.

## Blather

I tend to keep the yocto I produce up to date, so at the start (up through the middle) of a release cycle I will update yocto distros to drink the coalesced fixes/hotness from upstream. At some point, I became aware that simple leaf node changes were resulting in all of Qt being rebuilt. I don't actually increment build versions when testing, so this only manifested once it hit our build server, and that is beast enough, and yocto verbose enough that it was easy to overlook the rebuilding of the complete dependency graph from qtbase-native onwards. I took several stabs at trying to establish what could possibly be causing this, grepping in meta-qt for DISTRO_VERSION related considerations. I actually attempted this debugging on a couple occasions, never made much headway and lived with 40 min warm builds on a modern EPYC server.

This issue was not manifest in reference distros like [yoe](https://www.yoedistro.org/). What gives?

Well, it turns out DISTRO_VERSION was not actually the issue; in my distro I set SDK_VERSION = DISTRO_VERSION, and it was this setting of this SDK_VERSION which was driving the rebuilds. First I tried setting SDK_VERSION to a fixed value for non class-sdk components, since SDK_VERSION is used in the SDK path. This resulted in the most perplexing artifact generation I have seen in 7 years of working in yocto land. The resulting images contain binaries/artifacts from weeks/months of sstate cache. It is wicked, you will see the phantom of old 

 My then attempted to isolate this involved hardcoding SDK_VERSION in qtbase. This succeeded! I had to set SDK_VERSION for class-target and class-native, not class-sdk. And that appeared to work. Now that I had a solution, I went back and looked at yoe again in order to substantiate/communicate my discovery of a work around. That was when I noticed that they set SDKPATHINSTALL, where as I was setting SDKPATH for our distro. This value had not changed, it had not previously caused an issue, but lo and behold when I removed my SDK_VERSION hack, the issue remained solved.

I don't know why only Qt is hit by this

```
    # resolve absolute paths at runtime
    sed -i -e 's|${SDKPATH}/sysroots|\${SYSROOTS}|g' \
        ${SDK_OUTPUT}${NATIVE_SYSROOT}/usr/share/cmake/Qt6Toolchain.cmake
```

Maybe this is responsible?
