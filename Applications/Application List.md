# Application candidates

Working selections and applications to evaluate for Benchtop. This is not a manifest of installed packages. Feature requests below describe the behavior wanted from an application, not necessarily a feature it already has.

## Web

* Brave
* Bitwarden
* Linkwarden

## Publications

* Books — Calibre
* PDFs — Okular
* Papers — Zotero
* Ebooks - KOReader

## Education

* Atlas — Marble; GNOME Maps
* Encyclopedia — Kiwix
* Flash Cards — Anki

## Office

* Office — LibreOffice
* Notes — Obsidian

## Broadcasting

* Audacity
* OBS Studio

## Digital Audio and Video

* Kdenlive
* Blender
* Natron

## Digital Graphics

* digiKam
* Inkscape
* GIMP
* Krita
* Darktable

## Desktop Publishing

* Scribus

## CAD

* LibreCAD (2D)
* FreeCAD (3D)
* OpenSCAD (3D Printing)

## LEGO CAD

* LDraw
* LeoCAD
* LDCad
* LPub3D

## Science

### Astronomy

* Kstars — Astronomy/Astrophotography
* Stellarium — Sky Atlas

### Mathematics

* Matlab → Octave
* Minitab → JASP
* R Studio
* SPSS → PSPP
* Wolfram Language → [Woxi](https://github.com/ad-si/Woxi)

### Data Science

* Spyder

## Developer

* Cyberchef
* Kate
* VSCode
* virt-manager
* [Bustle](https://apps.gnome.org/Bustle/) — D-Bus Visualization
* [Meld](https://meldmerge.org/) — visual diff and merge for files, folders and version control (GNOME; on Flathub as org.gnome.meld)
* An open-source alternative to DataGrip, JetBrains' database IDE, which is not open source. Candidates to evaluate: DBeaver Community, Beekeeper Studio Community Edition. VERIFY their licenses and Flathub packages

## Security

* [Seer](https://github.com/epasveer/seer) — GDB GUI
* Wireshark
* Ghidra
* imHex

## Simple Apps (GTK)

* Screenshot/Snip Tool — Printscr
    * Printscr brings up the overlay selection
    * Shift+Printscr takes a screenshot of your entire desktop

## Advanced Apps (QT)

### Dolphin

File-manager candidate.

### Kate
Needs text filters:

* Add/Remove Line Numbers, Prefix/Suffix Lines, Sort Lines
* Process Duplicate Lines, Process Lines Containing, Remove Blank Lines
* Canonize, Text Merge, Increase/Decrease Quote Level, Strip Quotes
* Zap Gremlins, Convert Escape Sequences
* Convert Spaces to Tabs / Tabs to Spaces
* Strip Trailing Whitespace, Normalize Line Endings, Normalize Spaces
* Precompose/Decompose Unicode

### Okular

* Needs to be able to reorder, merge, and delete PDF pages
* Need to be able to change font on in-line comments
* Need to be able to import and attach signature to files

### Gwenview

* Need to be able to select OCRed text in images

## KDE Apps

* Dolphin, Konsole, Okular, Gwenview
* digiKam, Krita, Kdenlive, Elisa
* KDE Itinerary

## Maui Apps
Evaluate the [Maui](https://mauikit.org/) applications for the tablet interface:

* Index — File manager
* Nota — Text editor
* Station — Terminal emulator
* Clip — Video player
* Pix — Image viewer
* Shelf — Document viewer
* Communicator — Contacts manager

Check their desktop and touch behavior on the supported hardware before selecting them.

## X-Apps (Linux Mint)
X-Apps (https://github.com/linuxmint) are GTK apps from the Linux Mint project designed to be desktop-agnostic:

* Xed — Text editor (fork of gedit/pluma)
* Xreader — Document viewer (PDF, ePub, etc.)
* Xviewer — Image viewer
* Xplayer — Video player
* Pix — Image organizer

Compare their behavior with the selected GNOME and KDE applications before adding another application for the same task.

## Unsorted

* Thunderbird
* GDM Settings
* Balena Etcher
* KDE Education
* Tokodon, AudioTube, PlasmaTube, Kasts
* Neochat
* Zeal (documentation browser)
* Footage, Impression, Inspector
* Frog (OCR)
* Newsflash (RSS)
* Szyszka (batch renamer)
* Marker (Markdown editor)
* Librum (ebook reader)
* KiCad (electronics CAD)
* Mission Center (system monitor)
* RustDesk / RustConn

## Closed Source

* LM Studio
* Obsidian

## Equivalents Needed

* OmniFocus → Planify (https://github.com/alainm23/planify) — GTK4/libadwaita task manager with Todoist sync
* OmniGraffle → Dia (basic), yEd (free but not open source), or draw.io / diagrams.net (Electron app / web-based). For a native option, consider Minder (https://github.com/phase1geo/Minder) for mind maps, or simply use Inkscape for more complex diagrams.

## App installation

Use GNOME Software for Flatpak and the planned AppImage backend, including AppImageHub discovery. AppImages install in `~/Applications/` with generated `.desktop` entries. Bazaar is not needed. See [Application management](../System%20Core/Pre-Packaged%20Environment.md#application-management) for the intended behavior.

The links below are implementation references and earlier candidates, not a list of additional stores to ship:

* https://github.com/linuxmint/webapp-manager
* https://www.appimagehub.com/
* https://www.appimagehub.com/p/1228228
* https://github.com/prateekmedia/appimagepool
