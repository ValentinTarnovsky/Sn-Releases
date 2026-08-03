# Permissions

| Node | Default | Grants |
|---|---|---|
| `snkills.admin` | op | Everything below. Also what makes `/snkills` visible at all |
| `snkills.admin.reload` | op | `/snkills reload` |
| `snkills.admin.debug` | op | `/snkills debug` |
| `snkills.admin.update` | op | Receives the update notice in chat when a new version is out |

`snkills.admin` is declared as the parent of the other three, so granting it in LuckPerms
grants the whole set.

There is no player-side node. SnKills has nothing a player can run - the death messages
themselves are broadcast to everyone regardless of permissions.
