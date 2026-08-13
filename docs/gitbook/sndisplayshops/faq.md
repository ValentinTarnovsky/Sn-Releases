# FAQ

## How does a player make a shop?

They place a shop item. Give one out with `/dshop give <player>`, or hand it through a crate, a kit
or your server shop. Placing it creates the shop; right-clicking it opens the owner menu.

## A shop is not trading. Why?

A fresh shop starts paused on purpose, and it refuses to trade until it has an item, a price and a
currency. The owner menu shows which of those is missing, and the hologram says the shop is paused.

## How do owners set a price?

The "Set exact price" button closes the menu and asks in chat. What they type is consumed, never
broadcast, and the menu reopens once it is applied. Shorthand works: `1k`, `1.5m`, `2b` and the rest
of the `K/M/B/T/Qa/Qi` ladder, and a number written out in full is exact at any size -
`1500000`, `1.500.000` and `1,500,000` are the same price.

{% hint style="info" %}
A line the plugin cannot read as one number - two separators, a stray letter, anything past the
largest price there is - is refused and the price is left alone. It is never rounded into something
the owner did not type. The confirmation echoes every digit back so a mistyped price is visible
immediately.
{% endhint %}

## Can I change the shop block's material later?

Yes. A shop is identified by the tag the item carries, never by its material, so every shop already
placed keeps working after you change `shop-item.material`.

## Why did my material get replaced with an enchanting table?

You picked something affected by gravity. A falling block leaves its own coordinates and the shop
becomes unreachable, so those are refused at startup and logged.

Blocks that need support, blocks made of two halves, and blocks that melt or decay are accepted but
are a bad idea for the same family of reasons. Pick a plain solid block.

## A shop block is gone but the shop still counts against its owner's limit

The shop still exists at those coordinates. Put any block back at the same spot and it is reachable
again. This is what the material warnings above are about.

## How many shops can a player own?

`limits.max-shops-per-player` in `config.yml`, server-wide. `-1` is unlimited, `0` forbids creation.
There is no per-rank permission for it.

## Trading says the shop's currency no longer exists

That currency was skipped at startup. Check the log for a SEVERE naming it: a command-backed
currency is refused if `give-command`, `take-command` or `balance-placeholder` is blank, or if a
template is missing `{player}` or `{amount}`. Fix the entry and reload.

## Players can afford things and the shop still refuses

The plugin reads a command-backed balance through the placeholder you configured, so check
`balance-placeholder` resolves to a number for that player. `/papi parse <player> %your_placeholder%`
answers it directly.

## Can one server run several currencies?

Yes. Declare as many as you like under `currencies:`. Each shop picks one, and the owner menu cycles
through them in the order they are declared.

## Do I need DecentHolograms?

No. With it installed it draws the floating text; without it the plugin uses its own display entity
and looks the same.

## Why do the hologram placeholders show the owner's values?

The hologram is one shared entity that everybody nearby sees, so it cannot say different things to
different people. Its lines resolve against the shop's owner.

## A player picked up a shop and it refused

Picking a shop up hands its whole stock back in one action, and `limits.max-pickup-stacks` bounds
how much that may be. The owner withdraws or sells stock down until it fits. Nothing is destroyed to
make it fit, and there is no value that switches the limit off - below 1 reads as 256.

## An island was disbanded and the shops on it lost their stock

That is `integrations.superiorskyblock.delete-shops-on-island-disband`, and it does destroy stock:
not dropped, not returned, not logged. The same applies to `delete-shops-on-membership-loss`, where
the island survives and an ex-member loses their stock outright. Turn either off if your players
expect to keep what their shops held.

## Can I use MySQL?

Yes, set `database.type: mysql` and fill in the connection. Leave `pool-size` at `1`: a shop's stock
is written as a running total, so a second writer can land updates out of order and sold stock
reappears after a restart. The plugin logs a SEVERE if it finds MySQL above 1.

## Sold stock came back after a restart

Almost always `database.pool-size` above 1 on MySQL. See above.

## Do my config edits survive an update?

Yes. All four files are merged: new keys are added, your values, comments and additions are kept,
and example entries you deleted stay deleted. `currencies:` is marked extensible, so currencies you
remove are never re-inserted.

## Staff cannot run /dshop

`/dshop` is gated on `sndisplayshops.admin`, and every subcommand is a child of it. A rank without
that node cannot run the command at all, not even `/dshop help`.
