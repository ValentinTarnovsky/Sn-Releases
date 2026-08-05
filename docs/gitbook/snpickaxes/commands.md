# Commands

The root command is `/snpickaxes`. It ships with no aliases, and you can add your own under `command.aliases` in `config.yml`. Aliases are re-read whenever you reload.

Every subcommand is admin-only. There are no player-facing commands: players use the pickaxes, they do not run commands.

| Command | Permission | Description |
|---------|-----------|-------------|
| `/snpickaxes give <player> <area\|duplicator> [amount]` | `snpickaxes.admin.give` | Give a pickaxe to a player. `amount` defaults to 1 |
| `/snpickaxes reload` | `snpickaxes.admin.reload` | Reload the configuration, items and language files |
| `/snpickaxes debug` | `snpickaxes.admin.debug` | Toggle the runtime debug output on or off |
| `/snpickaxes help` | `snpickaxes.admin` | Show the command list |

{% hint style="info" %}
`/snpickaxes reload` applies changes to sizes, drop counts and region rules immediately. One setting is not reloadable: `advanced.block-break-event-priority` applies on server start, because the listener registers once at enable.
{% endhint %}

{% hint style="warning" %}
Pickaxes already in player inventories keep the name and lore they were given with. Placeholders such as `%size%` resolve once, when the item is created. After changing `size` or `extra-drops`, re-give the pickaxes if you want their text to match.
{% endhint %}
