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

## How do owners set or change the traded item?

The center cell of the owner menu, only: click it holding the item on the cursor, or drag one onto
it. The item is read, never taken. Shift-clicking a stack in your inventory ONLY deposits stock -
if it does not match the traded item it is refused with a message. Before 2.2.0 a non-matching
shift-click silently replaced the traded item; that gesture is gone on purpose.

## Why does pickup ask me to click twice?

The first click arms a red confirm cell for a few seconds; the second click removes the shop.
Pickup drains all stock and deletes the shop in one action, so it no longer fires on a single
(possibly accidental) click.

## Can I change the shop block's material later?

Yes. A shop is identified by the tag the item carries, never by its material, so every shop already
placed keeps working after you change `shop-item.material`.

## Why did my material get replaced with an enchanting table?

You picked something affected by gravity. A falling block leaves its own coordinates and the shop
becomes unreachable, so those are refused at startup and logged.

Blocks that need support, blocks made of two halves, and blocks that melt or decay are accepted but
are a bad idea for the same family of reasons. Pick a plain solid block.

## A shop block is gone but the shop still counts against its owner's limit

It should fix itself. With `shop.restore-missing-blocks` on (the default), the block is re-placed at
its own coordinates - at startup for chunks that are already loaded, otherwise the moment that chunk
loads - and a line in the console names the shop that was repaired.

If you turned that off, the shop still exists at those coordinates and putting any block back at the
same spot makes it reachable again. This is what the material warnings above are about.

## What exactly does the block repair replace?

Only air, water or lava at the shop's coordinates, and only with the CURRENT `shop-item.material`. A
standing block of any other material is left alone: a mismatch means you reconfigured the material,
not that a block went missing. A shop placed under an older material comes back as today's block,
which changes nothing, since a shop is identified by its tag and never by its material.

## How many shops can a player own?

`limits.max-shops-per-player` in `config.yml`, server-wide. `-1` is unlimited, `0` forbids creation.
A rank holding `sndisplayshops.limit.<n>` owns `<n>` instead: the highest node a player holds wins,
and `sndisplayshops.limit.unlimited` beats every number. See
[Permissions](permissions.md#how-many-shops-a-player-may-own).

## Trading says the shop's currency no longer exists

That currency is still declared in `config.yml` but was SKIPPED at startup. Check the log for a
SEVERE naming it: a command-backed currency is refused if `give-command`, `take-command` or
`balance-placeholder` is blank, or if a template is missing `{player}` or `{amount}`. Fix the entry
and reload, and the shops start trading again on their own.

Shops are deliberately NOT moved off a skipped currency, because the currency is coming back as soon
as you fix it - or as soon as EdTools is up - and a move cannot be undone.

## I deleted a currency from config.yml. What happened to its shops?

They were moved to the first currency in the file, at the next start or the next `/dshop reload`,
and each move is in the console at INFO with the shop, its owner and the currency it left.

{% hint style="danger" %}
The price NUMBER is carried over untouched. Nothing in `config.yml` says what a gem is worth in
coins, so a shop priced at 100 gems becomes a shop priced at 100 coins. Look at what your players
are selling before you delete a currency.
{% endhint %}

## Where can I see who bought what?

`plugins/SnDisplayShops/logs/<date>.log`, one file per day, one line per completed trade, with the
buyer, the owner, the shop, the item, the quantity, the unit price, the total and the currency. Turn
it off with `trade-log.enabled`. See [Configuration](configuration.md) for the line format and its
two caveats.

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

## A player left an island and their shops are still there

Both island toggles act only once SuperiorSkyblock has actually made the change, and only for a
change that fires its event: `/is leave`, a kick, a ban and a disband all do. A member removed by a
path that fires no event, such as an admin purge, leaves their shops in place. Check
`delete-shops-on-membership-loss` is on and look at `/dshop debug` output, which names every shop
the teardown considered and why it kept or removed it.

## Players are placing shops on other people's islands

`integrations.superiorskyblock.deny-placement-on-foreign-islands` refuses that, and it ships on.
Only the island's owner and its members may place there; visitors and coops are refused even where
the island lets them build. Outside every island, and at spawn, nothing changes. If it is on and a
placement still went through, look for a WARNING in the console: when SuperiorSkyblock cannot
answer, the plugin allows the placement rather than blocking every shop on the server.

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
