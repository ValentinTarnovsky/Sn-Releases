# Installation

1. Download the latest `snchat-v*` release from [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases/releases?q=snchat-).
2. Place the `.jar` file into your server's `plugins/` folder, together with SnLib and LuckPerms.
3. Start the server once. SnChat creates `plugins/.Sn-License/license.yml` and stops with `[Sn] License: FAIL` in the console. That is expected on a fresh install.
4. Paste your Sn license key into that file, replacing the placeholder value.
5. Restart the server. SnChat validates the key at startup.

{% hint style="info" %}
The license file is shared by every licensed Sn plugin, so one key unlocks the whole bundle. You only fill it in once per server.
{% endhint %}

{% hint style="warning" %}
Requires **SnLib** installed (`depend: [SnLib]`), version 1.23.0 or later.
{% endhint %}

The license is checked once, at startup. There are no periodic checks and nothing is disabled later. If the server starts, it stays running.

{% hint style="warning" %}
The check needs outbound HTTPS. A firewalled or offline server shows the same `[Sn] License: FAIL` line as a wrong key. If your key is correct and the plugin still refuses to enable, check that the server can reach the internet.
{% endhint %}

## Dependencies

| Plugin | Required |
|--------|----------|
| SnLib | Yes |
| LuckPerms | Yes |
| PlaceholderAPI | No |

SnChat is built against Java 21, as is SnLib. On 1.20.x your server itself has to run a Java 21 runtime. Paper supports it on every 1.20 build, but a server still on Java 17 will not load the plugin. 1.21.x requires Java 21 anyway.

## Generated files

`config.yml`, `lang/messages_en.yml` and both `guis/` files are created before the license is checked, so a failed first boot still leaves them on disk. `announcements.yml` and `blockcommands.yml` are created after it, so they appear only once the plugin enables.

## Read this before your first restart

{% hint style="danger" %}
The command whitelist is **on** out of the box. `command-blocker.enabled` defaults to `true` and `blockcommands.yml` ships with a populated `default` group.
{% endhint %}

From the very first boot:

- every player in the LuckPerms `default` group can only run the commands listed under it;
- every group **not** listed in `blockcommands.yml` falls back to the `default` list, so it is restricted too;
- operators hold `snchat.bypass.blockcommands` and notice nothing.

That combination is easy to miss until a player reports it. Either add your groups to `blockcommands.yml`, or set `command-blocker.enabled: false` in `config.yml`.

`blockcommands.yml` does not exist until the plugin has enabled once. Do this on the restart that follows a successful license check, not on the boot that failed it.
