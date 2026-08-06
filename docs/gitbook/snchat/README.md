# SnChat

SnChat formats your chat per LuckPerms group and moderates it. Players can show their held item, inventory or ender chest inline. It also broadcasts rotating announcements and restricts which commands each group may run.

## Features

- **Group chat formats.** One format per LuckPerms primary group, with a global fallback. Every line supports `&` codes, `&#RRGGBB` hex and PlaceholderAPI.
- **Line hover and click.** Attach a tooltip and a click action to the whole chat line.
- **Showcase tokens.** `[item]` embeds the held item with its real tooltip. `[inv]` and `[ec]` post a clickable tag that opens a frozen, view-only replica.
- **Chat moderation.** Caps, character repetition, message cooldowns, a `plugin:command` syntax block and a global mute. Each module corrects or denies, and can alert staff.
- **Announcements.** A rotating list on its own interval, each entry with its own hover and click.
- **Command whitelist.** A per-group allowlist of the commands a player may run and see in tab completion.

## Optional integrations

- **PlaceholderAPI**: resolves `%papi%` placeholders in chat formats, hover lines, click actions and announcements. Without it those placeholders reach chat as literal text, and everything else keeps working.

## Links

- Source and releases: part of the shared [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases) repo
