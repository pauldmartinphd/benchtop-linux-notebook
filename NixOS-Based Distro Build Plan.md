# NixOS build plan — not adopted

**Status, September 26, 2026:** Nix/NixOS is no longer part of the Benchtop direction. The active system is based on openSUSE Tumbleweed, with Flatpak and Homebrew for applications and additional tools. This document is retained as the record of an earlier proposal, not a future implementation plan. Its schedules, version claims, recommendations, and verification ledger are historical. In particular, its x86-64-v1 baseline is superseded: the supported baseline is now x86-64-v3 with 4 cores / 8 threads, 16 GB RAM (32 GB or more recommended), a 1 TB SSD, a 1920 × 1080 display, and hardware acceleration suitable for the supported GNOME/Wayland stack. See [Build Configuration](Build%20Configuration.md) for the active build.

The [distro vision](System%20Core/Distro%20Vision.md#why-nixos-was-rejected) records the reason for rejecting NixOS: KIWI already meets the declarative image-building need, while compatibility costs for existing administration tools and the unfamiliar configuration language add work that Benchtop does not need.

## Original proposal

**Prepared:** Wednesday, July 22, 2026 · rev. 1 · **Coauthors:** Paul Martin
**Decisions ratified today:** base = nixos-unstable, flake-pinned · kernel = upstream kernel.org LTS (6.18 line) · desktop = GNOME old-stable (n−1), self-maintained overlay · root FS = Btrfs on LUKS2 with RAID1 support; ZFS for arrays; these are the only two officially supported filesystems · first milestone = workstation **and** installer ISO in parallel, from the same flake

Cross-references: *Patching Research Plan.md* (which already cites this document by name), *System Core/Distro Vision.md*, and the per-domain notes cited throughout §5. Facts asserted in this plan were verified live today unless explicitly marked UNVERIFIED; the ledger is §10.

> **Earlier status (2026-08-15; superseded):** This plan is now the **long-term architecture target**, not the active build track. Per the 2026-08-15 decision ("Slowroll now, Maybe NixOS later"), v1 ships as an openSUSE derivative — **Technicomp Benchtop Linux**, since built on Tumbleweed rather than Slowroll (see *Build Configuration.md*) — retaining both boring-core pillars (upstream kernel.org LTS kernel; GNOME old-stable) via first-party OBS projects. Plan of record: `tc-benchtop-linux/docs/build-plan.md` (its §8 relates the two tracks). Re-evaluation checkpoint with real maintenance-cost data: M4, December 2026. Nothing below has been rewritten; the §7 phase dates are stale.

---

## 0. Summary

The distro is the architecture the research project bets on, made real: a **rolling userland** (nixpkgs unstable, on the theory that rolling patches security bugs faster — RQ1) combined with a **boring core** — an upstream-maintained kernel.org LTS kernel and an upstream-maintained GNOME old-stable branch, both tracked *verbatim* rather than soft-forked (the RQ2 model). Every stability-critical component follows an upstream security-fix branch; everything else rolls. No bespoke patch fork is maintained anywhere in the system, which is precisely the policy RQ2 evaluates against the enterprise status quo.

The base moves from openSUSE MicroOS to NixOS for four reasons. First, NixOS-unstable is a true rolling distribution with a mature update pipeline, satisfying the "fresh userland" half of the thesis. Second, Nix's model — every package carries its own dependency closure — is the only mainstream mechanism that lets an old-stable GNOME coexist with a fully rolling system without freezing shared libraries for everyone else; on a conventional distro, pinning GNOME n−1 drags glibc-adjacent compromises with it, while on NixOS the pin is surgical. Third, the entire OS is one declarative repository: the same flake emits Paul's workstation configuration and the installable ISO, which is what makes "both in parallel" cheap. Fourth — a research dividend — a flake-pinned system has *perfect* per-day state reconstruction (the git history of `flake.lock` is a better historical record than snapshot.debian.org), making the distro itself a clean measurement subject later.

MicroOS's headline features are not lost: NixOS generations give atomic upgrades and boot-menu rollback (transactional-update's job), the Nix store is immutable by construction, and configuration drift is impossible by design. What is lost is OBS and zypper-world tooling, which §1 maps to replacements.

A structural note discovered during today's verification, relevant to both this plan and the research: **NixOS now has an official per-CVE security tracker** (tracker.security.nixos.org, `NIXPKGS-YYYY-NNNN` identifiers, triage workflow). The Patching Research Plan §6 assumed this did not exist and scoped NixOS as a sidebar "unless wrong — then promote." It is wrong as of 2026; see §6 below.

---

## 1. What the move from MicroOS to NixOS changes

The notes accumulated under an OpenSUSE assumption. The concept map:

| MicroOS concept (in notes) | NixOS replacement |
|---|---|
| transactional-update + snapper rollback | `nixos-rebuild boot/switch`; generations in the boot menu; `nixos-rebuild --rollback`. Btrfs snapshots remain useful for **/home** only (system state is already versioned by generations) |
| transactional-update.timer + notify (Boot and Init.md) | `system.autoUpgrade` with `flake` + `allowReboot=false`; notify hook for "reboot pending" state |
| OBS custom repo (NVIDIA, binder, ZFS module, 1000 Hz kernel, system76-scheduler, adw-gtk3, pipewire codecs, dislocker) | The flake's own overlay + CI binary cache (Cachix or self-hosted). NVIDIA/ZFS/binder need no custom packaging at all on NixOS (§5.1); the rest are nixpkgs attrs or small overlay packages |
| zypper/Packman (codecs, `libavcodec-full`) | nixpkgs with `allowUnfree`; full ffmpeg/gstreamer without a third-party repo. Flatpak apps additionally carry their own codecs via the Freedesktop runtime |
| `sdbootutil update-all-entries` after cmdline edits | Nothing — `boot.kernelParams` is declarative; rebuild regenerates entries |
| zypper orphan cleanup (`solver.cleandepsOnRemove`) | `nix-collect-garbage -d` + `nix.gc.automatic`; orphans cannot exist in the running system by construction |
| OpenSUSE sudo `targetpw` + polkit wheel fixes (Security Architecture.md) | Moot. NixOS defaults already match the desired end state (wheel-based sudo asking the invoker's password; root login lockable via `users.users.root.hashedPassword = "!"`) |
| YaST / zypper patterns | NixOS modules and this repo's profiles |
| `/etc/kernel/cmdline`, `/etc/sysctl.d/*`, `/etc/udev/rules.d/*`, `/etc/tmpfiles.d/*` files scattered through the notes | `boot.kernelParams`, `boot.kernel.sysctl`, `services.udev.extraRules`, `systemd.tmpfiles.rules` — same content, one repo |

Notes that become historical with this decision: most of *OpenSUSE/OpenSUSE Configuration.md* (its still-live decisions carry over: dbus-broker → §5.1, and its Flatpak picks — Ventoy, SD Memory Card Formatter, Nautilus previews via Sushi — → §5.6); the transactional-update and sdbootutil fragments of *Boot and Init.md*; the zypper section of *Power Management.md*; the zypper/OBS package-name spellings throughout (the packages themselves all carry over). Recommendation: keep the OpenSUSE folder as an archive rather than deleting — it documents why decisions were made.

What NixOS makes harder, stated honestly: FHS-assuming binaries need `programs.appimage`/`steam-run`/patching (mitigated by the Flatpak-for-GUI policy, since Flatpak is its own FHS-ish world); Secure Boot is community tooling (lanzaboote), not yet in-tree (§3.6); anything we build custom (GNOME overlay, tuned kernel) is our CI's job to pre-build, or users compile locally; and the learning curve is real — this distro's target user is initially its author.

---

## 2. Load-bearing upstream facts (verified 2026-07-22)

| Fact | State today |
|---|---|
| GNOME versions | Stable **50** "Tokyo" (2026-03-18, at 50.2/50.3); old-stable **49** (2025-09-17, at **49.8**, point releases scheduled through **49.10 on 2026-09-12**, then EOL). **51 releases 2026-09-16** |
| GNOME support policy | Current + previous stable receive point releases; each release is maintained ~12 months (n goes EOL when n+2 ships). Documented via release.gnome.org/calendar |
| nixos-unstable GNOME | **50.2** (gnome-shell, mutter). Exactly **one** GNOME set per nixpkgs revision; no mechanism for parallel GNOME versions |
| nixpkgs GNOME extensions | Auto-generated from extensions.gnome.org; per-shell-version builds retained, top-level merges the **last three** shell versions (48/49/50) — so shell-49 extension builds exist in today's nixpkgs |
| kernel.org LTS lines | 6.18 (EOL Dec 2028), 6.12 (Dec 2028), 6.6, 6.1, 5.15, 5.10. Mainline is now the 7.x series (7.1.4 stable / 7.2-rc4) — numbering jumped 6.18 → 7.0 |
| nixos-unstable kernels | Default `linuxPackages` = **6.18.39 (LTS)**. Attrs for all six LTS lines + 7.1 + zen/xanmod. `linux_hardened` and all `linux_rt_*` attrs **removed** (unmaintained) |
| NixOS kernel config | CONFIG_HZ = **250** (upstream default; nixpkgs doesn't set it). Preemption on 6.18: **PREEMPT_LAZY** default, `preempt=full` selectable at boot on x86-64 |
| ZFS on unstable | OpenZFS 2.4.3; supports kernels ≤ 7.0 → fine with 6.18 LTS. `zfs.latestCompatibleLinuxPackages` is gone; policy is "run the LTS default or pin explicitly" |
| bcachefs | Removed from mainline in 6.18 (late 2025); out-of-tree DKMS-style module, packaged as `linuxPackages.bcachefs` |
| Waydroid | binder/binderfs enabled in the standard NixOS kernel — no custom kernel needed |
| Secure Boot | lanzaboote v1.1.0 (June 2026), active, still out-of-tree; bootspec (RFC 125) is in NixOS core. TPM2 LUKS auto-unlock supported via systemd initrd (scripted initrd deprecated, removal planned 26.11) |
| NixOS modules | firewalld module **new in 25.11**; tuned module **new in 25.11** (with power-profiles-daemon compatibility); flatpak, waydroid, cockpit, opensnitch, scx, system76-scheduler, sunshine, ipp-usb, snapper, etc. all present (§5). `programs.adb` removed (systemd 258 handles uaccess; use `android-tools`) |
| x86-64-v3 | No official or community binary cache; still an open pre-RFC. Setting `gcc.arch` means building the world locally |
| CachyOS-on-Nix | Chaotic-Nyx archived 2025-12-08 (dead); forks exist without confirmed caches |
| NixOS security data | Official **Nixpkgs Security Tracker** live at tracker.security.nixos.org (per-CVE, triage states, `NIXPKGS-YYYY-NNNN` IDs); vulnix and sbomnix/vulnxscan both actively maintained |

---

## 3. Core architecture decisions

### 3.1 Base: nixos-unstable, flake-pinned, CI-gated

The system is a flake whose primary input is `github:NixOS/nixpkgs/nixos-unstable`. We do not track the branch live; we ship **tested lock snapshots**. CI bumps `flake.lock` daily, evaluates, builds the full system closure for all hosts plus the ISO, runs a smoke test (boot the ISO in a VM), pushes to the binary cache, and auto-merges on green. A red bump stays open with the failure attached and retries next day.

Two consequences worth stating. First, this makes us — deliberately — a distribution with our own patch lead time: the delta between a fix landing in nixos-unstable and our lock advancing. The update policy is therefore part of the research posture: **fast by default; holds only by exception, with the reason recorded in the commit message** (that record is analyzable later). nixos-unstable itself already lags nixpkgs master by hours-to-days via Hydra gating channel advancement; our CI adds a second, measured gate. Second, an unstable base occasionally ships breakage waves (large staging merges). The lock gate plus generation rollback is the containment strategy; the LTS kernel and old-stable GNOME mean the two components most able to ruin a morning are exempt from the wave entirely. That is the whole point of the architecture.

### 3.2 Kernel: pin the 6.18 LTS line; stock config first, tuned config by measurement

`boot.kernelPackages = pkgs.linuxPackages_6_18` — pinned explicitly rather than relying on `linuxPackages` (which happens to be 6.18 today) so nixpkgs' choice of default can never move the kernel under us. Policy: track the newest kernel.org longterm line; evaluate the next LTS each December when kernel.org designates it, and move only after ZFS supports it and NVIDIA is clean on it. 6.18 is LTS until Dec 2028, so there is no urgency ever.

The notes' compile-time wishes (Kernel Tuning.md: HZ=1000; CachyOS envy) are deferred to a measurement, not adopted on vibes: stock nixpkgs 6.18 is HZ=250 with **PREEMPT_LAZY** — a 6.13+ mechanism specifically built to deliver full-preemption latency behavior with voluntary-preemption throughput, which materially weakens the old case for HZ=1000 + `preempt=full`. Phase C runs the *Benchmarking.md* latency protocol (p99 scheduling latency under load, frame times) comparing stock vs. a custom `structuredExtraConfig` build (HZ_1000, and optionally `preempt=full` via boot param — no rebuild needed for that one). If the custom config wins visibly, CI builds and caches it permanently; a locally-built kernel is a ~30–60 min CI job, not a user burden. Everything else from Kernel Tuning.md is runtime and lands in Phase A verbatim: `threadirqs`, `nowatchdog`, `nmi_watchdog=0`, watchdog module blacklists, `kernel.split_lock_mitigate=0`, irqbalance, rtkit (`security.rtkit.enable`). The `rcu_nocbs=all` + `rcu_lazy` items carry a VERIFY flag (§10) before shipping. PREEMPT_RT proper: deferred indefinitely — nixpkgs dropped its RT attrs, RT requires EXPERT-gated config surgery, and NVIDIA-on-RT has open crash bugs; musnix's RT option remains available for a future audio-workstation profile if a user accepts the tradeoffs.

**Hardening note:** nixpkgs also removed `linux_hardened`. Kernel hardening therefore happens via config/sysctl (KSPP-aligned settings, Kicksecure security-misc as reference — *Security Architecture.md*) in the hardening profile, not via a special kernel attr.

### 3.3 GNOME old-stable: the pin-or-build question, answered — build

Your rule was: *pin it if Nix maintains GNOME builds for the full year GNOME does; build it if not.* Verified answer: **Nix does not.** nixos-unstable carries exactly one GNOME (50 today; 51 within weeks of 2026-09-16), and stable NixOS channels hold n−1 only for their ~7-month support window (25.11, the last channel with GNOME 49, reached EOL around June 2026 under NixOS's standard ~7-month window — before 49's September EOL; exact channel state gets re-checked when resurrecting expressions, §10) with a frozen dependency closure besides. There is nothing upstream to pin that satisfies both "old-stable GNOME" and "rolling everything else." So we build: a **`gnome-oldstable` overlay** in our flake.

Mechanism. The overlay replaces the GNOME session core — gnome-shell, mutter, gdm, gnome-session, gnome-settings-daemon, gnome-control-center, nautilus, gnome-shell-extensions, xdg-desktop-portal-gnome, libadwaita and the handful of libraries version-locked to the shell — with the old-stable branch (49.8 today), built against **current rolling dependencies** (glibc, mesa, systemd, pipewire from unstable). Everything version-coupled stays internally consistent inside the overlay; everything not version-coupled rolls. Where a rolling app needs a newer libadwaita than the session's, Nix's parallel-closure property makes that a non-event — the app's closure carries its own; this is the capability no conventional distro has and the reason §0 chose NixOS.

Implementation is resurrection, not authorship: nixpkgs' own GNOME 49 expressions exist, fully debugged, in the `release-25.11` branch history. The overlay starts as those files, bumped to the latest 49.x tarballs, with dependency references repointed at unstable. Expected friction is real but bounded (a 10-month-old GNOME against ~2-months-newer libraries is the kind of skew Arch users live on the other side of); Phase B budgets for it and defines an abort criterion.

Cadence, steady-state: track old-stable point releases (49.8 → 49.9 → 49.10) as they appear; on 2026-09-16, when 51 releases and 50 becomes old-stable, the overlay flips to **holding 50.x** — which is nearly free, since 50's expressions are simply today's nixpkgs, retained while unstable moves on, then bumped along 50.x point releases for a year. After this first cycle, the overlay's job is permanently the cheap "hold n−1, track its point releases" mode; the expensive "backport an older release" mode only existed because we're entering mid-cycle. Sequencing consequence for launch week: **Phase A ships on GNOME 50 (flagged as a temporary, honest deviation — it is current-stable, not old-stable), Phase B lands 49 by mid-August, and Sept 16 begins the permanent rhythm.** If Phase B overruns badly, the fallback is to stay on 50 until Sept 16, at which point 50 *becomes* n−1 and the overlay starts in cheap mode — the architecture arrives ~5 weeks late with zero backport labor. Extensions: nixpkgs retains per-shell-version extension builds (48/49/50 are all in today's tree), so the curated extension set (§5.3) can follow the shell version; the exposure of the per-version attrset is a VERIFY (§10).

GDM and the session are wired through the standard NixOS GNOME module (`services.desktopManager.gnome`, `services.displayManager.gdm` — current option paths after the rename from `services.xserver.*`), which consumes the top-level GNOME attrs the overlay replaces; no module fork should be needed (VERIFY at build: the module's version assumptions).

### 3.4 Filesystems: Btrfs root on LUKS2 (RAID1-capable); ZFS for arrays; nothing else supported

Per today's decision, exactly two filesystems are officially supported. **Root: Btrfs**, with `compress=zstd`, subvolume layout `@ @home @nix @var @snapshots`, `relatime` (Storage and IO.md). Encryption is **LUKS2 under Btrfs** — Btrfs has no usable native encryption in 6.18 (fscrypt-for-btrfs remains unmerged work; VERIFY flag §10), so "encrypted btrfs directly" is implemented as dm-crypt beneath the filesystem. LUKS2 formatting defaults to `--sector-size 4096` on 4Kn drives (Storage and IO.md). **RAID1 root is supported**: two independently-LUKS'd devices opened in initrd, Btrfs `raid1` profile for data+metadata across them (`mkfs.btrfs -d raid1 -m raid1`); systemd stage-1 (`boot.initrd.systemd.enable = true`, mandatory for us anyway) handles multi-device unlock with one passphrase or TPM. Btrfs raid1 is the supported multi-device mode; raid5/6 remain explicitly unsupported. Snapshots: system rollback is NixOS generations' job; `services.snapper` snapshots **/home** (timeline + pre-post is meaningless without a package manager mutating /, so timeline only).

**Arrays/secondary storage: ZFS** via `boot.zfs` — OpenZFS 2.4.3 supports ≤ 7.0, so the 6.18 pin is comfortable, and the arrays-only policy (never root) means a future ZFS/kernel skew can never block booting; it only delays an array import. Poolsman noted as candidate GUI (Distro Vision.md). Hibernation interplay: swap lives outside ZFS always; with zram as primary swap (§5.2), hibernation is deferred as an open item (ZRAM Hibernate note) rather than a launch feature.

bcachefs (Distro Vision legacy note) is **excluded**: it left mainline in 6.18 and now lives as an out-of-tree module — the definition of a non-boring core. Stratis likewise dropped. Removable media: exFAT. Everything else — NTFS, HFS+, APFS, dislocker/BitLocker, encfs, sshfs and the whole Filesystems.md FUSE list — ships **userspace-only** (FUSE), which lands the "userspace-only support for legacy filesystems" security principle as a real attack-surface decision: no *legacy* in-kernel FS parsers reachable from removable media. Two deliberate in-kernel exceptions, named so the claim stays honest: vfat (the ESP requires it; also FAT-formatted sticks) and exfat (the modern Samsung-contributed in-tree driver — actively maintained, not legacy; flip it to FUSE per the stricter reading of the note if preferred, at a mount-performance cost).

### 3.5 Package-management split (per Distro Vision.md)

Nix for the system and CLI; **Flatpak for GUI applications** (declaratively, via the nix-flatpak module, so the app list is code too); Bazaar as the storefront with Warehouse and Flatseal for management; Homebrew explicitly not needed (Nix covers its niche). AppImage supported via `programs.appimage` with binfmt registration. Home Manager owns the per-user layer: dotfiles, dconf settings, shell config, and — via `programs.gnome-shell` — the enabled-extension set. Home Manager is chosen over chezmoi as the primary sync mechanism for coherence (one language, one repo); chezmoi remains the documented answer for syncing to non-Nix machines. The "fork of Extension Sync for GNOME state" idea is superseded for our own use (declarative extensions + dconf in HM is strictly stronger) and moves to the product backlog for non-declarative users. Brave ships from **nixpkgs, not Flatpak** (recommended; confirmation is open item §9.3), because the Browser Configuration.md enterprise policy is delivered as `environment.etc."brave/policies/managed/*.json"` — trivially declarative for the native package, awkward for the sandboxed one. Storefront: GNOME Software stays in the image initially (it also fronts fwupd firmware updates); Bazaar is evaluated as its Flatpak-storefront replacement in Phase B rather than assumed.

### 3.6 Boot chain: systemd-boot + Plymouth now; lanzaboote + TPM2 in Phase E

Phase A: systemd-boot, systemd stage-1 initrd, Plymouth with the flicker-free set from Boot and Init.md (`quiet loglevel=2 systemd.show_status=no splash`, early KMS via `boot.initrd.kernelModules` for the GPU driver, `DefaultTimeoutStopSec=15s`; `loglevel=0` reserved for release images per the note). systemd-bsod enabled if present in nixpkgs' systemd build (VERIFY §10). Phase E adds **lanzaboote** (v1.1.0) for Secure Boot signing and `systemd-cryptenroll` TPM2 auto-unlock (`crypttabExtraOpts = [ "tpm2-device=auto" ]`, PCR 7+11 policy), with a **recovery-key-first** enrollment flow designed around the documented Aeon PCR-validation failure mode (Security Architecture.md TPM Issues): every TPM enrollment is preceded by a mandatory recovery-key enrollment, and firmware-update-driven PCR breaks degrade to passphrase, never to lockout. CoreBoot/OpenBMC items in the note are hardware aspirations out of scope for the OS repo.

### 3.7 Firewall: firewalld (new module) + OpenSnitch

Security Architecture.md's reasoning (zones, NetworkManager integration, GUI availability) now maps cleanly to NixOS: the **firewalld module landed in 25.11** with declarative zone/service submodules, replacing NixOS's simpler default firewall for our purposes; `firewall-config` and Cockpit's panel serve as the GUI until the aspirational first-party Firewall GUI (product backlog). Home zone pre-opens ssh/mdns/cockpit per Network Services.md. **OpenSnitch** (`services.opensnitch` + the UI) ships in the security profile as the outbound, application-level layer.

### 3.8 Architecture baseline: x86-64 (v1), no microarch rebuild

There is no x86-64-v3 nixpkgs cache (still pre-RFC), so a v3 baseline would forfeit cache.nixos.org entirely — unacceptable. Ship baseline x86-64; revisit only if we ever run our own Hydra at scale. Selective per-package optimization (and BOLT/AutoFDO experiments from Gaming Mode.md) belongs in Phase F as a measured experiment, not a default. ARM64 (FEX-Emu note) is out of scope for v1.

### 3.9 Immutability model

NixOS's own: read-only `/nix/store`, declarative `/etc`, generation rollback. `users.mutableUsers = true` for v1 (a workstation, not a fleet). The impermanence module (ephemeral `/` with explicit persistence) is the *stronger* form of the MicroOS idea and is staged as a Phase E opt-in profile rather than a launch default, because its failure mode (silently losing state you forgot to persist) is hostile until the persistence list has matured on a daily driver.

---

## 4. Repository layout and the two build targets

One flake, several outputs. Sketch:

```
distro/                        # working name TBD (§9.1)
  flake.nix                    # inputs: nixpkgs(nixos-unstable), home-manager,
                               #   lanzaboote, nix-flatpak, disko, snapper?, musnix,
                               #   impermanence (later), nixos-generators (images)
  overlays/
    gnome-oldstable/           # §3.3 — the distro's one first-party maintenance duty
    default.nix                # misc: packages nixpkgs lacks (§5 flags them)
  modules/
    core/                      # boot, fs, kernel, zram, sysctl, udev, monitoring
    desktop/                   # GNOME(oldstable), gdm, fonts, theming, dconf defaults
    profiles/                  # gaming, audio-production, development, virtualization,
                               #   ml, security-tools, tablet, hardening, server-lite
  pkgs/                        # brave-policy, fontconfig-msmap, udev-rule sets,
                               #   sound theme (later), freetype-envision config
  hosts/
    magneto/                   # Paul's workstation (hardware.nix via nixos-facter/disko)
  images/
    installer-minimal.nix      # CLI installer ISO with the flake preloaded
    installer-calamares.nix    # Phase D: branded graphical installer
  home/                        # Home Manager: shell, CLI stack, extensions, dconf
  ci/                          # lock-bump workflow, eval+build gate, VM boot smoke
                               #   test, cache push, (later) lead-time telemetry
  docs/
```

**Workstation target (milestone A):** `nixosConfigurations.magneto`. Disk layout declared with disko (Btrfs-on-LUKS2 per §3.4; RAID1 iff the machine has two disks — hardware inventory is open item §9.2). Migration path: back up `/home`, boot our minimal ISO, `disko` + `nixos-install --flake`, restore. `nixos-anywhere` is the fallback for remote/scripted installs.

**ISO target (milestone A′):** the same modules composed into an installer image. Day one this is the standard NixOS installer ISO with our flake, cache config, and profiles baked in (near-zero marginal work — this is why "both in parallel" is affordable). Phase D turns it into a branded graphical installer by forking **calamares-nixos-extensions** (the supported branding/customization mechanism), including profile selection (§5.7) at install time. Distro identity strings via `system.nixos.distroName`/`distroId` once named — checking the NixOS trademark policy for derivative naming is open item §9.1.

**Binary cache from day one** (Cachix initially; self-hosted attic if volume demands). Without it, every ISO user rebuilds the GNOME overlay and any custom kernel; with it, the distro is install-and-go. The cache key is part of the ISO's nix config.

CI cadence per §3.1. The lock-bump commit log doubles as the distro's patch-lead-time dataset (§6).

---

## 5. Implementation map, domain by domain

Column key: **Source** = the note file the decision comes from; **Phase** per §7. Items marked ⚠ carry a §10 VERIFY flag; items marked ✎ need a small first-party package or config file in `pkgs/`.

### 5.1 System core

| Area | Implementation | Source | Phase |
|---|---|---|---|
| Boot chain | systemd-boot; systemd stage-1; Plymouth DRM; flicker-free cmdline; early-KMS module in initrd; `DefaultTimeoutStopSec=15s`; dbus-broker (`services.dbus.implementation = "broker"` — a NixOS non-default, unlike openSUSE); systemd-bsod ⚠ | Boot and Init, OpenSUSE Config | A |
| Kernel | `linuxPackages_6_18` pinned; params `threadirqs nowatchdog nmi_watchdog=0`; blacklist `sp5100_tco iTCO_wdt`; `libahci.ignore_sss=1` where needed; `rcu_nocbs`/`rcu_lazy` ⚠; irqbalance; rtkit | Kernel Tuning, Storage and IO | A |
| Resume quirks | Targeted systemd sleep hooks to reset BT/WiFi/trackpad/touchscreen on resume, per-hardware quirk list in `hosts/` | Boot and Init, Scratch | B |
| Filesystems | §3.4: Btrfs+LUKS2 root (disko-declared, RAID1-capable), ZFS arrays, snapper for /home, `fstrim.timer`, exFAT removables, FUSE stack in systemPackages (ntfs-3g, apfs-fuse, dislocker, encfs, sshfs, squashfuse…) ✎ for any missing from nixpkgs | Filesystems, Distro Vision | A |
| Compression/archives | cabextract, lzip, unrar (unfree), p7zip, pax, sharutils, xz, zstd… in systemPackages; File Roller/Nautilus integration comes free | Filesystems | A |
| Audio | PipeWire (NixOS default) + WirePlumber; rtkit; non-free BT codec state in nixpkgs' pipewire build ⚠ (AAC/aptX/LDAC/LC3 — believed largely enabled; verify list, fill gaps via overlay); pro-audio profile via **musnix** (memlock limits, `/dev/cpu_dma_latency` access, swappiness=10 override) as opt-in profile, resolving the swappiness conflict with zram (§5.2) by profile precedence | Audio, Memory Management | A (base) / E (profile) |
| Monitoring | `services.smartd`; rasdaemon (module verified present; option path `hardware.rasdaemon` vs `services.` ⚠); users in `systemd-journal` group at creation; tools: htop iotop-c nvtop iftop nethogs powertop atop | Drivers and Firmware | A |
| Drivers/firmware | `hardware.enableAllFirmware` (unfree on) covering the "proprietary firmware built-in" principle; NVIDIA via `hardware.nvidia` (open modules where supported, PRIME offload for Optimus); `services.fwupd` + firmware GUI in GNOME Software; Solaar + `services.ratbagd`; `hardware.openrgb` + udev rules; OpenRazer opt-in; fingerprint via `services.fprintd` (fingwit as the exclusion-UX reference) | Drivers and Firmware, Distro Vision, Input Devices, GNOME Configuration | A |
| Hardware enablement scope | v1 targets magneto's hardware only (`hosts/`); vendor-quirk stacks — linux-surface, asus-linux, t2linux, MrChromebox — are a Phase F hardware-enablement backlog, one opt-in profile each; Asahi/ARM64 excluded with §3.8 | Drivers and Firmware, Scratch | F |
| Codecs | ffmpeg-full, gstreamer full plugin set (incl. bad/ugly/libav), intel-media-driver + libva; VA-API/VDPAU verification per note; image formats: libheif/libavif/libjxl/libraw etc. so Nautilus/Loupe thumbnail everything; Flatpak apps self-carry codecs | Drivers and Firmware | A |
| Waydroid | `virtualisation.waydroid.enable` — stock kernel has binder; ARM NativeBridge translation deferred to backlog | Distro Vision, OpenSUSE Config | B |
| Virtualization | `virtualisation.libvirtd` + virt-manager + GNOME Boxes; Incus (`virtualisation.incus`); Podman with docker-compat + Podman Desktop; Distrobox; `services.cockpit` (+cockpit-machines ⚠ plugin coverage on NixOS) | Distro Vision, Network Services | B |
| ML/DS profile | CUDA (`cudaSupport`, nvidia-container-toolkit) and ROCm toolchains; R/RStudio, Spyder, JASP etc. via profile package sets | Distro Vision, Application List | E |

### 5.2 Performance and tuning

| Area | Implementation | Source | Phase |
|---|---|---|---|
| zram | `zramSwap.enable`, `memoryPercent=100`, zstd; sysctls: `vm.swappiness=180`, `vm.page-cluster=0`, watermark boost/scale, `dirty_bytes` 256M/128M, `vm.max_map_count` | Memory Management | A |
| THP | `systemd.tmpfiles.rules`: enabled=madvise, defrag=defer, shmem=advise (desktop default); gaming flips to `always` at runtime (below) | Memory Management, Gaming Mode | A |
| MGLRU | tmpfiles: `lru_gen/enabled=y`, `min_ttl_ms=1000` ⚠ (may already be kernel-default-on) | Memory Management | A |
| OOM | `systemd.oomd` (swap+memory-pressure policies); the Vision's OOM/low-mem **GUI** is product backlog | Memory Management, Distro Vision | A / F |
| I/O scheduler | udev rules: bfq for rotational/SD, kyber for SATA SSD, none for NVMe (amended from the note's kyber-for-NVMe per its own caveat) | Storage and IO | A |
| USB storage | `bdi/max_ratio=1` udev rule for removables; UAS/TRIM verification documented | Storage and IO | A |
| /tmp | tmpfs with 4G cap (`boot.tmp.useTmpfs`) | Storage and IO | A |
| Network | `tcp_mtu_probing=1`; CUBIC + fq_codel as default (mostly already kernel/systemd defaults — declared explicitly anyway); BBR profile documented as opt-in for lossy links | Network Tuning | A |
| Power | **tuned** (module new in 25.11) with `ppdSettings` compatibility so GNOME's power panel keeps working; profiles: desktop default, desktop-gaming (THP always, `compaction_proactiveness=0`, split-lock off), latency-performance for audio; powertop2tuned for the laptop case; thermald auto-enabled on Intel; hdparm/sdparm/smartmontools in base. Falls back to plain power-profiles-daemon if tuned integration disappoints ⚠ | Power Management, Kernel Tuning | B |
| Gaming mode | `programs.gamemode` with custom start/stop scripts implementing the note's THP/compaction toggle-and-restore; `programs.steam` (+ its controller udev rules), gamescope, MangoHUD, LACT, CPU-X, hardinfo2; scheduler experiment: `services.scx` (BORE/lavd) vs `services.system76-scheduler`, decided by Phase C benchmarks; ublue udev rule set for controllers ✎ | Gaming Mode, Distro Vision | B/C |
| Benchmark harness | Scripted per Benchmarking.md: cyclictest p99 under load matrix, MangoHUD frametime logging, fio/iperf3/stream/kernel-compile throughput; comparisons vs stock, CachyOS, Clear Linux, and Fedora defaults; the HZ=1000 experiment pairs with NO_HZ_FULL as the note prescribes; drives §3.2 and scheduler/THP decisions | Benchmarking | C |
| Preload/ananicy/nohang/bpftune/auto-cpufreq/zorin-exec-guard | **Not** shipped by default (overlap and regression risk with oomd/scx/tuned; exec-guard duplicates Flatpak sandboxing rationale); documented as opt-ins in docs | Gaming Mode, Power Management | — |

### 5.3 Desktop: GNOME (old-stable), theming, fonts, input

| Area | Implementation | Source | Phase |
|---|---|---|---|
| Session | GNOME old-stable per §3.3 via overlay; GDM; Wayland-only by default (Xwayland present) | Distro Vision | A/B |
| Default settings | `programs.dconf.profiles.user.databases`: `check-alive-timeout=60000`, Nautilus `open-folder-on-dnd-hover`, plus curated defaults; `environment.gnome.excludePackages` to slim stock apps where Flatpak versions are preferred | GNOME Configuration | A |
| Extensions (curated, shell-version-matched) | Ship enabled-by-default: AppIndicator, Dash to Dock, Blur My Shell, Just Perfection, Alphabetical App Grid, Bluetooth Battery Meter, Coverflow Alt-Tab, Rounded Window Corners Reborn, Pano or Clipboard Indicator, Quick Settings Tweaks, Tiling Shell, Transparent Window Moving, ScreenToSpace, GSConnect (via `programs.kdeconnect` with gsconnect package), Weather O'Clock, Caffeine; DING included but off; consideration list tracked in repo. Availability per shell version resolved at build from nixpkgs' per-version sets ⚠ | GNOME Extensions, GNOME Configuration | B |
| Tablet profile | Maximize-to-empty-workspace, Auto Activities, Always Show Titles, Improved OSK, Custom Hot Corners Extended + the settings list; ships as `profiles/tablet` | GNOME Tablet Mode | E |
| Theming | adw-gtk3 (nixpkgs ⚠) for GTK3 coherence; Qt/KDE coherence for the shipped Qt apps via KvLibadwaita/Libadwaita-KDE ⚠; per-app Adwaita themes (Firefox, Thunderbird, Steam, VSCode, GIMP) from the Theming.md list; MoreWaita icons ⚠/✎; active/inactive headerbar contrast tracked as a first-party theme tweak (Yaru's solution) in backlog | Theming | B |
| Fonts | `fonts.packages`: Noto (+CJK), Liberation, DejaVu, Carlito, Caladea, Open Sans, Overpass, Roboto, TeX Gyre, Ubuntu family, Courier Prime, Merriweather, Source Sans, Inter, IBM Plex, JetBrains Mono, Cascadia Code, Hack, Fira, Intel One Mono, Atkinson Hyperlegible, Unifont; missing faces (Signika, Gelasio, WeblySleek…) packaged in `pkgs/` ✎; `fonts.fontconfig`: hinting slight, lcdfilter default, plus the full MS-substitution map as a first-party fontconfig file ✎; freetype-envision settings evaluated ⚠; Infinality confirmed dead (note already says so) | Fonts | B |
| Input | libinput defaults; kinetic-scroll work is upstream R&D (backlog, not v1); modal-keys idea → backlog; middle-click-minimize etc. via dconf where GNOME exposes them | Input Devices | — |
| GNOME upstreamables | The bug list (Nautilus partial-move atomicity, notification hygiene, inhibit-sleep notification, GDM display choice…) becomes a tracked "upstream issues we care about" doc — distro carries **no** bespoke GNOME patches, by principle (§0) | GNOME Configuration | ongoing |

### 5.4 Security

| Area | Implementation | Source | Phase |
|---|---|---|---|
| Update posture | The headline control: rolling userland + upstream-LTS core, §3.1/§3.2 | Security Architecture | A |
| Attack surface | No GTK2/Python2 anywhere in the closure (CI assertion — evaluable in Nix: closure scan failing on denylisted store paths); legacy FS/protocols userspace-only (§3.4); Wayland-only session | Security Architecture, Distro Vision | B |
| Firewalls | firewalld + OpenSnitch per §3.7 | Security Architecture | A/B |
| Hardening profile | KSPP/security-misc-derived sysctl+boot set (kptr_restrict, dmesg_restrict, yama ptrace_scope, bpf hardening, io_uring policy decision ⚠ flatpak/userns interplay), opt-out-able; no linux_hardened attr exists anymore (§3.2) | Security Architecture | E |
| Disk crypto | LUKS2 (+TPM2/FIDO2 enrollment §3.6); Cryptomator for cloud; TCG Opal documented-only (cryptsetup Opal support ⚠) | Security Architecture | A/E |
| Keys/tokens | `security.tpm2`; YubiKey: pam_u2f for auth + FIDO2 LUKS slot; NitroKey equivalent | Security Architecture | E |
| Crypto policies | NixOS has **no** crypto-policies mechanism (verified) — honest gap; per-service settings + a doc page; candidate first-party module later | Security Architecture | F |
| Security tools | `profiles/security-tools`: Wireshark/termshark, Ghidra, imHex, Seer+GDB, mitmproxy, nmap/rustscan, Sleuthkit/Autopsy ⚠ (packaging state), Volatility, Hashcat, John, Cyberchef, fail2ban (for exposed hosts). The note's open categories: "OS hash-check installs" is satisfied natively (Nix verifies every store path by hash, and the ISO ships checksummed/signed); exploit-scanning and web-scanning tool selection (nuclei/ZAP class) is backlog | Security Tools | E |
| Rust replacements | Watch item (sudo-rs et al. per Ubuntu's path) — adopt when nixpkgs/NixOS offers first-class switches; not v1 | Security Architecture (Scratch) | — |

### 5.5 Networking, printing, sharing

| Area | Implementation | Source | Phase |
|---|---|---|---|
| mDNS | `services.avahi` + `nssmdns4` (+myhostname in NSS — NixOS composes nsswitch declaratively) | Network Services | A |
| Printing | Driverless-only stack exactly per Printing.md: cups + cups-filters, `services.ipp-usb`, avahi discovery; **no** HPLIP/foomatic/PPD packs shipped (available in nixpkgs if a user insists — documented) | Printing | A |
| Scanning | `hardware.sane` + sane-airscan (eSCL/WSD) + simple-scan; `sane-backends` present as legacy fallback | Printing | A |
| Remote mgmt | Cockpit (+tukit-less: our "transactions" are generations); RustDesk; Sunshine/Moonlight (`services.sunshine`) as the low-latency remote/game-stream answer; SSH + Mosh in base | Network Services, Self-Hosted, Terminal and CLI | B |
| VPN/mesh | `services.tailscale` (+ documented headscale recipe); NetworkManager ignore-rule if needed ⚠ (module may handle); WireGuard/OpenVPN via NM | Pre-Packaged Environment, Self-Hosted | B |
| File sync/share | Syncthing (module, declaratively configured); LocalSend; SMB3 via Samba for the File Sharing GUI story (GUI itself = backlog); rclone; restic (+ a default backup story: snapper for history, restic for off-machine) | Self-Hosted, Network Services, Pre-Packaged Env | B |
| Self-hosted services disposition | Server-side services in Self-Hosted Services.md (Nextcloud, Vaultwarden, headscale, Matrix homeservers) are out of distro scope — documented recipes, not preinstalled; their **clients** ship via the app manifests (Bitwarden, Signal, Nextcloud client, Tor Browser as Flatpak, Floccus as a browser extension doc note). Casting (CAST/AirPlay receive) → backlog | Self-Hosted Services | B/F |
| macOS-parity sharing matrix | Tracked as a feature checklist against the Network Services.md list (media sharing, content caching etc. are aspirational/backlog) | Network Services | F |

### 5.6 Applications and user environment

| Area | Implementation | Source | Phase |
|---|---|---|---|
| GUI apps | Declarative Flatpak (nix-flatpak) manifests generated from Application List.md: LibreOffice, Obsidian, Calibre, Okular, Zotero, Anki, Kiwix, GIMP, Inkscape, Krita, Darktable, digiKam, Scribus, Kdenlive, Blender, OBS, Audacity, FreeCAD/LibreCAD/OpenSCAD, KiCad, Stellarium/KStars, JASP/PSPP/Octave, VSCode, Kate, Dolphin+KDE set, Maui set (evaluate), Mission Center, Planify, Bitwarden, Thunderbird, LM Studio, etc. Plus the OpenSUSE-note carryovers: Ventoy, SD Memory Card Formatter, Nautilus with Sushi previews. Exact Flathub-vs-nixpkgs placement decided per app (rule: GUI→Flatpak unless policy/hardware integration argues otherwise — Brave §3.5, virt-manager, Wireshark stay native). webapp-manager/appimagepool → backlog | Application List, OpenSUSE Config | B |
| Firefox (secondary browser) | Home Manager `programs.firefox` policies/prefs implementing Browser Configuration.md: `apz.gtk.pangesture.delta_mode=2` + multiplier 25 (scroll fix), disk cache → RAM (`browser.cache.disk.enable=false`); font selection TODO carried over | Browser Configuration | B |
| Disk utilities | GNOME Disks in core; KDE Partition Manager and Cockpit's storage panel available via profiles (install-time partitioning itself is disko/calamares territory) | Filesystems, Scratch | B |
| Default footprint | **Minimal core + opt-in profiles** at install (recommended §9.4): core ships browser, files, terminal, office basics; everything else via profiles/Bazaar | Distro Vision | B/D |
| Terminal env | zsh default + completions + **Starship enabled by default**; modern CLI set in base: ripgrep fd fzf zoxide eza bat delta jq yq pandoc ImageMagick; neovim default editor; nushell/helix available; full TUI catalog (lazygit, termshark, bluetui, impala, sysz, harlequin…) in `profiles/development` and docs | Terminal and CLI, Distro Vision | A |
| Home Manager | Ships preconfigured user skeleton: shell, prompt, dconf, extensions, git defaults; the distro's "settings sync" story (§3.5) | Distro Vision | A |
| Windows subsystem | Bottles (Flatpak) as the friendly face over Wine/Proton; documented; deeper "subsystem" integration = backlog | Distro Vision | E |
| Audio cues | First-party granular sound theme (design per note) — backlog ✎ | Audio | F |

### 5.7 Profiles (Distro Configured Use Cases)

`profiles/` maps one-to-one to Distro Vision.md's use cases — gaming, audio/multimedia, development (git, Podman, k8s tooling, Distrobox, VSCode), virtualization, ML/data-science, sysadmin (Cockpit), security-tools, tablet, hardening — each a NixOS module the installer (Phase D) exposes as checkboxes and `docs/` explains for post-install enabling. This is the NixOS-native answer to the "patterns" concept, and keeping each profile a plain module means a profile is also a *documented capability list* for the research artifact.

---

## 6. Research integration

The distro is the RQ2 hybrid instantiated: upstream-LTS-tracked core (kernel 6.18 LTS; GNOME old-stable — both branches upstream maintains anyway) with zero bespoke soft-fork maintenance, on a rolling base whose lead time we control and measure. The Patching Research Plan's conflict-of-interest note (§10 there) already covers the reflexivity; this plan adds the dogfooding instrumentation: (a) CI records every lock bump with timestamps → our own per-package patch lead time vs. nixos-unstable and vs. upstream is a query, not a study; (b) `flake.lock` git history gives exact per-day shipped-state reconstruction, better than any archive service used in the study; (c) holds are recorded with reasons, making our own "policy exposure" auditable in the study's own vocabulary.

Two findings from today's verification feed **back** into the research plan and should be edited there (with your approval — I have not touched that file): first, the *NixOS security-tracker absence* VERIFY item in §6/§15 is now false — the official Nixpkgs Security Tracker is live (per-CVE, `NIXPKGS-YYYY-NNNN`, triage states), plus maintained vulnix/sbomnix tooling, which materially strengthens the case for promoting NixOS from sidebar toward a studied rolling distro (its per-day state reconstruction is the best of any candidate; the tracker was the missing instrument — its git history depth/point-in-time semantics need the same §7-style scrutiny as the other trackers before promotion). Second, minor ledger confirmations: post-6.18 kernel numbering is indeed 7.x (consistent with the "Ubuntu 26.04 kernel 7.0 (non-longterm)" entry), and the 6.18/6.12 LTS EOLs (both Dec 2028) sharpen the D4/D5 dispersion snapshot.

---

## 7. Phased work plan

Capacity assumptions match the research plan: Paul 5–15 h/week total across both projects, launch effort concentrated in the weeks of Jul 20 and Jul 27; research Phases 0–1 (archiving + pilot, Claude-heavy) run concurrently. Claude carries packaging, CI, and drafting; Paul carries decisions, hardware-specific testing, and anything requiring hands on the machine.

| Phase | Dates (2026) | Work | Exit criterion |
|---|---|---|---|
| **A — Bootstrap** | Jul 22–27 | Flake skeleton + repo + cache; `modules/core` (boot/fs/kernel/zram/sysctl/udev per §5); GNOME **50** desktop module (temporary, flagged); disko layout for magneto; minimal installer ISO building in CI; workstation migrated and daily-driving | Workstation boots our flake; ISO artifact in CI; rollback tested |
| **B — Old-stable + surface** | Jul 27 – ~Aug 21 | `gnome-oldstable` overlay: resurrect 49.x from nixpkgs 25.11 history, build against unstable, land 49.8+; extension set matched to shell 49; Flatpak app manifests; firewalld/opensnitch; tuned profiles; gaming mode; fonts/theming; printing/scanning | Workstation on GNOME 49.x with rolling deps; app + tuning surface installed. **Abort criterion:** if overlay cost exceeds ~2 focused weeks, fall back to GNOME 50-until-Sept-16 (architecture intact, labor deferred) |
| **C — First cycle + measurement** | Sep – Oct | 2026-09-16: GNOME 51 ships, 49 EOLs → flip overlay to **hold 50.x** (steady-state mode) as unstable moves to 51; run Benchmarking.md harness → decide HZ_1000/custom kernel, scx-vs-system76, THP defaults with data; CI lead-time telemetry live | Overlay in hold mode through one real transition; tuning decisions made by measurement; kernel decision recorded |
| **D — Distributable** | Oct – Nov | Branded calamares installer (fork calamares-nixos-extensions); profile selection at install; naming/branding/os-release; docs site; public repo decision executed (§9.1) | A stranger can install the distro from an ISO and pick profiles |
| **E — Hardening + depth** | Nov – Dec | lanzaboote Secure Boot + TPM2 auto-unlock with recovery-key-first flow; hardening profile; audio-production profile (musnix); tablet profile; Waydroid polish; impermanence opt-in | Secure-boot + TPM path documented and survivable through a firmware update |
| **F — Product backlog** | 2027, interleaved with research write-up | Firewall GUI, OOM GUI, file-sharing GUI, extension-sync-for-others, granular sound theme, crypto-policies module, filesystem-hierarchy and tag-FS experiments, BOLT/µarch experiments, macOS-sharing parity | Prioritized backlog, shipped opportunistically |

Collision management: research Phase 1 (pilot, Jul 27–Aug 24) overlaps distro Phase B — both Claude-heavy. Sequencing rule when they contend: research pilot archiving (time-sensitive, feeds a paper) outranks overlay polish (has a free fallback); Paul's scarce hours go to workstation validation and the research pilot's 20-case manual audit, neither of which Claude can do.

---

## 8. Risks

**GNOME overlay cost is the load-bearing unknown.** Bounded by the Phase B abort criterion and permanently cheaper after Sept 16 (hold-mode). Worst case is a 5-week-late n−1, never a broken system. **Unstable breakage waves** are contained by the CI gate + generations + the exempted core; the residual risk is *our* lead time inflating during long red streaks — which the telemetry will show honestly. **Secure Boot/TPM fragility** (Aeon-style PCR lockouts) is mitigated by recovery-key-first design and by keeping TPM unlock opt-in until Phase E exit. **NVIDIA on rolling Mesa/kernel skew**: LTS kernel pin actually helps here; NVIDIA-specific breakage gates the lock like anything else. **Single-maintainer bus factor** (both projects): everything is declarative and documented in-repo by construction; the distro degrades to "a NixOS config that stops updating," not to an unbootable system. **Scope creep** is the honest biggest risk given the research calendar; the backlog (Phase F) exists so that saying "later" has a place to point.

---

## 9. Open items (Paul's input; none block Phase A)

1. **Name** — needed by Phase D for branding, repo, os-release; also check NixOS trademark/derivative-naming policy ⚠. Repo public-from-day-one or private-until-stable mirrors research open item #4.
2. **Workstation hardware inventory** for `hosts/magneto` — GPU vendor (NVIDIA path?), disk count (RAID1 root or single-disk), TPM version, WiFi chipset. Ten minutes of `lshw` output settles §5.1's hardware lines.
3. **Brave from nixpkgs** (policy-manageable, recommended §3.5) vs Flatpak — confirm.
4. **Default footprint**: minimal-core-plus-profiles (recommended) vs batteries-included image — affects ISO size and Phase D installer design.
5. **tuned vs power-profiles-daemon** after Phase B testing — tuned's module is new (25.11); if its ppd-compat is rough on GNOME, we fall back and keep tuned for the gaming/audio profiles only.
6. **Hibernation**: require it (forces swap partition sizing + zram rework + no-ZFS-swap care) or accept suspend-only for v1 (recommended)?

---

## 10. Verification ledger

**Verified live 2026-07-22** (three parallel sweeps; agent reports preserved in session): GNOME 50 "Tokyo" 2026-03-18, 49 old-stable at 49.8 with 49.10 final on 2026-09-12, 51 on 2026-09-16, ~12-month per-release maintenance via release.gnome.org/calendar + handbook; nixos-unstable ships gnome-shell/mutter 50.2 (by-name tree), single GNOME set per revision, extensions generated per shell version with last-three merged (48/49/50); kernel.org longterm = 6.18/6.12 (Dec 2028), 6.6/6.1 (Dec 2027), 5.15/5.10 (Dec 2026), mainline 7.2-rc4/stable 7.1.4; nixpkgs default `linuxPackages` = 6.18.39, all six LTS attrs present, `linux_hardened` and `linux_rt_*` removed, zen/xanmod present; nixpkgs kernel config: HZ unset → 250 default, PREEMPT_LAZY on 6.18+, binder/binderfs enabled (Waydroid OK on stock kernel); PREEMPT_RT requires EXPERT-gated custom config, NVIDIA-on-RT has open crash bugs (open-gpu-kernel-modules #891); OpenZFS 2.4.3 (max kernel 7.0), `latestCompatibleLinuxPackages` removed with pin-explicitly guidance (rl-2411); bcachefs removed from mainline in 6.18, packaged out-of-tree as `linuxPackages.bcachefs`; lanzaboote v1.1.0 (2026-06-22) active/out-of-tree, bootspec in core, TPM2 unlock via systemd initrd + `crypttabExtraOpts`, scripted initrd deprecated (removal 26.11); NixOS modules verified present: flatpak (+nix-flatpak v0.7.0 active), programs.appimage(+binfmt), waydroid, cockpit, opensnitch (+HM ui), **firewalld (new 25.11)**, **tuned (new 25.11, ppdSettings)**, power-profiles-daemon, gamemode, system76-scheduler, scx, zramSwap, oomd, printing, ipp-usb, sane(+extraBackends), avahi(nssmdns4), tailscale, syncthing, sunshine, kdeconnect(gsconnect), smartd, rasdaemon, ratbagd, openrgb, fwupd, usbmuxd, snapper-class snapshots; `programs.adb` removed (android-tools + systemd 258); musnix maintained (2026-05); Home Manager standard w/ dconf + programs.gnome-shell; impermanence maintained; nixos-generators 1.8.0 + 25.05 image-infra rework; calamares-nixos-extensions is the branding mechanism; **no x86-64-v3 cache** (open pre-RFC); Chaotic-Nyx archived 2025-12-08; **Nixpkgs Security Tracker live** (tracker.security.nixos.org, NIXPKGS-YYYY-NNNN); vulnix (1.12.4) and sbomnix (1.8.0) active; NixOS has no crypto-policies equivalent.

**UNVERIFIED / verify at build time:** Btrfs native encryption still unmerged in 6.18 (assumed — LUKS2 plan does not depend on it changing); `gnome49Extensions`-style per-version attrset exposure (fallback: vendor the JSON filter); NixOS GNOME module version-assumption tolerance for the overlay; NixOS 25.11's exact GNOME 49 point level and channel EOL date (resurrection source, §3.3); nixpkgs pipewire's exact non-free BT codec matrix (AAC/aptX/LDAC/LC3); systemd-bsod presence in nixpkgs' systemd build; `rcu_nocbs=all` syntax validity and `rcutree.enable_rcu_lazy` on 6.18; MGLRU default state in the stock kernel; rasdaemon module's option path (`hardware.` vs `services.`); io_uring restriction interplay with Flatpak/user-namespaces (hardening profile); freetype-envision applicability to current freetype; cockpit plugin (machines/podman) packaging coverage on NixOS; tailscale module's NetworkManager interaction; cryptsetup Opal support state; adw-gtk3/KvLibadwaita/Sleuthkit/Autopsy/MoreWaita/Signika/Gelasio/WeblySleek packaging state in nixpkgs; disko/nixos-anywhere assumed current standard (not re-verified today); NixOS trademark policy terms for derivatives; magneto's hardware details (open item #2). Claims resting on prior knowledge rather than today's verification: btrfs raid1 maturity vs raid5/6 status, in-kernel exfat driver maturity, LUKS2 4Kn sector guidance, snapper timeline semantics, PREEMPT_LAZY having landed in 6.13, NixOS wheel/sudo defaults matching the note's desired end state, the virtualisation module set (libvirtd/incus/podman/waydroid are long-standing NixOS modules; waydroid re-verified today, the rest not), Bottles/Wine positioning — all standard, all checkable in Phase A/B.

---

*Next action on approval: Phase A starts immediately — repo + flake skeleton + core modules, workstation migration targeted within launch week. First checkpoint: workstation daily-driving our flake with ISO in CI, ~Jul 27.*
