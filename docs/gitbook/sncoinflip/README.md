# SnCoinFlip

SnCoinFlip is PvP coinflip gambling with as many currencies as you care to configure. A player stakes an amount, the wager goes up in a public lobby menu, another player accepts it, both stakes are debited, a fair 50/50 winner is drawn and an animation plays before the winner takes the whole pot.

## Features

- Any number of currencies, all declared in `config.yml`, each on one of three backends: your Vault economy (the default, and the one that needs no configuration at all), an EdTools currency id through that API, or a pair of give/take console commands with a balance placeholder. The plugin links no economy plugin of its own - Vault is a broker, not an economy.
- A lobby menu listing every open coinflip, a stats menu beside it, and a coin-flip animation on the reveal. All three layouts live in `guis/*.yml`.
- Lifetime totals per player per currency: wins, losses, amount wagered and amount won, exposed through eight PlaceholderAPI placeholders.
- One active coinflip per player, as creator or as joiner.
- Wager bounds, a creation cooldown and tab-completed suggested amounts.
- Independent chat floors for the creation and the result broadcast, global or per currency, so small wagers can stay out of chat without touching the game itself.
- Configurable sounds for the animation tick, the winner, the loser and the creation broadcast.
- English and Spanish language files, both shipped.

## How the money is handled

This is a gambling plugin, so the interesting part is not the coin flip, it is what happens when something goes wrong halfway through. The rules it holds to:

- A waiting coinflip holds no money. Nothing is debited until someone actually joins, so creating and cancelling costs nothing.
- A match that cannot confirm both wagers were taken pays nobody. It refunds what it can prove was taken, tells both players, and writes a named line to the console.
- A restart during an animation refunds both wagers, to whoever is online and by UUID to whoever is not.
- A disconnect during an animation does not stop the match. It resolves on schedule and the winner is paid even if they are offline.
- A payout takes the event-free path wherever the backend offers one, so server boosters, money enchants and pets cannot multiply a prize. On EdTools that is the booster flag; a Vault deposit is the one exception, because Vault exposes a single deposit primitive and no event-free form.
- A currency that cannot pay a winner is refused at startup rather than registered half-working. That includes an EdTools id EdTools cannot resolve, and a command-backed currency with no `give-command`.

## Integrations

- **SnLib**: required. It is the engine: configuration, language, database, menus, items, scheduling and the placeholder bridge.
- **Vault**: required to load, since v2.1.0. It brokers whatever economy plugin your server runs; a `vault: true` currency needs no configuration beyond its display name. The economy provider itself may register late - the currency picks it up with no restart and no reload.

## Optional integrations

- **PlaceholderAPI**: needed by every command-backed currency, because its balance is read through a placeholder, and needed by the eight placeholders SnCoinFlip provides. Without it a command-backed currency still registers, but no balance on it can be read and every wager on it is refused.
- **EdTools**: needed by every currency with `edtoolapi: true`. A currency whose id EdTools cannot resolve is skipped at startup with a console line naming it.

Both are soft dependencies, so the plugin starts without them. The currencies that need them do not work without them.

## Links

- Source and releases: part of the shared [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases) repo
