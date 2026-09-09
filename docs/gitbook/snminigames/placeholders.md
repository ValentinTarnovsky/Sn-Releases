# Placeholders

All placeholders require [PlaceholderAPI](https://www.spigotmc.org/resources/placeholderapi.6245/) and use the `%snminigames_` prefix. Values are live and never empty: game-scoped tokens return `idle`, `0` or an empty string when nothing is running. `<game>` is the game id: `parkour`, `tntrun`, `tnttag`, `spleef` or `fastmine`.

| Placeholder | Description |
|-------------|-------------|
| `%snminigames_state_<game>%` | Round state: `waiting`, `starting`, `running`, `ending` or `idle` |
| `%snminigames_queue_<game>%` | Live participant count of the round |
| `%snminigames_max_<game>%` | How many players the room holds: `queue.max-players`, lowered to the live round's own limit when that is smaller (FastMine seats one player per shaft). With no round running it is the configured value, which is always the upper bound |
| `%snminigames_countdown_<game>%` | Seconds left in the queue countdown |
| `%snminigames_map_<game>%` | Id of the map the current round plays |
| `%snminigames_ingame%` | `yes` or `no` for the viewing player |
| `%snminigames_game%` | The viewing player's current game id, or empty |

## Scheduled rounds

These read [`schedule.yml`](configuration.md#scheduleyml). With the schedule off, or with nothing left to open, each returns the `status.none` word from the language file (`None` by default) - except `%snminigames_next_game_id%`, which returns nothing, so a scoreboard condition comparing it does not start matching a translated word. `<game>` is a game id as above.

| Placeholder | Description |
|-------------|-------------|
| `%snminigames_next_game%` | Display name of the minigame that opens next, colour codes included |
| `%snminigames_next_game_id%` | The same game's plain id. Use this one for comparisons; the display name carries formatting. Empty when nothing is scheduled |
| `%snminigames_next_time%` | Clock time of that opening, rendered with the schedule's own `time-format` |
| `%snminigames_next_in%` | How long until it: `1d 3h 27m`, `2h 15m`, `23m`, and `45s` only inside the last minute |
| `%snminigames_next_time_<game>%` | When that ONE game next opens, so a board can show the whole grid |
| `%snminigames_next_in_<game>%` | How long until that one game opens |

An entry naming a game that is disabled or unknown is never reported as next: it can never open a round. Times follow the timezone of the server machine.

{% hint style="warning" %}
Every placeholder on this page needs a player context. Scoreboards and per-viewer holograms provide one; parsing from the console, or with a tool that passes no player, returns nothing.
{% endhint %}
