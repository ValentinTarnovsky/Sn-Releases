# Installation

1. Download the latest `snclans-v*` release from [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases/releases?q=snclans-).
2. Place the `.jar` file into your server's `plugins/` folder.
3. Start the server once. SnClans creates a shared license file at `plugins/.Sn-License/license.yml`.
4. Paste your Sn license key into that file, replacing the placeholder line.
5. Restart the server. SnClans validates the key at startup and then enables.

{% hint style="warning" %}
Requires **SnLib** installed (`depend: [SnLib]`), version 1.6.0 or later.
{% endhint %}

{% hint style="info" %}
SnClans is licensed. The key in `plugins/.Sn-License/license.yml` is shared by every bundled Sn plugin on the server, so you paste it once. Without a valid key the plugin refuses to enable.
{% endhint %}

## Dependencies

| Plugin | Required |
|--------|----------|
| SnLib | Yes |
| PlaceholderAPI | No (optional, enables placeholders) |
| WorldGuard | No (optional, enables region gating) |
| DecentHolograms | No (optional, alternative rally hologram backend) |

{% hint style="info" %}
When an optional plugin is absent, the matching feature degrades gracefully. Placeholders stay raw, region rules are skipped, and the rally hologram falls back to SnLib's built-in renderer.
{% endhint %}
