# Gaming Mode

**Status, October 25, 2026:** Resolved. THP is `always` system-wide ([memory management](Memory%20Management.md)), NTSYNC loads at every boot ([drivers and firmware](../System%20Core/Drivers%20and%20Firmware.md)), the pattern installs gamemode and steam-devices (udev rules for Steam controllers and VR hardware), tc-benchtop-settings carries game-devices-udev's controller rules (below), and MangoHud and gamescope come from Flathub. Not implemented: the per-session compaction toggle; the other tools below remain candidates in [future ideas](Future%20Ideas.md).

## Game Mode THP Settings

Transparent huge pages can reduce translation overhead by backing memory with larger pages. The available sizes depend on the architecture and kernel; modern kernels also support sizes below the traditional 2 MiB PMD-sized page on x86. THP policy is controlled through sysfs, while compaction also has sysctl controls. See the [kernel THP documentation](https://docs.kernel.org/admin-guide/mm/transhuge.html).

Proposal: disable proactive compaction for gaming sessions, restore to defaults when the session ends (some workloads such as databases can be negatively impacted by memory fragmentation). THP itself is already `always`, with `shmem_enabled` at `advise`, in the [memory management](Memory%20Management.md) settings.

Disable when gaming, restore when game ends:

    echo 0 | sudo tee /proc/sys/vm/compaction_proactiveness
    echo 0 | sudo tee /sys/kernel/mm/transparent_hugepage/khugepaged/defrag

See [Steam Deck Utilities](https://github.com/CryoByte33/steam-deck-utilities/blob/main/docs/tweak-explanation.md)
See [THP Gaming Performance](https://blog.patshead.com/2023/02/enabling-transparent-hugepages-can-provide-huge-gaming-performance-improvements.html)
See [Measuring THP Impact](https://alexandrnikitin.github.io/blog/transparent-hugepages-measuring-the-performance-impact/)

## Performance Tools and Applications

* [GameMode](https://github.com/FeralInteractive/gamemode/) — Feral Interactive's on-demand game optimizer
* [system76-scheduler](https://github.com/pop-os/system76-scheduler) — Process priority scheduling
* [MangoHUD](https://mangohud.com/) — Vulkan/OpenGL overlay for monitoring FPS, temps, etc.
* [LACT](https://github.com/ilya-zlobintsev/LACT) — Linux GPU Configuration Tool
* [CPU-X](https://thetumultuousunicornofdarkness.github.io/CPU-X/) — CPU/GPU/motherboard info
* [hardinfo2](https://github.com/hardinfo2/hardinfo2) — System profiler and benchmark

## Udev Rules for Game Controllers

tc-benchtop-settings carries all of game-devices-udev's rules, installed as `71-*`, for controllers, arcade sticks, flight and racing hardware and VR headsets, and two of Universal Blue's: the HORIPAD STEAM over its wireless receiver and Bluetooth, which game-devices-udev does not cover, and Wooting keyboards. They give the device to the user at the seat (`uaccess`). steam-devices covers some of the same controllers with the same access, which is harmless. The Universal Blue rules left out, and why, are in the settings repository's [UPSTREAM.md](https://github.com/TechnicompLabs/benchtop-settings/blob/main/UPSTREAM.md), which `scripts/check-upstream` compares with upstream.

* https://codeberg.org/fabiscafe/game-devices-udev
* https://github.com/ublue-os/packages/tree/main/packages/ublue-os-udev-rules/src/udev-rules.d

## Other experiments

The [future-ideas note](Future%20Ideas.md) holds the additional tuning daemons, BOLT workflow, and FEX-Emu evaluation. These are not part of the gaming-session configuration above.
