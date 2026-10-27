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

### Text Selection Right Click Options

* Define
* Speak
* Translate

## Icons in Menus
https://blog.jim-nielsen.com/2025/icons-in-menus/

## LibreOffice Issues

* LibreOffice Writer did not respect dark mode change and icons do not show up
