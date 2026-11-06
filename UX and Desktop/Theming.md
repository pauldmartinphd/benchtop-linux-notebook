# Theming and Visual Design

## In TCBL (November 5, 2026)

The themes below are built from their GitHub upstreams in benchtop-themes (see [Build Configuration](../Build%20Configuration.md)), all installed by the base pattern.

- GTK 3: adw-gtk3 is the default theme (tc-benchtop-settings' GSettings override), so GTK 3 applications match libadwaita ones. Flatpak applications cannot see the host's themes; `tcbl-themes.service` (a user service, at login and daily) installs Flathub's `org.gtk.Gtk3theme.adw-gtk3` and `adw-gtk3-dark` into each user's installation, once.
- Yaru, Ubuntu's themes, version 26.10.3, in place of openSUSE's 24.04.0: GTK 3 and 4, GNOME Shell, icons and cursors, sounds, GtkSourceView styles, in every accent color, and upstream's "Yaru" session in the login screen's session list. Inside Flatpak applications only Flathub's Yaru extensions of April 2022, without the accent colors, exist; a TCBL Flatpak repository for them is planned.
- Qt: Kvantum, with KvLibadwaita (the default) and KvYaru-Colors. Host applications get the Kvantum style from systemd's user environment (`QT_STYLE_OVERRIDE=kvantum-auto`, TCBL's patched kvantum, below); Flatpak applications get `kvantum` from a system-wide Flatpak override, which also lets them read `~/.config/Kvantum` (Flatpak binds it into each application's own configuration folder). `tcbl-themes.service` installs Flathub's `org.kde.KStyle.Kvantum` for each KDE runtime branch the user has, copies the Kvantum themes into `~/.config/Kvantum`, and selects KvLibadwaita when the user has chosen no theme.
- Dark style: Benchtop Theme Switcher, TCBL's fork of Legacy (GTK3) Theme Scheme Auto Switcher, switches the GTK 3 theme (`Name`, `Name-dark`) and the Kvantum theme (`Name`, `NameDark`) with the system's style. GTK 3 applications and Qt 6 applications on the host change at once (`kvantum-auto`); Qt 5 applications and Flatpak Qt applications change when they start.
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

### kvantum-auto (November 5, 2026)

- Kvantum reads its theme once, when a Qt application starts; it does not watch `kvantum.kvconfig` or the system's color scheme.
- TCBL's patch (`kvantum-auto-color-scheme.patch`, in benchtop-themes `kvantum/`, on openSUSE Factory's package) adds the style key `kvantum-auto`. With Qt 6.5 or later, it loads the theme's Dark variant when `QStyleHints::colorScheme()` is dark, as `kvantum-dark` does, and when the scheme changes it replaces the application's style with a new one, so running applications switch. It acts only while it is the application's style. With Qt 5 the key behaves as `kvantum`, so that a global `QT_STYLE_OVERRIDE` does not send Qt 5 applications to Fusion. The keys `kvantum` and `kvantum-dark` are unchanged.
- Tested (November 5) against Qt 6.12, Tumbleweed's version, under Xvfb with Qt's GNOME platform theme and a test theme: light to dark to light switched the palette each time and deleted the old style; after the application changed to Fusion, nothing followed. The Qt 5 build loads the light theme. Not tested: the scheme coming from the real desktop portal (the application requested it in the test). Qt 6.12's GNOME theme reads `org.freedesktop.appearance color-scheme` from the portal, asynchronously, so an application may start light and reload dark a moment later; whether that is visible is not known.
- A theme without a Dark variant (for example KvYaru-BlueLight) stays as it is.
- Possible race (not observed): Benchtop Theme Switcher still rewrites `theme=` in `kvantum.kvconfig` for Flatpak applications. On a change from dark to light, a host application that reloads before the file is rewritten loads `KvLibadwaitaDark` by name and stays dark until the next change or restart. A change from light to dark is not affected.
- Upstream: in tsujan/Kvantum issue #1148, tsujan wrote that `kvantum-dark` is at odds with Kvantum's flexibility and that he had wanted to remove it, so a key on the same mechanism may be declined. TCBL maintains the patch either way.

#TODO — Kvantum: build `org.kde.KStyle.Kvantum` with the patch in the TCBL Flatpak repository (with the Yaru GTK 3 extensions), set the Flatpak override to `kvantum-auto`, and remove the Kvantum part of Benchtop Theme Switcher, which ends the race above.

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
