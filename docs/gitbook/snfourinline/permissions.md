# Permissions

Player nodes default to `true`; admin nodes default to op. The bare `/fourinline` carries no permission, so every player can open the lobby. Each button inside it checks the same node as the command it stands for, so revoking a node revokes the feature.

| Permission | Default | Description |
|-----------|---------|-------------|
| `snfourinline.use` | `true` | Basic usage: `invite`, `accept`, `reject`, `create`, `cancel`, `lobby`, and the lobby's create and join buttons |
| `snfourinline.bet` | `true` | `bet` and `betcreate`, accepting a bet invite, and joining a bet game from the lobby |
| `snfourinline.spectate` | `true` | `spectate` and the lobby's spectate buttons |
| `snfourinline.top` | `true` | `top` and the lobby's leaderboard button |
| `snfourinline.admin` | op | Full administrative access; parent of every node below |
| `snfourinline.admin.reload` | op | `/fourinline reload` |
| `snfourinline.admin.debug` | op | `/fourinline debug` |
| `snfourinline.admin.update` | op | Receive the update notice in chat |
