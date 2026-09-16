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
| `/lootbox audit` | `lootboxes.admin.audit` | Lists reward items stored in several variants that look identical but do not stack. |
| `/lootbox sync` | `lootboxes.admin.sync` | Refreshes every reward of the item in your main hand, across every lootbox. |
| `/lootbox reload` | `snlootboxes.admin.reload` | Reloads the configuration and lootboxes. |
| `/lootbox debug` | `snlootboxes.admin.debug` | Toggles debug output. |

Key delivery is all-or-nothing. When the target inventory cannot absorb the whole stack, nothing is given. `/lootbox giveall` also honors the `delivery.max-accounts-per-ip` cap from `config.yml`.

## Keeping reward items stackable

An item reward stores a byte-exact snapshot of the item it was captured from. That is what lets a lootbox hand out an item owned by another plugin, intact. The snapshot is frozen, though: if the plugin that mints the item later changes how it renders the name or the lore, the stored copy keeps looking identical while its underlying components differ, and the game refuses to merge the two. What players report is "the item from the lootbox does not stack with the real one".

`/lootbox audit` finds them. It groups every loaded reward by what item it is meant to be and lists the ones stored in more than one non-stacking variant, with how many rewards carry each variant and where they live. Whether two items stack is decided with the server's own merge test, so a difference the game does not care about is never reported.

`/lootbox sync` repairs them. Hold one freshly obtained copy of the item and run it: every reward meant to be that item but no longer stacking with it is refreshed from what you are holding, across every lootbox at once. Each reward keeps its weight, delivered amount, delivery method, command template and enabled state. Rewards that already stack are left alone, so running it twice does nothing.

{% hint style="info" %}
Some items are unique by construction, such as a player head whose profile id is randomized on every mint. No two copies of those ever stack, from a lootbox or otherwise, and no snapshot refresh can change that. Give them out as a `COMMAND` reward instead, so the plugin that owns the item mints it at the moment of the win.
{% endhint %}

`/lootbox audit` prints at most `audit.max-items` items and `audit.max-variants-per-item` variants per item (`config.yml`); set either to `0` to remove the cap. A lootbox whose file loaded with skipped rewards is reported rather than rewritten, because a whole-file write would discard the entries the plugin never managed to read.

{% hint style="danger" %}
These commands are destructive and cannot be undone: `/lootbox delete`.
{% endhint %}
