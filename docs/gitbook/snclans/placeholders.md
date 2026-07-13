# Placeholders

All placeholders require [PlaceholderAPI](https://www.spigotmc.org/resources/placeholderapi.6245/) and use the `%snclans_` prefix.

Every placeholder resolves against the requesting player's clan. A player without a clan gets an empty or zero value.

| Placeholder | Description |
|-------------|-------------|
| `%snclans_name%` | Clan name |
| `%snclans_tag%` | Clan tag |
| `%snclans_leader%` | Clan leader name |
| `%snclans_motd%` | Clan message of the day |
| `%snclans_members%` | Total member count |
| `%snclans_online%` | Online member count |
| `%snclans_max%` | Maximum members allowed |
| `%snclans_role%` | Your role in the clan |
| `%snclans_kdr%` | Clan kill/death ratio |
| `%snclans_kills%` | Clan total kills |
| `%snclans_deaths%` | Clan total deaths |
| `%snclans_in_clan%` | Whether you are in a clan (true or false) |
| `%snclans_has_clan%` | Alias of `in_clan` (true or false) |
| `%snclans_pvp%` | Whether clan friendly fire is on (true or false) |
| `%snclans_allies%` | Number of allied clans |
| `%snclans_points_<id>%` | Clan points for point type `<id>`, for example `%snclans_points_kills%` |
| `%snclans_rank_<id>%` | Clan leaderboard rank for point type `<id>`, for example `%snclans_rank_kills%` |
