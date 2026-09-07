# Installation

1. Download the latest `sndiscordlink-v*` release from [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases/releases?q=sndiscordlink-).
2. Place the `.jar` file into your server's `plugins/` folder.
3. Start the server once. SnDiscordLink creates a shared license file at `plugins/.Sn-License/license.yml`.
4. Paste your Sn license key into that file, replacing the placeholder line.
5. Restart the server. SnDiscordLink validates the key at startup and then enables.

{% hint style="warning" %}
Requires **SnLib** installed (`depend: [SnLib]`), version 1.34.3 or later. SnDiscordLink refuses to enable against an older engine, so update `SnLib.jar` at the same time.
{% endhint %}

{% hint style="info" %}
SnDiscordLink is licensed. The key in `plugins/.Sn-License/license.yml` is shared by every bundled Sn plugin on the server, so you paste it once. Without a valid key the plugin refuses to enable.
{% endhint %}

{% hint style="warning" %}
The Discord library (JDA) is not bundled in the jar. Paper downloads it from Maven Central into `libraries/` on the first boot, so the server needs internet access that one time.
{% endhint %}

## Dependencies

| Plugin | Required |
|--------|----------|
| SnLib | Yes |
| LuckPerms | No (optional, enables the rank to Discord role sync) |
| PlaceholderAPI | No (optional, enables placeholders) |

{% hint style="info" %}
When an optional plugin is absent, the matching feature degrades gracefully. Without LuckPerms the plugin keeps linking, claims and role rewards, and simply never touches rank roles.
{% endhint %}

## Discord bot setup

1. Create an application at the Discord Developer Portal and add a bot to it.
2. Enable the **Server Members Intent** on the bot page. Role, boost and leave events need it.
3. Under Installation choose **Guild Install** only, with the `bot` and `applications.commands` scopes.
4. Give the bot the **Manage Roles** and **Manage Nicknames** permissions.
5. Invite the bot, then move its role above every role it must give or remove.
6. Fill `discord.token` and `discord.guild-id` in `config.yml`, then run `/discord reload`.

{% hint style="info" %}
Write `env:VARIABLE_NAME` in `discord.token` to read the token from an environment variable instead of the file. Slash commands register in your guild the first time the bot starts.
{% endhint %}

## Multi-server setup

Set `database.type: mysql` with the same connection on every server, and ship the same `config.yml` everywhere. There is no cross-server messaging: every signal travels through the shared database.

Leave `discord.mode: auto` so the servers elect one bot host through a database lease. Use `always` only on a single-server install, and `never` to keep a server from ever hosting.

{% hint style="danger" %}
Two servers set to `always` means two bots on one token. Keep `auto` on every server of a network.
{% endhint %}

Rewards run on the server where the player is online. Leave `rewards.claim-on` empty and use network-wide actions, or list the allowed server names and set the `SNDISCORDLINK_SERVER` environment variable on each server.
