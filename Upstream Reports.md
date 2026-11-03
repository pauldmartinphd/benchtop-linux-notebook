# Upstream reports

Reports drafted for filing upstream, with where each goes and what to check first. Each moves to the build log, with its link, once filed. Before filing any of them, search the project's open issues for a duplicate; that search has not been done for these four.

Source references are to the versions read on November 2, 2026: systemd `main`, tik 1.5.1 (the version in Tumbleweed), and yast-samba-server `master` (5.0.2).

## 1. yast2-samba-server: `Supplements: samba`

- Where: <https://github.com/yast/yast-samba-server>, as an issue or as a pull request that changes the one line in `package/yast2-samba-server.spec`. openSUSE Bugzilla (component YaST2) is the alternative; which of the two the YaST team prefers for packaging changes has not been checked.
- Status: confirmed on the laptop (update of October 30). TCBL is not affected any more: the base pattern conflicts with `yast2`.

Title:

~~~
yast2-samba-server: "Supplements: samba" pulls YaST onto systems that do not use YaST
~~~

Body:

~~~markdown
On openSUSE Tumbleweed, a system that has `samba` installed but does not use YaST gets `yast2-samba-server` on the next update that installs recommended packages, because the spec has

```
# bnc #386473, recommend yast2-samba-server when installaing these packages
Supplements:    samba
```

`yast2-samba-server` requires `yast2`, and once `yast2` is installed, other YaST modules follow through their own supplements, such as `yast2-snapper`'s `Supplements: (snapper and yast2)`. On my system one update installed YaST's core, its libraries and about a dozen modules.

Expected: `yast2-samba-server` is added only where YaST is already installed.

Proposed change, the same form `yast2-snapper` uses for snapper:

```
Supplements:    (samba and yast2)
```

Systems that use YaST would still get the module when they install samba, which is what bnc#386473 asked for.

Version: yast2-samba-server 5.0.2 (Tumbleweed).
~~~

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

## 3. tik: end the session instead of powering off when the installation fails

- Where: <https://github.com/sysrich/tik/issues> (feature request).
- Status: TCBL does this now by replacing tik's `cleanup()` (`01-tcbl-exit` in tik-config-benchtop). Tested only by running the function with stubs; not yet in a real failed installation.

Title:

~~~
Option to return to the session instead of powering off when an installation fails or is cancelled
~~~

Body:

~~~markdown
When tik exits with an error, `cleanup()` in `/usr/bin/tik` shows "Installation Failed" and runs `systemctl poweroff --force`.

Our image (built from Aeon's layout) is also used as a live system: the user logs in to a session on the USB and starts the installer from there. Powering off on a failure or a cancel ends that session, along with anything else the user was doing in it. By the time `cleanup()` runs, `error()` has already shown the specific error, and a cancel has already been confirmed, so the second dialog adds little.

Request: a setting in tik's config, for example `TIK_FAIL_ACTION=poweroff|logout`, defaulting to the present behavior. With `logout`, `cleanup()` would:

- call `tik_cleanup_mounts`, so the target disk is released and the installation can be retried;
- copy `tik.log` to `/ignition` as it does now;
- end the user's session (`loginctl terminate-user`) instead of powering off.

We do this now by redefining `cleanup()` in a module, which works because the EXIT trap looks the function up when it fires, but it means keeping a copy of the function in step with tik's.
~~~

## 4. tik: progress of the copy, and the installation log

- Where: <https://github.com/sysrich/tik/issues> (feature request). The TCBL code is MIT-licensed, as tik is, and could be offered as a pull request instead, if the maintainer wants it.
- Status: TCBL does this now (`90-tcbl-repart-prep` and `copy-progress` in tik-config-benchtop). Not yet seen in a real installation since it moved into the package.

Title:

~~~
Show the copy's progress and the installation log instead of a pulsing "Deploying OS Image" bar
~~~

Body:

~~~markdown
During the self-deploy, `dump_image_repart_self()` pipes systemd-repart's output into `zenity --progress --pulsate` with the text "Deploying OS Image". On a large or slow disk the copy takes many minutes with no sign of progress, and repart's standard output goes to the progress dialog, which does not display it.

systemd-repart reports a percentage only for `CopyBlocks=`; for a root populated with `CopyFiles=`, as in Aeon's layout, mkfs.btrfs copies the files and repart reports no progress.

What we do now, as a possible approach:

- Progress: the image build records the size of the system to be copied; during the copy, a helper reads the `rchar` counter of the mkfs.btrfs process from `/proc/<pid>/io` and shows the percentage, the amount copied and the time taken.
- Details: repart's standard output goes to `tik.log`, and an optional window follows `tik.log` (`tail -F` into `zenity --text-info`) for the rest of the installation.

Either would be useful in tik itself; the progress estimate needs the size recorded when the image is built, which the image's build would have to provide.
~~~
