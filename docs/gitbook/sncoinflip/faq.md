# FAQ

## A currency is missing from `/coinflip create` tab completion

It was refused at startup. Search the console for a SEVERE line naming it. The three reasons a currency is refused:

- `edtoolapi: true` and EdTools cannot resolve the `currency-id`.
- `edtoolapi: true` and EdTools is not installed at all.
- Command-backed and there is no `give-command`, which is the only way it could ever pay a winner.

The refusal is deliberate. A currency that can take a wager but not pay a prize is worse than a currency that does not exist.

A `vault: true` currency is never on that list. It is always registered, because the one thing that can be missing - a registered economy provider - is a question of load order rather than a mistake in your file, and it resolves itself the moment an economy plugin registers one. While there is nothing to read, the currency appears in tab completion and refuses wagers as unverifiable.

## Players are told they have no balance when they clearly do

Read the console line first: it says which of the two this is. A wager refused as **unverifiable** means the balance could not be READ, which is an outage on your side; a wager refused for **insufficient funds** means it was read and it was too small.

For a command-backed currency, the balance comes from `balance-placeholder`. Check it resolves to a plain number for that player with `/papi parse <player> %your_placeholder%`. Common causes:

- PlaceholderAPI is not installed, or the expansion that provides it is not. No balance can then be read, and every wager is refused as unverifiable.
- The placeholder renders a formatted string rather than a number.

For a `vault: true` currency there is nothing in your config to check: the balance is whatever your economy plugin reports. Confirm one has registered a provider with Vault - `/vault-info` on most builds - because Vault on its own provides no economy.

## Wagers are refused with a message about the balance not being precise enough

The currency's `balance-placeholder` renders an abbreviated number, such as `1.5M`. A render like that cannot show a small debit leaving, so the plugin cannot tell whether the wager was really charged, and it refuses the bet before charging anything rather than gamble on it. The console names the balance and the amount.

Point the currency at the raw placeholder instead of the formatted one. Larger wagers, big enough to move the render, still work meanwhile.

## Matches are being cancelled and nobody is paid, on a currency that works fine otherwise

The debit verification is losing a race with your balance provider. This happens when a `take-command` is forwarded to a proxy and applied there, while the placeholder reads a cache on the local server that refreshes afterwards. Both looks land before the truth arrives, both sides read as unpaid and the match aborts.

Raise `verification.confirm-look-ticks`. Then check that `animation.duration-ticks` plus `animation.reveal-delay-ticks` is still at or above `first-look-ticks` plus `confirm-look-ticks`; the console warns at startup if it is not.

## A player disconnected during the animation. What happens to the money?

The match resolves normally. The animation timer belongs to the plugin, not to the viewer, so it keeps running, and the winner is paid by UUID whether or not they are online. They see the balance when they log back in.

A player who quits while their coinflip is still **waiting** simply has it cancelled, and nothing was ever debited.

## The server restarted mid-animation

Both wagers are refunded, to whoever is online and by UUID to whoever is not, with a console line per side. The match does not resolve and nothing is written to the stats.

## Can a booster or a money enchant multiply a coinflip prize?

Not on an EdTools currency: payouts there are credited with the booster flag off, so nothing listening for a currency-add event sees a coinflip prize as earned income. This was the most expensive bug in this plugin's history and the guard is deliberate. On a command-backed currency the answer is whatever your `give-command` does, which is your choice to make.

Vault is the one backend that cannot promise it. Its API has a single deposit primitive and no event-free form, so whatever your economy plugin fires on a credit, it fires on a coinflip payout too. In practice a Vault deposit is a plain balance write and the things that multiply income listen to their own plugin's events instead. If that is not true on your server, put the currency on EdTools or on an explicit `give-command`.

## Can a player have two coinflips at once?

No. One active coinflip per player, as creator or as joiner.

## The lobby does not show all the coinflips

There is no pagination, so entries past 28 cells are not shown.

## Why does right-clicking a lobby entry do nothing?

The lobby item is the join button and joining spends real money, so it is left-click only. Right-click, shift-click and the number keys are ignored on purpose.

## `/coinflip` prints help instead of opening the lobby

`presentation.main` is set to `chat`. Set it to `gui` to open the lobby. Note that the lobby is the only way to join a coinflip, so on `chat` nobody can accept a wager through the root command.

## Menus are in Spanish

Menu text lives in `guis/lobby.yml`, `guis/stats.yml` and `guis/animation.yml`, not in the language file, and ships in Spanish. Edit those three files. The `lang` setting only covers chat messages.

## Do the stats show profit?

No. `won` is the gross prize, so a won 100 coinflip adds 100 to `wagered` and 200 to `won`. Profit is `won` minus `wagered`.

## The plugin will not enable and the console says `License: FAIL`

Either the key in `plugins/.Sn-License/license.yml` is missing, wrong or expired, or the server cannot reach `sn-license-server.okimc-dev.workers.dev`, or the jar has been modified. The check runs on every startup, retries three times, then disables the plugin. Allow the host through your firewall and use the jar exactly as downloaded.

## I deleted a currency from `config.yml` and it stays gone. Is that a bug?

No, that is the point. The `currencies:` block is marked extensible, so the updater never puts back entries you removed and never inserts new ones. The trade-off is that new currency documentation shipped in a later version never appears in your file either, which is why the currency keys are documented in [Configuration](configuration.md).
