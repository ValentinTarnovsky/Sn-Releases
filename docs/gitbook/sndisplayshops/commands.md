# Commands

Root command `/dshop`, with the aliases `/displayshop` and `/displayshops`.

{% hint style="info" %}
Every command here is for staff. Players never type anything: they place the shop item, right-click
their own shop to manage it, and right-click somebody else's to trade.
{% endhint %}

| Command | Permission | What it does |
|---|---|---|
| `/dshop give <player> [amount]` | `sndisplayshops.admin.give` | Gives a player shop items. Amount defaults to 1. |
| `/dshop bypass` | `sndisplayshops.admin.bypass` | Toggles staff bypass for yourself. While it is on you open the OWNER menu of any shop you right-click instead of its buyer menu. |
| `/dshop reload` | `sndisplayshops.admin.reload` | Reloads config, language, both menu layouts and the hologram settings. |
| `/dshop help` | `sndisplayshops.admin` | Lists the commands you may run. |
| `/dshop debug` | `sndisplayshops.admin.debug` | Toggles debug logging. |

## Aliases

The aliases live in `config.yml` under `command.aliases` and are re-read on `/dshop reload`, so you
can add your own without touching the jar:

```yaml
command:
  aliases:
    - displayshop
    - displayshops
```

{% hint style="info" %}
Aliases you ADD here work immediately after a reload. The two shipped above are also declared in the
plugin's own `plugin.yml`, so removing them from this list does not un-register them until the next
server start.
{% endhint %}

## Bypass

`/dshop bypass` is a toggle, not a per-shop command. While it is on, right-clicking any shop opens
its owner menu, so you can fix a price, unpause a shop or pick one up on a player's behalf.

{% hint style="warning" %}
Bypass survives a relog, deliberately, so a staff member who disconnects mid-job comes back where
they left off. It is checked again on every single click, so revoking the permission or toggling it
off takes effect immediately, even on a menu that is already open.
{% endhint %}
