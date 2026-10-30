# Build Log

One file per day of work on Technicomp Benchtop Linux, named by date.

Days through October 6 are reconstructed from the commit histories of the build repositories and this notebook (including the notes as committed), OBS package revisions, and the design notes and audit kept in the notes vault's outputs folder. They record what changed and when, and the reasoning where those sources give it. From October 12 the entries also draw on the working-session record, so they include findings, decisions and reversals. Times are US Eastern.

Commit hashes refer to these repositories: [benchtop-settings](https://github.com/TechnicompLabs/benchtop-settings), [benchtop-patterns](https://github.com/TechnicompLabs/benchtop-patterns), [benchtop-image](https://github.com/TechnicompLabs/benchtop-image), [benchtop-branding](https://github.com/TechnicompLabs/benchtop-branding) and [benchtop-linux-notebook](https://github.com/pauldmartinphd/benchtop-linux-notebook). OBS packages without a git repository (OpenRGB, nmap, health-checker) are cited by revision.

## Before September 3

There is no daily record before the notes were imported. This background comes from a design summary in the October 25 claude.ai data export (last updated September 15), and from the status note in the NixOS-Based Distro Build Plan. It is a summary, not a log, and parts of it were superseded later.

- The project: a Linux distribution under Technicomp Labs, intended for public distribution, with an immutable core from a minimal, well-tested base.
- Bases evaluated: NixOS Unstable against openSUSE Aeon/MicroOS, judged on the quality of upstream integration testing and the likelihood of package breakage. A concern about MicroOS snapshots (created but not marked bootable after extended inactivity) was recorded as resolved.
- August 15 decision: "Aeon now, NixOS later". v1 ships as an Aeon derivative, and the NixOS plan becomes the long-term target. The base later became Slowroll (September 4) and then Tumbleweed (September 7), and the reasons for rejecting NixOS were committed on October 21.
- Kernel: an LTS kernel adopted 3 to 6 months after its LTS designation and updated about once a year.
- GNOME: n−1 (old stable), updated twice a year.
- Applications: Flatpak and AppImage for GUI applications. Nix for user packages outside the immutable core was later replaced by Homebrew.
- Storage: Btrfs, with home on a separate subvolume for snapshots.

## Days

| Date | Work |
|---|---|
| [2026-09-03](2026-09-03.md) | Design notes imported |
| [2026-09-04](2026-09-04.md) | Settings package started; OBS push-to-build |
| [2026-09-07](2026-09-07.md) | Tumbleweed as the base |
| [2026-09-14](2026-09-14.md) | Pattern and image started |
| [2026-09-16](2026-09-16.md) | Settings corrected; install-time encryption |
| [2026-09-18](2026-09-18.md) | THP, Brave policy and tik-config fixes |
| [2026-09-19](2026-09-19.md) | GDM added |
| [2026-09-21](2026-09-21.md) | Webhooks fixed; tik runs |
| [2026-09-22](2026-09-22.md) | Installer modules and branding |
| [2026-09-23](2026-09-23.md) | Live login flow; systemd-resolved; upstream defaults removed |
| [2026-09-24](2026-09-24.md) | Repository, update policy, OpenRGB, branding |
| [2026-09-25](2026-09-25.md) | Manual pages and toolchain |
| [2026-09-26](2026-09-26.md) | Notes reorganized; languages in the pattern |
| [2026-09-27](2026-09-27.md) | Pattern in 28 categories; Flathub per user; nmap |
| [2026-09-29](2026-09-29.md) | Kernel command line moved into configuration files |
| [2026-10-03](2026-10-03.md) | Hibernation TODO |
| [2026-10-06](2026-10-06.md) | IOMMU; health-checker rollback without reboot |
| [2026-10-12](2026-10-12.md) | Pattern split; snapshot gate designed; raw_gadget found in the image |
| [2026-10-13](2026-10-13.md) | Connect idea; USB emulation and flashrom notes |
| [2026-10-19](2026-10-19.md) | kmscon considered |
| [2026-10-21](2026-10-21.md) | Snapshot gate reverted; Tumbleweed repositories added; notebook renamed |
| [2026-10-22](2026-10-22.md) | JACK documented |
| [2026-10-25](2026-10-25.md) | Hardware rules from other projects; Homebrew per user; rasdaemon; install test |
| [2026-10-26](2026-10-26.md) | Synchronous discard and SELinux labels on the installed system |
| [2026-10-27](2026-10-27.md) | Installer fixes: discard, progress, host name, live-session cleanup, SELinux relabel; GUI launchers of Racket and amdgpu_top hidden |
| [2026-10-28](2026-10-28.md) | Notebook licensed CC BY-SA 4.0; Agama as the eventual installer; Easy Anti-Cheat requirements; AI launcher and Emacs text keys noted |
| [2026-10-29](2026-10-29.md) | First-boot checks confirmed; Arch Wiki gaming tweaks considered |
| [2026-10-30](2026-10-30.md) | System configuration moved from config.sh into packages |
