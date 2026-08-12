# Configuration

Four files, all merged on update:

| File | Holds |
|---|---|
| `config.yml` | the shop item, the hologram, limits, currencies, the database, integrations |
| `lang/messages_en.yml` | every line the plugin sends, plus the hologram text |
| `guis/buyer.yml` | the layout of the menu everyone but the owner sees |
| `owner-menu.yml` | the layout of the owner's management menu |

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

## Shop defaults

```yaml
shop:
  defaults:
    price: 0
    mode: SELL
    start-paused: true
```

`mode` is from the SHOP's point of view: `SELL` means the shop sells to players, `BUY` means it buys
from them. A fresh shop starts paused so nobody trades with it before its owner has set the item,
the price and the currency.

## Currencies

Each entry is one currency, and the order they are declared in is the order the owner menu cycles
through. A currency is backed either by commands or by EdTools.

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

## Limits

```yaml
limits:
  max-shops-per-player: 100
  external-interaction-cooldown-seconds: 5
  max-pickup-stacks: 256
```

`max-shops-per-player`: `-1` is unlimited, `0` forbids creation.

`max-pickup-stacks` bounds how much one pickup may move. Picking a shop up hands its whole stock
back at once, and stock is unbounded, so a shop holding ten million items would be a hundred and
fifty thousand ground entities in a single tick.

{% hint style="warning" %}
There is no value that switches `max-pickup-stacks` off. Below 1 is read as 256, because "no limit"
here means "one click can hang the server". Raise the number instead if you mean it. Over the limit
the pickup button is refused and says so; nothing is ever destroyed to make a shop fit.
{% endhint %}

## Hologram

`hologram:` in `config.yml` holds the geometry - the item scale, the two vertical offsets and the
rotation timing. The TEXT lives in `lang/messages_en.yml` under `hologram.text`, because it is
language, not layout.

{% hint style="info" %}
Rotation values are clamped to a sane range at startup and a substituted value is logged. The
hologram lines resolve their placeholders against the shop's OWNER, not against whoever is looking.
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

## Reloading

`/dshop reload` re-reads all four files, both menu layouts and the hologram settings. Changing the
database section needs a restart.
