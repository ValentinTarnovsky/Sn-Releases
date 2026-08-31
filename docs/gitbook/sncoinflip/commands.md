# Commands

The root command is `/coinflip`, with `/cf` as its built-in alias. Add further aliases under `command.aliases` in `config.yml`: they are registered on `/coinflip reload`, with no restart, and unregistered again when you remove them. `/cf` itself is declared in the jar's `plugin.yml`, so deleting it from that list does not remove it.

| Command | Permission | Description |
|---------|------------|-------------|
| `/coinflip` | `sncoinflip.use` | Opens the lobby with every active coinflip. Prints the generated help instead when `presentation.main` is `chat`. |
| `/coinflip create <currency> <amount>` | `sncoinflip.use` | Creates a coinflip for that amount of that currency. Nothing is debited yet. |
| `/coinflip cancel` | `sncoinflip.use` | Cancels your pending coinflip. Costs nothing, because a waiting coinflip holds no money. |
| `/coinflip stats` | `sncoinflip.use` | Shows your lifetime totals per currency, as a menu or as chat depending on `presentation.stats`. |
| `/coinflip reload` | `sncoinflip.admin.reload` | Reloads configuration, language and menus. |
| `/coinflip help` | `sncoinflip.use` | Shows the generated help. |
| `/coinflip debug` | `sncoinflip.admin.debug` | Toggles debug output live. |

{% hint style="info" %}
`<currency>` is the currency **id**, which is the map key in `config.yml`, not the display name. Tab completion offers every registered id, so a currency that was skipped at startup is also absent from the completion list.
{% endhint %}

### Writing the amount

`<amount>` accepts abbreviated wagers from **2.4.0** on: `1k`, `1.5m`, `2B`. The suffixes are `k`, `m`, `b`, `t`, `qa` and `qi`, case-insensitive. Comma grouping (`1,000`) and the European decimal comma (`1,5k`) are understood too.

The wager is always a whole number, so `1.5k` is 1500 and is accepted, while a bare `1.5` is refused rather than quietly rounded.

{% hint style="warning" %}
`10.000` is refused. A lone dot is read as a decimal point, so that input would otherwise become a wager of **ten**, not ten thousand. Write `10000` or `10k`.
{% endhint %}

`bet.suggested-amounts` may use the abbreviated forms as well.

## Creating and joining

`/coinflip create <currency> <amount>` puts your wager in the lobby and takes nothing from you. The lobby entry is the join button: another player opens `/coinflip`, left-clicks your entry, and that is the moment both stakes are debited and the animation starts for both of you.

{% hint style="warning" %}
Joining is left-click only. The lobby item is a commit button that spends real money, so it deliberately ignores right-click, shift-click and the number keys.
{% endhint %}

You may hold one active coinflip at a time, as the creator or as the joiner.

## The amount

The wager must be a whole number between `bet.min` and `bet.max` inclusive. Tab completion suggests the values in `bet.suggested-amounts`, but any amount inside the bounds is accepted.

A wager can also be refused before anything is charged when the currency's balance placeholder is too coarse to prove that the debit happened. The player is told, and the console names the balance and the amount. Raising the wager past what the render can display makes it work, so this costs exactly the bets that could not have been verified.

## During the animation

Neither player can close the animation menu or start another coinflip until the reveal has passed. The hold lasts `animation.duration-ticks` plus `animation.reveal-delay-ticks`.

Disconnecting does not cancel anything. The match resolves on schedule and the winner is paid, offline if necessary.
