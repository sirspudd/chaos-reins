---
title:  "Qt 5.13 rc3 on the Nvidia Jetson Nano"
published: true
tags: [linux, qt, nvidia, jetson, artriculate]
---

<iframe width="560" height="315" src="https://www.youtube.com/embed/hxTvs5CSIS4" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

# Goals

* Getting Qt running on the jetson-nano without X11
* Get pet project running on the jetson
* Get pet project running well on the jetson

# TL&DR

* Pros
    * Glorious kit; I have grown so accustom to the vc4 (raspberry pi) and ARM Mali devices, that I am blown out of the water by the performance of this hardware, and the fact the drivers largely just work *golf clap* and work well. My application runs at 4K@60fps _out of the box_ when I had personally blamed myself (and joylessly pursued) 1080p@60fps on the raspberry pi (proprietary and vc4 drm)
* Gotchas
    * toolchains ate my face
    * Configuring Qt (and the associated mkspec) ate my face
    * Updating the image took me back several configuration steps
    * They actually ship weston along with their binary drivers, so using it as fair Wayland reference was actually moot. #JustAnotherOpaqueBinary
    * That heat sink aint there for outrageous loads; my app was sufficient to make it cook

# Shout outs

* [https://github.com/madisongh/meta-tegra/wiki/Wayland-Weston-support-on-TX1-TX2-Xavier-Nano](https://github.com/madisongh/meta-tegra/wiki/Wayland-Weston-support-on-TX1-TX2-Xavier-Nano) (for documenting a bunch of valuable otherwise painful to earn/aggregate; and potentially a Yocto channel)
* The Qt peeps who have been publicly blogging about their active use of Nvidia targets and making them pretty easy to bring up

# All you need

* [https://github.com/sirspudd/mkspecs/blob/master/linux-jetson-nano-g%2B%2B/qmake.conf](https://github.com/sirspudd/mkspecs/blob/master/linux-jetson-nano-g%2B%2B/qmake.conf)
* [https://developer.nvidia.com/embedded/dlc/kernel-gcc-6-4-tool-chain](https://developer.nvidia.com/embedded/dlc/kernel-gcc-6-4-tool-chain)
* An adequate Qt source bundle, I tend to live on the bleeding edge of things
* Optional: [https://github.com/sirspudd/qt-sdk-raspberry-pi](https://github.com/madisongh/meta-tegra/wiki/Wayland-Weston-support-on-TX1-TX2-Xavier-Nano) (aur recipe I use to build target centric sdks for Arch; read bash script to the uninitiated)

# Bring up

Grabbed the only game (OS image) in town: [LFT](https://developer.nvidia.com/embedded/learn/get-started-jetson-nano-devkit#setup)
Booted into L4T
enabled NFS (for my dev flow)

/etc/exports
```
/ *(ro,fsid=root,no_root_squash,nohide,insecure)
```

uncommented all deb-src in /etc/apt/sources.list
```
deb http://ports.ubuntu.com/ubuntu-ports/ bionic main restricted
deb-src http://ports.ubuntu.com/ubuntu-ports/ bionic main restricted

## Major bug fix updates produced after the final release of the
## distribution.
deb http://ports.ubuntu.com/ubuntu-ports/ bionic-updates main restricted
deb-src http://ports.ubuntu.com/ubuntu-ports/ bionic-updates main restricted
.....
```

apt-get build-dep qtdeclarative5-dev

This installed all the extended build dependencies for Qt, saving me the hassle of knowing them or tracking them down.

Protip #1

apt-get install symlinks

I can't recall the exact symlinks call, but it roughly amounts to:

```
symlinks -c -r /
```

This is to avoid absolute symlinks screwing you when you are NFS mounted into a remote alien desktop environment and someone tries to dereference it.

Mounted jetson against /mnt/pi5 to use as a sysroot to compile against.

Started cobbling together a jetson-nano Qt mkspec from the existing Nvidia ones

* Tried to use a vendor agnostic aarch64 toolchain (one I compiled/built for the pi3) already on my machine; that barfed
* Tried to use community/aarch64-linux-gnu-gcc (Arch, aur)
* Tried aarch64 toolchain directly from ARM
* Went back in time, tried aarch64 toolchain from Linaro
* Tried toolchain from Nvidia on their jetson-nano page
* Tried older (gcc-linaro-7.1.1-2017.08-x86_64_arm-linux-gnueabihf toolchain from jetson-nano page.)[https://developer.nvidia.com/embedded/dlc/kernel-gcc-6-4-tool-chain]

Success! Your toolchain can compile binaries and does not explode in a plethora of unreadable error messages about a wealth of issues. (The most interesting being a disagreement about native primate integer types.

This toolchain hopping cost me around 3-4 hours of my thirties and I will bite someone if I ever find an assailant other than my own shitty choice of hobbies.

The final mkspec looks so:

```
#
# qmake configuration for the Jetson TK1 boards running Linux For Tegra
#
# Note that this environment has been tested with X11 only.
#
# A typical configure line might look like:
# configure \
#   -device linux-jetson-tk1-g++ \
#   -device-option CROSS_COMPILE=/opt/nvidia/toolchains/tegra-4.8.1-nv/usr/bin/arm-cortex_a15-linux-gnueabi/arm-cortex_a15-linux-gnueabi- \
#   -sysroot /opt/nvidia/l4t/targetfs

include(../common/linux_device_pre.conf)

QMAKE_INCDIR_POST += \
    $$[QT_SYSROOT]/usr/include/$${GCC_MACHINE_DUMP}

QMAKE_LIBDIR_POST += \
    $$[QT_SYSROOT]/usr/lib \
    $$[QT_SYSROOT]/lib/$${GCC_MACHINE_DUMP} \
    $$[QT_SYSROOT]/usr/lib/$${GCC_MACHINE_DUMP}

QMAKE_RPATHLINKDIR_POST += \
    $$[QT_SYSROOT]/usr/lib \
    $$[QT_SYSROOT]/usr/lib/$${GCC_MACHINE_DUMP} \
    $$[QT_SYSROOT]/lib/$${GCC_MACHINE_DUMP}

DISTRO_OPTS += aarch64
# gcc -march=native -Q --help=target
COMPILER_FLAGS               += -march=armv8-a+crypto+crc
# -mstrict-align

EGLFS_DEVICE_INTEGRATION = eglfs_kms_egldevice

include(../common/linux_arm_device_post.conf)
load(qt_config)
```

The crux is:

```
include(../common/linux_device_pre.conf)

# deb-multi-arch madness
QMAKE_INCDIR_POST += \
    $$[QT_SYSROOT]/usr/include/$${GCC_MACHINE_DUMP}

QMAKE_LIBDIR_POST += \
    $$[QT_SYSROOT]/usr/lib \
    $$[QT_SYSROOT]/lib/$${GCC_MACHINE_DUMP} \
    $$[QT_SYSROOT]/usr/lib/$${GCC_MACHINE_DUMP}

QMAKE_RPATHLINKDIR_POST += \
    $$[QT_SYSROOT]/usr/lib \
    $$[QT_SYSROOT]/usr/lib/$${GCC_MACHINE_DUMP} \
    $$[QT_SYSROOT]/lib/$${GCC_MACHINE_DUMP}
# end: deb-multi-arch madness

# Qt is gonna hurt you if you dont supply this for aarch64 targets
DISTRO_OPTS += aarch64
# end: Qt is gonna hurt you if you dont supply this for aarch64 targets

# sane baseline CXXFLAG
COMPILER_FLAGS               += -march=armv8-a+crypto+crc
# end: sane baseline CXXFLAG

# use eglfs + egldevice not X11 by default
EGLFS_DEVICE_INTEGRATION = eglfs_kms_egldevice
# end: use eglfs + egldevice not X11 by default

include(../common/linux_arm_device_post.conf)
load(qt_config)
```

Groovy; you can compile binaries but are you gonna get what you want. Probably not. The next step is configure Qt repeatedly until you see the functionality you want get detected and pass initial inspection. This is gloriously complicated by the fact that this is an Ubuntu image and they use the multiarch arrangement, which means an agnostic toolchain is toast until you enlighten it as to where to look.

This is further complicated by the fact that for deb-multi-arch Qt appears to expect pkg-config to be part of your toolchain and will actually explicitly attempt to use ${TOOLCHAIN_PREFIX}pkg-config

mkspecs/devices/common/linux_device_pre.conf
```
contains(DISTRO_OPTS, deb-multi-arch): \
    QMAKE_PKG_CONFIG = $${CROSS_COMPILE}pkg-config
```

You dont want to be exploring Qt configure without pkg-config support; my first couple builds were deceptively functional, but for some reason I could still not run applications without X11. Turns out that libdrm was infinitely far away from passing the compile test as a result of pkg-config not being part of the toolchain (#1 problem) and the generic link line for libdrm not being up to the task.

Problem #1 is skirted by not using DISTRO_OPTS += deb-multi-arch in the mkspec; I just sucked the desired functionality up into the mkspec itself (searching /usr/include/aarch64-linux-gnu, /usr/lib/aarch64-linux-gnu/)
If your toolchain does not understand deb-multi-arch, you are gonna need:

export PKG_CONFIG_LIBDIR=${_sysroot}/usr/lib/$(${_toolchain}gcc -dumpmachine)/pkgconfig/:${_sysroot}/usr/lib/pkgconfig/
export PKG_CONFIG_SYSROOT_DIR=${_sysroot}

on your build server, prior to configuring Qt.

The end goal is:

```
sirspudd@beast /opt/dev/src/arch/qt-sdk-raspberry-pi (git)-[master] % ls build/config.tests 
alloca_h         cloexec       drm_atomic     eventfd     ipv6ifname     opengl_es2         reduce_relocations   xcb_xlib
arch             conftest-out  egl            futimens    libatomic      opengles3          renameat2            xkbcommon
atomicfptr       conftest.cpp  egl-egldevice  getauxval   libdl          opengles31         separate_debug_info  xlib
c++14            cxx11_future  egl-mali       getentropy  librt          opengles32         statx                zlib
c++1z            cxx11_random  egl-mali-2     getifaddrs  libudev        posix_fallocate    stl
c11              d3d12         egl-viv        glibc       linkat         ppoll              verifyspec
c99              dbus          egl-x11        inotify     linux-netlink  precompile_header  x86_simd
clock-monotonic  drm           evdev          ipc_sysv    linuxfb        reduce_exports     xcb
```

If you want life without X11, you really want:

```
sirspudd@beast /opt/dev/src/arch/qt-sdk-raspberry-pi (git)-[master] % ls build/config.tests/drm                          :(
Makefile  drm  drm.pro  main.cpp  main.o
```
to have successfully generated artifacts.

I must admit, at this point I kinda grew frustrated at propagating variables through Qt to the respective mkspecs and on to the actually CFLAGS/LIB variables.

I ended up just modifying: /usr/lib/aarch64-linux-gnu/pkgconfig/libdrm.pc

```
prefix=/usr
libdir=${prefix}/lib/aarch64-linux-gnu
includedir=${prefix}/include

Name: libdrm
Description: Userspace interface to kernel DRM services
Version: 2.4.95
Libs: -L${libdir} -L${libdir}/tegra -ldrm -ldrm -lnvll -lnvrm -lnvdc -lnvos -lnvrm_graphics -lnvimp
Cflags: -I${includedir} -I${includedir}/drm -I${includedir}/libdrm
```

please also note the addition of the -I${includedir}/drm cflag which I found to be necessary. I am also not certain why -I${includedir} is not blowing up the build, as I have seen it blow up many a build with an inability to forward stdint to the correct header. Niiiiiiicccceeee.

In any case, with this arrangement we are off to the races:

```
/opt/dev/src/arch/qt-sdk-raspberry-pi/src/qt-everywhere-src-5.13.0-rc3/configure -ltcg -qt-freetype -qt-harfbuzz -qt-libpng -qt-libjpeg -qt-pcre -pkg-config -prefix /opt/qt/qt-sdk-raspberry-pi5 -opengl es2 -no-xcb -qpa eglfs -egl -qt-sqlite -optimized-qmake -optimized-tools -optimize-size -confirm-license -opensource -v -silent -release -pch -make libs -nomake tools -nomake examples -reduce-exports -qtlibinfix Pi5 -sysroot /mnt/pi5 -device linux-jetson-nano-g++ -device-option CROSS_COMPILE=/opt/gcc-linaro-6.4.1-2017.08-x86_64_aarch64-linux-gnu/bin/aarch64-linux-gnu- -skip qtwebengine -no-icu -skip qtscript -no-widgets -force-debug-info -separate-debug-info -no-direct2d -no-directfb -no-mirclient -no-cups -no-iconv -no-gif -no-sql-mysql -no-sql-psql -no-tslib -no-feature-bearermanagement -no-qml-debug -no-ico -no-glib -hostprefix /opt/qt/qt-sdk-raspberry-pi5 -no-gbm
```

```
Building on: linux-g++ (x86_64, CPU features: mmx sse sse2)
Building for: devices/linux-jetson-nano-g++ (arm64, CPU features: neon crc32)
Target compiler: gcc 6.4.1
Configuration: cross_compile compile_examples enable_new_dtags force_debug_info largefile ltcg neon optimize_size precompile_header silent shared rpath release c++11 c++14 concurrent dbus reduce_exports reduce_relocations release_tools separate_debug_info stl no-widgets
Build options:
  Mode ................................... release (with debug info); optimized tools
  Optimize release build for size ........ yes
  Building shared libraries .............. yes
  Using C standard ....................... C11
  Using C++ standard ..................... C++14
  Using ccache ........................... no
  Using new DTAGS ........................ yes
  Generating GDB index ................... no
  Using precompiled headers .............. yes
  Using LTCG ............................. yes
  Target compiler supports:
    NEON ................................. yes
  Build parts ............................ libs
Qt modules and options:
  Qt Concurrent .......................... yes
  Qt D-Bus ............................... yes
  Qt D-Bus directly linked to libdbus .... yes
  Qt Gui ................................. yes
  Qt Network ............................. yes
  Qt Sql ................................. yes
  Qt Testlib ............................. yes
  Qt Widgets ............................. no
  Qt Xml ................................. yes
Support enabled for:
  Using pkg-config ....................... yes
  udev ................................... yes
  Using system zlib ...................... yes
  Zstandard support ...................... no
Qt Core:
  DoubleConversion ....................... yes
    Using system DoubleConversion ........ no
  GLib ................................... no
  iconv .................................. no
  ICU .................................... no
  Built-in copy of the MIME database ..... yes
  Tracing backend ........................ <none>
  Logging backends:
    journald ............................. no
    syslog ............................... no
    slog2 ................................ no
  Using system PCRE2 ..................... no
Qt Network:
  getifaddrs() ........................... yes
  IPv6 ifname ............................ yes
  libproxy ............................... no
  Linux AF_NETLINK ....................... yes
  OpenSSL ................................ no
    Qt directly linked to OpenSSL ........ no
  OpenSSL 1.1 ............................ no
  DTLS ................................... no
  OCSP-stapling .......................... no
  SCTP ................................... no
  Use system proxies ..................... yes
Qt Gui:
  Accessibility .......................... yes
  FreeType ............................... yes
    Using system FreeType ................ no
  HarfBuzz ............................... yes
    Using system HarfBuzz ................ no
  Fontconfig ............................. no
  Image formats:
    GIF .................................. no
    ICO .................................. no
    JPEG ................................. yes
      Using system libjpeg ............... no
    PNG .................................. yes
      Using system libpng ................ no
  EGL .................................... yes
  OpenVG ................................. no
  OpenGL:
    Desktop OpenGL ....................... no
    OpenGL ES 2.0 ........................ yes
    OpenGL ES 3.0 ........................ yes
    OpenGL ES 3.1 ........................ yes
    OpenGL ES 3.2 ........................ yes
  Vulkan ................................. no
  Session Management ..................... yes
Features used by QPA backends:
  evdev .................................. yes
  libinput ............................... no
  INTEGRITY HID .......................... no
  mtdev .................................. no
  tslib .................................. no
  xkbcommon .............................. yes
  X11 specific:
    XLib ................................. yes
    XCB Xlib ............................. yes
    EGL on X11 ........................... yes
QPA backends:
  DirectFB ............................... no
  EGLFS .................................. yes
  EGLFS details:
    EGLFS OpenWFD ........................ no
    EGLFS i.Mx6 .......................... no
    EGLFS i.Mx6 Wayland .................. no
    EGLFS RCAR ........................... no
    EGLFS EGLDevice ...................... yes
    EGLFS GBM ............................ no
    EGLFS VSP2 ........................... no
    EGLFS Mali ........................... no
    EGLFS Raspberry Pi ................... no
    EGLFS X11 ............................ yes
  LinuxFB ................................ yes
  VNC .................................... yes
  Mir client ............................. no
Qt Sql:
  SQL item models ........................ yes
Qt Widgets:
  GTK+ ................................... no
  Styles ................................. Fusion Windows
Qt PrintSupport:
  CUPS ................................... no
Qt Sql Drivers:
  DB2 (IBM) .............................. no
  InterBase .............................. no
  MySql .................................. no
  OCI (Oracle) ........................... no
  ODBC ................................... no
  PostgreSQL ............................. no
  SQLite2 ................................ no
  SQLite ................................. yes
    Using system provided SQLite ......... no
  TDS (Sybase) ........................... no
Qt Testlib:
  Tester for item models ................. yes
Qt QML:
  QML network support .................... yes
  QML debugging and profiling support .... no
  QML tracing JIT support ................ no
  QML sequence object .................... yes
  QML list model ......................... yes
  QML XML http request ................... yes
  QML Locale ............................. yes
  QML delegate model ..................... yes
Qt Quick:
  Direct3D 12 ............................ no
  AnimatedImage item ..................... yes
  Canvas item ............................ yes
  Support for Qt Quick Designer .......... yes
  Flipable item .......................... yes
  GridView item .......................... yes
  ListView item .......................... yes
  TableView item ......................... yes
  Path support ........................... yes
  PathView item .......................... yes
  Positioner items ....................... yes
  Repeater item .......................... yes
  ShaderEffect item ...................... yes
  Sprite item ............................ yes

Note: Also available for Linux: linux-clang linux-icc

Note: -optimized-tools is not useful in -release mode.
```

The critical pieces above are:

```
  Qt Widgets ............................. no
  EGL .................................... yes
  OpenGL:
    Desktop OpenGL ....................... no
    OpenGL ES 2.0 ........................ yes
    OpenGL ES 3.0 ........................ yes
    OpenGL ES 3.1 ........................ yes
    OpenGL ES 3.2 ........................ yes
  evdev .................................. yes
  xkbcommon .............................. yes
  EGLFS .................................. yes
    EGLFS EGLDevice ...................... yes
```

libinput is great too, but it was excluded from this build due to muppetry on my part. What we want is EGL, OpenGL* _and_ EGLFS EGLDevice.

Once this was accomplished, Qt (base and declarative) built successfully, and I could successfully compile/deploy my application to the tool and run it sans X11.

# interest
## thermals

The device does get hot running my art application:

```
root@boombox:~# cat /sys/kernel/debug/soctherm/cpu/temp
74500
root@boombox:~# cat /sys/kernel/debug/soctherm/gpu/temp
73500
root@boombox:~# cat /sys/kernel/debug/soctherm/mem/temp
74500
root@boombox:~# cat /sys/kernel/debug/soctherm/pll/temp
71000
root@boombox:~# sensors
thermal-fan-est-virtual-0
Adapter: Virtual device
temp1:        +74.0°C
```

Nvidia's thermal design guide:

https://developer.download.nvidia.com/assets/embedded/secure/jetson/Nano/docs/Jetson_Nano_Thermal_Design_Guide.pdf?aSH8MZC4et2iweqxWEN0qG6qYpnibw27V4CQYp7TNqUao0yf_VoGOpdk4WWjz6MT96TBt4ywjPEzfCJtSyMKT2gbryx40jIWGalsxmTeY8NJzlgavrD-r240-UZYfxazDqBVHQ4utI-R6o8NO24RCE7OLYmzqBZXZJ3RLjHfCbIXA_ivFPPyfZyt2w

Does give one the impression that I have another 25 degrees celsius of head room before shit gets critical.

# ugly

```
root@tegra-ubuntu:/usr/lib/aarch64-linux-gnu/pkgconfig# ls -la /usr/bin/weston
lrwxrwxrwx 1 root root 46 Jun 12 02:37 /usr/bin/weston -> /usr/lib/aarch64-linux-gnu/tegra/weston/weston
```

```
head -n1 /etc/nv_tegra_release
ls
tar xf Jetson-Nano-Tegra210_Linux_R32.1.0_aarch64.tbz2
ls
cd Linux_for_Tegra/
ls
./apply_binaries.sh
./apply_binaries.sh -r /
apt install lbzip2
./apply_binaries.sh -r /
```

They ship an ubuntu image, then provide updates as a tarball?! which you unpack and update by running a bash script which engages in inplace binary replacement?
I have worked on mom&pop projects with 1 user where you package assets and spin up an Artifactory/Deb server. Not only that, but running this glorious script to update your GL libraries blows away your hostname and changes all manner of other shite. The device is lovely, but hiring a dev ops critter to manage this broadly used spin/distro would make this a great deal less thorny.

# Further reading/resources

* [https://blog.qt.io/blog/2016/09/19/qt-graphics-with-multiple-displays-on-embedded-linux/](https://blog.qt.io/blog/2016/09/19/qt-graphics-with-multiple-displays-on-embedded-linux/) (The Qt blog is generally great and people like Laszlo Agocs are actively improving/extending Qt on any embedded device with a GPU and actively sharing insights on said blog; saves you time and sheds direct light into an otherwise arcane area)
* [https://blog.qt.io/blog/2016/11/10/qt-nvidia-jetson-tx1-device-creation-style/](https://blog.qt.io/blog/2016/11/10/qt-nvidia-jetson-tx1-device-creation-style/)
* [https://developer.nvidia.com/embedded/linux-tegra](https://developer.nvidia.com/embedded/linux-tegra) (Where I got the functional toolchain)
* [https://doc.qt.io/qt-5/embedded-linux.html#eglfs-with-eglfs-kms-backend](https://doc.qt.io/qt-5/embedded-linux.html#eglfs-with-eglfs-kms-backend) (Such glorious docs)
* [https://rudd-o.com/linux-and-free-software/how-to-make-pulseaudio-run-once-at-boot-for-all-your-users](https://rudd-o.com/linux-and-free-software/how-to-make-pulseaudio-run-once-at-boot-for-all-your-users) (I have had to find this page so many times; install shairport-sync and you now have an airport sound sink dumping to HDMI (there is no 3.5 mm jack) #ItJustDuckingWorks)
