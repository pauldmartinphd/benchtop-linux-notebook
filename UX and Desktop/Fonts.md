# Fonts

## Font Rendering Configuration

### 1. Freetype2 Symlinks
Create symlinks for good-looking rendering defaults:

    ln -s /etc/fonts/conf.avail/11-lcdfilter-default.conf /etc/fonts/conf.d
    ln -s /etc/fonts/conf.avail/10-hinting-slight.conf /etc/fonts/conf.d

### 2. Fontconfig: /etc/fonts/local.conf

**Status, September 29, 2026:** Resolved. Noto is the default for serif, sans-serif and monospace: serif through OpenSUSE's own list, sans-serif and monospace through tc-benchtop-settings (below). The replacement list is not implemented, because Flatpak apps do not read the host's fontconfig configuration, and the pattern no longer carries the fonts that were there only for it (Courier Prime, Merriweather, Overpass).

Default font families:

* serif → Noto Serif (OpenSUSE's default, so no rule)
* sans-serif → Noto Sans
* monospace → Noto Sans Mono
* fantasy, cursive → no rule (only web pages use them, and browsers are Flatpak apps)

CJK text is left to fontconfig, which picks the Japanese, Korean or Chinese Noto CJK font by the language of the text.

The imported XML had missing opening and closing tags. The complete example below retains its family mappings, including entries omitted from the earlier summary. `Corberl` and `SegoeUI` are corrected to `Corbel` and `Segoe UI`. These are selected substitutions; they are not a claim of metric compatibility with every original face.

```xml
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
  <alias>
    <family>serif</family>
    <prefer>
      <family>Heuristica</family>
    </prefer>
  </alias>
  <alias>
    <family>sans-serif</family>
    <prefer>
      <family>Noto Sans</family>
      <family>Noto Sans CJK SC</family>
    </prefer>
  </alias>
  <alias>
    <family>monospace</family>
    <prefer>
      <family>Liberation Mono</family>
      <family>Noto Sans Mono CJK SC</family>
    </prefer>
  </alias>
  <alias>
    <family>fantasy</family>
    <prefer>
      <family>Signika</family>
    </prefer>
  </alias>
  <alias>
    <family>cursive</family>
    <prefer>
      <family>TeX Gyre Chorus</family>
    </prefer>
  </alias>
  <match target="pattern">
    <test name="family"><string>Arial</string></test>
    <edit name="family" mode="assign" binding="strong"><string>Liberation Sans</string></edit>
  </match>
  <match target="pattern">
    <test name="family"><string>Arial Narrow</string></test>
    <edit name="family" mode="assign" binding="strong"><string>Liberation Sans Narrow</string></edit>
  </match>
  <match target="pattern">
    <test name="family"><string>Book Antiqua</string></test>
    <edit name="family" mode="assign" binding="strong"><string>TeX Gyre Bonum</string></edit>
  </match>
  <match target="pattern">
    <test name="family"><string>Calibri</string></test>
    <edit name="family" mode="assign" binding="strong"><string>Carlito</string></edit>
  </match>
  <match target="pattern">
    <test name="family"><string>Cambria</string></test>
    <edit name="family" mode="assign" binding="strong"><string>Caladea</string></edit>
  </match>
  <match target="pattern">
    <test name="family"><string>New Century Schoolbook</string></test>
    <edit name="family" mode="assign" binding="strong"><string>TeX Gyre Schola</string></edit>
  </match>
  <match target="pattern">
    <test name="family"><string>Comic Sans MS</string></test>
    <edit name="family" mode="assign" binding="strong"><string>Signika</string></edit>
  </match>
  <match target="pattern">
    <test name="family"><string>Consolas</string></test>
    <edit name="family" mode="assign" binding="strong"><string>Droid Sans Mono Slashed</string></edit>
  </match>
  <match target="pattern">
    <test name="family"><string>Constantia</string></test>
    <edit name="family" mode="assign" binding="strong"><string>Merriweather</string></edit>
  </match>
  <match target="pattern">
    <test name="family"><string>Corbel</string></test>
    <edit name="family" mode="assign" binding="strong"><string>Merriweather Sans</string></edit>
  </match>
  <match target="pattern">
    <test name="family"><string>Courier New</string></test>
    <edit name="family" mode="assign" binding="strong"><string>Courier Prime</string></edit>
  </match>
  <match target="pattern">
    <test name="family"><string>Geneva</string></test>
    <edit name="family" mode="assign" binding="strong"><string>Noto Sans</string></edit>
  </match>
  <match target="pattern">
    <test name="family"><string>Georgia</string></test>
    <edit name="family" mode="assign" binding="strong"><string>Gelasio</string></edit>
  </match>
  <match target="pattern">
    <test name="family"><string>Helvetica</string></test>
    <edit name="family" mode="assign" binding="strong"><string>Liberation Sans</string></edit>
  </match>
  <match target="pattern">
    <test name="family"><string>Helvetica Narrow</string></test>
    <edit name="family" mode="assign" binding="strong"><string>Liberation Sans Narrow</string></edit>
  </match>
  <match target="pattern">
    <test name="family"><string>Helvetica Neue</string></test>
    <edit name="family" mode="assign" binding="strong"><string>Open Sans</string></edit>
  </match>
  <match target="pattern">
    <test name="family"><string>Impact</string></test>
    <edit name="family" mode="assign" binding="strong"><string>Oswald</string></edit>
  </match>
  <match target="pattern">
    <test name="family"><string>ITC Zapf Chancery</string></test>
    <edit name="family" mode="assign" binding="strong"><string>TeX Gyre Chorus</string></edit>
  </match>
  <match target="pattern">
    <test name="family"><string>Lucida Calligraphy</string></test>
    <edit name="family" mode="assign" binding="strong"><string>Quintessential</string></edit>
  </match>
  <match target="pattern">
    <test name="family"><string>Lucida Handwriting</string></test>
    <edit name="family" mode="assign" binding="strong"><string>Quintessential</string></edit>
  </match>
  <match target="pattern">
    <test name="family"><string>Lucida Casual</string></test>
    <edit name="family" mode="assign" binding="strong"><string>CantoraOne</string></edit>
  </match>
  <match target="pattern">
    <test name="family"><string>Lucida Console</string></test>
    <edit name="family" mode="assign" binding="strong"><string>Droid Sans Mono</string></edit>
  </match>
  <match target="pattern">
    <test name="family"><string>Lucida Sans Typewriter</string></test>
    <edit name="family" mode="assign" binding="strong"><string>Liberation Sans Mono</string></edit>
  </match>
  <match target="pattern">
    <test name="family"><string>Lucida Fax</string></test>
    <edit name="family" mode="assign" binding="strong"><string>Luxi Mono</string></edit>
  </match>
  <match target="pattern">
    <test name="family"><string>Lucida Sans</string></test>
    <edit name="family" mode="assign" binding="strong"><string>Droid Sans</string></edit>
  </match>
  <match target="pattern">
    <test name="family"><string>Lucida Grande</string></test>
    <edit name="family" mode="assign" binding="strong"><string>Droid Sans</string></edit>
  </match>
  <match target="pattern">
    <test name="family"><string>Palatino Linotype</string></test>
    <edit name="family" mode="assign" binding="strong"><string>TeX Gyre Pagella</string></edit>
  </match>
  <match target="pattern">
    <test name="family"><string>Segoe UI</string></test>
    <edit name="family" mode="assign" binding="strong"><string>WeblySleek UI</string></edit>
  </match>
  <match target="pattern">
    <test name="family"><string>Symbol</string></test>
    <edit name="family" mode="assign" binding="strong"><string>Symbola</string></edit>
  </match>
  <match target="pattern">
    <test name="family"><string>Tahoma</string></test>
    <edit name="family" mode="assign" binding="strong"><string>DejaVu Sans Condensed</string></edit>
  </match>
  <match target="pattern">
    <test name="family"><string>Times New Roman</string></test>
    <edit name="family" mode="assign" binding="strong"><string>Liberation Serif</string></edit>
  </match>
  <match target="pattern">
    <test name="family"><string>Trebuchet MS</string></test>
    <edit name="family" mode="assign" binding="strong"><string>Ubuntu</string></edit>
  </match>
  <match target="pattern">
    <test name="family"><string>Verdana</string></test>
    <edit name="family" mode="assign" binding="strong"><string>DejaVu Sans</string></edit>
  </match>
  <match target="pattern">
    <test name="family"><string>Wingdings</string></test>
    <edit name="family" mode="assign" binding="strong"><string>Symbola</string></edit>
  </match>
</fontconfig>
```

Source: bohoomil's replacement list from the infinality-bundle documentation (http://bohoomil.com/doc/05-fonts/, until ibfonts-meta-extended), by way of cryzed's Infinality-like fontconfig configuration
[Link](https://gist.github.com/cryzed/4f64bb79e80d619866ee0b18ba2d32fc)

#### Implemented in tc-benchtop-settings

##### /usr/share/fontconfig/conf.avail/59-tcbl-family-prefer.conf (linked into /etc/fonts/conf.d)

    <alias>
      <family>sans-serif</family>
      <prefer><family>Noto Sans</family></prefer>
    </alias>
    <alias>
      <family>monospace</family>
      <prefer><family>Noto Sans Mono</family></prefer>
    </alias>

Noto for sans-serif and monospace, as upstream fontconfig has set them since 2.14 (60-latin.conf), which is the configuration Flatpak runtimes ship.  OpenSUSE's fonts-config would otherwise pick Roboto and Source Code Pro.  The file sorts after local.conf (55), the user's configuration (56) and fonts-config's settings (58), so their preferences come first

The replacement list above is not implemented.  Flatpak apps get the host's fonts but not its fontconfig configuration, and TCBL's GUI apps come from Flathub, so the rules would only reach GNOME itself, which names its own fonts (Adwaita Sans and Adwaita Mono), and command-line tools.  fontconfig's own metric-compatible aliases (30-metric-aliases.conf) still replace Arial, Times New Roman, Calibri, Cambria and Georgia, in Flatpak apps too

### 3. Install Distro Fonts

    zypper in google-noto-fonts
    zypper in texlive-tex-gyre-fonts

### 4. Font package references

The names below were collected from multiple distributions, including Arch/AUR. They identify fonts to locate and package; they are not an openSUSE installation command.

* Caladea (ttf-caladea)
* Carlito (ttf-carlito)
* DejaVu (ttf-dejavu)
* Impallari Cantora (aur/ttf-impallari-cantora)
* Liberation (ttf-liberation)
* Noto (noto-fonts)
* Open Sans (ttf-opensans)
* Overpass (otf-overpass)
* Roboto (ttf-roboto)
* TeX Gyre (tex-gyre-fonts)
* Ubuntu (ttf-ubuntu-font-family)
* Courier Prime (aur/ttf-courier-prime)
* Gelasio (aur/ttf-gelasio-ib)
* Merriweather (aur/ttf-merriweather)
* Source Sans Pro (aur/ttf-source-sans-pro-ibx)
* Signika (aur/ttf-signika)

### 5. Envision Settings
[Reddit](https://www.reddit.com/r/linux/comments/1bh1x80/tweaks_for_the_freetype_font_rendering/) / [GitHub](https://github.com/maximilionus/freetype-envision)

## Ubuntu and Intel One Mono (November 5, 2026)

- openSUSE's `ubuntu-fonts` is the original Ubuntu family (0.83, 2011, with Ubuntu Mono), which the base pattern already requires. Ubuntu Sans and Ubuntu Sans Mono, Ubuntu's fonts since 23.10, are not in openSUSE Factory or M17N:fonts; benchtop-themes builds them from Canonical's GitHub releases (Ubuntu Font Licence 1.0). Each family is two packages that conflict, so that it is never listed twice: the variable fonts (`ubuntu-sans-fonts`, `ubuntu-sans-mono-fonts`, installed by default) and the static TrueType fonts (`ubuntu-sans-static-fonts`, `ubuntu-sans-mono-static-fonts`). The releases' OpenType and web fonts are the same designs again and are left out.
- Intel One Mono 1.5 (November 4) waits for openSUSE's `intel-one-mono-fonts`, at 1.4.0 in Factory and M17N:fonts; decided November 5. A plain version bump will not build: the release archive was renamed (`intel-one-mono-ttf.zip`), as was its folder, and `OFL.txt` is no longer in it.

## Open Source Font Stack (Curated)

* Atkinson Hyperlegible (accessibility)
* Fira
* GNU Unifont
* Google Noto (universal coverage)
* Hack Pro
* IBM Plex
* Intel One Mono
* Inter
* JetBrains Mono
* Microsoft Cascadia Code
* NebulaSans (from the original font list)
* Ubuntu

## Other font questions

The original vision also called for ClearType-style rendering, Microsoft TTF support, and Microsoft-compatible defaults for office documents. These remain evaluation questions alongside the substitution configuration.

## Infinality Font Rendering
Infinality and Infinality Remix were earlier candidates. Keep their references for comparison, but verify compatibility before using old FreeType patches. The working configuration above uses Fontconfig settings; freetype-envision remains an option to evaluate.

* https://github.com/pdeljanov/infinality-remix/issues/13
* https://github.com/pdeljanov/infinality-remix

## Reference Links

* https://news.ycombinator.com/item?id=30705078
* https://gist.github.com/cryzed/e002e7057435f02cc7894b9e748c5671
* https://wiki.archlinux.org/title/Font_configuration
