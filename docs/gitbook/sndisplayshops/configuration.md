# Configuration

Four files, all merged on update:

| File | Holds |
|---|---|
| `config.yml` | the shop item, the hologram, limits, currencies, the database, integrations |
| `lang/messages_en.yml` | every line the plugin sends, plus the hologram text |
| `guis/buyer.yml` | the layout of the menu everyone but the owner sees |
| `guis/owner.yml` | the layout of the owner's management menu |

## The shop item

```yaml
shop-item:
  material: ENCHANTING_TABLE
  display-name: "&#FFD700&lDisplay Shop"
  lore:
    - "&7Place this block to create"
    - "&7a player shop with a floating"
    - "&7display."
  glow: true
  custom-model-data: 0
```

A shop is identified by the tag the item carries, never by its material, so changing `material`
later leaves every shop already in the world working.

{% hint style="warning" %}
A material affected by gravity is refused at startup and falls back to `ENCHANTING_TABLE`. A block
that falls out of its own coordinates leaves a shop nobody can reach.

Three more families are accepted but are a bad idea, and the plugin cannot tell them apart to warn
you: blocks that need support (torch, carpet, sapling, banner), blocks made of two halves (door,
bed, tall flower), and blocks that melt or decay (ice, coral, leaves). Pick a plain solid block.
{% endhint %}

## Shop defaults and block repair

```yaml
shop:
  defaults:
    price: 0
    mode: SELL
    start-paused: true
  restore-missing-blocks: true
```

`mode` is from the SHOP's point of view: `SELL` means the shop sells to players, `BUY` means it buys
from them. A fresh shop starts paused so nobody trades with it before its owner has set the item,
the price and the currency.

The three `defaults` are read once, when a block is placed; changing them never touches a shop that
already exists. `restore-missing-blocks` is read live and applies to every shop.

`restore-missing-blocks` re-places the block of a shop whose coordinates are empty - broken while
the plugin was not running, or removed by something that fires no event, such as a world edit or
another plugin calling `setType`. Before it existed, that left a "ghost": the shop alive in every
index and still counting against its owner's limit, its hologram spinning over air, and recovery a
matter of putting a block back at coordinates nobody knew. The pass runs at startup for chunks that
are already loaded, and then per shop as each chunk loads.

{% hint style="info" %}
Only air, water or lava counts as missing. A standing block of any other material is left exactly
where it is, because a material that does not match `shop-item.material` means a reconfigured
server, not a missing block. What gets placed is the CURRENT `shop-item.material` - what a shop was
originally placed as is not recorded anywhere - and nothing is placed into a player.
{% endhint %}

## Currencies

Each entry is one currency, and the order they are declared in is the order the owner menu cycles
through. A currency is backed either by commands or by EdTools.

The file ships one entry of each shape and no more. They are examples, not a currency list to keep:
copy the shape you need as many times as your server has currencies and delete the one you do not.
The section is marked extensible, so an entry you remove stays removed.

```yaml
currencies:
  okicoins:
    display-name: "&#FF9B00okicoins"
    give-command: "eco give {player} {amount}"
    take-command: "eco take {player} {amount}"
    balance-placeholder: "%vault_eco_balance%"
  gems:
    display-name: "&dGems"
    edtoolapi: true
    currency-id: gems
```

{% hint style="warning" %}
A command-backed currency missing `give-command`, `take-command` or `balance-placeholder`, or with a
template that has no `{player}` or no `{amount}`, is SKIPPED at startup with a SEVERE naming the
missing piece. That is deliberate: a currency that is skipped leaves its shops visibly
unconfigured, which you can fix, while one that registers half-configured looks fine and moves no
money.
{% endhint %}

{% hint style="info" %}
An `&` written directly before a placeholder colors the name with whatever color code that
placeholder returns, which is how `"&%itsmyconfig_dcolor%NAME"` works.
{% endhint %}

{% hint style="danger" %}
**Deleting a currency moves its shops, and keeps their prices.** A currency you remove from
`currencies:` no longer strands the shops that traded in it: at the next start, and at the next
`/dshop reload`, they are moved to the first currency in the file, and every move is logged at INFO
naming the shop, its owner and the currency it left.

The NUMBER is not converted, because nothing in the file says what one currency is worth in another.
A shop priced at 100 gems becomes a shop priced at 100 coins. Check what deleting a currency is
about to do to your economy before you delete it.

A currency that is still declared but was SKIPPED at startup - EdTools down, a broken command
template - is left alone, and its shops wait. Only a currency you actually removed from the file
migrates. If nothing is left to move shops TO, the sweep does nothing and says so with a WARNING.
{% endhint %}

## Limits

```yaml
limits:
  max-shops-per-player: 100
  max-shops-permission-prefix: "sndisplayshops.limit"
  external-interaction-cooldown-seconds: 5
  max-pickup-stacks: 256
```

`max-shops-per-player`: `-1` is unlimited, `0` forbids creation. This is the number for a player
holding none of the permission nodes below.

`max-shops-permission-prefix` is the prefix of the per-rank override node - see
[Permissions](permissions.md#how-many-shops-a-player-may-own) for the full node shape and how
several matching nodes resolve. It REPLACES `max-shops-per-player` for that player rather than
adding to it. Leave it blank to disable the permission override entirely, so every player is
governed by `max-shops-per-player` alone.

`max-pickup-stacks` bounds how much one pickup may move. Picking a shop up hands its whole stock
back at once, and stock is unbounded, so a shop holding ten million items would be a hundred and
fifty thousand ground entities in a single tick.

{% hint style="warning" %}
There is no value that switches `max-pickup-stacks` off. Below 1 is read as 256, because "no limit"
here means "one click can hang the server". Raise the number instead if you mean it. Over the limit
the pickup button is refused and says so; nothing is ever destroyed to make a shop fit.
{% endhint %}

## Hologram

`hologram:` in `config.yml` holds the geometry - the item scale, the two vertical offsets, the
rotation timing and the bob. The TEXT lives in `lang/messages_en.yml` under `hologram.text`,
because it is language, not layout.

```yaml
hologram:
  item-scale: 1.2
  item-y-offset: 1.3
  text-y-offset: 3.0
  rotation-interval-ticks: 10
  rotation-period-ticks: 200
  rotation-interpolation-extra-ticks: 2
  bounce-amplitude: 0.1
  bounce-period-ticks: 80
```

The floating item spins and bobs, the way a dropped item does on the ground. `bounce-amplitude` is
how far it rises and falls around `item-y-offset`, in blocks; `0` leaves it hanging still.
`item-y-offset` stays the MIDDLE of the bob rather than its floor, so raising the amplitude swings
the item further around where it already hangs instead of pushing it up into the text.

{% hint style="info" %}
The bob is free. It rides on the same update the spin already sends, so it does not scale with the
number of loaded shops - `rotation-interval-ticks` remains the plugin's single biggest CPU knob.
For the smoothest result keep `bounce-period-ticks` a multiple of four times
`rotation-interval-ticks`; the defaults are exactly eight pushes per bob.
{% endhint %}

{% hint style="info" %}
Rotation and bounce values are clamped to a sane range at startup and a substituted value is
logged. The hologram lines resolve their placeholders against the shop's OWNER, not against whoever
is looking.
{% endhint %}

## Database

```yaml
database:
  type: sqlite
  pool-size: 1
```

SQLite needs no setup. For MySQL, fill in the host, port, database, username and password.

{% hint style="danger" %}
`pool-size` ships at `1` and should stay there. A shop's stock is written as a running total, so two
workers can land the same row's updates out of order and the losing write is the one that survives a
restart - sold stock reappears. The plugin logs a SEVERE at startup if it finds MySQL configured
above 1.
{% endhint %}

## Island integrations

```yaml
integrations:
  superiorskyblock:
    delete-shops-on-island-disband: true
    delete-shops-on-membership-loss: true
```

{% hint style="danger" %}
Removing a shop this way DESTROYS its stock. Everything the shop held is deleted with it: not
dropped, not returned, not logged, and the owner is not told. For a disband that is defensible,
since the island is being wiped anyway. For membership loss the island survives, so an ex-member
loses stock outright. Both are switches for exactly that reason.
{% endhint %}

## Trade log

```yaml
trade-log:
  enabled: true
  flush-seconds: 10
```

One line per COMPLETED trade, appended to `plugins/SnDisplayShops/logs/<date>.log`, a new file per
day, the way the server writes its own logs. A refused or aborted trade writes nothing.

```
[2026-08-15 14:03:11] BUY buyer=Steve buyer-uuid=0a1b… owner=Alex owner-uuid=7f3c… shop=2d9e…
  loc=world:120:64:-338 item="Diamond Sword" qty=3 unit=1000 total=3000 currency=okicoins
```

(one line in the file; wrapped here to fit). `BUY` and `SELL` are the BUYER's side of the trade, so
`BUY` is a player buying from a shop that is in SELL mode. The uuids, `loc`, `shop` and `currency`
are the machine columns; the names beside them are for reading. `unit` and `total` are what actually
moved, not what the shop advertises.

`flush-seconds` is how long lines wait in memory before being written, and it is clamped to 1-300.
Lines are written off the main thread; the server never waits on the disk to finish a trade.

{% hint style="info" %}
Old files are never deleted or compressed. Prune the folder yourself if the server is busy enough
for it to matter.
{% endhint %}

{% hint style="warning" %}
An owner the server has not named yet can appear as `owner=-` on the first line after a restart -
resolving an offline player's name reads from disk, and that is not done during a click. The
`owner-uuid` beside it is always correct, and the name is there from the next trade onward.

Stock removed through the developer API (a sell wand, for instance) is NOT a trade and is not
logged.
{% endhint %}

## Reloading

`/dshop reload` re-reads all four files, both menu layouts and the hologram settings. Changing the
database section needs a restart.
