# Commands

The root command is `/fourinline`, with the aliases listed in `command.aliases` (shipped as `fil`, `connect4`, `c4`, `4enlinea`, `4l`). Edit that list and run `/fourinline reload` to apply it live. Every subcommand is player-only.

## Player commands

| Command | Description |
|---------|-------------|
| `/fourinline` | Open the lobby (or print the help, when `presentation.main` is `chat`). While you are in a game, it reopens your board |
| `/fourinline invite <player>` | Invite a player to a friendly game |
| `/fourinline accept` | Accept the invite you were sent |
| `/fourinline reject` | Reject the invite you were sent |
| `/fourinline create` | List a public game anyone can join |
| `/fourinline cancel` | Cancel your pending invite or your public game |
| `/fourinline lobby` | Open the lobby, whatever `presentation.main` says |
| `/fourinline bet <player> <currency> <amount>` | Invite a player to a game with a bet |
| `/fourinline betcreate <currency> <amount>` | List a public game with a bet |
| `/fourinline spectate <player>` | Watch the game a player is in |
| `/fourinline top` | Open the leaderboard |
| `/fourinline help [page]` | Show the generated help |

## Admin commands

| Command | Permission | Description |
|---------|-----------|-------------|
| `/fourinline reload` | `snfourinline.admin.reload` | Reload the configuration, the language file and the menus |
| `/fourinline debug` | `snfourinline.admin.debug` | Toggle the runtime debug output |

Amounts accept the usual shorthands (`500`, `2.5k`, `1m`). The smallest stake is `0.01`, and a stake may not be finer than the `decimals` of its currency.

{% hint style="info" %}
Both bet flows stake the same amount from each player and pay the winner twice the stake. `bet` charges both players when the invite is accepted; `betcreate` charges the creator immediately and the joiner when they join. A draw refunds both stakes. Surrender, turn timeout and disconnect all pay the opponent like a normal win.
{% endhint %}
