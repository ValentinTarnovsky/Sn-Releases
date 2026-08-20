# Commands

SnPacketBosses ships one root command, `/packetbosses`, with the alias `/pbosses`. The alias list lives in two places. `plugin.yml` declares `aliases: [pbosses]` as a fallback, and `config.yml` declares `command.aliases` under the `command` section. The config key is authoritative at runtime, and it is re-read on `/packetbosses reload`. Add or drop an alias there instead of editing `plugin.yml`.

The root itself requires `snpacketbosses.admin`. Every leaf is an administration tool, so players without that node see nothing and run nothing.

## Command table

| Command | Permission | Description |
| --- | --- | --- |
| `/packetbosses` | `snpacketbosses.admin` | Opens the generated help page (same output as `help`). |
| `/packetbosses give <player> <boss> [amount]` | `snpacketbosses.admin.give` | Gives an online player the summoning eggs of a boss. |
| `/packetbosses kill <player\|all>` | `snpacketbosses.admin.kill` | Ends a player's boss fight, or every fight with `all`. |
| `/packetbosses list` | `snpacketbosses.admin.list` | Lists every configured boss and whether it is enabled. |
| `/packetbosses view <player\|stop>` | `snpacketbosses.admin.view` | Spectates a player's boss fight, or `stop` to stop watching. |
| `/packetbosses reload` | `snpacketbosses.admin.reload` | Reloads the plugin configuration. Injected by SnLib. |
| `/packetbosses help [page]` | `snpacketbosses.admin` | Shows the available commands, paginated. Injected by SnLib. |
| `/packetbosses debug` | `snpacketbosses.admin.debug` | Toggles runtime debug output. Injected by SnLib. |

{% hint style="info" %}
`reload`, `help` and `debug` are not declared in the plugin source. SnLib injects them into every root command it builds. `help` carries no permission of its own, so it inherits the root permission.
{% endhint %}

## Argument shapes

`give` takes a required online player, a required boss id and an optional amount. The boss id is a closed set: an id that names no boss file is rejected before the command runs. Tab completion offers disabled bosses too, so you can mint an egg to test a boss you switched off. The amount defaults to one and accepts any value of one or higher. Eggs that do not fit in the receiver's inventory drop at their feet.

`kill` takes one required argument. Tab completion offers `all` plus the online player names, but any name is accepted. That is deliberate: an offline player with a stored fight never appears in the suggestions. The literal `all` is matched first, so a player named "All" cannot shadow the wipe.

`view` takes one required argument. Tab completion offers `stop` plus the online player names. Watching somebody new stops your previous view automatically. The watched player is never told that you are spectating.

`list` and `debug` take no arguments. `help` takes an optional page number.

## Console and player use

`give`, `kill`, `list`, `reload`, `help` and `debug` all run from the console. `view` is player only. Spectating means receiving the boss packets on a client, and the console has no client to receive them.

{% hint style="info" %}
The fight lockdown can block commands while a player has an active boss. The root `packetbosses` stays runnable no matter what the `locks.allowed-commands` list says. That guarantee is the admin recovery path out of a stuck fight.
{% endhint %}

{% hint style="danger" %}
`/packetbosses kill all` cannot be undone. It ends every live fight and then deletes every stored session row from the database, paused fights included. Owners lose the boss health and the timer they had banked, and the attempts already spent are not refunded. Use `/packetbosses kill <player>` when you only mean to end one fight.
{% endhint %}
