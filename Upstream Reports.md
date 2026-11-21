# Upstream reports

Reports drafted for filing upstream, with where each goes and what to check first. Each moves to the build log, with its link, once filed. Before filing one, search the project's issues for a duplicate (done for 9). Reports decided against are listed at the end; their drafts are in this file's history (commit `f2a8d9e`).

Source references are to the versions read on November 2, 2026: systemd `main`, tik 1.5.1 (the version in Tumbleweed), and yast-samba-server `master` (5.0.2).

## 2. systemd: mkfs.btrfs discards again after `--empty=force` has discarded the whole disk

- Where: <https://github.com/systemd/systemd/issues> (bug report form; component systemd-repart).
- Status: the code path is confirmed in `main`; that it causes the pause is likely, not proven. TCBL's installer already passes `--nodiscard` to mkfs.btrfs; the install test that would show whether the pause is gone is still pending. Before filing: fill in the systemd version from the installer (`systemctl --version`), and add the result of the install test.

Title:

~~~
repart: with --empty=force, mkfs still discards partitions after the whole block device was discarded
~~~

Body:

~~~markdown
systemd version: [fill in]
Distribution: openSUSE Tumbleweed

With `--empty=force`, systemd-repart discards the entire block device (`context_discard_range(context, 0, context->total)`, logged as "Discarded entire block device."). `context_wipe_and_discard()` then skips the per-partition discards, because `context->from_scratch` is set, so `p->discarded` is never set for any partition, and `partition_mkfs_flags()` still adds `MKFS_DISCARD`. Each new file system is therefore discarded a second time by mkfs.

The "discard only once" change (#40579) covers only the case where `context_discard_partition()` did the discard.

Observed: tik (the installer of openSUSE Aeon) runs `systemd-repart --empty=force` with a LUKS2-encrypted Btrfs root that is populated with `CopyFiles=`. On an 8 TB WD_BLACK SN850X there is a pause of several minutes, with nothing read or written, before the copy starts. I believe the second discard (mkfs.btrfs discarding the whole new root through dm-crypt) is the cause, but I have not traced it. [Add: whether `SYSTEMD_REPART_MKFS_OPTIONS_BTRFS=--nodiscard` removes the pause.]

Suggested fix: when the whole-device discard succeeds, mark every new partition as discarded (set `p->discarded`), so that `partition_mkfs_flags()` leaves out `MKFS_DISCARD`.
~~~

## 5. openSUSE chrony: `-s` by default sets the clock from the hardware clock as local time

- Where: <https://bugzilla.opensuse.org>, product openSUSE Tumbleweed, package chrony; the change was made by Reinhard Max. Search for a duplicate first.
- Status: confirmed on the laptop (journal timestamps 5 hours ahead from chronyd's start until its first synchronization, on every boot). chrony behaves as documented; the report is about openSUSE's configuration. TCBL works around it with `rtconutc` and `hwclockfile /etc/adjtime` in `/usr/etc/chrony.d/90-tcbl-rtc.conf` (tc-benchtop-settings).
- Bugzilla search, November 21: no report of this. The change is from the Tumbleweed chrony 4.9-3 changelog (Reinhard Max, September 14, 2026): `OPTIONS="-s"` was made the default as part of the rework for bsc#1279495 and bsc#1145193 (chronyd and chrony-wait ordering). bsc#1279495 is still IN_PROGRESS, assigned to him. In 2020 (bsc#1173276) he had argued against `-s` by default because it also sets the clock from the RTC. File as a new bug referring to bsc#1279495, or as a comment there.

Title:

~~~
chrony: default OPTIONS="-s" sets the system clock from a UTC hardware clock as local time
~~~

Body:

~~~markdown
Since the change of September 14, 2026 ("Default OPTIONS to "-s" in the chronyd sysconfig to seed the clock from the RTC (or driftfile) at start-up"), chronyd sets the system clock from the hardware clock when it starts.

chrony reads the hardware clock as local time unless `rtconutc` or `hwclockfile` is configured (chrony.conf(5): "chronyd assumes by default that the RTC keeps local time"). openSUSE's chrony.conf has neither, while openSUSE systems keep the hardware clock in UTC (systemd's default; `timedatectl` shows "RTC in local TZ: no", and there is no /etc/adjtime).

Result: on every boot, chronyd sets the clock ahead or behind by the time zone's offset (5 hours in America/New_York), and the clock stays wrong until chronyd's first NTP step. In the meantime, journal entries carry the wrong time, snapper reports "time shift detected" for snapshots taken then, and anything that checks certificates or file ages sees the wrong time.

Suggested fix: add to openSUSE's chrony.conf (or a file in /usr/etc/chrony.d):

```
rtconutc
hwclockfile /etc/adjtime
```

`rtconutc` matches openSUSE's default UTC hardware clock; `hwclockfile` makes chronyd follow /etc/adjtime when it exists, which `timedatectl set-local-rtc 1` writes on computers that also run Windows, and it takes precedence over `rtconutc`.

chrony 4.9, openSUSE Tumbleweed [fill in the snapshot: `cat /etc/os-release`].
~~~

## 9. sdbootutil: a drop-in in both the snapshot and the running system is added twice

- Where: <https://github.com/openSUSE/sdbootutil/issues> (bug report form).
- Status: seen on silversurfer on November 15, 2026 (sdbootutil `1+git20261026.4e266f0`): after `sdbootutil update-all-entries`, the entries of snapshots 59 and 61 carried `90-tcbl.conf`'s options twice, the running snapshot's once. The same code is in `main` (`75bd88a`, November 21). Related, and not a duplicate: issue 411, fixed by pull request 460 (merged October 26), was the running snapshot reading the same directory twice; pull request 458, the alternative, stated that entries of other snapshots were left as before. Duplicate search done November 21.
- Fix prepared November 21: branch `fix-cmdline-d-duplicate` in Paul's sdbootutil checkout (commit `81775d3`, on `main` 75bd88a): the running system's drop-in is skipped when the snapshot has one with the same name, and a unit case, `t_kcf_dropins_once`, which fails without the change. Issue and pull request texts in `~/Projects/_upstream/`.
- Filed November 21 as [issue 483](https://github.com/openSUSE/sdbootutil/issues/483), with [pull request 484](https://github.com/openSUSE/sdbootutil/pull/484) (commit `8f8a6a5`): the running system's drop-ins are added only when the snapshot has no file of the same name; unit case `t_kcf_same_name`.

Title:

~~~
kernel_cmdline_files: a cmdline.d drop-in present in both a snapshot and the running system is added twice to that snapshot's entry
~~~

Body:

~~~markdown
When `kernel_cmdline_files` builds the command line for a snapshot that is not the running one, it lists the drop-ins of that snapshot and then those of the running system:

```
local usr_dropins=("${subvol:1}"/usr/lib/kernel/cmdline.d/*)
local etc_dropins=("${subvol:1}"/etc/kernel/cmdline.d/*)
if ! [ "${subvol:1}/" -ef / ]; then
	usr_dropins+=(/usr/lib/kernel/cmdline.d/*)
	etc_dropins+=(/etc/kernel/cmdline.d/*)
fi
```

A drop-in that exists in both, such as one shipped by a package installed before the snapshot was taken, is listed twice, and its options appear twice in the entry. #411 / #460 fixed the case where both paths are the same directory (the running snapshot); this is the case where they are different directories holding a file of the same name.

Steps:

1. Have a package install `/usr/lib/kernel/cmdline.d/90-example.conf` (for example containing `quiet`), and keep a snapper snapshot N, other than the running one, that contains it.
2. `sdbootutil update-all-entries`
3. The `options` line of snapshot N's entry has the file's options twice; the running snapshot's entry has them once.

Seen with `sdbootutil-1+git20261026.4e266f0` on Tumbleweed (which still reads the vendor files from `/usr/kernel/cmdline.d`), with a vendor drop-in of 10 options: the entries of the two snapshots that contained the file listed all 10 twice. The code is unchanged in `main`.

Expected: each drop-in name contributes once. One way: skip a running-system drop-in whose name was already listed from the snapshot, so that the snapshot's own file wins, in the same way an `/etc` file replaces the `/usr/lib` file of the same name.
~~~

## Not filed (decided November 20, 2026)

- 1. yast2-samba-server, `Supplements: samba`: YaST is being retired, and TCBL is not affected (its base pattern conflicts with `yast2`).
- 3. tik, end the session instead of powering off on a failure: specific to TCBL's image, which is also a live system; Aeon's installer image is not.
- 4. tik, the copy's progress and the installation log: a TCBL refinement of its own installer, and TCBL leaves tik for Agama.
- 6. tik, the recovery key under sdbootutil's current keyring names: a deprecation that sdbootutil's own developers will meet, and TCBL leaves tik for Agama.
- 7. Kvantum, a style key that follows the color scheme: a TCBL design choice, which the maintainer has said he dislikes; TCBL keeps the patch.
- 8. patterns-aeon, `Requires: malcontent-control`: it broke builds, so Aeon's maintainers would see it at once.
