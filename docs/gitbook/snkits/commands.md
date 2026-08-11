# Commands

Everything lives under `/kit`. The alias `/kits` ships by default and is configurable under
`command.aliases` in `config.yml`, which is re-read on `/kit reload`.

Players claim from the menus. `/kit` opens one and that is the intended path. `/kit claim <kit>`
exists for command blocks, shops and anyone who prefers typing.

Every argument tab completes for real: kit ids, menu ids, online players, and the `overwrite`,
`*` and `confirm` tokens. Kit id completion hides the kits you may not claim.

## Player commands

| Command | Permission | Description |
|---------|-----------|-------------|
| `/kit` | `snkits.use` | Open the menu named by `default-gui` |
| `/kit claim <kit>` | `snkits.use` | Claim a kit |
| `/kit help` | none | Paginated help, filtered to what you may run |

## Admin commands

| Command | Permission | Description |
|---------|-----------|-------------|
| `/kit gui <gui> [player]` | `snkits.admin.gui` | Open a specific menu, optionally for another player |
| `/kit create <id>` | `snkits.admin.create` | Create a kit from your current inventory |
| `/kit edit <kit>` | `snkits.admin.edit` | Edit a kit in game |
| `/kit delete <kit>` | `snkits.admin.delete` | Delete a kit, confirmed by repeating the command |
| `/kit enable <kit>` | `snkits.admin.toggle` | Make a kit claimable |
| `/kit disable <kit>` | `snkits.admin.toggle` | Make a kit unclaimable, without hiding it |
| `/kit give <player> <kit>` | `snkits.admin.give` | Hand a kit to a player, bypassing every claim rule |
| `/kit reset <player> <kit>` | `snkits.admin.reset` | Clear one player's stored usage of one kit |
| `/kit reset <player> *` | `snkits.admin.reset` | Clear all of one player's kit usage |
| `/kit reset all confirm` | `snkits.admin.reset` | Clear the stored usage of every player |
| `/kit import [overwrite]` | `snkits.admin.import` | Import the kits of PlayerKits2 |
| `/kit reload` | `snkits.admin.reload` | Reload the config, language, menus and kit files |

`/kit help` and `/kit reload` are provided by SnLib.

## Deleting a kit

`/kit delete <kit>` does not delete anything on the first call. It arms a confirmation and waits
`delete-confirmation.timeout-seconds`, and repeating the same command inside that window performs
the deletion. After the window the handshake starts over.

## Resetting usage

`/kit reset` clears the stored claim history: cooldown timestamps and spent one-time claims. It
never touches permissions, so taking a bought kit away means removing the permission as well.

`/kit reset all` without `confirm` prints a warning and changes nothing.

{% hint style="danger" %}
`/kit reset all confirm` wipes the stored kit usage of every player on the server, and cannot be
undone.
{% endhint %}
