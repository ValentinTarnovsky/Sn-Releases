# Placeholders

All placeholders require [PlaceholderAPI](https://www.spigotmc.org/resources/placeholderapi.6245/)
on the backend, and use the `%sncredits_` prefix. The proxy half registers nothing: the expansion
belongs to the backend.

| Placeholder | Description |
|-------------|-------------|
| `%sncredits_balance%` | The balance as a plain integer, for example `100` |
| `%sncredits_balance_formatted%` | The balance with thousands separators, for example `1,234` |
| `%sncredits_balance_short%` | The balance shortened, for example `1.2K` |
| `%sncredits_coinflip_wins%` | How many coinflips this player has won |
| `%sncredits_coinflip_losses%` | How many coinflips this player has lost |
| `%sncredits_coinflip_profit%` | Net coinflip result, with thousands separators |

{% hint style="info" %}
Every value is read from the copy the backend caches, which the proxy sends once the player is
connected. A player whose data has not arrived yet reads `0`, never a blank or an error.
{% endhint %}
