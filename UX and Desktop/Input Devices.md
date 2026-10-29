# Input Devices

## Touchpad / Kinetic Scrolling

The aim is consistent kinetic scrolling across applications, with a way to adjust speed for different touchpads and touchscreens. The original notes question whether more of that behavior should live in the compositor rather than being implemented separately by applications and toolkits. The proposal also needs to prevent momentum from transferring unexpectedly between windows.

Input passes through libinput, the display/compositor stack, and the application toolkit. The exact responsibilities need to be traced for the target applications before choosing an implementation.

### Momentum algorithm references

The collected PastryKit/iOS discussions describe decay factors of 0.9 and 0.95 per frame. One reconstruction uses 1:1 movement during touch, estimates momentum from pixels divided by swipe time, clamps it to zero below 10 pixels or 0.5 units of time, then multiplies by 0.95 each frame. Those are source descriptions with unspecified timing units, not a verified account of current macOS or iOS behavior. The [imported notes](../Reference/Imported%20Notes.md) preserve the excerpts.

* https://news.ycombinator.com/item?id=39607747
* https://stackoverflow.com/questions/38619717/need-help-dissecting-and-recreating-the-perfect-scroll-easing-based-on-pastrykit
* https://medium.com/homullus/recreating-native-ios-scroll-and-momentum-2906d0d711ad

## Keyboard Shortcuts

### Idea: Modal Keys

* Window key for Window Management
    * Window key: launch overview
    * Window-tab: switch between windows
    * Window-left/right: tiling
    * Window-up/down: minimize/maximize
* Alt key for Environment management (all apps)
    * Alt key: open app view
    * Alt-tab: switch between applications
* Ctrl key for Running Application (control focused app)
    * Ctrl-tab: switch between application tabs
* Caps key for controlling terminal

### Idea: Emacs text keys, as on macOS

This records an idea, not a decision.

macOS gives Cocoa text fields a subset of Emacs's editing keys on Control: Control-A and Control-E go to the start and end of the line, Control-B, F, P and N move the cursor, Control-D and Control-H delete forward and back, Control-K cuts to the end of the line into a kill buffer that is separate from the clipboard, Control-Y puts it back, Control-T swaps the characters around the cursor, and Control-O opens a new line after the cursor. AppKit reads them from its `StandardKeyBinding.dict`, and a user can add more in `~/Library/KeyBindings/DefaultKeyBinding.dict`. They work because Command, not Control, carries application commands. Apps that do not use AppKit's text views only have them if they implement them.

On GNOME, Control is the application command key (select all, find, new, print, new tab, open, and so on), the same role it has in Modal Keys above, so the Emacs keys collide with it. What TCBL has now:

* Terminal: bash's readline uses Emacs keys by default, with a kill ring (Control-K, Control-Y, and Alt-Y for earlier kills). Control-P and Control-N step through history, and Control-O runs the line and fetches the next one from history, not open-line.
* GTK 3 apps: `gsettings set org.gnome.desktop.interface gtk-key-theme 'Emacs'` loads GTK's Emacs key theme (`gtk/gtk-keys.css.emacs`). It covers text entries and text views only: Control-A, E, B and F, Alt-B and Alt-F, Control-P and N in multi-line text, Control-D, Control-H, Control-K and Control-U. Control-K deletes without keeping the text, Control-Y pastes the clipboard, there is no Control-T or Control-O, and in those widgets Control-A no longer selects all.
* GTK 4 and libadwaita apps, which are most of GNOME's own: no key themes. GTK 4 removed binding sets and the CSS key bindings they used, and its settings have no key theme.
* Firefox, Electron and Qt apps: their own bindings; not checked.

The macOS arrangement in every app would need a remapper below the toolkits (keyd or xremap, for example) that moves application commands to Super and turns Control-letter into navigation keys (Home, End, the arrows, Delete, BackSpace, and Shift-End then Delete for Control-K), except in terminals and Emacs. Toshy and Kinto.sh do this to give Linux a macOS layout. The costs: Super is GNOME Shell's key, and the Window key in Modal Keys above; skipping terminals needs the remapper to know the focused app, which on GNOME under Wayland probably means a Shell extension (not verified); and there is still no kill buffer, so Control-K would overwrite the clipboard.

#TODO — Decide what Control means: application commands (Modal Keys above, and GNOME's default) or text editing, with application commands on Super (macOS). The two ideas conflict on Control and on Super.

* GTK 3 Emacs key theme: https://gitlab.gnome.org/GNOME/gtk/-/blob/gtk-3-24/gtk/gtk-keys.css.emacs
* GTK 4 migration, key bindings: https://docs.gtk.org/gtk4/migrating-3to4.html

### Mouse Actions

* Middle Click → Minimize
* Double Click → Maximize

### CLI Interface UI Paradigms

* Space shows Command Palette (on desktop for GUI or in modal-CLI apps)
* Mode displayed on screen

## Gaming Mouse Configuration

    sudo systemctl enable --now ratbagd
* Solaar — included for managing Logitech mice along with libratbagd

## Game Controllers

* 8BitDo Ultimate Software Online — 8BitDo's controller configuration tool (button/stick remapping, profiles, firmware updates), historically Windows/Android-only; the "Online" version is announced as browser-based, which would make it usable from Linux (likely via WebHID/WebUSB) without a native app. Evaluate it alongside the controller udev rules in [Gaming Mode](../Performance/Gaming%20Mode.md).
    * https://www.reddit.com/r/linux/comments/1w0lxdh/8bitdo_announce_ultimate_software_online_to/ — link filed from title/announcement; thread not yet reviewed (Reddit blocks automated fetch). VERIFY whether it needs Chromium/WebHID and whether firmware flashing works from Linux.
