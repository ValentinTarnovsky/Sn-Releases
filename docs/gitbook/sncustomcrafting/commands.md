# Commands

The root command is `/customcrafting`, aliased to `/cc`. The alias is editable in `config.yml` under `command.aliases` and is re-read on reload.

| Command | Permission | Description |
|---------|-----------|-------------|
| `/customcrafting` | `customcrafting.use` | Show the command list |
| `/customcrafting set <id> [recipe-set]` | `customcrafting.admin.set` | Register the block you are looking at as a workstation |
| `/customcrafting remove <id>` | `customcrafting.admin.remove` | Remove a workstation |
| `/customcrafting bypass` | `customcrafting.admin.bypass` | Finish your own active crafts instantly |
| `/customcrafting give <player> <item> [amount]` | `customcrafting.admin.give` | Give one of this plugin's items |
| `/customcrafting reload` | `customcrafting.admin.reload` | Reload configuration, items, recipes and language files |
| `/customcrafting debug` | `customcrafting.admin.debug` | Toggle runtime debug output |

Crafting itself needs no command. Players right-click a registered workstation to open its menu, and right-click it again to claim a finished craft.

## Workstation ids

A workstation id may contain lowercase letters, numbers, `_` and `-`. It is the key the plugin stores the block under, so a separator such as a dot would be read back as a nested path instead of as an id.

Passing an id that is already registered re-points it at the block you are looking at.

## Handing out items from elsewhere

`customcrafting give` works from the console, so recipe rewards, crates and other plugins hand out ingredients and results with it. A vanilla `/give` cannot produce a plugin item.

{% hint style="warning" %}
Write the full command name in those lines, never the `cc` alias. The alias is yours to rename, and other plugins ship a `/cc` of their own. When one already owns it, yours is dropped and the reward line pays out nothing while the craft is still consumed.
{% endhint %}
