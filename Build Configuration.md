# Technicomp Benchtop Linux — Build Configuration

Technicomp Benchtop Linux is an immutable GNOME desktop image built with kiwi on the openSUSE Build Service. It is based on openSUSE Tumbleweed. The image is assembled against a tested Tumbleweed snapshot, and each package's source is held in a GitHub repository that the Build Service retrieves automatically through scmsync. Because the image is immutable, the entire package set must resolve as a single unit; when it does, the Build Service produces the image, and the running system is updated afterwards by transactional-update.

## Base distribution

The build uses openSUSE Tumbleweed through the repository `openSUSE:Factory/snapshot`. That repository is the openQA-tested Tumbleweed snapshot, and it provides every package as a real, built binary. A kiwi image build requires real binaries, which is the reason Tumbleweed is used rather than Slowroll.

Slowroll was evaluated first and rejected as a build base. On the Build Service, Slowroll's packages are provided through download-on-demand repositories (`openSUSE:Tumbleweed/slowroll` and `openSUSE:Tumbleweed/slowroll-next`), and image builds cannot install download-on-demand binaries. openSUSE produces its own Slowroll installation image only by first copying the required binaries into a staging project. Building against `openSUSE:Factory/snapshot` avoids that entirely, and it is the same base openSUSE's Aeon image uses, from which this image is derived.

## Source repositories

The build is composed of four public repositories under github.com/TechnicompLabs, each mapping to a single Build Service package.

- `benchtop-settings` builds `tc-benchtop-settings`, a configuration package whose files are installed directly from `Source0` onward, without a tarball. It also adds the TCBL package repository (`repo-tcbl`, at priority 90, above the openSUSE repositories) and ships the repository's signing key, which the image build imports.
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
