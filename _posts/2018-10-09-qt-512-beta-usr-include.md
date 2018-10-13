---
title:  "stdlib.h: No such file or directory"
published: true
tags: [linux, qt]
---

The Qt 5.12 beta was just [released](http://blog.qt.io/blog/2018/10/05/qt-5-12-lts-beta-released/)

I fed this beta to my qt-sdk-raspberry-pi [aur](https://www.chaos-reins.com/qpi/#aur-recipes) recipe, and:

```
compiling /opt/dev/src/arch/qt-sdk-raspberry-pi/src/qt-everywhere-src-5.12.0-beta1/qtbase/src/corelib/global/qt_pch.h
In file included from /opt/armv7-rpi2-linux-gnueabihf/armv7-rpi2-linux-gnueabihf/include/c++/6.1.0/bits/stl_algo.h:59:0,
                 from /opt/armv7-rpi2-linux-gnueabihf/armv7-rpi2-linux-gnueabihf/include/c++/6.1.0/algorithm:62,
                 from /opt/dev/src/arch/qt-sdk-raspberry-pi/src/qt-everywhere-src-5.12.0-beta1/qtbase/include/QtCore/../../src/corelib/global/qglobal.h:142,
                 from /opt/dev/src/arch/qt-sdk-raspberry-pi/src/qt-everywhere-src-5.12.0-beta1/qtbase/include/QtCore/qglobal.h:1,
                 from /opt/dev/src/arch/qt-sdk-raspberry-pi/src/qt-everywhere-src-5.12.0-beta1/qtbase/src/corelib/global/qt_pch.h:56:
/opt/armv7-rpi2-linux-gnueabihf/armv7-rpi2-linux-gnueabihf/include/c++/6.1.0/cstdlib:75:25: fatal error: stdlib.h: No such file or directory
 #include_next <stdlib.h>
                         ^
```

shit promptly barfed. This is a crappy issue to hit, I have seen it myself numerous times and keep on having to google it due to murkiness. I wont try to formalize why this has changed (my compiler is fixed, so changes in the sysroot and Qt are both up for imolation), what is very clear is that -I/usr/include or even -I/mnt/pi2/usr/include in your compiler line is gonna make the compiler blow chunks.

To be clear:

```
/opt/armv7-rpi2-linux-gnueabihf/bin/armv7-rpi2-linux-gnueabihf-g++ -pipe -pipe -march=armv7ve -mfpu=neon-vfpv4 -mthumb -mfloat-abi=hard --sysroot=/mnt/pi2 -Os -fPIC -std=c++1y -fvisibility=hidden -fvisibility-inlines-hidden -ffunction-sections -fdata-sections -flto -fno-fat-lto-objects -Wall -W -Wvla -Wdate-time -Wshift-overflow=2 -Wduplicated-cond -D_REENTRANT -DQT_NO_USING_NAMESPACE -DQT_NO_FOREACH -DQT_NO_NARROWING_CONVERSIONS_IN_CONNECT -DQT_BUILD_CORE_LIB -DQT_BUILDING_QT -DQT_NO_CAST_TO_ASCII -DQT_ASCII_CAST_WARNINGS -DQT_MOC_COMPAT -DQT_USE_QSTRINGBUILDER -DQT_DEPRECATED_WARNINGS -DQT_DISABLE_DEPRECATED_BEFORE=0x050000 -D_LARGEFILE64_SOURCE -D_LARGEFILE_SOURCE -DQT_NO_DEBUG -I/opt/dev/src/arch/qt-sdk-raspberry-pi/src/qt-everywhere-src-5.12.0-beta1/qtbase/src/corelib -I. -Iglobal -I/opt/dev/src/arch/qt-sdk-raspberry-pi/src/qt-everywhere-src-5.12.0-beta1/qtbase/src/3rdparty/harfbuzz/src -I/opt/dev/src/arch/qt-sdk-raspberry-pi/src/qt-everywhere-src-5.12.0-beta1/qtbase/src/3rdparty/md5 -I/opt/dev/src/arch/qt-sdk-raspberry-pi/src/qt-everywhere-src-5.12.0-beta1/qtbase/src/3rdparty/md4 -I/opt/dev/src/arch/qt-sdk-raspberry-pi/src/qt-everywhere-src-5.12.0-beta1/qtbase/src/3rdparty/sha3 -I/opt/dev/src/arch/qt-sdk-raspberry-pi/src/qt-everywhere-src-5.12.0-beta1/qtbase/src/3rdparty/forkfd -I/opt/dev/src/arch/qt-sdk-raspberry-pi/src/qt-everywhere-src-5.12.0-beta1/qtbase/src/3rdparty/tinycbor/src -I/opt/dev/src/arch/qt-sdk-raspberry-pi/src/qt-everywhere-src-5.12.0-beta1/qtbase/include -I/opt/dev/src/arch/qt-sdk-raspberry-pi/src/qt-everywhere-src-5.12.0-beta1/qtbase/include/QtCore -I../../include -I../../include/QtCore -I/opt/dev/src/arch/qt-sdk-raspberry-pi/src/qt-everywhere-src-5.12.0-beta1/qtbase/include/QtCore/5.12.0 -I/opt/dev/src/arch/qt-sdk-raspberry-pi/src/qt-everywhere-src-5.12.0-beta1/qtbase/include/QtCore/5.12.0/QtCore -I../../include/QtCore/5.12.0 -I../../include/QtCore/5.12.0/QtCore -I.moc -I.tracegen -isystem /mnt/pi2/usr/include -isystem /mnt/pi2/usr/include -I/opt/dev/src/arch/qt-sdk-raspberry-pi/src/qt-everywhere-src-5.12.0-beta1/qtbase/mkspecs/devices/linux-rpi2-g++ -x c++-header -c /opt/dev/src/arch/qt-sdk-raspberry-pi/src/qt-everywhere-src-5.12.0-beta1/qtbase/src/corelib/global/qt_pch.h -o .pch/Qt5CorePi2.gch/c++
In file included from /opt/armv7-rpi2-linux-gnueabihf/armv7-rpi2-linux-gnueabihf/include/c++/6.1.0/bits/stl_algo.h:59:0,
                 from /opt/armv7-rpi2-linux-gnueabihf/armv7-rpi2-linux-gnueabihf/include/c++/6.1.0/algorithm:62,
                 from /opt/dev/src/arch/qt-sdk-raspberry-pi/src/qt-everywhere-src-5.12.0-beta1/qtbase/include/QtCore/../../src/corelib/global/qglobal.h:142,
                 from /opt/dev/src/arch/qt-sdk-raspberry-pi/src/qt-everywhere-src-5.12.0-beta1/qtbase/include/QtCore/qglobal.h:1,
                 from /opt/dev/src/arch/qt-sdk-raspberry-pi/src/qt-everywhere-src-5.12.0-beta1/qtbase/src/corelib/global/qt_pch.h:56:
/opt/armv7-rpi2-linux-gnueabihf/armv7-rpi2-linux-gnueabihf/include/c++/6.1.0/cstdlib:75:25: fatal error: stdlib.h: No such file or directory
 #include_next <stdlib.h>

```

That -isystem /mnt/pi2/usr/include is fucking us. But who introduced it. Say goodbye to time better spent elsewhere. At first I assumed pkg-config was the perp, or Qt files in /usr/lib/pkgconfig, so I squandered some youth there, making sure they were (not) to blame. Hilariously, pkg-config offers:

```
  --keep-system-cflags              keep -I/usr/include entries in cflags output
```

Qt isn't using this argument. Another smoking gun was:

```
QMAKE_INCDIR_LIBUDEV = /mnt/pi2/usr/include
```

in qtbase/mkspecs/qmodule.pri, which certainly bolsters the assumption that something which automatically provisions paths is awry. After a fair amount of bashing, I accepted pkgconfig as innocent enough to warrant searching onwards and started exploring other options. In these circumstances, it is wise to check the hidden qmake files at various heights of your build tree. It also clearly helps to be using shadow builds, in order to differentiate the shipped source from the build artifacts. As it turns out, after I configured Qt, I had to manually remove a line from a hidden file.
QMAKE_CXX.INCDIRS = \
    /opt/armv7-rpi2-linux-gnueabihf/armv7-rpi2-linux-gnueabihf/include/c++/6.1.0 \
    /opt/armv7-rpi2-linux-gnueabihf/armv7-rpi2-linux-gnueabihf/include/c++/6.1.0/armv7-rpi2-linux-gnueabihf \
    /opt/armv7-rpi2-linux-gnueabihf/armv7-rpi2-linux-gnueabihf/include/c++/6.1.0/backward \
    /opt/armv7-rpi2-linux-gnueabihf/lib/gcc/armv7-rpi2-linux-gnueabihf/6.1.0/include \
    /opt/armv7-rpi2-linux-gnueabihf/lib/gcc/armv7-rpi2-linux-gnueabihf/6.1.0/include-fixed \
    /opt/armv7-rpi2-linux-gnueabihf/armv7-rpi2-linux-gnueabihf/include \
    /mnt/pi2/usr/include

Leading to:

```
commit bc5c9cfe534c87d6754932de53cd2949dc792244
Author: Donald Carr <d@chaos-reins.com>
Date:   Tue Oct 9 00:01:19 2018 -0700

    Remove gnarly /usr/include breaking the build

diff --git a/PKGBUILD b/PKGBUILD
index eaf1189..e2a6a33 100644
--- a/PKGBUILD
+++ b/PKGBUILD
@@ -416,6 +416,7 @@ fi
   echo ${_configure_line} > ${_configure_line_fn}
   set &> configure_env
   ${_configure_line} || exit 1
+  sed -i "/\/usr\/include$/d" ${_bindir}/.qmake.stash
   make || exit 1
 }
```

This bug deals with this issue:

https://bugreports.qt.io/browse/QTBUG-53367

there are many variations of it.
