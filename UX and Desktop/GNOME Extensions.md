# GNOME Extensions

## Recorded extension set

This was labeled “Installed” in the combined notes. It describes the working desktop selection, not a verified manifest of the Benchtop image. Entries marked “consider” remain candidates.

### Alphabetical App Grid

### AppIndicator and KStatusNotifierItem Support

### Bluetooth Battery Meter
https://extensions.gnome.org/extension/6670/bluetooth-battery-meter/

### Blur My Shell

* https://github.com/aunetx/blur-my-shell/issues/455
* https://github.com/aunetx/blur-my-shell/issues/757

### Coverflow Alt-tab

* https://github.com/dsheeler/CoverflowAltTab/issues/13

### Dash to Dock

* Launch Feedback: https://github.com/micheleg/dash-to-dock/issues/49
    * https://github.com/home-sweet-gnome/dash-to-panel/issues/2218
* https://github.com/micheleg/dash-to-dock/pull/574
* https://github.com/micheleg/dash-to-dock/issues/1029
https://extensions.gnome.org/extension/8640/dynamic-calendar-clocks-and-weather-icons-reborn/

### Dock integration idea

Consider combining the desired behavior of Dash to Dock, Dash to Panel, dynamic icons, and Dash2Dock Animated. The list records an integration idea rather than a decision to run all four together.

### Drop to Shell
https://gitlab.com/gunwald/drop-to-shell
https://www.reddit.com/r/gnome/comments/1xuepz4/drop_to_shell_applying_gnomes_hig_to_the_shell_or/ (discussion: applying GNOME's Human Interface Guidelines to the Shell)

* Replaces the top bar's dropdowns with larger views: Quick Settings as a resizable side pane, the clock as a Today page in the overview
* drop-to-shell@gunwald.gitlab.com, GNOME 50 and 51, GPL-2.0
* From the author's post (November 13, 2026; marked LLM-assisted):
    * Quick Settings pane: opens on the right with the system indicators or Super+S, and stays open while focus changes; windows make room for it, it can be resized, and Super+S or Esc closes it
    * Today page: the clock or Super+M shows notifications, a month view with events, the day's schedule, weather and world clocks across the screen
    * GNOME's own widgets, moved rather than rebuilt: other extensions' Quick Settings toggles keep working, and Blur My Shell blurs the pane; sections collapse when space runs out
    * Built-in tray icons (StatusNotifierItem), optional, so they can be left to the AppIndicator extension; the pane and the Today page can each be switched off
    * Companion extensions: Glean, and Tally (media players, file copies with pause and cancel, a system monitor)
    * Caveats: not on extensions.gnome.org yet; it hooks into Shell internals (it has a test suite that runs in a headless Shell)
* To check in TCBL: Tiling Shell with the pane (untested by the author), the tray icons against AppIndicator, and whether a pane that stays open suits everyone; one commenter found having to close it for every toggle worse than the dropdown

### Glean
https://gitlab.com/gunwald/glean

* Clipboard history (the clipboard and the middle-click selection, kept across locking and restarts) and quick notes; a drawer in the Drop to Shell pane
* glean@gunwald.gitlab.com, GNOME 50 and 51, GPL-2.0
* Replaces Pano (November 6, 2026)
* Inserts a chosen entry where you were typing (Drop to Shell's post, November 13, 2026); works on its own, without Drop to Shell

### GTK4 Desktop Icons Next Generation (DING)
Include but toggle off by default.

### GSConnect
https://extensions.gnome.org/extension/1319/gsconnect/

### Connect integration idea

Consider merging the phone-integration ("Connect") functionality. GSConnect covers Android through the KDE Connect protocol; [Tether](https://github.com/zackb/tether) ([write-up](https://zackbartel.com/blog/2026/08/tether/)) brings Apple Continuity features to the Wayland desktop for iPhone users: clipboard sync, file transfer, SMS and iMessage, notification mirroring, and calls. Tether is MIT-licensed and runs as a daemon with a GTK app rather than a GNOME extension, shipping as a Flatpak and an AppImage, so it would ride Flathub rather than the package set. The entry records the idea of a single Connect story covering both phone ecosystems, not a decision to adopt Tether.

### Just Perfection

* Click to Close Overview (replaces click-to-close-overview@l3nn4rt.github.io)
* Restore Desktop Thumbnails (replaces GNOME 4X UI Improvements)

### Rounded Window Corners Reborn

* Unround the bottom corners (not currently possible)
* Corner radius of 15 for consistency between GTK3 and libadwaita
* https://github.com/flexagoon/rounded-window-corners/issues/36
* https://gitlab.gnome.org/GNOME/gnome-shell/-/issues/7903
* https://github.com/flexagoon/rounded-window-corners/issues/43

### Quick Settings Tweaks

### Search Light

* Consider
* Integrate with Blur My Shell

### Tiling Shell

* Switch desktop while dragging and hovering on screen edge
* Switch desktops with keyboard while dragging

### Transparent Window Moving

* Switch desktop while dragging and hovering on screen edge
* Switch desktops with keyboard while dragging

### ScreenToSpace

### Weather o'Clock

## Also installed in the August 2025 prototype

These extensions were installed on Galactus in August 2025, during the hand-configured openSUSE Aeon prototype, and are not recorded elsewhere in these notes. The list records what was installed, not a Benchtop selection.

* Bluetooth Quick Connect (bluetooth-quick-connect@bjarosze.gmail.com)
* Dim Background Windows (dim-background-windows@stephane-13.github.com)
* Reboot to UEFI (reboottouefi@ubaygd.com)
* Tiling Assistant (tiling-assistant@leleat-on-github)
* Tray Icons: Reloaded (trayIconsReloaded@selfmade.pl)
* Vitals (Vitals@CoreCoding.com)

## Under Consideration

* Caffeine (caffeine@patapon.info)
* Clipboard Indicator (clipboard-indicator@tudmotu.com)
* AlphabeticalAppGrid@stuarthayhurst
* Always-Show-Titles-In-Overview
* applications-overview-tooltip@RaphaelRochet
* Gnome Fuzzy App Search
* ESP (Extension Search Provider)
* WSP (Window Search Provider)
* https://extensions.gnome.org/extension/8226/maximize-to-empty-workspace-2025/
* WiFi QR Code
* Bangs Search
