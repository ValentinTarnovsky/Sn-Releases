# Commands

The root command is `/minigames`, alias `/mg` (configurable via `command.aliases` in `config.yml`). `help`, `reload` and `debug` are provided by SnLib on the same root.

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
| `/mg admin start <game>` | `snminigames.admin.start` | Force-open a round now |
| `/mg admin stop <game>` | `snminigames.admin.stop` | Stop the running round and restore everyone |
| `/mg admin forcejoin <player> [game]` | `snminigames.admin.forcejoin` | Force a player into a round |
| `/mg admin wand` | `snminigames.admin.setup` | Get the map setup wand |
| `/mg admin parkour create <map>` | `snminigames.admin.setup` | Create a new parkour map |
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
| `/mg reload` | `snminigames.admin.reload` | Reload configuration |

{% hint style="danger" %}
These commands are destructive and cannot be undone: `/mg admin parkour delete`, `/mg admin parkour clearwaypoints`.
{% endhint %}
