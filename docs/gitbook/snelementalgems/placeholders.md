# Placeholders

All placeholders require [PlaceholderAPI](https://www.spigotmc.org/resources/placeholderapi.6245/) and use the `%snelementalgems_` prefix.

Balance and upgrade placeholders resolve against the requesting player. Leaderboard placeholders read a cached snapshot.

| Placeholder | Description |
|-------------|-------------|
| `%snelementalgems_balance%` | Your raw gem balance |
| `%snelementalgems_balance_formatted%` | Your balance grouped, with decimals per config |
| `%snelementalgems_balance_compact%` | Your balance in compact form (k, M, B, T, P) |
| `%snelementalgems_rank%` | Your position on the gem leaderboard (0 when unranked) |
| `%snelementalgems_top_<n>_name%` | Player name at leaderboard position `<n>`, for example `%snelementalgems_top_1_name%` |
| `%snelementalgems_top_<n>_value%` | Balance at leaderboard position `<n>`, for example `%snelementalgems_top_1_value%` |
| `%snelementalgems_upgrade_<type>_level%` | Your level of upgrade `<type>`: `fortune`, `luck`, `discount`, `charity`, or `execution` |
