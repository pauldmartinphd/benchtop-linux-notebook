# Storage and I/O

The scheduler and removable-drive writeback rules retain the September 16 corrections. `/tmp` remains on tmpfs, limited to 20% of RAM and cleaned after 3 days (October 30).

## I/O scheduler

BFQ is used for rotational disks and slow removable flash because it prioritizes fairness and interactive latency when individual I/O operations are expensive. Kyber is used for SSDs and NVMe because it provides low-overhead queue management suited to fast multiqueue storage. Since USB SSDs and flash drives both appear as `sd*` devices, non-rotational `sd*` devices default to Kyber, with removable USB media explicitly overridden back to BFQ.

CachyOS-Settings has its own scheduler rules (`60-ioschedulers.rules`, compared October 25): mq-deadline for SATA SSDs and SD/eMMC, kyber for NVMe, BFQ for hard disks. Benchtop's rules stay as they are; they also keep slow removable USB flash on BFQ. CachyOS's SATA link power management at max_performance (`50-sata.rules`) and its rule that keeps internal hard disks from spinning down (`69-hdparm.rules`) were not adopted either: both trade power, and the second also noise and wear, for latency, which is system policy rather than hardware support.

## /etc/udev/rules.d/70-iosched.rules

    # /etc/udev/rules.d/10-iosched.rules

	#
	# Slow storage
	#
	# BFQ prioritizes fairness and interactive latency and is well suited to
	# rotational disks and relatively slow flash media.
	#

	# Rotational disks: SATA/SAS/USB HDDs.
	ACTION=="add|change", SUBSYSTEM=="block", ENV{DEVTYPE}=="disk", \
	    ATTR{queue/rotational}=="1", \
	    ATTR{queue/scheduler}="bfq"

	# SD/eMMC storage.
	ACTION=="add|change", SUBSYSTEM=="block", ENV{DEVTYPE}=="disk", \
	    KERNEL=="mmcblk*", \
	    ATTR{queue/scheduler}="bfq"


	#
	# Fast solid-state storage
	#
	# Kyber is a low-overhead scheduler intended for fast multiqueue devices.
	#

	# NVMe SSDs.
	ACTION=="add|change", SUBSYSTEM=="block", ENV{DEVTYPE}=="disk", \
	    KERNEL=="nvme*n*", \
	    ATTR{queue/rotational}=="0", \
	    ATTR{queue/scheduler}="kyber"

	# Non-rotational SCSI-family devices. This includes SATA SSDs as well as
	# USB-attached SSDs and flash devices, so slow removable USB media is
	# overridden by the following rule.
	ACTION=="add|change", SUBSYSTEM=="block", ENV{DEVTYPE}=="disk", \
	    KERNEL=="sd*", \
	    ATTR{queue/rotational}=="0", \
	    ATTR{queue/scheduler}="kyber"

	# Slow/removable USB mass storage.
	#
	# This rule intentionally comes after the generic non-rotational sd* rule.
	# USB SSD/NVMe enclosures normally report removable=0 and retain Kyber;
	# removable USB flash media gets BFQ.
	ACTION=="add|change", SUBSYSTEM=="block", ENV{DEVTYPE}=="disk", \
	    KERNEL=="sd*", SUBSYSTEMS=="usb", ATTR{removable}=="1", \
	    ATTR{queue/scheduler}="bfq"
[Link](https://github.com/pop-os/default-settings/pull/149)

### USB Disks

USB flash drives can have extremely slow write performance, allowing a copy dialog to disappear while data is still being written from the kernel cache. Since a user may reach for and remove the drive within roughly a second of seeing the copy complete, slow removable media should have a small per-device dirty-data limit so the outstanding write tail is kept short. This should not affect USB SSDs or NVMe storage, which should retain normal high-performance writeback behavior.

#### /etc/udev/rules.d/71-removable-writeback.rules

	# Slow removable USB storage can accumulate a large amount of dirty data in
	# the page cache. Limit its per-device dirty budget so userspace cannot get
	# far ahead of the physical device.
	#
	# Do not change the global vm.dirty_* timers here; those would also affect
	# high-performance storage such as NVMe.

	ACTION=="add|change", SUBSYSTEM=="block", ENV{DEVTYPE}=="disk", \
	    KERNEL=="sd*", SUBSYSTEMS=="usb", ATTR{removable}=="1", \
	    ATTR{bdi/max_bytes}="8388608", \
	    ATTR{bdi/strict_limit}="1"

## Staggered Spin-Up

### /etc/modprobe.d/ahci.conf

	options libahci ignore_sss=1

"Some hardware implements staggered spin-up, which causes the OS to probe ATA interfaces serially, which can spin up the drives one-by-one and reduce the peak power usage. This slows down the boot speed, and on most consumer hardware provides no benefits at all since the drives will already spin-up immediately when the power is turned on."
[Arch Wiki](https://wiki.archlinux.org/title/Improving_performance/Boot_process)

Note: libahci is a module in the OpenSUSE kernel, so this goes in modprobe.d instead of the kernel command line.  The ahci driver reads the option when it loads either way, since modprobe also takes module options from the kernel command line.  Dracut copies modprobe.d into the initramfs

## TRIM Timer

    sudo systemctl enable --now fstrim.timer

Installed systems mount Btrfs with `discard=async`, the kernel's default for Btrfs since 6.2 on devices that support discard. systemd-repart's generated fstab puts a synchronous `discard` on every entry, so the installer's repart definition adds `discard=async` after it (see the Installer section of [Build Configuration](../Build%20Configuration.md#installer)). Synchronous discard trims every freed extent at once; the Btrfs documentation warns that it can have a severe performance hit, and on the first installed test system it did.

Note: Many USB-attached SSDs support TRIM/UNMAP via UAS (USB Attached SCSI). Ensure the device is detected as UAS rather than BOT (Bulk-Only Transport):

    lsusb -t   # Check for "Driver=uas"
Note: Some USB-SATA bridges do not pass through TRIM commands. Check with `lsblk --discard` — non-zero values in DISC-GRAN and DISC-MAX columns indicate TRIM support.

## /tmp on tmpfs

Keep `/tmp` on tmpfs. The earlier scratch proposal to disable it is superseded. tmpfs allocates memory as files are written and releases it when they are removed; the mount’s size limit is a ceiling, not a reservation. Its pages can also be swapped. See the [kernel tmpfs documentation](https://cdn.kernel.org/doc/html/latest/filesystems/tmpfs.html).

Tumbleweed already mounts `/tmp` as tmpfs: systemd's `tmp.mount` with `size=50%`, `nr_inodes=1m` and per-user quota (each user who logs in may fill 80% of it, since systemd 258). TCBL's root is a read-only snapshot with no `/tmp` subvolume, so `/tmp` cannot simply move to disk. TCBL's only swap is zram, so tmpfs pages that are swapped out still occupy (compressed) memory.

Decided October 30 (tc-benchtop-settings):

- Size: at most 20% of RAM (`tmp.mount.d/90-tcbl-size.conf`): 3.2 GB on the minimum 16 GB, 6.4 GB on the recommended 32 GB. Desktop use keeps `/tmp` small; what could outgrow it on 16 GB is mostly development work, such as large C/C++ builds with link-time optimization and pip building big Python packages from source (estimates, not measured on TCBL). GNOME's archive manager puts its temporary files wherever there is the most free space, so in practice in the home folder, and GNU sort only spills to `$TMPDIR` when the input exceeds its memory buffer (at least 1/8 of RAM).
- Cleanup: files in `/tmp` unused for 3 days are removed (openSUSE: 10 days), and in `/var/tmp` after 30 days (systemd's own default; openSUSE removes nothing there) (`tmpfiles.d/90-tcbl-tmp.conf`, applied daily by `systemd-tmpfiles-clean.timer`). Sockets in use are never removed. Each reboot also empties `/tmp`.
- When a job needs more: `TMPDIR=/var/tmp <command>` (on disk); `sudo mount -o remount,size=50% /tmp` for the current boot; or a drop-in in `/etc/systemd/system/tmp.mount.d/`.

## Access Times

Use `relatime` as the default mount option (this is already the kernel default since Linux 2.6.30). `relatime` only updates the access time (atime) if the previous atime is older than the modify or change time, or if the previous atime is more than 24 hours old. This reduces write overhead vs. the legacy `atime` behavior while still satisfying applications that depend on atime (e.g., tmpwatch, mutt).

For workloads that never need atime (e.g., build servers, databases), `noatime` eliminates the overhead entirely but can break some tools that depend on it. `relatime` is the safe default for a general-purpose desktop distro.

### /etc/fstab

    # Example: ensure relatime is set (should be default, but be explicit)
    UUID=xxx  /  btrfs  defaults,relatime,compress=zstd  0  0

## Optimal Encryption Sector Size

Modern drives with 4096-byte physical sectors should use a 4096-byte encryption sector size for LUKS to avoid read-modify-write overhead. Fedora 35+ defaults to 4096 for new LUKS volumes on drives that report 4K sectors:

    cryptsetup luksFormat --sector-size 4096 /dev/sdX

Check drive sector size:

    cat /sys/block/sdX/queue/physical_block_size
    cat /sys/block/sdX/queue/logical_block_size

NVMe drives almost universally use 4096-byte or 512-byte logical sectors. When logical=512 but physical=4096, aligning the encryption sector size to 4096 still provides a performance benefit.

Note: This only applies to new LUKS volumes. Existing volumes cannot be migrated without re-encryption.
