# GNOME configuration

These notes collect desired behavior, observed problems, and settings to apply. The issue links record the original reports; their presence does not imply that every issue remains open in the current GNOME release. Extension-specific choices live in [GNOME Extensions](GNOME%20Extensions.md), and visual integration in [Theming](Theming.md).

## Global Behavior

### Out of Memory User Dialog
https://gitlab.gnome.org/Teams/Design/whiteboards/-/issues/340

### Notifications
Apps should dismiss outdated notifications.

### App Grid

* Alphabetical App Grid (AlphabeticalAppGrid@stuarthayhurst)
* App Hider
* Applications Overview Tooltip (applications-overview-tooltip@RaphaelRochet)
* Vertical App Grid (vertical-app-grid)

Terminal programs don't get launchers in the app grid.  rpm skips the launchers of htop, nvtop and atop, both of amdgpu_top's, and Racket's DrRacket and Slideshow.  The programs stay installed; `amdgpu_top --gui`, `drracket` and `slideshow` still start from a terminal:

##### /usr/lib/rpm/macros.d/macros.tcbl-excludes

    %_netsharedpath /usr/share/applications/htop.desktop:/usr/share/applications/nvtop.desktop:/usr/share/applications/atop.desktop:/usr/share/applications/amdgpu_top-tui.desktop:/usr/share/applications/amdgpu_top.desktop:/usr/share/applications/drracket.desktop:/usr/share/applications/slideshow.desktop

### Search

* ESP (Extension Search Provider)
* Gnome Fuzzy App Search (possibly integrated into Shell in future: https://gitlab.gnome.org/GNOME/glib/-/issues/1152)
* WSP (Window Search Provider)

### Drag and Drop Issues

* Drag files to dock
* Drag window while changing desktops
* Drag icons to sidebar
* Drag text/images to desktop
* Cannot drag icon from background window without raising the window
    * https://gitlab.gnome.org/Teams/Design/whiteboards/-/issues/255

### Cloud Sync

* https://gitlab.gnome.org/Teams/Design/whiteboards/-/issues/335
* https://discourse.gnome.org/t/proposal-integrate-syncthing-into-gnome-settings/15387
* Dotfiles, Flatpak Apps, GNOME Shell Extensions, GNOME Shell Configuration

### Fingerprint Reader
https://github.com/xapp-project/fingwit

### Other

* Color Picker: Standalone App + QT + GTK
* Global option to open all files in tabs in QT + GTK applications
* Default templates in Template folder

## Bugs

### GDM

* GDM login screen on wrong display
    * https://gitlab.gnome.org/GNOME/gnome-shell/-/issues/3867
    * https://github.com/thiggy01/change-gdm-background/issues/15

### Endeavor (Tasks App)

* Nested Tasks: https://gitlab.gnome.org/World/Endeavour/-/issues/488

### GNOME Initial Setup

* Wrong WiFi password cannot be re-entered

### Nautilus

* When moving a directory containing a file the user lacks permissions for, half the directory gets moved before the error, leading to an inconsistent state. Moves should verify 100% success before any files are deleted.
* When copying or moving files, copy stops on first error. Copy should continue while error dialog is displayed.

#### My Computer view (idea)

This records an idea, not a decision.  [nautilus-my-computer](https://github.com/yannmasoch/nautilus-my-computer) (MIT, Python) adds a page that lists the drives with usage bars, sidebar options, and a beta column view with a preview column ([OMG! Ubuntu](https://www.omgubuntu.co.uk/2026/10/nautilus-my-computer-view)).  It is not an extension through a public API: Nautilus no longer has one for custom views, so it injects itself into Nautilus's widget tree at runtime and depends on internal structures that can change between releases.  A Nautilus update can break it, which is at odds with relying on upstream interfaces.  Its author tests it with GNOME Files 50 and on Tumbleweed, and packages it on OBS.

The fit is a Nautilus Flatpak built by TCBL: the add-on would be built into it and tested against that exact Nautilus version, so an update cannot break it unseen, and Sushi, the previewer that Nautilus calls over D-Bus, would go into the same Flatpak.  Today Nautilus and Sushi are RPMs in the base pattern.

#TODO — Decide whether to build Nautilus as a Flatpak, and check first what a sandboxed Nautilus loses: access to host file systems and gvfs mounts, the trash, thumbnailers, extensions installed on the host (GSConnect's), and Open in Terminal.

### Shell

* When an application inhibits sleep or reboot there should be a notification.

## Settings Configuration

### User-Configurable (needs GNOME GUI)

* Hostname
* Sync(thing)
* SMB
* Firewall

### System Settings (user shouldn't change)

* Bootloader
* NetworkManager
* Sysctl
* SystemD

## Default Settings

    gsettings set org.gnome.mutter check-alive-timeout 60000
[Link](https://askubuntu.com/questions/412917/how-to-increase-waiting-time-for-non-responding-programs)

    gsettings set org.gnome.nautilus.preferences open-folder-on-dnd-hover true
[Link](https://www.omgubuntu.co.uk/2023/02/ubuntu-open-folder-on-drag-drop-hover)

## GNOME Settings App Additions

* https://github.com/pop-os/firmware-manager
* https://www.reddit.com/r/gnome/comments/qyi9lc/does_gnome_plans_to_integrate_firewall_settings/

## Long Term Improvements

### AI launcher (tracking)

Canonical's Didier Roche-Tolomelli published `warty-prototype`, a snap its store listing calls a "design-discussion prototype of the next Ubuntu launcher" (October 2026).  It is a scripted mockup of a launcher that works with an on-device model; no model runs.  It is a separate app opened from the top panel, not a GNOME Shell extension.  No source repository or license is given, and Canonical has not committed to shipping it ([OMG! Ubuntu](https://www.omgubuntu.co.uk/2026/10/ubuntu-ai-launcher-prototype)).

#TODO — Follow it: whether it becomes a working launcher, whether the model runs on the device, whether it is an app or a Shell extension, and under what license.  A Snap-only release would not reach TCBL; what matters is the design and any code released under a license TCBL can ship.

### Text Selection Right Click Options

* Define
* Speak
* Translate

## Icons in Menus
https://blog.jim-nielsen.com/2025/icons-in-menus/

## LibreOffice Issues

* LibreOffice Writer did not respect dark mode change and icons do not show up
