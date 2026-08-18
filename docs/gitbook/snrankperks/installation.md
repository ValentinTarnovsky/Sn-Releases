# Installation

1. Download the latest `snrankperks-v*` release from [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases/releases?q=snrankperks-).
2. Place the `.jar` file into your server's `plugins/` folder.
3. Start the server once. The plugin creates `plugins/.Sn-License/license.yml` if it does not
   already exist.
4. Paste your Sn license key into that file.
5. Restart the server. The plugin validates the key at startup.

{% hint style="info" %}
SnRankPerks is sold as part of a bundle: the license key lives in the shared
`plugins/.Sn-License/license.yml` file, not in the plugin's own folder. One key unlocks every
plugin in the bundle. The plugin refuses to enable without a valid key.
{% endhint %}

{% hint style="warning" %}
Requires **SnLib** installed (`depend: [SnLib]`), version 1.28.0 or later.
{% endhint %}

## Dependencies

| Plugin | Required |
|--------|----------|
| SnLib | Yes |
| LuckPerms | No |
| PlaceholderAPI | No |
| TAB | No |
