# Installation

1. Download the latest `snantixray-v*` release from [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases/releases?q=snantixray-).
2. Place the `.jar` file into your server's `plugins/` folder.
3. Start the server once. The plugin creates `plugins/.Sn-License/license.yml`.
4. Paste your Sn license key into that file.
5. Restart the server. The plugin validates the key at startup.

{% hint style="info" %}
SnAntiXray uses bundle licensing. `plugins/.Sn-License/license.yml` is one shared file per server,
not one per plugin, and a single bundle key unlocks every Sn plugin in your bundle. The plugin
refuses to enable without a valid key.
{% endhint %}

{% hint style="warning" %}
Requires **SnLib** installed (`depend: [SnLib]`), version 1.17.0 or later. SnAntiXray checks the
SnLib API level at startup and disables itself with a console notice on an older engine, so update
both jars together.
{% endhint %}

{% hint style="danger" %}
**PacketEvents must be installed.** It is a hard dependency on purpose. Without it your server
refuses to enable SnAntiXray rather than starting with every protection layer silently switched
off. A protection plugin that looks installed but hides nothing is worse than one that refuses to
start.
{% endhint %}

## Dependencies

| Plugin | Required |
|--------|----------|
| SnLib | Yes |
| PacketEvents | Yes |

## Requirements

- Java 21 or newer
- Paper 1.20.4 or newer, including the 1.21 line
- Folia is supported

## First steps after installing

1. Open `plugins/SnAntiXray/worlds.yml` and set your own world names. Decoy veins are the only
   signal the detection layer scores, so this file decides whether detection works at all.
2. Check the startup log. The plugin prints which layers are on and whether any world carries a
   usable decoy rule.
3. Give yourself `snantixray.admin` and run `/antixray stats` to confirm chunks are being rewritten.
