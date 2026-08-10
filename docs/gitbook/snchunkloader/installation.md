# Installation

1. Download the latest `snchunkloader-v*` release from [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases/releases?q=snchunkloader-).
2. Place the `.jar` file into your server's `plugins/` folder.
3. Start the server once. SnChunkLoader creates a shared license file at `plugins/.Sn-License/license.yml`.
4. Paste your Sn license key into that file, replacing the placeholder line.
5. Restart the server. SnChunkLoader validates the key at startup and then enables.

{% hint style="warning" %}
Requires **SnLib** installed (`depend: [SnLib]`), version 1.24.1 or later. SnChunkLoader refuses to enable against an older engine, so update `SnLib.jar` at the same time.
{% endhint %}

{% hint style="info" %}
SnChunkLoader is licensed. The key in `plugins/.Sn-License/license.yml` is shared by every bundled Sn plugin on the server, so you paste it once. Without a valid key the plugin refuses to enable.
{% endhint %}

{% hint style="warning" %}
The licence is checked on every startup, not only the first one. The check is an outbound HTTPS call to `sn-license-server.okimc-dev.workers.dev`, retried three times before the plugin gives up and disables itself with a single `[Sn] License: FAIL` line in the console. Allow that host through your firewall. Expect the same failure if the jar has been repacked or modified, because its hash is part of the check.
{% endhint %}

## Dependencies

| Plugin | Required |
|--------|----------|
| SnLib | Yes |
| SnSuperiorSkyblock | No (optional, adds the island rules) |
| DecentHolograms | No (optional, alternative floating display backend) |
| PlaceholderAPI | No (optional, resolves PAPI tokens in the shipped files) |

{% hint style="info" %}
When an optional plugin is absent, the matching feature degrades gracefully. Without SnSuperiorSkyblock the plugin runs as a plain chunk loader and the island rules are skipped. The floating display falls back to SnLib's built-in renderer, and PlaceholderAPI tokens stay raw.
{% endhint %}

## First boot

SnChunkLoader generates its own files on first boot: `config.yml`, `lang/messages_en.yml` and `guis/loader.yml`. New keys are auto-merged on later boots, and your values and comments are preserved.

{% hint style="danger" %}
Decide `chunk-loader.material` before any loader exists, and then leave it alone. A placed loader is recognised by its block material, so changing that value makes every existing loader unrecognisable.

Loaders already **placed** are recoverable: the reconciliation sweep unplaces them and owes each item back to its owner, re-minted in the new material. Loader items already **handed out** are not. Every loader sitting in an inventory, a chest or an ender chest carries the old material, and placing one is refused, so those stacks become unplaceable for good and nothing refunds them. Buy them back or re-issue them by hand first.

A typo does the same thing, because an unknown value falls back to `BEACON`.
{% endhint %}
