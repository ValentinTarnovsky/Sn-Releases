# Placeholders

All placeholders require [PlaceholderAPI](https://www.spigotmc.org/resources/placeholderapi.6245/) and use the `%customcrafting_` prefix. Replace `<id>` with a workstation id, the same one you passed to `/customcrafting set`.

Every placeholder answers about the player it is resolved for, so the same scoreboard line shows each player their own craft on that workstation.

| Placeholder | Description |
|-------------|-------------|
| `%customcrafting_status_<id>%` | Display name of the recipe being crafted, taken from its `display` block in `recipes.yml` |
| `%customcrafting_time_<id>_short%` | Remaining time, short format |
| `%customcrafting_time_<id>_long%` | Remaining time, full format |
| `%customcrafting_progress_<id>%` | Crafting progress percent |
| `%customcrafting_time_<id>%` | Remaining time, short format (alias) |
| `%customcrafting_<id>_time%` | Remaining time, short format (alias) |

## What they show when nothing is running

The stand-in strings live under `status:` in `lang/messages_en.yml`, because they are text a player reads:

| Key | Answers for |
|-----|-------------|
| `status.no-craft` | The status placeholder with no craft, and every placeholder for a player who is offline or whose data is still loading |
| `status.time-fallback` | The time placeholders with no craft to report |
| `status.progress-fallback` | The progress placeholder with no craft to report |

Because a player whose data is still being read has no craft state that can be trusted, `status.no-craft` is what every slot shows for the first moment after they join.

## Formatting

Both duration formats are templates in `config.yml` under `time-format`. The `full` block is used by `_long` and by chat messages, and `short` by the rest. `progress-format` shapes the progress placeholder, with `{percent}` substituted.

{% hint style="info" %}
Placeholder output is written straight into whatever scoreboard, tab list or hologram asked for it. Plain colour codes such as `&7` and `&#RRGGBB` work in these values; MiniMessage tags do not.
{% endhint %}
