# SnFourInLine

Connect Four played inside chest GUIs: invite a friend or list a public game in the lobby, bet in any server currency, let others spectate, and climb the leaderboard.

## Features

- Chest-GUI board with a falling-piece animation, turn indicators and a per-move turn timer
- Friendly games by invite, or public listings anyone can join from the lobby
- Betting in any currency you configure, with every transfer verified by a balance read-back
- Escrow that settles exactly once on every ending: win, draw, surrender, timeout, disconnect and shutdown
- Spectator mode with live board updates
- Persistent stats on SQLite or MySQL, a leaderboard menu and a PlaceholderAPI expansion

## Optional integrations

- **PlaceholderAPI**: unlocks betting (a balance is read through a placeholder) and the `%snfourinline_...%` placeholders. Without it every balance reads as zero and bets are refused, never granted. Friendly games need nothing.

## Links

- Source and releases: part of the shared [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases) repo
