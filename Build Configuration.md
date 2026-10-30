# Technicomp Benchtop Linux — Build Configuration

Technicomp Benchtop Linux is an immutable GNOME desktop image built with kiwi on the openSUSE Build Service. It is based on openSUSE Tumbleweed. The image is assembled against a tested Tumbleweed snapshot, and each package's source is held in a GitHub repository that the Build Service retrieves automatically through scmsync. Because the image is immutable, the entire package set must resolve as a single unit; when it does, the Build Service produces the image, and the running system is updated afterwards by transactional-update.

## Base distribution

The build uses openSUSE Tumbleweed through the repository `openSUSE:Factory/snapshot`. That repository is the openQA-tested Tumbleweed snapshot, and it provides every package as a real, built binary. A kiwi image build requires real binaries, which is the reason Tumbleweed is used rather than Slowroll.

Slowroll was evaluated first and rejected as a build base. On the Build Service, Slowroll's packages are provided through download-on-demand repositories (`openSUSE:Tumbleweed/slowroll` and `openSUSE:Tumbleweed/slowroll-next`), and image builds cannot install download-on-demand binaries. openSUSE produces its own Slowroll installation image only by first copying the required binaries into a staging project. Building against `openSUSE:Factory/snapshot` avoids that entirely, and it is the same base openSUSE's Aeon image uses, from which this image is derived.

## Source repositories

The build is composed of four public repositories under github.com/TechnicompLabs, each mapping to a single Build Service package.

- `benchtop-settings` builds `tc-benchtop-settings`, a configuration package whose files are installed directly from `Source0` onward, without a tarball. It also adds the TCBL package repository (`repo-tcbl`, at priority 90, above the openSUSE repositories) and ships the repository's signing key, which the image build imports. It also carries udev rules and programs copied unchanged from other projects for hardware support, which `upstream.list` records and `scripts/check-upstream` compares with upstream before every push, from a pre-push hook (see the repository's `UPSTREAM.md`), and the `brew` command that installs Homebrew for each user on first use.
- `benchtop-patterns` builds `patterns-tc-benchtop`, which produces the metapackages `patterns-tc-benchtop-base`, `patterns-tc-benchtop-admin` and `patterns-tc-benchtop-languages`. The image depends on the base metapackage. The other two are optional sets, which the installer offers and installs during the installation, or which can be installed later: remote administration and servers (Cockpit, the Samba and NFS servers, container image and infrastructure tools, database clients), and programming languages.
- `benchtop-branding` builds `tc-benchtop-branding`, the wallpaper, and `distribution-logos-tc-benchtop`, the Technicomp logos, which take the place of openSUSE's `distribution-logos-openSUSE-Tumbleweed` so that openSUSE's boot splash, login screen and icons show them.
- `benchtop-image` builds `tc-benchtop-image`, the kiwi image description. It is an oem, btrfs read-only-snapshot layout derived from Aeon.

Each package is connected to its repository by an scmsync entry in its `_meta`:

```
<scmsync>https://github.com/TechnicompLabs/<repo>#main</scmsync>
```

The image description uses `<source path="obsrepositories:/">` for its repository, so the packages come entirely from the repository paths configured in the project metadata. The base is therefore controlled by the project metadata, and `config.kiwi` contains no repository URLs.

## Build Service project structure

```
home:technicomp:benchtop            # repository openSUSE_Tumbleweed -> path openSUSE:Factory/snapshot
  tc-benchtop-settings
  patterns-tc-benchtop
  tc-benchtop-branding
home:technicomp:benchtop:images     # Type: kiwi; repository openSUSE_Tumbleweed
  tc-benchtop-image                 #   paths: home:technicomp:benchtop/openSUSE_Tumbleweed, openSUSE:Factory/snapshot
```

The image resides in its own subproject because a kiwi build requires `Type: kiwi` in the project configuration, and applying that setting to the base project would interfere with the ordinary RPM builds there.

## Provider preferences

Several capabilities that the package set requires can be satisfied by more than one package. The Build Service reports these as "have choice" and does not choose on its own. They are resolved in the project configuration of `home:technicomp:benchtop:images`:

```
Prefer: helm
Prefer: plymouth-branding-openSUSE
Prefer: openSUSE-release-appliance
Prefer: tik-config-generic
```

`helm` (Helm 4) is preferred over `helm3`, the Helm 3 series, which also provides `helm`. `plymouth-branding-openSUSE` selects the openSUSE boot-splash branding. `openSUSE-release-appliance` selects the appliance release flavor; it is an interim choice, and the image identifies as openSUSE Tumbleweed until a dedicated `tc-benchtop-release` package is created. `tik-config-generic` selects tik's generic configuration rather than Aeon's.

The logos need no preference line, although openSUSE:Factory's own configuration prefers `distribution-logos-openSUSE-Tumbleweed`: the pattern requires `tc-benchtop-branding`, which requires `distribution-logos-tc-benchtop` by name, and the Build Service resolves such single-provider requirements before it makes any choice, so the `distribution-logos` capability is already provided when a choice would arise.

## Automatic rebuilds

A push to any repository rebuilds its package immediately rather than waiting for the scmsync poll. This is configured with one Build Service workflow token and one organization-level GitHub webhook. The GitHub token requires only the `repo:status` scope, so that the Build Service can report build results back onto the commits.

```
osc token --create --operation workflow --scm-token <GITHUB_PAT>   # prints an id and a secret
```

A single webhook is added under TechnicompLabs -> Settings -> Webhooks, using that id and secret:

- Payload URL: `https://build.opensuse.org/trigger/workflow?id=<id>`
- Content type: `application/json`
- Secret: `<secret>`
- Event: push

If the GitHub token expires, pushes stop reaching the Build Service with no error shown on GitHub. The first one, a classic token named "OBS Workflow", expired on October 24, 2026, and none of the pushes from October 25 to 27 synced. Its replacement (October 27) has no expiration date and, like the first, only the `repo:status` scope. To replace a token: create a new classic token with only `repo:status`, create a new workflow token with it (above), put the new id and secret into the webhook, then delete the old workflow token (`osc token --delete <id>`) and the old GitHub token. Neither token is stored in any repository. When the webhook is not working, `osc service remoterun` (below) syncs a package by hand.

Each repository contains a `.obs/workflows.yml` naming its own project and package. The `trigger_services` step makes the Build Service retrieve the pushed commit before it builds; `rebuild_package` would only rebuild the source it already holds:

```yaml
rebuild_on_push:
  steps:
    - trigger_services:
        project: <project>
        package: <package>
  filters:
    event: push
    branches:
      only:
        - main
```

## Common commands

```
osc results home:technicomp:benchtop:images                                    # image status
osc buildinfo home:technicomp:benchtop:images tc-benchtop-image openSUSE_Tumbleweed x86_64 \
  2>&1 | grep -iE "nothing provides|unresolvable|have choice"                   # unmet dependencies or provider choices
osc getbinaries home:technicomp:benchtop:images tc-benchtop-image openSUSE_Tumbleweed x86_64   # download the built image
osc service remoterun home:technicomp:benchtop <package>                        # force scmsync to retrieve source again
osc rebuild <project> <package>                                                 # force a rebuild
osc cat <project> <package> <file>                                             # display the source the Build Service holds
```

## Notes on build behaviour

The image resolves against a single tested Tumbleweed snapshot, so the whole package set is internally consistent at build time. The update cadence is determined by which snapshot the build targets; it currently follows the latest tested Tumbleweed snapshot and can be slowed by pinning to a fixed snapshot.

When a package requires a capability that several packages provide, the build fails with "have choice" until a `Prefer` line selects one. This is expected, and is not a missing-package error.

## Current state

The image resolves cleanly against `openSUSE:Factory/snapshot` and builds. Remaining work, in rough order: a `tc-benchtop-release` package to give the system its own identity in place of `openSUSE-release-appliance`; a custom kernel; and implementing the GNOME old stable (n−1) policy. Pinning the whole Tumbleweed snapshot was noted as an interim option, but it also holds back the rest of the package set; it does not by itself implement a separately maintained GNOME branch.

### Custom kernel

#TODO — Hibernation with Secure Boot

- openSUSE's kernels lock themselves down whenever they boot with Secure Boot (a SUSE patch, `CONFIG_LOCK_DOWN_IN_EFI_SECURE_BOOT`), and a locked-down kernel refuses to hibernate. Upstream has no such trigger. Build the TCBL kernel without it, and sign it with TCBL's own key, enrolled once per machine through MOK.
- Module and kexec signatures stay enforced without lockdown. With Secure Boot on, the upstream IMA architecture policy (`CONFIG_IMA_ARCH_POLICY`, already on in openSUSE's config) requires signed modules and kexec images and refuses the older `kexec_load` call, which cannot be signature-checked. `CONFIG_STRICT_DEVMEM` and `CONFIG_IO_STRICT_DEVMEM` are on as well. The rest of what lockdown blocks, such as MSR writes, direct I/O port and PCI access, ACPI table overrides and full debugfs access, is allowed unless the setting below turns lockdown on.
- Setting toggle: hibernation (no lockdown, the default) or lockdown. The setting raises lockdown early in the initrd by writing `integrity` to `/sys/kernel/security/lockdown`, with no kernel command line flag. It has to run before systemd looks for a hibernation image, so that a locked-down boot never resumes one. Root can turn the setting off for the next boot, so unlike openSUSE's automatic lockdown it protects only the running kernel.
- Hibernation swap: a swap file sized to RAM, in its own Btrfs subvolume on the encrypted root (`btrfs filesystem mkswapfile`; a subvolume that holds a swap file cannot be snapshotted). zram stays the everyday swap at the higher priority; systemd ignores zram when it picks the swap to hibernate to.
- Resume: systemd stores the swap file's location in the `HibernateLocation` EFI variable and resumes from it in the initrd once the TPM has unlocked the disk, so no `resume=` flag is needed.
- TPM: the disk key is sealed to PCRs 4, 5, 7 and 9 (tik's `post/15-encrypt`), and shim records in PCR 7 the certificate that verified each image it checks. A kernel signed with TCBL's key instead of openSUSE's changes PCR 7, so the seal has to be updated when a system switches kernels. Check whether sdbootutil's prediction handles that; if not, that boot asks for the recovery key.
- Test on hardware that hibernation still succeeds when zram is well filled. See ZRAM Hibernate in [Memory Management](Performance/Memory%20Management.md#zram-hibernate).

#TODO — IOMMU defaults

- Keep upstream's `CONFIG_INTEL_IOMMU_DEFAULT_ON=y`, which openSUSE's config turns off, and then drop `intel_iommu=on` from the kernel command line in `config.sh`.
- Keep openSUSE's `CONFIG_IOMMU_DEFAULT_PASSTHROUGH=y` (upstream's x86 default is `CONFIG_IOMMU_DEFAULT_DMA_LAZY`). This is what `iommu=pt` sets: devices inside the machine get unrestricted DMA, and devices behind ports the firmware marks as external (Thunderbolt, USB4) are always translated.

#TODO — USB device emulation (dummy_hcd)

- `raw-gadget` (`CONFIG_USB_RAW_GADGET`) lets userspace emulate arbitrary USB devices. openSUSE builds it as a module, and it is already in the image (kernel-default), along with the configfs and legacy gadget modules (`g_hid`, `g_ether`, `g_mass_storage`, the `usb_f_*` functions, `gadgetfs`, `libcomposite`). There is no separate kernel-default-extra in Tumbleweed; kernel-default carries them.
- What is missing is a controller to bind to. A desktop or laptop has host ports, not a USB Device Controller, so raw-gadget and the configfs gadgets have nothing to attach to. The physical UDCs in the image (`dwc3`, `musb_hdrc`, `pch_udc`) only matter where the board firmware exposes a peripheral or OTG controller, which is uncommon on x86.
- `dummy_hcd` (`CONFIG_USB_DUMMY_HCD`) is the virtual controller that removes the hardware requirement: it emulates a host and a device controller in software on the same machine, so a gadget can be emulated and used locally. openSUSE does not build it at any level, so the custom kernel adds it as `CONFIG_USB_DUMMY_HCD=m`.
- Permissions and on-demand loading of `/dev/raw-gadget` are in [Drivers and Firmware](System%20Core/Drivers%20and%20Firmware.md#usb-device-emulation-raw-gadget).

#TODO — Firmware flashing (flashrom)

- `flashrom` is in the base set. It works now with external USB programmers; the internal programmer, the board's own SPI flash, is blocked by two kernel defaults.
- Kernel lockdown: openSUSE raises integrity lockdown under Secure Boot (`CONFIG_LOCK_DOWN_IN_EFI_SECURE_BOOT`, see Hibernation above), which blocks `/dev/mem`, `/dev/port` and `ioperm`/`iopl`. The internal programmer needs those, so on the current image flashrom cannot reach the onboard flash. Building the kernel without that trigger, which hibernation needs anyway, restores the access; `CONFIG_DEVMEM` and `CONFIG_DEVPORT` are already on.
- The spi-intel driver: `CONFIG_SPI_INTEL_PCI=m` binds the PCH flash controller at `00:1f.5` and routes flashrom through the kernel MTD device, which drops flash-descriptor regions, protected-range handling and write-protection control; with `CONFIG_IO_STRICT_DEVMEM=y` its MMIO is also closed to `/dev/mem`. Keep the module from auto-binding (a modprobe blacklist, in [Drivers and Firmware](System%20Core/Drivers%20and%20Firmware.md#firmware-flashing-flashrom)), or unbind it before flashing.
- `CONFIG_STRICT_DEVMEM=y` is on, but it still allows device (non-RAM) memory through `/dev/mem`, so it is not itself the blocker and the old `iomem=relaxed` workaround is not needed.

### Installer

The image is its own installer. tik, Aeon's installer, copies the built image to the target disk with systemd-repart, encrypts the root as it writes it, and enrolls the TPM and a recovery key. TCBL tik modules offer the optional package sets (admin, languages) during the installation and ask for the host name. Storage is limited to one whole disk.

TCBL changes to the installation, all in `config.sh`:

- Discard. systemd-repart's generated fstab puts a plain `discard` on every Btrfs entry, which is synchronous discard ([systemd issue 38829](https://github.com/systemd/systemd/issues/38829)); on the first installed test system it made the desktop stall, with services timing out and crashing. Each `MountPoint=` in `repart.d/50-root.conf` ends in `discard=async`, which Btrfs applies after repart's `discard`, as the issue suggests. With `--empty=force`, repart discards the whole disk and still has mkfs.btrfs discard the new root again (its discard-only-once check, systemd 260, counts only per-partition discards); the copy step sets `SYSTEMD_REPART_MKFS_OPTIONS_BTRFS=--nodiscard`, which is systemd-repart's documented way to pass options to mkfs.btrfs. Aeon uses the same tik, the same `50-root.conf` and the same systemd, so its installs probably get the same fstab; not confirmed on an Aeon system.
- Progress. A window follows tik's log for the whole installation, and the copy shows a percentage, the amount copied and the time taken, from the bytes mkfs.btrfs has read against the copy's size, which `config.sh` measures when the image is built. Upstream showed only "Deploying OS Image" for the whole copy.
- The live session. Its zypper and man caches and its journal are not copied (`ExcludeFiles=`), and its log files are removed from the installed system; accounts made with "Create User" and their homes are kept.
- SELinux. The copy carries the USB's labels, including what the live session created, and the optional sets are installed with the USB's policy loaded. The installer creates `/etc/selinux/.autorelabel`, so openSUSE's selinux-autorelabel (required by microos-tools) relabels the root snapshot, `/etc` and `/var` in the initrd at the first boot and the other file systems before `local-fs.target`. On the first installed test system, `/var/log/tuned` had the wrong type and tuned-ppd could not start.

#TODO — Move to Agama

- Decided October 28: TCBL will eventually move to Agama. tik stays until Agama is Tumbleweed's installer and has multi-device Btrfs; the points below are what the move has to solve.
- Agama, SUSE's installer for SLES 16 and Leap 16, was considered on October 25 and not adopted for now. A TCBL product definition would list the optional patterns as `user_patterns`, which Agama shows as checkboxes on its software page, in place of the TCBL tik modules. It also offers LVM, reuse of existing MD RAID, LUKS2 with TPM unlocking, unattended installation from a JSON profile, and installation from a browser on another machine.
- Against it, for now: Agama installs packages from repositories instead of copying the tested image, so each system is assembled at install time, and an offline install needs a package repository on the media. `config.sh` runs only in the KIWI image build, so its system configuration (os-release branding, snapper, services, keys, kernel command line) would have to move into packages or Agama post-install scripts. TCBL would maintain a branched Agama live image and its own product definition across Agama releases, whose formats have changed. No immutable desktop ships with Agama: Kalpa has a product definition, but its maintainer does not support installing it that way, and Aeon uses tik. Agama is not yet Tumbleweed's installer; Agama 25 (October 2026) gives no date.
- Move when Agama is Tumbleweed's installer and has multi-device Btrfs (`btrfsRaids`, planned and not implemented), ideally with RAID creation in its interface. Check which PCRs Agama seals to on systemd-boot; tik seals to 4, 5, 7 and 9.
- Whatever the installer: move the system configuration from `config.sh` into packages. Changes to `config.sh` never reach installed systems, while package changes do. This is also most of the work a switch to Agama would need.
- Started October 30. In packages now: NetworkManager, systemd-resolved and zram swap enabled by tc-benchtop-settings' presets (its post-transaction script applies them on a new install, and systemd at the first boot); zypper's single rpm transaction; the SELinux booleans `selinuxuser_execmod`, `execheap` and `execstack`, set with selinux-policy's `%selinux_set_booleans`; rtkit in the pattern. Deleted from `config.sh` because they restated upstream: enabling chronyd, irqbalance and fstrim.timer (openSUSE's presets), SELinux enforcing and targeted (what selinux-policy writes), `preempt=full` (the default of openSUSE's kernel) and the extra import of the repo-tcbl key; the wicked DHCP host-name setting did nothing with NetworkManager. chrony stays, as openSUSE's default.
- os-release: a link to openSUSE:Factory/000release-packages in home:technicomp:benchtop, building only openSUSE-release, with a patch that changes NAME and PRETTY_NAME (files in `_obs-meta/000release-packages`). The sed in `config.sh` goes once the link builds.
- Kernel command line: moved October 30 into tc-benchtop-settings as `/usr/kernel/cmdline.d/90-tcbl.conf`, which sdbootutil (since March 2026) adds to every boot entry, so changes reach installed systems. `config.sh` keeps only `ignition.platform.id`; `rcutree.enable_rcu_lazy=1` was dropped (no effect without `CONFIG_RCU_LAZY`). The vendor directory is undocumented, sits oddly beside `/usr/lib/kernel/cmdline`, and may be a mistake: issue 362 asked for `/usr/lib/kernel/cmdline.d`, and its discussion mentioned `/usr/etc/kernel/cmdline.d`. The package therefore also links the file into `/usr/lib/kernel/cmdline.d`, and its `%check` fails the build if sdbootutil reads neither directory; OBS rebuilds the package when sdbootutil changes. Systems installed before this have the options in `/etc/kernel/cmdline` too, so they appear twice until that file is trimmed. The kernel measures its command line into PCR 9, which the TPM seal includes, so keep a recovery key at hand when the command line changes.
#TODO — Remove the `/usr/kernel/cmdline.d` link from tc-benchtop-settings once Tumbleweed's sdbootutil includes the fix for [issue 467](https://github.com/openSUSE/sdbootutil/issues/467) ([pull request 468](https://github.com/openSUSE/sdbootutil/pull/468), merged October 30): the file itself is in `/usr/lib/kernel/cmdline.d`, and the package's `%check` fails the build when the link is no longer needed. Still open upstream: whether a vendor drop-in should count as a configured command line during installation (which matters for Agama).
- Also moved October 30: the name in os-release (the link builds in repo-tcbl); the image's zypper locks, replaced by tc-benchtop-settings conflicting with `flatpak-remote-flathub` (it adds Flathub system-wide and supplements flatpak) and nothing for `iwd` (nothing in Tumbleweed recommends or supplements it, and NetworkManager requires wpa_supplicant); `baseSetRunlevel graphical.target` deleted, since systemd's `default.target` already points to `graphical.target`. `config.sh` is down from 1,089 to 992 lines, almost all of the rest being the installer.
- Not solved for Agama: `tc-benchtop-settings-rpm`, whose `%_netsharedpath` keeps the terminal programs' launchers out, works only when it is installed before the other packages, as KIWI's bootstrap does. An installer that installs everything in one transaction would install the launchers, and rpm never removes them afterwards.
- Stay with the installer: snapper's limits, the console font, the build stamp, the machine-id reset and the ignition settings. Check what Agama writes for snapper and the console font.
- The TCBL tik changes under the move: the package-set modules become `user_patterns`, and the host name question becomes Agama's own setting (to confirm in its interface). The copy progress and the live-session cleanup do not apply, because Agama installs packages instead of copying the live system. Discard and SELinux have to be checked again: what mount options Agama writes to fstab, and whether its installed files get the target's labels.

#TODO — RAID at installation

- Btrfs RAID1 for the root: no openSUSE installer creates multi-device Btrfs. Agama creates MD RAID 0/1/5/6/10 from a profile but not from its interface, which only reuses existing arrays, and the EFI system partition cannot be on the array.
- RAIDZ for /home or data: no openSUSE installer supports ZFS. openSUSE leaves it out of the distribution, but builds and publishes it in the OBS `filesystems` project: `zfs`, `zfs-kmp-default`, `zfs-kmp-longterm`, and `zfs-ueficert`, which enrolls that project's Secure Boot certificate through MOK. ZFS's license (CDDL) permits redistribution. What is disputed is distributing the kernel module built against the GPL kernel: Canonical ships it prebuilt, the Software Freedom Conservancy and the FSF hold that it violates the GPL, and Debian ships only the source, built on the user's machine. Decided October 25: TCBL ships ZFS in repo-tcbl, as Ubuntu has for years. Pools are created after installation; see ZFS in repo-tcbl below.
- Btrfs RAID5/6 is not an alternative for data; the Btrfs documentation says not to use it in production.
- For data arrays now: Cockpit, in the admin pattern, creates MD RAID from its storage page after installation.

#TODO — ZFS in repo-tcbl

- Build `zfs` in home:technicomp:benchtop, linked from the `filesystems` project as nmap is from openSUSE:Factory:NonFree, so that the KMP is built against the kernel the image ships (and kernel-lts once it replaces kernel-default) rather than whatever the `filesystems` repository last built against.
- Secure Boot: OBS signs the module with the home:technicomp key, so each machine enrolls that certificate once through MOK, the TCBL counterpart of `zfs-ueficert`. The custom kernel needs TCBL's key enrolled anyway, so one enrollment can cover both.
- Kernel support: OpenZFS supports a range of kernel versions, and a kernel newer than it supports leaves the KMP unbuildable. Building in the same project shows that before anything is published, and the LTS kernel makes it rare.
- Which pattern carries it: an optional set offered in the installer, so it is installed only where wanted.
- Pools, including a RAIDZ /home, are created after installation with `zpool`; no installer creates them, and Cockpit has no ZFS support of its own.

#TODO — Maybe: a passphrase and a recovery-key check in the installer

- This records an idea, not a decision. In TPM mode tik generates a recovery key, shows it once and asks for nothing else; it asks for a passphrase only when it cannot use the TPM.
- A TCBL post-module could ask for a passphrase in TPM mode as well, entered twice, and add it with tik's temporary key file, which tik removes only after every post module has run. It could also have part of the recovery key typed back before continuing, so the dialog cannot be dismissed without recording the key. Either could be proposed to tik instead.
- On an installed system: `sdbootutil enroll --method=password --ask-pw` adds a passphrase, and `sdbootutil unenroll --method=recovery-key` followed by `sdbootutil enroll --method=recovery-key` replaces the recovery key. sdbootutil unlocks through the TPM to authorize both.

#TODO — Upstream reports after the next installation confirms the fixes

- systemd: add this case to issue 38829; a new issue for mkfs.btrfs discarding again after `--empty=force` has discarded the whole disk.
- Aeon: the same `50-root.conf`, with a check of an Aeon system's fstab first.
