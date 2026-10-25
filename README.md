# Technicomp Benchtop Linux — design notes

Benchtop Linux is a GNOME workstation built on openSUSE Tumbleweed, with an immutable base updated through transactional-update. The design keeps the development tools and applications current while taking a more conservative approach to the kernel and desktop: an upstream kernel.org LTS kernel and GNOME’s old stable release (n−1). GNOME Software is the application manager, with Flatpak for GUI applications and Homebrew for additional CLI tools. Planned AppImage support adds an AppImage backend and AppImageHub integration to GNOME Software, with per-user installation under `~/Applications/` and generated desktop launchers.

These notes record the design, configuration choices, experiments, and unresolved questions. [Build Configuration](Build%20Configuration.md) describes the working Tumbleweed/OBS image and its source repositories. It also identifies work still needed for the custom kernel and GNOME release policy; those design goals should not be read as completed build features.

Start with the [distro vision](System%20Core/Distro%20Vision.md) for the intended workstation, then use the topic notes below. Package lists are working selections and candidates, not an inventory verified against a released image. Configuration snippets describe the intended settings or experiments; the implementation lives in the separate build repositories.

## Supported hardware baseline

The minimum design and test target is:

* CPU: x86-64-v3, 4 cores / 8 threads or better
* Memory: 16 GB minimum; 32 GB or more recommended
* Storage: 1 TB SSD minimum
* Display: 1920 × 1080 minimum
* Graphics: hardware acceleration suitable for the supported GNOME/Wayland stack

Lower-spec hardware may work, but it is outside the design and test target. The project does not aim to make this workstation environment suitable for very limited hardware.

## Notes by topic

| Topic | Notes |
|---|---|
| Build | [Current OBS build](Build%20Configuration.md), [openSUSE integration](OpenSUSE/OpenSUSE%20Configuration.md), [rejected NixOS proposal](NixOS-Based%20Distro%20Build%20Plan.md) |
| System | [Boot](System%20Core/Boot%20and%20Init.md), [filesystems](System%20Core/Filesystems.md), [drivers and firmware](System%20Core/Drivers%20and%20Firmware.md), [audio](System%20Core/Audio.md), [power](System%20Core/Power%20Management.md), [user environment](System%20Core/Pre-Packaged%20Environment.md) |
| Desktop | [GNOME behavior](UX%20and%20Desktop/GNOME%20Configuration.md), [extensions](UX%20and%20Desktop/GNOME%20Extensions.md), [tablet mode](UX%20and%20Desktop/GNOME%20Tablet%20Mode.md), [input](UX%20and%20Desktop/Input%20Devices.md), [fonts](UX%20and%20Desktop/Fonts.md), [theming](UX%20and%20Desktop/Theming.md) |
| Performance | [Kernel](Performance/Kernel%20Tuning.md), [memory](Performance/Memory%20Management.md), [storage](Performance/Storage%20and%20IO.md), [network](Performance/Network%20Tuning.md), [gaming](Performance/Gaming%20Mode.md), [benchmarking](Performance/Benchmarking.md), [future experiments](Performance/Future%20Ideas.md) |
| Security | [Architecture](Security/Security%20Architecture.md), [analysis tools](Security/Security%20Tools.md) |
| Networking | [Network services](Networking/Network%20Services.md), [printing and scanning](Networking/Printing.md), [self-hosted services](Networking/Self-Hosted%20Services.md) |
| Applications | [Application candidates](Applications/Application%20List.md), [terminal and CLI](Applications/Terminal%20and%20CLI.md), [browsers](Applications/Browser%20Configuration.md) |
| History | [Build log](Build%20Log/README.md), one file per day |
| Reference | [Research links](Reference/Research%20Links.md), [scratch notes](Scratch.md), [imported source notes](Reference/Imported%20Notes.md) |

Repeated source excerpts have been collected in the imported-notes archive so that the topic pages can be read without a second copy of each note. The archive preserves the wording needed to check how the material was combined.

## Status and license

These are pre-release working notes. The separate vulnerability-divergence research plan is intentionally excluded while unpublished. No license has been declared; treat the contents as all rights reserved until a license is added.
