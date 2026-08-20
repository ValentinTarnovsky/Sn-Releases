# Installation

SnPacketBosses is a licensed Sn plugin. Install it, boot once to generate the license file, paste your key, then restart.

1. Download the latest `snpacketbosses-v*` release from [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases/releases?q=snpacketbosses-).
2. Place the `.jar` file into your server's `plugins/` folder.
3. Start the server once. The plugin creates `plugins/.Sn-License/license.yml`.
4. Open that file and paste your Sn license key in place of the placeholder line.
5. Restart the server. The plugin validates the key at startup.

{% hint style="warning" %}
SnLib is required and must be installed as its own plugin in `plugins/`. This build targets `com.sn:snlib` 1.29.0, so run that version or newer. SnLib is never bundled inside the jar, so an older SnLib will fail the startup API check.
{% endhint %}

{% hint style="info" %}
The key file at `plugins/.Sn-License/license.yml` is shared by every Sn bundle plugin on the server. One key unlocks all of them, so you only paste it once, no matter how many bundle plugins you run. Without a valid key the plugin refuses to enable and disables itself during startup.
{% endhint %}

## Dependencies

Every entry below is a hard dependency, declared under `depend` in `plugin.yml`. The plugin declares no soft dependencies at all. If any of these is missing, the server will not load SnPacketBosses.

| Plugin | Required |
|--------|----------|
| SnLib | Yes |
| EdTools | Yes |
| packetevents | Yes |

SnLib is the engine behind the config, language, database, item and command layers. EdTools supplies the block break events that damage a boss while a player mines. packetevents sends the boss to its owner and reads the interact packets used by the armored phase.

{% hint style="warning" %}
The real platform floor is Paper 1.21.2, not 1.21. The manifest says `api-version: '1.21'`, but the plugin repositions bosses with the entity position sync packet. That packet does not exist below 1.21.2, so an older server cannot run this plugin. The floor is a hard technical limit, not a cautious guess.
{% endhint %}

{% hint style="warning" %}
Folia is not supported. The plugin does not declare `folia-supported`, so run it on Paper.
{% endhint %}

Java 21 or newer is also required, since the jar is compiled at release 21.

## What the first boot creates

On a clean install the plugin writes its own files into `plugins/SnPacketBosses/`:

| File | Purpose |
|------|---------|
| `config.yml` | Global settings, command aliases and shared defaults |
| `lang/messages_en.yml` | Every user facing message and status word |
| `bosses/guardian.yml` | A fully commented example boss you can copy |

The `bosses/` folder is seeded exactly once, on a true fresh install. After that the folder belongs to you. A boss file you delete stays deleted and is never restored on the next boot.

{% hint style="success" %}
Once the server starts cleanly, run `/packetbosses list` to confirm the example boss loaded, then `/packetbosses give <player> guardian` to hand out your first boss egg.
{% endhint %}
