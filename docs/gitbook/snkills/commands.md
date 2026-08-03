# Commands

SnKills has no player-facing commands. Everything under `/snkills` is administration, so the
whole tree is gated on `snkills.admin` and players do not see it in `/help` or in their client
command tree.

| Command | Permission | What it does |
|---|---|---|
| `/snkills` | `snkills.admin` | Shows the help page |
| `/snkills help [page]` | `snkills.admin` | Paginated help |
| `/snkills reload` | `snkills.admin.reload` | Re-reads `config.yml` and the language file |
| `/snkills debug` | `snkills.admin.debug` | Toggles verbose per-death diagnostics |

Default alias: `/skills`.

## `/snkills reload`

Picks up template edits, weapon-display changes and added aliases without a restart. What it
does **not** pick up: damage causes newly added by a Minecraft version - those are inserted at
boot, and a Minecraft upgrade means a restart anyway.

## `/snkills debug`

Turns on a per-death trace: the damage cause resolved, which template matched, what
PlaceholderAPI returned, the weapon that was found, and whether the message was broadcast. It
is the first thing to turn on when a death produces nothing and you cannot see why.

The toggle persists to `debug.enabled` in `config.yml`, so it survives a restart. Turn it off
when you are done - it writes a handful of lines per death.

## Aliases

`command.aliases` in `config.yml` registers extra aliases live on `/snkills reload`.

{% hint style="info" %}
`skills` also ships in `plugin.yml`, which is what Bukkit registers at load. Removing it from
`command.aliases` does **not** unregister it. Adding new aliases there works normally.
{% endhint %}
