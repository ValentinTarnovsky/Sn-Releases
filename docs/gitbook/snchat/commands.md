# Commands

The root command is `/snchat`. It ships with no aliases: set your own under `command.aliases` in `config.yml`, and they are re-read on reload.

`/mutechat` and `/clearchat` are standalone shortcuts for the matching subcommands. They take no arguments.

| Command | Permission | Description |
|---------|-----------|-------------|
| `/snchat` | `snchat.use` | Show the showcase tokens and the command help |
| `/snchat announce <name> [player]` | `snchat.admin.announce` | Send a configured announcement, to everyone or to one player |
| `/snchat clearchat` | `snchat.admin.clearchat` | Clear chat for every online player |
| `/snchat mutechat` | `snchat.admin.mutechat` | Toggle the global chat mute |
| `/snchat alerts` | `snchat.notify` | Turn your own violation alerts on or off. They start **off**, and the choice is saved across relog and restart |
| `/snchat blockcommands` | `snchat.admin.blockcommands` | Show the command whitelist status |
| `/snchat reload` | `snchat.admin.reload` | Reload the configuration |
| `/snchat help [page]` | `snchat.use` | Show the command help |
| `/snchat debug` | `snchat.admin.debug` | Toggle runtime debug output |
| `/mutechat` | `snchat.admin.mutechat` | Toggle the global chat mute |
| `/clearchat` | `snchat.admin.clearchat` | Clear chat for every online player |

`reload`, `help` and `debug` are provided by SnLib on every command tree.

## Chat tokens

These are not commands. Players type them inside a normal chat message, and they need `snchat.showcase`.

| Token | Default aliases | Effect |
|-------|-----------------|--------|
| Item | `[item]`, `[i]`, `[itm]` | Embeds the item the sender is holding, with its real tooltip |
| Inventory | `[inv]`, `[inventory]` | Posts a clickable tag opening a frozen replica of the sender's inventory |
| Ender chest | `[ec]`, `[enderchest]` | Posts a clickable tag opening a frozen replica of the sender's ender chest |

All three token sets are renameable in `config.yml`. Matching is case-insensitive.

The replica is view-only and frozen at the moment the message was sent. Every click and drag inside it is cancelled, and no item can leave it. Each snapshot expires after its configured TTL, and the tag stops working at exactly the same moment.
