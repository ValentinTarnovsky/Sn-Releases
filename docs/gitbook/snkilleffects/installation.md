# Installation

1. Download the latest `snkilleffects-v*` release from [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases/releases?q=snkilleffects-).
2. Place the `.jar` file into your server's `plugins/` folder.
3. Start the server once. The plugin creates `plugins/.Sn-License/license.yml`.
4. Paste your Sn license key into that file.
5. Restart the server. The plugin validates the key at startup.

{% hint style="warning" %}
Requires **SnLib** installed (`depend: [SnLib]`), version 1.20.0 or later.
{% endhint %}

{% hint style="info" %}
SnKillEffects is part of the Sn bundle. The key lives in `plugins/.Sn-License/license.yml`, one shared file for every bundle plugin on the server, so a single key unlocks the whole pack. The plugin refuses to enable without a valid key.
{% endhint %}

## Dependencies

| Plugin | Required |
|--------|----------|
| SnLib | Yes |
| PlaceholderAPI | No |
