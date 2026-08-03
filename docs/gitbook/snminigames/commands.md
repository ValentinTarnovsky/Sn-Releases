# Commands

The root command is `/minigames`, alias `/mg` (configurable via `command.aliases` in `config.yml`). `help`, `reload` and `debug` are provided by SnLib on the same root.

Every help listing echoes the alias you typed. `/mg help` lists `/mg ...` and `/minigames help` lists `/minigames ...`, down to the per-game setup help. The descriptions and argument labels below are translatable: see [Configuration](configuration.md).

{% hint style="info" %}
Each minigame keeps its map setup commands out of `/mg help` and lists them behind its own
`/mg admin <game> help` instead. That way the root help grows by one line per minigame rather
than one line per subcommand. The setup commands still run and still tab-complete: type
`/mg admin parkour ` and press TAB to see them.
{% endhint %}

## Player commands

| Command | Description |
|---------|-------------|
| `/mg join <game>` | Join the open queue of a minigame |
| `/mg leave` | Leave the queue or the running round |
| `/mg list` | List games, their state and queue count |
| `/mg help` | Show the help menu |

## Admin commands

| Command | Permission | Description |
|---------|-----------|-------------|
| `/mg admin start <game> [map]` | `snminigames.admin.start` | Force-open a round now; the optional map forces a specific map instead of the rotation pick (without advancing the rotation) |
| `/mg admin forcestart <game>` | `snminigames.admin.forcestart` | Skip the remaining waiting countdown and begin the round now with whoever is queued (bypasses the minimum-players requirement) |
| `/mg admin stop <game>` | `snminigames.admin.stop` | Stop the running round and restore everyone |
| `/mg admin forcejoin <player> [game]` | `snminigames.admin.forcejoin` | Force a player into a round |
| `/mg admin wand` | `snminigames.admin.setup` | Toggle the single-block setup wand (given when absent, removed when present; also removed on quit) |
| `/mg admin wand region` | `snminigames.admin.setup` | Toggle the cuboid region wand: left-click one corner, right-click the other, edges rendered with particles. Running it again takes the wand back and clears the selection - and clears the selection even when the wand is already gone, which is how you stop leftover particles |
| `/mg admin parkour help [page]` | `snminigames.admin.setup` | List every parkour setup command with its usage, paginated |
| `/mg admin parkour create <name>` | `snminigames.admin.setup` | Create a new parkour map |
| `/mg admin parkour delete <map>` | `snminigames.admin.setup` | Delete a map (not while a round plays it) |
| `/mg admin parkour list` | `snminigames.admin.setup` | List maps with waypoint and winner counts |
| `/mg admin parkour setstart <map>` | `snminigames.admin.setup` | Set the race start to the wand-selected block |
| `/mg admin parkour addwaypoint <map>` | `snminigames.admin.setup` | Append a checkpoint at the wand-selected block |
| `/mg admin parkour clearwaypoints <map>` | `snminigames.admin.setup` | Remove every checkpoint of the map |
| `/mg admin parkour setwin <map>` | `snminigames.admin.setup` | Set the win block to the wand-selected block |
| `/mg admin parkour setwaiting <map>` | `snminigames.admin.setup` | Set the queue waiting lobby to where you stand |
| `/mg admin parkour setend <map>` | `snminigames.admin.setup` | Set the end teleport to where you stand |
| `/mg admin parkour sethold <map>` | `snminigames.admin.setup` | Set the multi-winner hold spot to where you stand |
| `/mg admin parkour setwinners <map> <n>` | `snminigames.admin.setup` | Set how many finishers end the round |
| `/mg admin tntrun help [page]` | `snminigames.admin.setup` | List every TNT Run setup command with its usage, paginated |
| `/mg admin tntrun create <name>` | `snminigames.admin.setup` | Create a new TNT Run map |
| `/mg admin tntrun delete <map>` | `snminigames.admin.setup` | Delete a map (not while a round plays it) |
| `/mg admin tntrun list` | `snminigames.admin.setup` | List maps with their region and survivor counts |
| `/mg admin tntrun setregion <map>` | `snminigames.admin.setup` | Save your region-wand selection as the arena |
| `/mg admin tntrun setstart <map>` | `snminigames.admin.setup` | Set the round start to where you stand |
| `/mg admin tntrun setwaiting <map>` | `snminigames.admin.setup` | Set the queue waiting lobby to where you stand |
| `/mg admin tntrun setend <map>` | `snminigames.admin.setup` | Set the survivors' end teleport to where you stand |
| `/mg admin tntrun setelimy <map> <y>` | `snminigames.admin.setup` | Set the Y below which a player is eliminated |
| `/mg admin tntrun setdelay <map> <ticks>` | `snminigames.admin.setup` | Set how long a stepped-on block survives |
| `/mg admin tntrun setdepth <map> <blocks>` | `snminigames.admin.setup` | Set how many blocks deep to remove (1-16; a sand+TNT layer clears both at 2) |
| `/mg admin tntrun setwinners <map> <n>` | `snminigames.admin.setup` | Set how many survivors end the round |
| `/mg admin tntrun settimelimit <map> <s>` | `snminigames.admin.setup` | Set the round time limit (0 disables it) |
| `/mg admin tnttag help [page]` | `snminigames.admin.setup` | List every TNT Tag setup command with its usage, paginated |
| `/mg admin tnttag create <name>` | `snminigames.admin.setup` | Create a new TNT Tag map |
| `/mg admin tnttag delete <map>` | `snminigames.admin.setup` | Delete a map (not while a round plays it) |
| `/mg admin tnttag list` | `snminigames.admin.setup` | List maps with their region, tagger and survivor counts |
| `/mg admin tnttag setregion <map>` | `snminigames.admin.setup` | Save your region-wand selection as the arena boundary |
| `/mg admin tnttag setstart <map>` | `snminigames.admin.setup` | Set the round start to where you stand (must be INSIDE the region) |
| `/mg admin tnttag setwaiting <map>` | `snminigames.admin.setup` | Set the queue waiting lobby to where you stand |
| `/mg admin tnttag setend <map>` | `snminigames.admin.setup` | Set the survivors' end teleport to where you stand |
| `/mg admin tnttag settaggers <map> <n>` | `snminigames.admin.setup` | Set how many players start each round holding the TNT (1-16) |
| `/mg admin tnttag setroundtime <map> <s>` | `snminigames.admin.setup` | Set the first round's length in seconds (5-600) |
| `/mg admin tnttag setdecrement <map> <s>` | `snminigames.admin.setup` | Set how many seconds each following round loses (0-60) |
| `/mg admin tnttag setminroundtime <map> <s>` | `snminigames.admin.setup` | Set the shortest a round may get (1-600) |
| `/mg admin tnttag setgrace <map> <s>` | `snminigames.admin.setup` | Set the counter floor applied when a loose tag is reassigned (0-60) |
| `/mg admin tnttag setbreak <map> <s>` | `snminigames.admin.setup` | Set the observation break between a detonation and the next round (0-30, 0 = none) |
| `/mg admin tnttag setwinners <map> <n>` | `snminigames.admin.setup` | Set how many survivors end the match |
| `/mg admin tnttag settimelimit <map> <s>` | `snminigames.admin.setup` | Set the match time limit (0 disables it) |
| `/mg admin spleef help [page]` | `snminigames.admin.setup` | List every Spleef setup command with its usage, paginated |
| `/mg admin spleef create <name>` | `snminigames.admin.setup` | Create a new Spleef map |
| `/mg admin spleef delete <map>` | `snminigames.admin.setup` | Delete a map (not while a round plays it) |
| `/mg admin spleef list` | `snminigames.admin.setup` | List maps with their region, floor materials and survivor counts |
| `/mg admin spleef setregion <map>` | `snminigames.admin.setup` | Save your region-wand selection as the breakable arena (select the floor **and** the headroom above it) |
| `/mg admin spleef setstart <map>` | `snminigames.admin.setup` | Set the round start to where you stand (must be over the arena's footprint) |
| `/mg admin spleef setwaiting <map>` | `snminigames.admin.setup` | Set the queue waiting lobby to where you stand |
| `/mg admin spleef setend <map>` | `snminigames.admin.setup` | Set the survivors' end teleport to where you stand |
| `/mg admin spleef setelimy <map> <y>` | `snminigames.admin.setup` | Set the Y below which a player is eliminated |
| `/mg admin spleef setsnowballs <map> <min> <max>` | `snminigames.admin.setup` | Set the snowballs a **shovel** break pays out (0-16; snowball and melt breaks pay nobody) |
| `/mg admin spleef setmeltstart <map> <s>` | `snminigames.admin.setup` | Set how long before the floor starts melting by itself (0-3600, 0 = never) |
| `/mg admin spleef setmeltrate <map> <n>` | `snminigames.admin.setup` | Set how many random floor blocks the melt takes per second (1-64) |
| `/mg admin spleef setwinners <map> <n>` | `snminigames.admin.setup` | Set how many survivors end the round |
| `/mg admin spleef settimelimit <map> <s>` | `snminigames.admin.setup` | Set the round time limit (0 disables it) |
| `/mg admin fastmine help [page]` | `snminigames.admin.setup` | List every FastMine setup command with its usage, paginated |
| `/mg admin fastmine create <name>` | `snminigames.admin.setup` | Create a new FastMine map |
| `/mg admin fastmine delete <map>` | `snminigames.admin.setup` | Delete a map (not while a round plays it) |
| `/mg admin fastmine list` | `snminigames.admin.setup` | List maps with their shaft count, depth and palette size |
| `/mg admin fastmine addshaft <map>` | `snminigames.admin.setup` | Mark a shaft where you stand - one per player slot. Refused within one block of an existing shaft: a shaft is a 3x3 footprint |
| `/mg admin fastmine removeshaft <map> <index>` | `snminigames.admin.setup` | Remove a shaft by its number (1-based, as shown by `shafts`) |
| `/mg admin fastmine shafts <map>` | `snminigames.admin.setup` | List the marked shafts with their positions |
| `/mg admin fastmine setwaiting <map>` | `snminigames.admin.setup` | Set the queue waiting lobby to where you stand |
| `/mg admin fastmine setend <map>` | `snminigames.admin.setup` | Set the end teleport to where you stand |
| `/mg admin fastmine setdepth <map> <blocks>` | `snminigames.admin.setup` | Set how many blocks each column holds, which is also how deep the shaft is dug (1-64) |
| `/mg admin fastmine setcasing <map> <material>` | `snminigames.admin.setup` | Set the shaft wall and landing floor material (glass keeps the race watchable) |
| `/mg admin fastmine settimelimit <map> <s>` | `snminigames.admin.setup` | Set the round time limit (0 disables it) |
| `/mg reload` | `snminigames.admin.reload` | Reload configuration |

{% hint style="info" %}
Each game hides its setup commands behind its own `help` leaf, so `/mg help` stays short: it shows one line per game (`/mg admin tntrun help`, `/mg admin tnttag help`, `/mg admin spleef help`, `/mg admin fastmine help`) instead of one line per subcommand. The hidden leaves still tab-complete.
{% endhint %}

{% hint style="info" %}
Spleef's floor materials have **no** subcommand: `removable-materials` is a list, so edit it in `games/spleef.yml` and run `/mg reload`. That list is also what the shovel is allowed to break, which is why it can never be empty. FastMine's `blocks` palette works the same way for the same reason - edit it in `games/fastmine.yml` and reload.
{% endhint %}

{% hint style="info" %}
FastMine has no `setregion` and no `setstart`: its arena is generated, not built. `addshaft` is the whole setup - one shaft per player slot - and the map then seats exactly that many players.
{% endhint %}

{% hint style="danger" %}
These commands are destructive and cannot be undone: `/mg admin parkour delete`, `/mg admin parkour clearwaypoints`, `/mg admin tntrun delete`, `/mg admin tnttag delete`, `/mg admin spleef delete`, `/mg admin fastmine delete`, `/mg admin fastmine removeshaft`.
{% endhint %}
