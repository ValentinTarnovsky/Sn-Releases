# Configuration

SnCoinFlip ships `config.yml`, two language files under `lang/` and three menu layouts under `guis/`. New keys are auto-merged on boot and your values and comments are preserved.

## config.yml

### General

| Key | Default | Description |
|-----|---------|-------------|
| `lang` | `en` | Language code. Loads `lang/messages_<code>.yml`, falling back to `en`. Does not affect the menus. |
| `update-configs` | `true` | Master switch of the auto-updater for this plugin's managed files. `false` freezes them and only warns about missing keys. |
| `debug.enabled` | `false` | Runtime debug output. Also toggleable live with `/coinflip debug`. |
| `debug.level` | `DEBUG` | `OFF`, `INFO`, `DEBUG` or `TRACE`. |
| `debug.categories` | `[]` | Category filter, empty lets everything through. Emitted categories: `coinflip`, `economy`, `currency`, `commands`, `stats`. |
| `command.aliases` | `[cf]` | Extra aliases of `/coinflip`, re-read on reload. Anything you add is registered live and removed again when you take it out. `cf` is also declared in `plugin.yml`, so removing it here does **not** remove `/cf`. |

### Database

| Key | Default | Description |
|-----|---------|-------------|
| `database.type` | `sqlite` | `sqlite` or `mysql`. SQLite needs nothing else. |
| `database.host` | `localhost` | MySQL only. |
| `database.port` | `3306` | MySQL only. |
| `database.database` | `sncoinflip` | MySQL only. |
| `database.username` | `root` | MySQL only. |
| `database.password` | `""` | MySQL only. |

### Presentation

| Key | Default | Description |
|-----|---------|-------------|
| `presentation.main` | `gui` | Bare `/coinflip`. `gui` opens the lobby, `chat` prints the generated help. |
| `presentation.stats` | `gui` | `/coinflip stats`. `gui` opens the stats menu, `chat` prints the text view. |

{% hint style="warning" %}
Setting `presentation.main: chat` makes the lobby unreachable through the root command, and the lobby is the only place a coinflip can be joined. Both values are legal and nothing warns, so if you want the chat help on the bare command you also need to give players another way into the lobby.
{% endhint %}

### Currencies

The map key is the currency id: it is the `<currency>` argument of `/coinflip create`, the PlaceholderAPI suffix and the key stats are stored under. There is no limit on how many you declare.

There are three backends, chosen with two flags: `vault` and `edtoolapi`.

**Vault-backed** (`vault: true`) - the default, and the only one that needs no configuration:

| Key | Required | Description |
|-----|----------|-------------|
| `display-name` | Yes | Name shown wherever the currency is named. PlaceholderAPI is resolved before colour, so a placeholder that returns a colour code may be written right after a `&`. |
| `vault` | Yes | `true` selects this backend. No id, no commands and no placeholder are read. |
| `broadcast-min` | No | Overrides `broadcast.min-amount` (see [Broadcasts](#broadcasts)) for this currency alone. |
| `result-broadcast-min` | No | Overrides `broadcast.result-min-amount` (see [Broadcasts](#broadcasts)) for this currency alone. |

**Command-backed** (neither flag):

| Key | Required | Description |
|-----|----------|-------------|
| `display-name` | Yes | Name shown wherever the currency is named. PlaceholderAPI is resolved before colour, so a placeholder that returns a colour code may be written right after a `&`. |
| `give-command` | Yes | Console command that credits a payout. `{player}` and `{amount}` are replaced. |
| `take-command` | Yes | Console command that debits a wager. `{player}` and `{amount}` are replaced. |
| `balance-placeholder` | Yes | Placeholder read to learn a player's balance. |
| `broadcast-min` | No | Overrides `broadcast.min-amount` (see [Broadcasts](#broadcasts)) for this currency alone. |
| `result-broadcast-min` | No | Overrides `broadcast.result-min-amount` (see [Broadcasts](#broadcasts)) for this currency alone. |

**EdTools-backed** (`edtoolapi: true`, `vault` absent or false):

| Key | Required | Description |
|-----|----------|-------------|
| `display-name` | Yes | As above. |
| `currency-id` | Yes | EdTools currency id. The three command fields are ignored. |
| `broadcast-min` | No | Overrides `broadcast.min-amount` (see [Broadcasts](#broadcasts)) for this currency alone. |
| `result-broadcast-min` | No | Overrides `broadcast.result-min-amount` (see [Broadcasts](#broadcasts)) for this currency alone. |

{% hint style="info" %}
Vault is a hard dependency of SnCoinFlip, but Vault itself provides no economy - it brokers whatever an economy plugin (EssentialsX, CMI, XConomy, ...) registers with it. A `vault: true` currency is never skipped at startup over a missing provider: it registers, refuses wagers as unverifiable while there is nothing to read, and starts working the moment a provider appears. No restart and no `/coinflip reload` are needed, which is what makes an economy plugin that enables after SnCoinFlip a non-event.

Setting both `vault: true` and `edtoolapi: true` on one currency is a mistake. Vault wins, `currency-id` is ignored, and the console says so by name at startup.
{% endhint %}

{% hint style="danger" %}
`give-command` is not optional and its absence is not cosmetic. It is the only way a prize, an abort refund, a shutdown refund or a refused-join rollback reaches a player. A command-backed currency without it is refused at startup with a SEVERE line naming it, rather than registered as a currency that takes wagers and pays nobody. The same rule applies to an EdTools `currency-id` that EdTools cannot resolve.
{% endhint %}

{% hint style="danger" %}
`balance-placeholder` should be an exact, unabbreviated number. This is not a display string. For a command-backed currency it is the only evidence that a wager really left the player, because a console command reports nothing back. A render that rounds, such as `1.5M`, cannot show a 100 debit leaving at all, and a formatter that rounds up reports more money than the player holds. Point this at the raw placeholder, not the pretty one: `%sncredits_balance%`, not `%sncredits_balance_formatted%`.
{% endhint %}

### Wager bounds

| Key | Default | Description |
|-----|---------|-------------|
| `bet.min` | `100` | Smallest wager, inclusive. |
| `bet.max` | `1000000000` | Largest wager, inclusive. |
| `bet.suggested-amounts` | `[100, 1000, 10000, 100000, 1000000]` | Tab completion suggestions only. Any amount inside the bounds is accepted. May use the abbreviated forms (`1k`, `1.5m`) from 2.4.0 on. |

{% hint style="info" %}
The suggestions are not filtered against `bet.min` and `bet.max`. If you raise the minimum above 100, the default suggestion list still offers 100 and the command then refuses it. Edit both together.
{% endhint %}

### Animation

Every value is in server ticks. 20 ticks is one second.

| Key | Default | Description |
|-----|---------|-------------|
| `animation.duration-ticks` | `100` | Length of the spin before the reveal. Clamped to 1..1200. |
| `animation.reveal-delay-ticks` | `30` | How long the revealed winner stays on screen before the menu closes. Clamped to 0..1200. |
| `animation.fast-phase-pct` | `50` | Percent of the duration running at `fast-interval-ticks`. |
| `animation.medium-phase-pct` | `25` | Percent running at `medium-interval-ticks`. The remainder runs slow. |
| `animation.fast-interval-ticks` | `2` | Ticks between head swaps during the fast phase. |
| `animation.medium-interval-ticks` | `4` | Ticks between head swaps during the medium phase. |
| `animation.slow-interval-ticks` | `6` | Ticks between head swaps during the slow phase. |

{% hint style="warning" %}
`duration-ticks` plus `reveal-delay-ticks` is also the time the plugin has to confirm that both wagers were really charged on a command-backed currency. Keep the sum at or above `verification.first-look-ticks` plus `verification.confirm-look-ticks`, or a debit that silently failed is paid out as a win. The console says so at startup if they are too low.

Both keys are clamped to 1200 ticks because neither player can close the menu until the reveal has passed. They are ticks, not milliseconds.
{% endhint %}

### Debit verification

Only ever runs for command-backed currencies. An EdTools currency confirms its own debit as it makes it, and a Vault currency gets the economy provider's own verdict on the withdrawal.

A console `take-command` reports nothing about whether money moved, so the plugin reads both balances twice after a match starts and decides from what changed.

| Key | Default | Description |
|-----|---------|-------------|
| `verification.first-look-ticks` | `4` | Ticks after the match starts before the first look. Decides nothing on its own; it only decides whether the second look is worth taking. |
| `verification.confirm-look-ticks` | `16` | Ticks after the first look before the confirming one, which is the look that acts. |

{% hint style="warning" %}
The right numbers depend on your balance provider, not on this plugin. If a take-command is forwarded to a proxy and applied to a database before the placeholder catches up, both looks can land before the truth arrives, and a match that was paid for correctly is then cancelled. If you see that in the console, raise `confirm-look-ticks`.
{% endhint %}

### Cooldowns

| Key | Default | Description |
|-----|---------|-------------|
| `cooldowns.create-seconds` | `5` | Seconds before a player may create another coinflip. Stamped for both players when a match starts, checked only on creation. |

### Broadcasts

| Key | Default | Description |
|-----|---------|-------------|
| `broadcast.min-amount` | `0` | Smallest wager whose *creation* is announced in chat, inclusive. `0` announces every coinflip. |
| `broadcast.result-min-amount` | `0` | Smallest wager whose *result* (who won) is announced in chat, inclusive. `0` announces every finished match. |

A currency's own entry under `currencies:` may also carry either floor, overriding the global value for that currency alone:

| Key | Required | Description |
|-----|----------|-------------|
| `broadcast-min` | No | Overrides `broadcast.min-amount` for this currency, in this currency's own units. Omit it to follow the global value. |
| `result-broadcast-min` | No | Overrides `broadcast.result-min-amount` for this currency, in this currency's own units. Omit it to follow the global value. |

{% hint style="warning" %}
`currencies:` is owner-owned (`# sn:extensible`), so SnLib never inserts a new key into a currency entry that already exists on disk. On a server upgrading from an earlier version, neither `broadcast-min` nor `result-broadcast-min` will appear on its own - add the line by hand to the currencies that need their own floor.
{% endhint %}

{% hint style="info" %}
The two floors are **independent**: a currency may throttle its creation noise and still announce every winner, or the reverse. Neither one implies the other, and both are `0` out of the box, so a server that never touches them keeps announcing everything exactly as before.

Both compare against the **wager**, not against the doubled prize, so the same number means the same match in either key. A result floor of `1000` silences exactly the matches a creation floor of `1000` silences.
{% endhint %}

{% hint style="info" %}
A coinflip below the creation floor is otherwise unaffected: it is still created, still listed in the lobby and still joinable. Only its own creation announcement is silenced - the creator still gets their normal confirmation message.

A match below the result floor is still played and still paid. The winner receives the pot and the statistics are recorded before the announcement is ever reached, and both players still see their own outcome in the animation. Only the server-wide chat line is skipped.
{% endhint %}

### Sounds

An empty value disables that sound.

| Key | Default | Description |
|-----|---------|-------------|
| `sounds.animation-tick` | `UI_BUTTON_CLICK` | Played to both participants on every head swap. |
| `sounds.winner` | `ENTITY_PLAYER_LEVELUP` | Played to the winner on the reveal. |
| `sounds.loser` | `ENTITY_VILLAGER_NO` | Played to the loser on the reveal. |
| `sounds.broadcast-created` | `""` | Played to every online player when a coinflip is created. Disabled by default. |

{% hint style="info" %}
An unresolvable sound name is tolerated rather than fatal: the plugin logs one warning per bad sound at load and carries on silent. Check the console after editing these.
{% endhint %}

## Language

`lang/messages_en.yml` and `lang/messages_es.yml` both ship, and `lang` in `config.yml` picks between them. It ships as `en`, so switching to Spanish is `lang: es` plus a `/coinflip reload`. Add your own by copying one to `messages_<code>.yml` and pointing `lang` at it; new keys from later versions merge into it and your wording is never overwritten.

{% hint style="info" %}
`lang` covers chat messages only. Menu text lives in `guis/*.yml` and is translated by editing those three files directly, whatever `lang` is set to.
{% endhint %}

## Menus

| File | Menu |
|------|------|
| `guis/lobby.yml` | The lobby listing every open coinflip. |
| `guis/stats.yml` | The per-currency statistics view. |
| `guis/animation.yml` | The coin-flip animation. |

{% hint style="warning" %}
The lobby has no pagination. Past 28 entries, the overflow is not shown.
{% endhint %}
