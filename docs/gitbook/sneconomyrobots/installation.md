# Installation

1. Download the latest `sneconomyrobots-v*` release from [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases/releases?q=sneconomyrobots-).
2. Place the `.jar` file into your server's `plugins/` folder.
3. Start the server once. The plugin creates `plugins/.Sn-License/license.yml`.
4. Paste your Sn license key into that file.
5. Restart the server. The plugin validates the key at startup.

{% hint style="info" %}
SnEconomyRobots is a bundle plugin. Its licence lives in the shared file
`plugins/.Sn-License/license.yml`, not in the plugin's own folder, so one bundle key unlocks every
plugin of the bundle on that server. The plugin refuses to enable without a valid key.
{% endhint %}

{% hint style="warning" %}
Requires **SnLib** installed (`depend: [SnLib]`), version 1.20.2 or later.
{% endhint %}

## Dependencies

| Plugin | Required |
|--------|----------|
| SnLib | Yes |
| EdTools | Yes |
| PlaceholderAPI | No |
| Vault | No |

SnEconomyRobots targets Paper and supports both 1.20.x and 1.21.x.
