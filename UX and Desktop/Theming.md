# Theming and Visual Design

## In TCBL (November 5, 2026)

The themes below are built from their GitHub upstreams in benchtop-themes (see [Build Configuration](../Build%20Configuration.md)), all installed by the base pattern.

- GTK 3: adw-gtk3 is the default theme (tc-benchtop-settings' GSettings override), so GTK 3 applications match libadwaita ones. Flatpak applications cannot see the host's themes; `tcbl-themes.service` (a user service, at login and daily) installs Flathub's `org.gtk.Gtk3theme.adw-gtk3` and `adw-gtk3-dark` into each user's installation, once.
- Yaru, Ubuntu's themes, version 26.10.3, in place of openSUSE's 24.04.0: GTK 3 and 4, GNOME Shell, icons and cursors, sounds, GtkSourceView styles, in every accent color, and upstream's "Yaru" session in the login screen's session list. Inside Flatpak applications only Flathub's Yaru extensions of April 2022, without the accent colors, exist; a TCBL Flatpak repository for them is planned.
- Qt: Kvantum, with KvLibadwaita (the default) and KvYaru-Colors. Host applications get the Kvantum style from systemd's user environment (`QT_STYLE_OVERRIDE=kvantum`); Flatpak applications from a system-wide Flatpak override, which also lets them read `~/.config/Kvantum` (Flatpak binds it into each application's own configuration folder). `tcbl-themes.service` installs Flathub's `org.kde.KStyle.Kvantum` for each KDE runtime branch the user has, copies the Kvantum themes into `~/.config/Kvantum`, and selects KvLibadwaita when the user has chosen no theme.
- Dark style: Benchtop Theme Switcher, TCBL's fork of Legacy (GTK3) Theme Scheme Auto Switcher, switches the GTK 3 theme (`Name`, `Name-dark`) and the Kvantum theme (`Name`, `NameDark`) with the system's style. GTK 3 applications change at once, Qt applications when they start (see the Kvantum TODO below).
- Fonts: Ubuntu Sans and Ubuntu Sans Mono, in [Fonts](Fonts.md).

## Adwaita Ecosystem

* https://github.com/eylles/adw-gtk2-colorizer
* https://github.com/lassekongo83/adw-gtk3
* https://github.com/dp0sk/adw-gimp3
* https://github.com/RichardSepsi/adw-inkscape
* https://github.com/nukusaba/Libadwaita-KDE/
* https://github.com/GabePoel/KvLibadwaita
* https://github.com/rafaelmardojai/firefox-gnome-theme
* https://github.com/rafaelmardojai/thunderbird-gnome-theme
* https://github.com/tkashkin/Adwaita-for-Steam
* https://github.com/piousdeer/vscode-adwaita
* https://github.com/ricewind012/discord-gnome-theme
* https://github.com/birneee/obsidian-adwaita-theme

## Yaru Theme

* https://github.com/ubuntu/yaru
* Reference for distinguishing the active window from inactive windows
* The recorded example uses a black headerbar on the active window and white on inactive windows

## Kvantum (Qt)

#TODO — Kvantum: follow the system's dark style in running applications

- Kvantum reads its theme once, when a Qt application starts; it does not watch `kvantum.kvconfig` or the system's color scheme. Benchtop Theme Switcher (benchtop-themes) switches the theme in `kvantum.kvconfig` with the system style, but running Qt applications change only when they are restarted; GTK 3 applications change at once.
- The patch: since Qt 6.5, Qt reports the system's color scheme (`QStyleHints::colorScheme`) and tells running applications when it changes. Kvantum would load the theme's Dark variant when the scheme is dark, as its `kvantum-dark` style key already does at startup, and repolish when it changes. Not confirmed: that Qt 6 applications on GNOME and in Flatpak get the scheme from the desktop portal. Qt 5 has no color scheme, so Qt 5 applications would keep switching only at startup.
- Propose it upstream (tsujan/Kvantum), for Paul to file, and carry it meanwhile in a TCBL build of openSUSE's kvantum for host applications. Flatpak applications get it when Flathub's `org.kde.KStyle.Kvantum` is updated. Then remove the Kvantum part of Benchtop Theme Switcher.

## macOS-Inspired

* https://github.com/vinceliuice/MacTahoe-gtk-theme
* https://news.ycombinator.com/item?id=46031208

## Icon Set

* https://github.com/somepaulo/MoreWaita

## Window Decoration Improvements

* Dim parent windows with detached modals
* Tooltips on all apps
* App icon on all window summaries
* Window/tab preview on hover; window preview on window-tab
* App icon on alt-tab
