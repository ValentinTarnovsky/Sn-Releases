# Installation

1. Download the latest `snlootboxes-v*` release from [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases/releases?q=snlootboxes-).
2. Place the `.jar` file into your server's `plugins/` folder.
3. Start the server once; the plugin creates `plugins/.Sn-License/license.yml`.
4. Paste your Sn license key into that file.
5. Restart the server; the plugin validates the key at startup.

{% hint style="warning" %}
Requires **SnLib** installed (`depend: [SnLib]`), version 1.28.0 or later.
{% endhint %}

{% hint style="info" %}
Your key lives in the shared `plugins/.Sn-License/license.yml` file. The plugin refuses to enable without a valid key. One bundle key unlocks every Sn bundle plugin on the server.
{% endhint %}

## Dependencies

| Plugin | Required |
|--------|----------|
| SnLib | Yes |
| PlaceholderAPI | No |
| WorldGuard | No |
