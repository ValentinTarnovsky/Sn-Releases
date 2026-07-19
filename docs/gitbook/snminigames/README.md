# SnMiniGames

Scalable minigame framework for Paper servers. Adding a new game costs one game class plus one YAML file. The framework provides the queue and waiting lobby, countdowns, teleports, crash-safe player-state snapshots, full event protection, rewards and commands. The first game is Parkour.

## Features

- Automatic rounds with a clickable **[JOIN]** chat announcement and a live queue countdown.
- Crash-safe player state: inventory, armor, XP, effects, health, food, location and more are snapshotted to disk before a game touches them, and restored exactly.
- Full in-game protection: block changes, damage, drops, pickups, inventory clicks and commands are blocked; chat and leaving always work.
- Parkour with multi-map rotation, ordered checkpoints, fall respawn, and single or multi-winner finishes with a finish-hold wait.
- Per-position rewards as action lists (console commands, broadcasts) with placeholders.
- In-game map setup: a selection wand plus `/minigames admin parkour ...` subcommands, no file editing required.

## Optional integrations

- **PlaceholderAPI**: live `%snminigames_*%` placeholders for scoreboards, tab and chat. Without it the plugin works normally and registers no expansion.

## Links

- Source and releases: part of the shared [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases) repo
