# Placeholders

All placeholders require [PlaceholderAPI](https://www.spigotmc.org/resources/placeholderapi.6245/) and use the `%snminigames_` prefix. Values are live and never empty: game-scoped tokens return `idle`, `0` or an empty string when nothing is running. `<game>` is the game id, for example `parkour`.

| Placeholder | Description |
|-------------|-------------|
| `%snminigames_state_<game>%` | Round state: `waiting`, `starting`, `running`, `ending` or `idle` |
| `%snminigames_queue_<game>%` | Live participant count of the round |
| `%snminigames_max_<game>%` | Configured maximum players |
| `%snminigames_countdown_<game>%` | Seconds left in the queue countdown |
| `%snminigames_map_<game>%` | Id of the map the current round plays |
| `%snminigames_ingame%` | `yes` or `no` for the viewing player |
| `%snminigames_game%` | The viewing player's current game id, or empty |
