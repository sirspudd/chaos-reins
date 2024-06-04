---
layout: post
title: Get rid of the KDE 6 prerelease banner
subtitle: OLED friendly KDE ahoy
date:   2023-12-02
published: true
tags: [KDE 6, Qt, Beta]
---

# KDE 6 installation/adoption

After hearing about the KDE 6 beta, I went sniffing around in Arch and sure enough kde-unstable has something readily consumable.
KDE is already my daily driver; it is a thing of glory and should be a point of pride for everyone who contributed to it, and to anyone Qt affiliated. What a groovy techincal showcase/shell.

In any case; KDE 6 is gorgeous, I git one issue switching to it, caused by me explicitly overriding sddm to run under wayland. This configuration snippet

```
[0] <git:(master 0dad43a✱) > cat /etc/10-wayland.conf
[General]
DisplayServer=wayland
GreeterEnvironment=QT_WAYLAND_SHELL_INTEGRATION=layer-shell

[Wayland]
CompositorCommand=kwin_wayland --drm --no-lockscreen --no-global-shortcuts --locale1
```

did not prevent sddm from launching, but it did prevent key strokes from being received and my attempts to input my password were moot until I pulled this from play.

All in all, pretty bloody smooth.

# OLED hostile prerelease banner ahoy

I run KDE with 3x3 virtual desktops on a 55inch LG CX; when I leave my desktop, I tend to park my screen on a purely black background to avoid any chance of burn in and since it nicely parks my screen in a state which is visually indistinguishable from turned off (the backlight after all is at zero).

Unfortunately, the peeps who hack on this stuff have seen fit to include a fixed offset snipped of text in the bottom right hand side of the plasma workspace UI, utter ripe for burning in:

![KDE Preview Release Banner](/img/plasma-prerelease-banner.jpg)

I went spelunking to find the source code responsible for this atrocity, and to find out if there was a trivial way to toggle it.

```
https://invent.kde.org/plasma/plasma-workspace.git
shell/desktopview.cpp

#if PROJECT_VERSION_PATCH >= 80 || PROJECT_VERSION_MINOR >= 80
bool DesktopView::showPreviewBanner() const
{
    static const bool shouldShowPreviewBanner =
        !KConfigGroup(KSharedConfig::openConfig("kdeglobals"), u"General"_s).readEntry("HideDesktopPreviewBanner", false);
    return shouldShowPreviewBanner;
}

QString DesktopView::previewBannerTitle() const
{
    // Plasma 6 pre-release versions
    if constexpr (PROJECT_VERSION_MAJOR == 5 && PROJECT_VERSION_MINOR >= 80) {
        if constexpr (PROJECT_VERSION_PATCH == 80) {
            // Development
            return i18nc("@label %1 is the Plasma version", "KDE Plasma 6.0 Dev");
        } else if constexpr (PROJECT_VERSION_MINOR == 80) {
            // Alpha, 5.80.0
            return i18nc("@label %1 is the Plasma version", "KDE Plasma 6.0 Alpha");
        } else if constexpr (PROJECT_VERSION_MINOR == 90) {
            // Beta 1, 5.90.0
            return i18nc("@label %1 is the Plasma version", "KDE Plasma 6.0 Beta 1");
        } else if constexpr (PROJECT_VERSION_MINOR == 91) {
            // Beta 2, 5.91.0
            return i18nc("@label %1 is the Plasma version", "KDE Plasma 6.0 Beta 2");
        } else if constexpr (PROJECT_VERSION_MINOR == 92) {
            // RC1, 5.92.0
            return i18nc("@label %1 is the Plasma version, RC meaning Release Candidate", "KDE Plasma 6.0 RC1");
        } else if constexpr (PROJECT_VERSION_MINOR == 93) {
            // RC2, 5.93.0
            return i18nc("@label %1 is the Plasma version, RC meaning Release Candidate", "KDE Plasma 6.0 RC2");
        }
    }
```

so we can gloriously removed this crud with

```
[0] <git:(master cf62a04f78) > cat /home/sirspudd/.config/kdedefaults/kdeglobals
[General]
ColorScheme=BreezeDark
HideDesktopPreviewBanner=true
```

Leaving nothing but joy
