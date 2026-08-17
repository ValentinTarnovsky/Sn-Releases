# Commands

The root command is `/rotatinheads`, with the aliases `/rheads` and `/rh` (edit `command.aliases`
in `config.yml` to change them, then `/rh reload`). Every command is an admin command; there are no
player commands. `help`, `reload` and `debug` are provided by SnLib.

Head ids are letters, digits, `_` and `-`, up to 32 characters, and are stored in lower case. Ids
are case-insensitive when you type them.

## Admin commands

| Command | Permission | Description |
|---------|-----------|-------------|
| `/rh help [page]` | `snrotatinheads.admin` | Generated help, filtered by your permissions |
| `/rh reload` | `snrotatinheads.admin.reload` | Reload `config.yml`, the language file and `heads.yml`, then rebuild every head |
| `/rh debug` | `snrotatinheads.admin.debug` | Toggle debug output in the console |
| `/rh create <id> [texture]` | `snrotatinheads.admin.create` | Create a head at your position, facing the way you face |
| `/rh remove <id>` | `snrotatinheads.admin.remove` | Remove a head (`delete` also works) |
| `/rh list` | `snrotatinheads.admin.list` | List every head with its world and coordinates |
| `/rh info <id>` | `snrotatinheads.admin.info` | Show the settings of a head and its line and action counts |
| `/rh movehere <id>` | `snrotatinheads.admin.movehere` | Move a head to your position |
| `/rh tp <id>` | `snrotatinheads.admin.tp` | Teleport yourself to a head (`teleport` also works) |
| `/rh texture <id> <texture>` | `snrotatinheads.admin.texture` | Set the skin: a base64 textures value or a skin URL (visible while the head shows a skull) |
| `/rh model <id> <material[:model]/hand>` | `snrotatinheads.admin.model` | Set the item the head shows; `hand` reads it off your main hand, `player_head` goes back to the skull |
| `/rh transform <id> <mode>` | `snrotatinheads.admin.transform` | Item display context: `ground` (default), `fixed`, `head`, `gui`, `none`, `thirdperson_righthand`, ... |
| `/rh size <id> <value>` | `snrotatinheads.admin.size` | Scale of the head, minimum `0.05` |
| `/rh rotationspeed <id> <value>` | `snrotatinheads.admin.rotationspeed` | Radians per animation frame; negative reverses, `0` stops |
| `/rh bouncespeed <id> <value>` | `snrotatinheads.admin.bouncespeed` | Bounce speed multiplier; `0` stops the bounce |
| `/rh bounceheight <id> <value>` | `snrotatinheads.admin.bounceheight` | Peak height of the bounce, in blocks |
| `/rh viewrange <id> <blocks>` | `snrotatinheads.admin.viewrange` | How far the head renders, in blocks (approximate) |
| `/rh hologram list <id>` | `snrotatinheads.admin.hologram` | Show the label lines with their index (starting at 0) |
| `/rh hologram add <id> <text>` | `snrotatinheads.admin.hologram` | Append a label line (DecentHolograms text syntax) |
| `/rh hologram set <id> <line> <text>` | `snrotatinheads.admin.hologram` | Replace the line at that index |
| `/rh hologram clear <id>` | `snrotatinheads.admin.hologram` | Remove every label line and the label itself |
| `/rh hologram followbounce <id> <true/false>` | `snrotatinheads.admin.hologram` | Make the label bob with the head |
| `/rh action add <id> <left/right/any> <[tag]> [argument]` | `snrotatinheads.admin.action` | Add a click action to one side or both |
| `/rh action remove <id> <left/right/any> <index>` | `snrotatinheads.admin.action` | Remove the action at that index (starting at 0) |
| `/rh action clear <id> <left/right/any>` | `snrotatinheads.admin.action` | Remove every action of that side |
| `/rh action list <id> <left/right/any>` | `snrotatinheads.admin.action` | Show the actions of that side with their index |

## Custom models

A head shows the textured player head unless you give it a model. A model is one token:

| Token | What the head shows |
|-------|---------------------|
| `paper:1001` | The item `paper` with `custom-model-data` 1001 (every server version) |
| `paper:mypack:crown` | The item `paper` with the `item_model` component `mypack:crown` (1.21.2 or newer; older servers show the plain item) |
| `diamond_sword` | The plain item |
| `player_head:7` | A player head with a model selector; the texture still applies |
| `player_head` | Back to the textured player head |
| `hand` | Whatever you hold in your main hand: its material and its `item_model` (1.21.2+) or `custom-model-data` |

`hand` is the easy way in: hold the ItemsAdder, Nexo or resource-pack item, run `/rh model <id> hand`,
done. The head keeps its texture, size, rotation, bounce, label and actions; only the item changes.
The texture is only visible while the head shows a player head, and `/rh texture` tells you so.

Most custom models are designed for one display context. `ground` is the default and is what a
skull reads best in; if a model looks small, offset or tilted, try `/rh transform <id> fixed` or
`/rh transform <id> head`. Every value of the vanilla item display context is accepted.

## Click actions

An action is one line in the SnLib `[tag] argument` syntax. The most useful tags:

| Line | What happens on click |
|------|-----------------------|
| `[player] spawn` | Runs `/spawn` as the clicking player |
| `[player-as-op] warp hub` | Runs the command with a temporary op grant, restored afterwards |
| `[console] give {player} diamond` | Runs the command from the console |
| `[message] &aWelcome, {player}!` | Sends a message to the clicker |
| `[broadcastmessage] {player} found the secret head` | Sends a message to everyone |
| `[sound] entity.experience_orb.pickup 1 1.2` | Plays a sound to the clicker (volume and pitch optional) |
| `[actionbar] &eClick!` and `[title] Hi;there;10;40;10` | Action bar and title |
| `[right-click] [message] Only on right click` | Guards: `[left-click]`, `[right-click]`, `[chance=50]` and the rest of the SnLib catalogue |

`{player}` and `%player%` are the clicking player. PlaceholderAPI placeholders resolve against them.
Actions run one tick after the click, so a command that opens a menu opens it. Every line must start
with a `[tag]`; a plain command without a tag is rejected.

`any` adds the same line to both sides, removes the index from each side, or lists the left block
then the right block.
