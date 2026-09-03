# Placeholders

## Provided by SnDisplayShops

Needs PlaceholderAPI. Usable anywhere on the server: scoreboards, tab, chat, other plugins' menus.

| Placeholder | Returns |
|---|---|
| `%sndisplayshops_shops_count%` | How many shops the viewing player owns right now. |
| `%sndisplayshops_shops_max%` | How many that player is allowed to own: their [permission override](permissions.md#how-many-shops-a-player-may-own) if they hold one, otherwise `limits.max-shops-per-player`. |
| `%sndisplayshops_shops_total%` | How many shops exist on the whole server. |

A scoreboard line reading `Shops: %sndisplayshops_shops_count%/%sndisplayshops_shops_max%` is the
common use.

{% hint style="info" %}
`shops_max` is read from a cache refreshed every 5 seconds, not resolved live on every render -
walking permissions on every scoreboard tick for every online player is not something PlaceholderAPI
can afford. A rank granted mid-session reaches this placeholder within that window, no relog needed.
{% endhint %}

## Tokens inside SnDisplayShops files

These are not PlaceholderAPI placeholders. They are substituted by the plugin inside its own
messages, menus and hologram lines, and they work whether or not PlaceholderAPI is installed.

### Hologram lines (`lang/messages_en.yml`, `hologram.text`)

| Token | Becomes |
|---|---|
| `{owner}` | the shop owner's name |
| `{item}` | the item the shop trades |
| `{price}` | the price of one unit |
| `{currency}` | the currency's display name |
| `{stock}` | how much the shop holds |
| `{mode}` | the shop's direction, from `status.mode-buy` / `status.mode-sell` |

{% hint style="info" %}
Hologram lines resolve PlaceholderAPI against the shop's OWNER, not against the player reading them.
The hologram is one shared entity that everyone nearby sees, so it cannot say different things to
different people.
{% endhint %}

### Buyer menu (`guis/buyer.yml`)

`{owner}`, `{mode}`, `{action}`, `{price}`, `{currency}`, `{stock}`, `{have}`, `{amount}`.

`{action}` is the INVERSE of the shop's mode, because it describes what the button does to the
person clicking it: a shop in SELL mode offers a stranger a buy.

{% hint style="info" %}
On the offer item, `{price}` is the price of one unit. On the two trade buttons it is the TOTAL that
button charges. The shipped file uses it both ways on purpose.
{% endhint %}

### Owner menu (`guis/owner.yml`)

`{item}` in the title, `{currency}`, `{mode}`, `{price}` and `{stock}` on the control items,
`{amount}` on the stock grid entries, and `{page}` / `{pages}` on the arrows and the page indicator.

{% hint style="info" %}
`{stock}` and `{amount}` are different numbers. `{stock}` is EVERYTHING the shop holds, every
variant summed and written out in full (`1.500.000`) - the amount the withdraw button empties, so
the shipped withdraw button shows it. `{amount}` is one grid cell's own variant, shortened
(`1.2M`), because that one is read across many cells at a glance.
{% endhint %}

## Consumed by SnDisplayShops

A command-backed currency reads its balance through a placeholder you supply, for example
`%vault_eco_balance%`, `%essentials_balance%` or the one your economy plugin documents. That is what
makes PlaceholderAPI effectively required for command-backed currencies. An EdTools-backed currency
reads through the EdTools API instead and needs no placeholder.
