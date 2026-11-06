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

#TODO — Firefox ESR instead of the rapid release

- Wanted: Firefox ESR. Mozilla publishes no ESR Flatpak, and Flathub has none (Mozilla bug 1779150, unconfirmed). Tumbleweed has `firefox-esr` (153.6.0 on November 6) as an rpm, and Mozilla's own zypper repository has one too.
- Options: the Tumbleweed rpm in the base pattern (an exception to no GUI applications in the pattern; system-wide policies in one host file), or Mozilla's ESR binaries repackaged as a Flatpak in the TCBL Flatpak repository. Paused November 6; until then each user gets Flathub's Firefox.

#TODO — Firefox: preinstalled extensions (an ad blocker such as uBlock Origin)

- Possible route: Firefox's enterprise policies (`ExtensionSettings`, with an addons.mozilla.org install URL), in a system-wide `policies.json` for the Flatpak, which Firefox reads from the `org.mozilla.firefox.systemconfig` extension point since Firefox 115 (Mozilla bug 1814897): `/var/lib/flatpak/extension/org.mozilla.firefox.systemconfig/x86_64/stable/policies/policies.json`, with preference files in `defaults/pref` beside it. The same place could carry the preferences below.
- To check: whether a Firefox installed in the user's own installation, as TCBL installs it, reads that system-wide extension. Which extensions, and whether users may remove them, is not decided.

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
