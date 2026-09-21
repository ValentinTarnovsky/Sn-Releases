# Installation

1. Download the latest `snbattlepass-v*` release from [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases/releases?q=snbattlepass-).
2. Place the `.jar` file into your server's `plugins/` folder.
3. Start the server once. The plugin creates `plugins/.Sn-License/license.yml`.
4. Paste your Sn license key into that file.
5. Restart the server. The plugin validates the key at startup.

{% hint style="info" %}
The license key lives in `plugins/.Sn-License/license.yml`, shared by every plugin of the Sn bundle on this server, so one key unlocks all of them. The plugin refuses to enable without a valid key.
{% endhint %}

{% hint style="warning" %}
Requires **SnLib** installed (`depend: [SnLib]`), version 1.28.0 or later, and one of **EdTools** or **RivalHarvesterHoes**. Passive XP comes from the block breaks of one of those two, so the plugin will not enable without SnLib or with neither of them installed. With both installed, `xp.passive.source` in `config.yml` picks one (EdTools by default).
{% endhint %}

## Dependencies

| Plugin | Required |
|--------|----------|
| SnLib | Yes |
| EdTools **or** RivalHarvesterHoes | One of the two |
| SnCrates | No |
| SnGens | No |
| SnEnvoys | No |
| PlaceholderAPI | No |
| RivalPets | No |
| SnPets | No |

The plugin writes `config.yml`, `challenges.yml`, `rewards.yml`, `lang/messages_en.yml` and `guis/*.yml` on first boot, so there is nothing to create by hand.
