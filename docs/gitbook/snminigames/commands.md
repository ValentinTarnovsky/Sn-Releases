# Commands

The root command is `/minigames`, alias `/mg` (configurable via `command.aliases` in `config.yml`). `help`, `reload` and `debug` are provided by SnLib on the same root.

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
| `/mg admin wand region` | `snminigames.admin.setup` | Toggle the cuboid region wand: left-click one corner, right-click the other, edges rendered with particles |
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
| `/mg admin tntrun setwinners <map> <n>` | `snminigames.admin.setup` | Set how many survivors end the round |
| `/mg admin tntrun settimelimit <map> <s>` | `snminigames.admin.setup` | Set the round time limit (0 disables it) |
| `/mg reload` | `snminigames.admin.reload` | Reload configuration |

{% hint style="danger" %}
These commands are destructive and cannot be undone: `/mg admin parkour delete`, `/mg admin parkour clearwaypoints`, `/mg admin tntrun delete`.
{% endhint %}
