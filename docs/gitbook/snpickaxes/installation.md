# Installation

1. Download the latest `snpickaxes-v*` release from [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases/releases?q=snpickaxes-).
2. Place the `.jar` file into your server's `plugins/` folder.
3. Start the server once. The plugin creates `plugins/.Sn-License/license.yml`.
4. Paste your Sn license key into that file.
5. Restart the server. The plugin validates the key at startup and generates its own `config.yml`, `items.yml` and `lang/messages_en.yml`.

{% hint style="warning" %}
Requires **SnLib** installed (`depend: [SnLib]`), version 1.20.3 or later.
{% endhint %}

{% hint style="info" %}
The license key lives in `plugins/.Sn-License/license.yml`, shared by every plugin in the bundle. One key unlocks all of them, so if you already run another bundle plugin the key is already in place. The plugin refuses to enable without a valid key, and it needs outbound internet access at startup to verify one.
{% endhint %}

## Dependencies

| Plugin | Required |
|--------|----------|
| SnLib | Yes |
| WorldGuard | No, only for region restrictions |

**Requirements:** Java 21 or later, Paper 1.20.x or 1.21.x.
