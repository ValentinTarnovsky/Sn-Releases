# Installation

1. Download the latest `sncustomcrafting-v*` release from [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases/releases?q=sncustomcrafting-).
2. Place the `.jar` file into your server's `plugins/` folder.
3. Start the server once. The plugin creates `plugins/.Sn-License/license.yml`.
4. Paste your Sn license key into that file.
5. Restart the server. The plugin validates the key at startup and generates its own configuration files.

{% hint style="warning" %}
Requires **SnLib** installed (`depend: [SnLib]`), version 1.24.0 or later. On an older SnLib this plugin refuses to enable and says so in the console. Update SnLib first.
{% endhint %}

{% hint style="info" %}
The license key lives in `plugins/.Sn-License/license.yml`, shared by every plugin of the bundle. One key unlocks all of them, so if you already run another Sn bundle plugin the key is already in place. The plugin refuses to enable without a valid key: no command, menu or placeholder is registered.
{% endhint %}

## Dependencies

| Plugin | Required |
|--------|----------|
| SnLib  | Yes      |
| PlaceholderAPI | No |

## Generated files

On the first successful boot the plugin writes `config.yml`, `items.yml`, `recipes.yml`, `guis/default.yml` and `lang/messages_en.yml` into `plugins/SnCustomCrafting/`.

`items.yml` and `recipes.yml` ship a working example: five items and two recipes. Rename them, restyle them or delete them. A deleted entry stays deleted.

Crafts are stored in SQLite by default, which needs no setup. To share them across a network, set `database.type: mysql` and fill in the connection block.
