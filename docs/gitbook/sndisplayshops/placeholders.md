# Placeholders

## Provided by SnDisplayShops

Needs PlaceholderAPI. Usable anywhere on the server: scoreboards, tab, chat, other plugins' menus.

| Placeholder | Returns |
|---|---|
| `%sndisplayshops_shops_count%` | How many shops the viewing player owns right now. |
| `%sndisplayshops_shops_max%` | How many that player is allowed to own (`limits.max-shops-per-player`). |
| `%sndisplayshops_shops_total%` | How many shops exist on the whole server. |

A scoreboard line reading `Shops: %sndisplayshops_shops_count%/%sndisplayshops_shops_max%` is the
common use.

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

### Owner menu (`owner-menu.yml`)

`{currency}`, `{mode}` and `{price}` on the control items, and `{amount}` on the stock grid entries.

{% hint style="info" %}
Items under `decorative:` are built without a viewer, so per-player PlaceholderAPI does not resolve
there. Keep those to static text.
{% endhint %}

## Consumed by SnDisplayShops

A command-backed currency reads its balance through a placeholder you supply, for example
`%vault_eco_balance%`, `%essentials_balance%` or the one your economy plugin documents. That is what
makes PlaceholderAPI effectively required for command-backed currencies. An EdTools-backed currency
reads through the EdTools API instead and needs no placeholder.
