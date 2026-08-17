# Configuration

SnFourInLine ships `config.yml`, one language file under `lang/` and three menu layouts under `guis/`. New keys are auto-merged on boot; your values and comments are preserved.

## config.yml

### General

| Key | Default | Description |
|-----|---------|-------------|
| `lang` | `en` | Language code. Loads `lang/messages_<code>.yml`, falling back to `en`. |
| `update-configs` | `true` | Auto-merge new keys into the managed files on boot. `false` freezes them. |
| `debug.enabled` | `false` | Runtime debug output. Also toggleable live with `/fourinline debug`. |
| `debug.level` | `DEBUG` | `OFF`, `INFO`, `DEBUG` or `TRACE`. |
| `debug.categories` | `[]` | Category filter; empty lets everything through. |
| `command.aliases` | `[fil, connect4, c4, 4enlinea, 4l]` | Aliases of `/fourinline`, re-read on reload. |

### Database

| Key | Default | Description |
|-----|---------|-------------|
| `database.type` | `sqlite` | `sqlite` or `mysql`. SQLite needs nothing else. |
| `database.host` | `localhost` | MySQL only. |
| `database.port` | `3306` | MySQL only. |
| `database.database` | `snfourinline` | MySQL only. |
| `database.username` | `root` | MySQL only. |
| `database.password` | `""` | MySQL only. |

### Presentation and game

| Key | Default | Description |
|-----|---------|-------------|
| `presentation.main` | `gui` | Bare `/fourinline`: `gui` opens the lobby, `chat` prints the help. |
| `game.turn-timeout` | `30` | Seconds a player has to move before auto-surrendering (minimum 5). |
| `game.turn-warning-threshold` | `5` | Seconds left at which one warning is sent. `0`, or `turn-timeout` and above, sends none. |
| `game.falling-animation-interval` | `3` | Ticks between two steps of the falling piece. |
| `game.end-close-delay` | `60` | Ticks the board stays open after a game ends. |
| `game.win-highlight-delay` | `40` | Extra ticks when there is a winning line to show. |

### Invites, lobby and leaderboard

| Key | Default | Description |
|-----|---------|-------------|
| `invites.expire-seconds` | `60` | Lifetime of a pending invite (minimum 5). Expiry is silent. |
| `lobby.refresh-interval` | `20` | Ticks between two in-place lobby refreshes. `0` or less disables; positive values are floored at 5. |
| `leaderboard.refresh-interval` | `60` | Seconds between two refreshes of the cached top ten. |

### Restrictions and broadcasts

| Key | Default | Description |
|-----|---------|-------------|
| `restrictions.worlds` | `[]` | Worlds where the plugin is refused (exact, case-sensitive names). |
| `broadcasts.public-game-created` | `true` | Announce a new public listing server-wide. |
| `broadcasts.game-won` | `true` | Announce a won game server-wide. |

### Sounds

The format is `"SOUND_ID [volume] [pitch]"`, or `none` to disable one.

| Key | Default | Played |
|-----|---------|--------|
| `sounds.piece-place` | `BLOCK_STONE_PLACE 1.0 1.2` | To every viewer when a piece lands. |
| `sounds.piece-fall` | `BLOCK_NOTE_BLOCK_HAT 0.5 1.5` | To every viewer on each step of the falling animation. |
| `sounds.game-win` | `ENTITY_PLAYER_LEVELUP 1.0 1.0` | To the winner. |
| `sounds.game-lose` | `ENTITY_VILLAGER_NO 1.0 0.8` | To the loser. |
| `sounds.surrender` | `ENTITY_VILLAGER_HURT 1.0 1.0` | To a player who surrenders (button, timeout or disconnect). |

Every timing applies on `/fourinline reload`, including the three repeating tasks: leaderboard refresh, invite expiry sweep and lobby refresh.

### Currencies

The `currencies:` block is yours: it is marked extensible, so entries you add are never pruned and entries you delete stay deleted. One entry per currency players may bet in, keyed by the id they type on the command line:

```yaml
currencies:
  coins:
    display-name: "&#FFD700Coins"
    give-command: "eco give {player} {amount}"
    take-command: "eco take {player} {amount}"
    balance-placeholder: "%vault_eco_balance%"
    decimals: 2
```

- `balance-placeholder` is a PlaceholderAPI token (`%...%`) whose value is the player's balance as a **number**. Color codes and a currency symbol around it are stripped, and `1,234.5`, `3.0E7` or `10.5k` all read correctly.
- `decimals` is how finely the currency counts: `2` for money, `0` for a points economy or a placeholder that answers whole numbers (such as `%vault_eco_balance_fixed%`). A stake may not be finer than this, because a fractional charge could not be verified against a whole-number balance.
- `give-command` and `take-command` are dispatched from the console. `{player}` and `{amount}` are the only tokens; the amount is always plain decimal, never scientific notation. Right after a take the balance is read again and the stake only counts if it dropped. A take the economy refuses (a minimum-balance rule, a command that only prints its usage) never starts a game whose stake nobody paid. An economy that applies console commands asynchronously cannot be used for bets.
- `display-name` is shown in messages and menus and may itself contain colors. A PlaceholderAPI token inside it resolves in chat messages, not inside menu items.
- A currency is only offered for new bets when all three fields are set. A half-filled entry is reported at boot and on `/fourinline reload`, and so is an empty `currencies:` block.

## lang/messages_en.yml

Every line the plugin sends, plus a `status:` block with the short state words spliced into the menus and into `%snfourinline_playing%`. Restyle any value freely; your edits survive updates. Colors accept `&`, `&#RRGGBB` hex, MiniMessage, and the `[center]`, `[rgb]` and `[small]` tags.

## guis/board.yml, guis/lobby.yml, guis/leaderboard.yml

The three menus. The layout mask, the materials, the names, the lore and which cell carries which click action are all yours. The code reads the cells from the mask, so moving the sidebar or repositioning the 42 board cells works. The grid itself is always 7 columns by 6 rows.
