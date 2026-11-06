# Browser Configuration

## Brave

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

Decided November 6, 2026: Firefox is TCBL's browser, installed from Flathub for every user (`tcbl-firefox.service`), because it is what most people expect. With no other browser installed, GNOME uses it as the default without further configuration.

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
