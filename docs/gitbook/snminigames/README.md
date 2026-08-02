# SnMiniGames

Scalable minigame framework for Paper servers. Adding a new game costs one game class plus one YAML file. The framework provides the queue and waiting lobby, countdowns, teleports, crash-safe player-state snapshots, full event protection, rewards and commands. It ships with four games: Parkour, TNT Run, TNT Tag and Spleef.

## Features

- Automatic rounds with a clickable **[JOIN]** chat announcement and a live queue countdown.
- Crash-safe player state: inventory, armor, XP, effects, health, food, location and more are snapshotted to disk before a game touches them, and restored exactly.
- Full in-game protection: block changes, damage, drops, pickups, inventory clicks and commands are blocked; chat and leaving always work.
- **Parkour** with multi-map rotation, ordered checkpoints, fall respawn, and single or multi-winner finishes with a finish-hold wait.
- **TNT Run** with cuboid arenas: a one-block-wide trail vanishes behind each runner, falling below the arena eliminates you, and the last player standing wins. The arena restores itself completely at the end of the round, on reload and on shutdown.
- **TNT Tag**: one or more players start each round holding the TNT - a TNT hat, a hotbar full of TNT and a glow - and whoever still holds it when the round counter hits zero explodes and is out. Pass it by hitting another player, **with or without server PvP** - the hit keeps its knockback and hurt animation and loses only the damage, and the holder gets a speed boost to chase with. Each explosion is followed by a few seconds of observation, then the next round opens: everyone is teleported back to the start spawn, a new TNT is handed out and the clock is shorter than the last one, until only the winners are left. The counter draws on a boss bar or the action bar, and the "explosion" is a particle plus a sound: it never damages a single block.
- **Spleef**: everyone spawns on a snow floor holding a shovel that breaks a block in **one click**, and every block broken with it pays out 1-3 snowballs. Throw those and they break the floor where they land and shove whoever they hit, so the ground can go out from under someone across the arena. Fall below the elimination height and you are out. Nobody can hurt anybody - punching does nothing at all and a snowball costs zero hearts, so the snowball's push is the only force in the game and it feels the same **with or without server PvP**. Camping is answered by a global melt: past a configurable delay, random floor blocks vanish every second until someone falls. The arena restores itself completely at the end of the round, on reload and on shutdown.
- Per-position rewards as action lists (console commands, broadcasts) with placeholders.
- In-game map setup: a single-block selection wand, a cuboid region wand and per-game `/minigames admin <game> ...` subcommands, no file editing required.

## Optional integrations

- **PlaceholderAPI**: live `%snminigames_*%` placeholders for scoreboards, tab and chat. Without it the plugin works normally and registers no expansion.

## Links

- Source and releases: part of the shared [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases) repo
