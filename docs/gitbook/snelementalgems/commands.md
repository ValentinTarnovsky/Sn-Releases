# Commands

The root command is `/gems`. It ships with the aliases `/gem` and `/eg`, which you can change under `command.aliases` in `config.yml`. That alias list is authoritative and is re-read on `/gems reload`. Running `/gems` with no argument opens the shop when `presentation.main` is `gui`, or prints the help list otherwise. The `help`, `reload`, and `debug` subcommands are provided automatically by SnLib.

## Player commands

| Command | Description |
|---------|-------------|
| `/gems balance` | Show your own gem balance |
| `/gems balance <player>` | Show another player's balance (needs `snelementalgems.balance.others`) |
| `/gems top` | Show the gem leaderboard |
| `/gems shop [category]` | Open the gem shop, or a specific category |
| `/gems upgrades` | Open the upgrades menu |
| `/gems withdraw <amount>` | Withdraw gems as a physical item |
| `/gems pay <player> <amount>` | Pay gems to an online player |

## Admin commands

Every `/gems admin` action needs the `snelementalgems.admin` parent node plus its own leaf node below.

| Command | Permission | Description |
|---------|-----------|-------------|
| `/gems admin give <player\|*> <amount> [-s] [-item]` | `snelementalgems.admin.give` | Give gems to a player, or to everyone online with `*` |
| `/gems admin add <player> <amount> [-s]` | `snelementalgems.admin.add` | Add gems to a player's balance (offline players allowed) |
| `/gems admin remove <player> <amount> [-s]` | `snelementalgems.admin.remove` | Remove gems from a player's balance |
| `/gems admin set <player> <amount>` | `snelementalgems.admin.set` | Set a player's balance to an exact value |
| `/gems reload` | `snelementalgems.admin.reload` | Reload configuration and language files |
| `/gems debug` | `snelementalgems.admin.debug` | Toggle runtime debug output |

{% hint style="info" %}
The `-s` flag runs an admin action silently, skipping the target's message. The `-item` flag on `/gems admin give` hands physical gem items instead of adding to the balance. The `*` target gives every online player the same amount, never a compounded one.
{% endhint %}
