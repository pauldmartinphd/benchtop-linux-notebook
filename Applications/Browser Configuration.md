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
