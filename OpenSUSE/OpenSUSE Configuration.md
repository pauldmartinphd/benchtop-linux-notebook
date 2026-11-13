# openSUSE integration notes

[Build Configuration](../Build%20Configuration.md) describes the active Tumbleweed/OBS build. This page records integration work and package candidates, including earlier material collected while evaluating Aeon and MicroOS.

## Areas needing integration work

* RAID
* Disk Partitioning
* ZFS
* Cryptography
* Remote Management
* Automatic Disk Unlock

## Kernel support to verify in the image

* NVIDIA Module
* Binder and ASHMEM Modules (for Waydroid): checked November 13, 2026, openSUSE's kernel 7.3.3 has no binder (`CONFIG_ANDROID_BINDER_IPC` is not set), so Waydroid needs the TCBL kernel
* ZFS Module
* 1000 Hz tick rate (a kernel build setting, not a module)

## OBS Packages

* system76-scheduler (packaged November 5, benchtop-scheduler)
* adw-gtk3 (packaged November 5, benchtop-themes)
* pipewire-nonfree-codecs
* dislocker

## Application and integration candidates

The original list grouped these under Flatpak, but their delivery method is not established. Waydroid and desktop integration components in particular need checking against the image design.

* Waydroid:
    * Status, November 13, 2026: Tumbleweed has no Waydroid package (only Plasma Mobile's integration with it) and its kernel no binder; LXC 7.0 is packaged. Waydroid in TCBL therefore needs binder in the TCBL kernel and a Waydroid package in the TCBL repository.
    * Debian now packages Waydroid, and Ubuntu 26.04 and later install it from their archive ([OMG! Ubuntu, November 12, 2026](https://www.omgubuntu.co.uk/2026/11/android-apps-on-ubuntu-waydroid)): LineageOS in a container, GPU acceleration on Intel and AMD, a shared clipboard and Android notifications in GNOME. The article reports the Android container keeps running, and using power, until its service is stopped, and that about 4 GB of disk went to the image and a few games.
    * Google apps: TCBL can only ship the plain LineageOS image; the "with Google apps" image is what Waydroid downloads at its own setup when the user chooses it.
    * Idea, not a decision: stop the Android container when its last app closes, for battery life.
    * https://gist.github.com/Saren-Arterius/c5bc39199552a5c244449b0ce467d6b6
    * https://bugzilla.opensuse.org/show_bug.cgi?id=1189456
    * https://linux32bituefi.blogspot.com/2021/12/install-waydroid-in-opensuse-tumbleweed.html
    * https://github.com/SGNight/Arm-NativeBridge
* Ventoy
* Cyberchef
* SD Memory Card Formatter for Linux
* RStudio
* Nautilus w/ Sushi
* GNOME Software — selected application manager; an AppImage backend and AppImageHub integration are planned (see [Application management](../System%20Core/Pre-Packaged%20Environment.md#application-management))

## Zypper: Automatically Remove Orphaned Packages

By default, zypper does not remove orphaned dependencies when you remove a package. To enable automatic cleanup:

##### /etc/zypp/zypp.conf

    solver.cleandepsOnRemove = true

Alternatively, manually clean orphans:

    sudo zypper packages --orphaned
    sudo zypper remove --clean-deps <package>
