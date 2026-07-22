# Installation

1. Download the latest `snelementalgems-v*` release from [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases/releases?q=snelementalgems-).
2. Place the `.jar` file into your server's `plugins/` folder.
3. Start the server once. SnElementalGems creates the shared bundle license file at `plugins/.Sn-License/license.yml`.
4. Paste your Sn bundle key into that file, replacing the placeholder line.
5. Restart the server. SnElementalGems validates the key at startup and then enables.

{% hint style="warning" %}
Requires **SnLib** installed (`depend: [SnLib]`), version 1.11.0 or later.
{% endhint %}

{% hint style="info" %}
SnElementalGems uses a bundle license. The key lives in the shared `plugins/.Sn-License/license.yml`, so one bundle key unlocks every bundle plugin on the server. Without a valid key the plugin refuses to enable.
{% endhint %}

## Dependencies

| Plugin | Required |
|--------|----------|
| SnLib | Yes |
| PlaceholderAPI | No (optional, enables placeholders) |
| Vault | No (optional, registers gems as the server economy) |
| HeadDatabase | No (optional, custom head for the gem item) |
| RoseStacker | No (optional, stacked-mob drop sizing) |
| WildStacker | No (optional, stacked-mob drop sizing) |
| WildTools | No (optional, harvester-hoe drops) |
