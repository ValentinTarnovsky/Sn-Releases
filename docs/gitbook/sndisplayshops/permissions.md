# Permissions

| Node | Default | What it grants |
|---|---|---|
| `sndisplayshops.use` | everyone | Place a shop, manage your own, and trade at anyone else's. |
| `sndisplayshops.admin` | op | The `/dshop` command itself, and every node below. |
| `sndisplayshops.admin.give` | op | `/dshop give` |
| `sndisplayshops.admin.bypass` | op | `/dshop bypass`, and opening any shop's owner menu while it is on. |
| `sndisplayshops.admin.reload` | op | `/dshop reload` |
| `sndisplayshops.admin.debug` | op | `/dshop debug` |
| `sndisplayshops.admin.update` | op | Receives the update notice on join when a new version is released. |
| `sndisplayshops.limit` | false | Parent of `sndisplayshops.limit.<n>`. Grants no allowance by itself. |
| `sndisplayshops.limit.unlimited` | false | Unlimited shops for this rank, whatever `max-shops-per-player` says. Beats every numeric node. |

{% hint style="warning" %}
`sndisplayshops.admin` is the command root. A staff member who does not hold it cannot run `/dshop`
at all, not even `/dshop help`.
{% endhint %}

## Revoking `sndisplayshops.use`

The node gates two things at once: creating a shop, and trading at someone else's. Revoking it from
a rank stops that rank from opening any shop, not just from making one.

{% hint style="info" %}
There is no separate node for "may create a shop" versus "may buy from one". If you want to sell
shop creation as a rank perk, gate the shop ITEM instead - hand it out through a crate, a kit or a
shop, and leave `sndisplayshops.use` alone.
{% endhint %}

## How many shops a player may own

`limits.max-shops-per-player` in `config.yml` is the number for a player holding none of the
nodes below. `-1` is unlimited and `0` forbids creation entirely.

A rank can override it with `sndisplayshops.limit.<n>` - for example
`/lp group vip permission set sndisplayshops.limit.25` lets VIP own 25 shops, whatever the config
number says. Grant `sndisplayshops.limit.unlimited` for no cap at all. If a player holds more than
one matching node, the HIGHEST amount wins, and `unlimited` beats every number.

{% hint style="info" %}
A node REPLACES the config number for that player, it does not add to it: a rank granted
`sndisplayshops.limit.5` while the config default is 100 owns 5 shops, not 105. This is also why
`sndisplayshops.limit.*` grants nothing - it is not a wildcard, unlike the node prefix itself.
{% endhint %}

{% hint style="warning" %}
Demoting a rank below what a player already owns does not touch their existing shops - they keep
trading exactly as before. Only NEW placements are refused until the count drops back under the
new limit.
{% endhint %}
