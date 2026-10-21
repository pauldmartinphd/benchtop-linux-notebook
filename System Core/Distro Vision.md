# Distro vision

Benchtop is intended to be a complete technical workstation: an immutable base with the drivers, firmware, development tools, and desktop integration needed for ordinary work already configured. The active build uses openSUSE Tumbleweed. The kernel and GNOME are intended to follow upstream-maintained releases more conservatively than the rest of the system: kernel.org LTS and GNOME old stable (n−1). [Build Configuration](../Build%20Configuration.md) records what is implemented and what remains to be built.

The base should be small enough to understand and maintain, without requiring users to assemble basic hardware support themselves. That includes GPU and Wi-Fi drivers, printer and scanner support, function keys, and proprietary drivers or firmware where needed. Remove obsolete dependencies such as GTK2 and Python 2 where the supported application set allows it. Prefer userspace implementations for legacy filesystems and file-sharing protocols.

## Why NixOS was rejected

KIWI already provides a declarative description of the system image, including its package selection and build configuration. NixOS offers additional control over dependencies and system configuration, but those benefits do not justify changing the distribution for Benchtop.

Compatibility with existing system-administration tools and practices is a substantial concern. Tools that assume conventional filesystem paths, package management, or directly editable system configuration can need adaptation under NixOS. Its Nix language and module system also introduce a configuration syntax and model far removed from conventional Linux administration. For Benchtop, the resulting integration and maintenance work outweighs the additional declarative benefits. The project will continue with Tumbleweed, OBS, and KIWI.

## Supported hardware baseline

The minimum design and test target is:

* CPU: x86-64-v3, 4 cores / 8 threads or better
* Memory: 16 GB minimum; 32 GB or more recommended
* Storage: 1 TB SSD minimum
* Display: 1920 × 1080 minimum
* Graphics: hardware acceleration suitable for the supported GNOME/Wayland stack

Lower-spec hardware may work, but it is outside the design and test target. The project does not aim to make this workstation environment suitable for very limited hardware.

## Applications and settings

The current direction is GNOME Software for application management, Flatpak and AppImage for GUI applications, and Homebrew for additional CLI tools. Bazaar is not part of the plan. GNOME Software needs an AppImage backend with AppImageHub integration; AppImages should install in `~/Applications/` with generated desktop launchers. The [packaged-environment note](Pre-Packaged%20Environment.md#appimage-backend-and-appimagehub) records the intended behavior. Nix and Home Manager are no longer part of the direction; the [rejected NixOS proposal](../NixOS-Based%20Distro%20Build%20Plan.md) is retained for reference. Chezmoi remains a candidate for dotfile synchronization. GNOME state and extension synchronization need their own design, including the earlier idea of a fork of Extension Sync.

Provide GUI administration for low-memory conditions, the firewall, and file sharing. The terminal should have Starship configured by default. Include Solaar, libratbag support, and the udev rules needed for supported mice, game controllers, and other peripherals.

## Intended work

The system should support machine learning with CUDA and ROCm, data science, software development, virtualization, system administration, gaming, and audio and multimedia work. The notes identify Git, Podman, Kubernetes, Podman Desktop, KVM/libvirt, virt-manager, Boxes, Incus, Cockpit, and Distrobox/Distrosheff as relevant tools. These use cases guide package selection; their presence here does not settle which applications belong in the base image.

Windows compatibility and Android through Waydroid are also intended capabilities. The [application list](../Applications/Application%20List.md) and [packaged environment](Pre-Packaged%20Environment.md) keep the candidate tools separate from the build description.

## Filesystem ideas

Several earlier ideas go beyond the active image: searchable text formats for personal data; tags represented through hard links and virtual folders; a separate `/home` subvolume or array; and a clearer filesystem hierarchy. The notes also consider two modes for home storage: cross-platform compatibility with restricted names and case-insensitive behavior, or a single-platform mode with tags, case-sensitive names, and fewer portability constraints. Whether all file metadata is needed, rather than creation and modification times alone, remains a question.

These are design explorations. The original list also proposed ZFS root with a low-latency kernel, keeping the kernel in the ESP, and no distributed root filesystem. The active image instead has a Btrfs read-only-snapshot layout. The [filesystem note](Filesystems.md) retains the alternatives without treating them as implemented decisions.

## References and research questions

Bluefin is a reference for providing upstream tools and keeping application workflows portable across distributions. Benchtop uses GNOME Software rather than adopting Bluefin’s Bazaar choice. See its [introduction](https://docs.projectbluefin.io/introduction), [developer environment](https://docs.projectbluefin.io/bluefin-dx), and [command-line tools](https://docs.projectbluefin.io/command-line).

Two questions in the original notes remain research hypotheses: whether rolling distributions deliver security fixes faster than distributions maintaining older releases, and whether language-native package managers such as pip and npm provide a better update path than distro-managed libraries. Neither should be presented as an established result of these design notes.
