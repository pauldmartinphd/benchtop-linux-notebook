# Browser Configuration

## Brave

Not part of TCBL (November 6, 2026); tc-benchtop-settings no longer ships the policy below. For reference: Flathub's Brave (`com.brave.Browser`) does read the host's `/etc/brave/policies`, through `--filesystem=host-etc` and its launcher script.

The selected policy disables Rewards, Wallet, VPN, AI Chat, and Tor, and leaves DNS-over-HTTPS in automatic mode:

```json
{
    "BraveRewardsDisabled": true,
    "BraveWalletDisabled": true,
    "BraveVPNDisabled": true,
    "BraveAIChatEnabled": false,
    "TorDisabled": true,
    "DnsOverHttpsMode": "automatic"
}
```

Note: BraveVPNDisabled is a boolean policy, so it takes `true`, not `1`

## Firefox (Default Browser)

Decided November 6, 2026: Firefox is TCBL's browser, installed from Flathub for every user (`tcbl-default-applications.service`), because it is what most people expect. With no other browser installed, GNOME uses it as the default without further configuration.

#TODO — The browser: Firefox ESR as a Flatpak, with the Firefox GNOME theme

Decided November 7, 2026. Until it is done, each user gets Flathub's Firefox (rapid release), unthemed.

- Firefox ESR, as a Flatpak, for every user (`tcbl-default-applications.service`, once). Mozilla publishes no ESR Flatpak and Flathub has none (Mozilla bug 1779150, unconfirmed), so it has to come from the TCBL Flatpak repository, built from Mozilla's ESR binaries. Firefox ESR stays free; the paid Firefox Enterprise (October 13, 2026) is a management console and support around it.
- The Firefox GNOME theme (rafaelmardojai/firefox-gnome-theme), installed into each user's profile the way Add Water does it, and kept up to date, without Add Water's application. The theme's release has to match the ESR version (tag v153 for ESR 153), not the newest release that Add Water installs.
- Yaru: the theme follows Adwaita only, but takes its accent from Firefox's `AccentColor`, which follows the GTK theme, so under Yaru it should get Yaru's accent by itself (the removed Yaru variant changed only the accent). This needs the Yaru GTK 3 Flatpak extensions from the TCBL Flatpak repository; without them the Flatpak falls back to Adwaita's blue. If `AccentColor` does not follow, override the theme's `--gnome-accent-bg` and `--gnome-accent`. To check on the laptop.
- Preinstalled extensions and preferences: the TODO below.
- Details in [Theming](../UX%20and%20Desktop/Theming.md) (the Firefox TODO).

#TODO — Firefox: extensions packaged into TCBL's Firefox ESR Flatpak

Decided November 7, 2026: the extensions come with TCBL's own Firefox ESR Flatpak (the TODO above), not from each user's profile setup.

- uBlock Origin: `uBlock0@raymondhill.net`, GPL-3.0-only, <https://github.com/gorhill/uBlock>
- Shut Up (comment blocker): `{b6840179-45a1-4a2d-a4ef-e2815c1faa28}`, MIT, <https://github.com/RickyRomero/shut-up-webextension>
- Others: to be listed. Each needs a license that allows redistribution, and Mozilla's signed build from addons.mozilla.org (Firefox loads only signed extensions).
- How, to decide: the signed `.xpi` files in the Flatpak's `distribution/extensions` folder, which Firefox installs into new profiles, or `ExtensionSettings` in a `policies.json` inside the Flatpak (installation mode and install URL per extension); the same `policies.json` can carry the preferences below. Not decided: whether users may remove them, and whether they then update from addons.mozilla.org or only with the Flatpak.

### Fix Scrolling

    apz.gtk.pangesture.delta_mode=2
    apz.gtk.pangesture.pixel_delta_mode_multiplier=25
[Bug](https://bugzilla.mozilla.org/show_bug.cgi?id=1752862)

### Switch Browser Cache from Disk (SSD) to RAM

    browser.cache.disk.enable=false
    browser.cache.disk_cache_ssl=false

### Font Selection
#TODO

### Browser Cache
#TODO — Investigate optimal cache settings
