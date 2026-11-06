# Future performance experiments

These tools and optimization ideas need evaluation; they are not selected defaults. The gaming-session settings are in [Gaming Mode](Gaming%20Mode.md).

## Performance Tools and Applications

* [GameMode](https://github.com/FeralInteractive/gamemode/) — Feral Interactive's on-demand game optimizer
* [system76-scheduler](https://github.com/pop-os/system76-scheduler) — Process priority scheduling. Packaged and on by default since November 5, with Benchtop Focus Priority for the focused window (benchtop-scheduler; see [Build Configuration](../Build%20Configuration.md))
* [MangoHUD](https://mangohud.com/) — Vulkan/OpenGL overlay for monitoring FPS, temps, etc.
* [LACT](https://github.com/ilya-zlobintsev/LACT) — Linux GPU Configuration Tool
* [CPU-X](https://thetumultuousunicornofdarkness.github.io/CPU-X/) — CPU/GPU/motherboard info
* [hardinfo2](https://github.com/hardinfo2/hardinfo2) — System profiler and benchmark

## Other Performance Applications

* [Preload](https://wiki.archlinux.org/title/Preload) — Adaptive readahead daemon
* ~~[Prelink](https://handwiki.org/wiki/Prelink)~~ — ELF prelinking (DEPRECATED: Prelink is no longer maintained and is incompatible with modern security features like ASLR and PIE. Do not use.)
* [Ananicy-cpp](https://gitlab.com/ananicy-cpp/ananicy-cpp) — Auto nice daemon (note: the original [Ananicy](https://github.com/Nefelim4ag/Ananicy) in Python is abandoned; use the C++ rewrite instead)
* [nohang](https://github.com/hakavlad/nohang) — OOM prevention daemon
* [auto-cpufreq](https://github.com/AdnanHodzic/auto-cpufreq) — Automatic CPU speed/power optimizer
* [bpftune](https://github.com/oracle/bpftune) — BPF-driven auto-tuning
* [zorin-exec-guard](https://github.com/ZorinOS/zorin-exec-guard) — Execution guard



## BOLT (Binary Optimization and Layout Tool)

[BOLT](https://github.com/llvm/llvm-project/tree/main/bolt) is a post-link binary optimizer from Meta (now part of LLVM). It uses hardware performance counters (perf data) to rearrange the code layout of already-compiled binaries for better instruction cache utilization and branch prediction. Unlike PGO (which requires recompilation), BOLT operates on final binaries.

Potential use: Profile the distro's critical binaries (systemd, GNOME Shell, Mesa, kernel) and optimize their layout. CachyOS uses a similar technique with AutoFDO for the kernel. The earlier notes cite 5–15% speedups, but contain no Benchtop measurement supporting that range.

Workflow:
1. Build the binary with relocations preserved (`-Wl,--emit-relocs`)
2. Collect perf data: `perf record -e cycles:u -j any,u -- <binary>`
3. Convert perf data: `perf2bolt -p perf.data -o perf.fdata <binary>`
4. Optimize: `llvm-bolt <binary> -o <binary>.bolt -data=perf.fdata -reorder-blocks=ext-tsp -reorder-functions=hfsort+`

## FEX-Emu (Fast x86 Emulation on AArch64)

[FEX-Emu](https://fex-emu.com/) is a usermode x86 and x86-64 emulator for AArch64 Linux. It allows running x86/x86-64 Linux binaries on ARM64 hardware (e.g., Apple Silicon via Asahi Linux, Qualcomm Snapdragon laptops, Ampere servers).

This is only relevant if the distro targets ARM64 hardware. FEX is comparable to Apple's Rosetta 2 but for Linux. It supports running Steam, Wine/Proton, and many native x86 Linux applications on ARM.

FEX runs x86/x86-64 applications on an AArch64 host. It is relevant only if Benchtop targets that host architecture.
