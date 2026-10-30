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

## Easy Anti-Cheat v2

Epic's Easy Anti-Cheat v2 for Linux, which Fortnite on Linux will use, has the requirements below. Epic asks distributions to contact it about supporting them, and suggests a separate boot entry with these settings for protected games. No release date was given (October 2026). Sources: [OMG! Ubuntu](https://www.omgubuntu.co.uk/2026/10/epic-confirm-fortnite-linux), [Epic's EAC v2 announcement](https://onlineservices.epicgames.com/news/introducing-easy-anti-cheat-linux-v2) (not read directly).

| Requirement | TCBL |
|---|---|
| Secure Boot on | Supported |
| TPM 2.0 present | Depends on the machine; tik uses it when present |
| IOMMU enabled in the firmware | A firmware setting on each machine |
| BPF LSM loaded | openSUSE builds it in and lists `bpf` in `CONFIG_LSM`. TCBL's `security=selinux` is believed not to turn it off; not checked on an installed system |
| kexec disabled | openSUSE builds kexec in (`CONFIG_KEXEC=y`); under Secure Boot only signed images load. Whether Epic counts that as disabled is not known. `kernel.kexec_load_disabled=1` turns it off until the next boot |
| Kernel signed with a well-known key | openSUSE's kernel-default is signed with openSUSE's key; whether Epic accepts it is not known (the article names Canonical's). The planned TCBL kernel, signed with TCBL's own key, probably would not qualify |
| Added modules signed by a reputable party | openSUSE's modules are signed with openSUSE's key. ZFS from repo-tcbl is signed with the home:technicomp key, so a system using it probably fails this, and /home on ZFS would need ZFS in the gaming entry too |

The last two rows are inferences from the requirements as reported, not from Epic's checks.

#TODO — Easy Anti-Cheat: contact Epic as a distribution; check `cat /sys/kernel/security/lsm` on an installed system; decide on `kernel.kexec_load_disabled=1`, on every boot or only in the gaming entry; plan the gaming entry (openSUSE's kernel-default beside the TCBL kernel, with openSUSE-signed modules only) and how it works with ZFS.

## Arch Wiki gaming tweaks (considered, not adopted)

**AI-generated analysis, not independently corroborated.**

Checked October 29 against the Arch Wiki's Gaming page (sound, memory, network, PCI and scheduler sections), the kernel and upstream sources, and TCBL's settings. Nothing below is adopted except where TCBL already has an equivalent.

### Sound

- OpenAL HRTF: openSUSE's openal-soft (1.25.2) ships its default HRTF data set, and its default (`auto`) uses HRTF on stereo output that is detected as headphones. The PipeWire backend detects headphones from the device's form factor (`headphones` or `headset`), which USB and Bluetooth headsets report; headphones in a laptop's jack are probably not detected, because that backend does not look at the jack (inference from the backend's code). Per user, `stereo-encoding = hrtf` under `[general]` in `~/.config/alsoft.conf` forces it; the wiki's `hrtf = true` is deprecated, and `~/.alsoftrc` is still read. No system-wide setting, which would apply HRTF to speakers too. It affects only native games that use OpenAL, not Proton games. Which configuration file Flatpak Steam's runtime reads is not checked.
- Source engine `dsp_slow_cpu` and related settings are in-game console settings, not distribution configuration.
- PulseAudio realtime priority, resampling and buffer settings do not apply to PipeWire. PipeWire's `libpipewire-module-rt` already runs it at nice -11 with realtime priority through RTKit, which TCBL enables, and the audio group has rtprio 99 and nice -11 ([audio](../System%20Core/Audio.md)). PipeWire resamples with its own resampler, not speex, at `resample.quality` 4 (0 to 14); a higher value costs CPU, with no measurement of an audible gain. Buffering is PipeWire's quantum.

### Memory

| Setting | Arch Wiki | TCBL | Reason |
|---|---|---|---|
| `vm.swappiness` | 10 | 180 | Arch assumes swap on disk; TCBL swaps to zram ([memory management](Memory%20Management.md)) |
| `vm.watermark_boost_factor` | 1 | 0 | The kernel documentation says a value below a pageblock still reclaims a pageblock (2 MiB) per fragmentation event; 0 turns boosting off |
| `vm.watermark_scale_factor` | 500 | 125 | 500 keeps 5% of memory between the watermarks, so kswapd reclaims much earlier |
| `vm.min_free_kbytes` | 1048576 | kernel's value | Writing it stops the kernel adjusting it, including the increase for THP; 1 GiB is 12.5% of an 8 GB machine |
| `vm.zone_reclaim_mode` | 0 | kernel default | Already the kernel default |
| `vm.compaction_proactiveness` | 0 | 20 (default) | Proposed above for gaming sessions only |
| `vm.page_lock_unfairness` | 1 | 5 (default) | From the Steam Deck tweaks, whose author says it can have negative side effects; no measurement. A candidate for the gaming session only |

### Network

TCBL already sets `net.core.default_qdisc = fq_codel` ([network tuning](Network%20Tuning.md)); openSUSE's kernel would otherwise use pfifo_fast (`CONFIG_NET_SCH_DEFAULT` is not set). cake on a computer does not shape traffic unless given a bandwidth below the link's. Bufferbloat is almost always in the router or modem, which is where it has to be fixed.

### PCI latency timers

CachyOS's `pci-latency` lowers the latency timer of every device except sound cards. The timer applies only to conventional PCI devices; PCI Express devices hardwire it to 0 (as understood from the PCIe specification, not checked against the text), so on current machines it changes almost nothing. The wiki notes it conflicts with the professional-audio advice. `sudo lspci -vv | grep -i latency` shows the values on a machine.

### Scheduler

None of the six settings works as written on TCBL:

- `kernel.sched_child_runs_first` is not in the current kernel source.
- `kernel.sched_autogroup_enabled` does not exist: openSUSE builds the kernel without `CONFIG_SCHED_AUTOGROUP`.
- `kernel.sched_cfs_bandwidth_slice_us` (default 5000) affects only cgroups with a CPU quota.
- `base_slice_ns`, `migration_cost_ns` and `nr_migrate` are in debugfs, which is not a stable interface, and writes there fail under the lockdown that openSUSE's kernel raises with Secure Boot. `migration_cost_ns` 500000 is already the default. `base_slice_ns` defaults to 0.7 ms times 1 + log2 of the CPU count (capped at 8 CPUs), so 2.8 ms on 8 or more CPUs; 3 ms is about the same, and a longer slice favors throughput, not latency. `nr_migrate` 8 is the realtime kernel's value; the default is 32.
- The wiki's references describe CFS; the scheduler has been EEVDF since kernel 6.6.

Sources: [Arch Wiki, Gaming](https://wiki.archlinux.org/title/Gaming), [OpenAL Soft alsoftrc.sample](https://github.com/kcat/openal-soft/blob/master/alsoftrc.sample), [kernel vm sysctls](https://docs.kernel.org/admin-guide/sysctl/vm.html), [kernel/sched/fair.c](https://github.com/torvalds/linux/blob/master/kernel/sched/fair.c), [PipeWire pipewire.conf](https://gitlab.freedesktop.org/pipewire/pipewire/-/blob/master/src/daemon/pipewire.conf.in), [CachyOS pci-latency](https://github.com/CachyOS/CachyOS-Settings/blob/master/usr/bin/pci-latency).

## Other experiments

The [future-ideas note](Future%20Ideas.md) holds the additional tuning daemons, BOLT workflow, and FEX-Emu evaluation. These are not part of the gaming-session configuration above.
