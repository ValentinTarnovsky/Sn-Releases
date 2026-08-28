# Commands

The root command is `/lootbox`. It ships with no aliases; add your own under `command.aliases` in `config.yml`. Running `/lootbox` with no arguments shows the generated help, filtered to the subcommands the sender has permission for; the root itself needs no permission.

| Command | Permission | Description |
|---------|------------|-------------|
| `/lootbox list` | `lootboxes.admin.list` | Lists the loaded lootboxes. |
| `/lootbox give <player> <lootbox> [amount]` | `lootboxes.admin.give` | Gives lootbox keys to a player. |
| `/lootbox giveall <lootbox> [amount]` | `lootboxes.admin.giveall` | Gives lootbox keys to every online player. |
| `/lootbox create <id>` | `lootboxes.admin.create` | Creates a new lootbox from the bundled example. |
| `/lootbox delete <id>` | `lootboxes.admin.delete` | Deletes a lootbox and its file. |
| `/lootbox editor` | `lootboxes.admin.editor` | Opens the in-game lootbox editor. |
| `/lootbox reload` | `snlootboxes.admin.reload` | Reloads the configuration and lootboxes. |
| `/lootbox debug` | `snlootboxes.admin.debug` | Toggles debug output. |

Key delivery is all-or-nothing. When the target inventory cannot absorb the whole stack, nothing is given. `/lootbox giveall` also honors the `delivery.max-accounts-per-ip` cap from `config.yml`.

{% hint style="danger" %}
These commands are destructive and cannot be undone: `/lootbox delete`.
{% endhint %}
