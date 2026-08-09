# Placeholders

Requires PlaceholderAPI. `<currency>` is a currency **id** as declared in `config.yml`, which is the map key, not the display name.

## Per currency

| Placeholder | Value |
|---|---|
| `%sncoinflip_wins_<currency>%` | Matches won in that currency |
| `%sncoinflip_losses_<currency>%` | Matches lost in that currency |
| `%sncoinflip_wagered_<currency>%` | Total staked in that currency |
| `%sncoinflip_won_<currency>%` | Total winnings in that currency |

## Totals

| Placeholder | Value |
|---|---|
| `%sncoinflip_wins_total%` | Matches won across every currency |
| `%sncoinflip_losses_total%` | Matches lost across every currency |
| `%sncoinflip_wagered_total%` | Total staked across every currency |
| `%sncoinflip_won_total%` | Total winnings across every currency |

{% hint style="info" %}
`won` is the **gross** prize, not net profit. A player who stakes 100 and wins has `wagered` 100 and `won` 200. This is deliberate and it is what the scoreboards on the network that this plugin was written for already show.
{% endhint %}

{% hint style="warning" %}
The four `_total` placeholders are summed across currencies on every read rather than cached, so putting all four on a fast-refreshing scoreboard is more work than putting the per-currency ones there.
{% endhint %}
