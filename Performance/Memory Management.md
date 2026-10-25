# Memory management

These settings retain the corrections recorded on September 16. The audio note uses the same zram and memory-locking policy.

## Virtual memory

### /etc/sysctl.d/30-vm-default-settings.conf

    vm.swappiness = 180
    vm.watermark_boost_factor = 0
    vm.watermark_scale_factor = 125
    vm.dirty_bytes = 268435456
    vm.dirty_background_bytes = 134217728
    vm.max_map_count = 2147483642
	vm.page-cluster = 0
[Link](https://github.com/pop-os/default-settings/issues/111), [Link](https://github.com/pop-os/default-settings/pull/172)

The selected `vm.swappiness=180` assumes zram-backed swap. The kernel accepts 0–200 independently of whether zram is present; the value expresses the relative cost of swap and filesystem paging. Values above 100 can make sense when swap I/O is cheaper. `vm.page-cluster=0` disables swap readahead, which is unnecessary for zram. See the [kernel VM documentation](https://kernel.org/doc/html/latest/admin-guide/sysctl/vm.html).

Explanation of key settings:

* vm.watermark_boost_factor=0 — disables watermark boosting which can cause unnecessary reclaim on desktop
* vm.watermark_scale_factor=125 — increases the gap between low and high watermarks to reduce reclaim stalls
* vm.dirty_bytes=268435456 (256MB) — limits dirty page cache before synchronous writeback starts
* vm.dirty_background_bytes=134217728 (128MB) — starts background writeback at 128MB of dirty pages
* vm.max_map_count=2147483642 — maximum number of memory map areas per process; required for some games and applications (e.g., Proton/Wine, Elasticsearch)

#### Note on swappiness for audio workloads:
The older audio-system check recommended `vm.swappiness=10`. That recommendation concerned avoiding disk-backed swap; it does not replace the selected zram setting. In this configuration, vm.swappiness=180 assumes zram-backed swap, where swap I/O is substantially cheaper than filesystem paging. This differs from traditional realtime-audio recommendations such as `swappiness=10`, which are intended to avoid latency from disk-backed swap. Realtime audio applications should lock latency-critical memory rather than relying on low swappiness to prevent paging.
See https://wiki.linuxaudio.org/wiki/system_configuration#sysctlconf

### Transparent Hugepages

###### /etc/tmpfiles.d/30-thp.conf

	# Write Transparent Huge Pages policy at boot
	# Format: type path mode user group age argument
	w! /sys/kernel/mm/transparent_hugepage/enabled       - - - - always
	# Improve performance for applications that use tcmalloc
	# https://github.com/google/tcmalloc/blob/master/docs/tuning.md#system-level-optimizations
	w! /sys/kernel/mm/transparent_hugepage/defrag        - - - - defer+madvise
	w! /sys/kernel/mm/transparent_hugepage/shmem_enabled - - - - advise

###### /etc/tmpfiles.d/30-thp-shrinker.conf

	# THP Shrinker has been added in the 6.12 Kernel
	# Default Value is 511
	# THP=always policy vastly overprovisions THPs in sparsely accessed memory areas, resulting in excessive memory pressure and premature OOM killing
	# 409 means that any THP that has more than 409 out of 512 (80%) zero filled pages will be split.
	# This reduces the memory usage, when THP=always used and the memory usage goes down to around the same usage as when madvise is used, while still providing an equal performance improvement
	w! /sys/kernel/mm/transparent_hugepage/khugepaged/max_ptes_none - - - - 409
[Link](https://github.com/CachyOS/CachyOS-Settings/blob/master/usr/lib/tmpfiles.d/thp-shrinker.conf)

Note: there is a lot of debate about these settings. Depending on the workload it can help performance, hurt performance (on memory pressure) or make no difference. `always` is opt-out and `madvise` is opt-in. The selected desktop policy is `always`, with the shrinker splitting huge pages that are mostly zeros. The [gaming proposal](Gaming%20Mode.md) disables proactive compaction during a session and restores the previous settings afterward; its benefit still needs measurement on the target workload.

###### 2024-09-07 Note: I think maybe we want 2MB THP to minimize TLB use
See also: https://www.phoronix.com/news/Glibc-malloc-2MB-THP-AArch64

[Kernel Docs](https://www.kernel.org/doc/html/latest/admin-guide/mm/transhuge.html) [Arch Reddit](https://www.reddit.com/r/archlinux/comments/1atueo0/higher_ram_usage_since_kernel_67_and_the_solution/) [Steam Deck](https://github.com/CryoByte33/steam-deck-utilities/blob/main/docs/tweak-explanation.md) [Nelhage](https://blog.nelhage.com/post/transparent-hugepages/) [Evan Jones](https://www.evanjones.ca/hugepages-are-a-good-idea.html) [StackOverflow](https://stackoverflow.com/questions/11543748/why-is-the-page-size-of-linux-x86-4-kb-how-is-that-calculated/50033983#50033983)

### ZRAM Swap (Memory Compression)

	sudo systemctl enable --now zramswap.service

### Default Settings
Recorded settings for systemd-zram-service:

	#. Amount of memory to use for zram, from 1 to 200.
    PORTION=100
    #. Default to compress with zstd, which has an average compression ratio of 3.37.
    ALGO=zstd
    # Higher values encourage the kernel to be more eager to move pages to swap.
    SWAPPINESS=180

These values describe what TCBL gets, but systemd-zram-service reads no settings file, so they are not settings it takes. Its `zramswapon` script fixes them: one device, zram0, the size of MemTotal, compressed with zstd and switched on at swap priority 100. It leaves `vm.swappiness` alone; the 180 comes from `90-tcbl-vm.conf` in tc-benchtop-settings. The image enables `zramswap.service` in `config.sh`, so the command above is not needed on an installed system (checked against systemd-zram-service 0.2.1, October 25).

CachyOS-Settings sets `vm.swappiness` to 150 when zram0 starts and switches zswap off at the same time (`30-zram.rules`, compared October 25). It is not adopted, because it would override the 180 above. Its reason for switching zswap off with zram, that zswap can keep pages from reaching zram and from zramctl's counts, is input for the zswap question.

### ZRAM Hibernate

#TODO — With the custom kernel, zram stays the everyday swap and a swap file on disk holds the hibernation image. See [Build Configuration](../Build%20Configuration.md#custom-kernel)

https://github.com/gissf1/zram-hibernate/issues

### Compressed RAM Devices (CRAM)

#TODO — Track upstream. Meta (Gregory Price) has proposed kernel support for compressed RAM: memory devices with inline hardware compression, today CXL memory expanders, which report more capacity than they physically have. The kernel demotes cold pages to the device, keeps them mapped but write-protected, and moves a page back to regular RAM when it is written, so the compression ratio cannot run away. It is not a replacement for zram or zswap, and it needs CXL hardware, which desktops and laptops do not have

**Status, October 6, 2026:** Not in mainline (7.3-rc6 has no mm/cram.c). The base series, Private Memory NUMA Nodes, is at v5 (July 2026), with CRAM split out to be submitted separately. Talk at Linux Plumbers on October 5, 2026

[LPC 2026 talk](https://lpc.events/event/20/contributions/2424/) [RFC v4 mm/cram patch](https://lkml.iu.edu/2602.2/06848.html) [v5 thread](https://lkml.iu.edu/2607.2/15194.html) [LWN](https://lwn.net/Articles/1053508/) [Reddit](https://www.reddit.com/r/linux/comments/1wyz82o/meta_developing_compressed_ram_cram_for_linux/)

## Resource Limits

### /etc/security/limits.d/20-audio.conf

	# Realtime scheduling privileges for members of the audio group.
	@audio   -   rtprio   99
	@audio   -   nice    -11

### /etc/security/limits.d/30-memlock.conf

	# Allow processes to lock memory without an RLIMIT_MEMLOCK ceiling.
	*   soft   memlock   unlimited
	*   hard   memlock   unlimited

## MGLRU

##### /etc/tmpfiles.d/30-mglru.conf

	# Enable Multi-Gen LRU at boot and set a mild thrash guard
	w! /sys/kernel/mm/lru_gen/enabled    - - - - y
	w! /sys/kernel/mm/lru_gen/min_ttl_ms - - - - 1000

## OOM Killer

    systemctl enable --now systemd-oomd.service
