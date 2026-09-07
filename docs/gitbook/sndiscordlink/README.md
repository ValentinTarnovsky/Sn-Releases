# SnDiscordLink

SnDiscordLink links Minecraft accounts to Discord accounts with one-time codes. Several servers share one MySQL database and elect a single embedded Discord bot, so a player links once and every server follows.

## Features

- Two-way linking with one-time 6-digit codes: start the flow in game, or start it in Discord.
- One shared database for a whole network: links, codes, rewards, cooldowns and the bot lease.
- Automatic bot failover: exactly one server runs the bot, and a dead host is replaced in about 30 seconds.
- A linked role given on link and removed on unlink, with optional nickname sync.
- In-game claims gated by a Discord boost or a Discord role, each with its own cooldown.
- LuckPerms ranks mirrored to Discord roles, and Discord roles that run actions in game.
- A webhook embed log of link, unlink, claim and leave events.

## Optional integrations

- **LuckPerms**: unlocks `rank-roles`, the permanent group to Discord role sync. Without it the plugin degrades to linking, claims and role rewards only.
- **PlaceholderAPI**: unlocks the `%sndiscordlink_...%` placeholders. Without it the placeholders are left as raw text.

## Links

- Source and releases: part of the shared [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases) repo
