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

That is a config number, not a permission: `limits.max-shops-per-player` in `config.yml`. It is
server-wide. `-1` is unlimited and `0` forbids creation entirely.
