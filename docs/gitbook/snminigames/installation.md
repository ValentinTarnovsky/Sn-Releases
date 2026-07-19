# Installation

1. Download the latest `snminigames-v*` release from [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases/releases?q=snminigames-).
2. Place the `.jar` file into your server's `plugins/` folder.
3. Start the server once; the plugin creates `plugins/.Sn-License/license.yml`.
4. Paste your Sn license key into that file.
5. Restart the server; the plugin validates the key at startup.

{% hint style="warning" %}
Requires **SnLib** installed (`depend: [SnLib]`), version 1.7.1 or later.
{% endhint %}

{% hint style="info" %}
SnMiniGames is part of the Sn bundle: one shared key in `plugins/.Sn-License/license.yml` unlocks every bundle plugin on the server. The plugin refuses to enable without a valid key.
{% endhint %}

## Dependencies

| Plugin | Required |
|--------|----------|
| SnLib  | Yes      |
| PlaceholderAPI | No (optional) |
