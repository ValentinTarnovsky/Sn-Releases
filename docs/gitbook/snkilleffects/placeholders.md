# Placeholders

All placeholders require [PlaceholderAPI](https://www.spigotmc.org/resources/placeholderapi.6245/) and use the `%snkilleffects_` prefix.

| Placeholder | Description |
|-------------|-------------|
| `%snkilleffects_active%` | Display name of the player's active effect, or the "no effect" label |
| `%snkilleffects_active_id%` | Raw id of the active effect, `random` in random mode, empty when none |
| `%snkilleffects_owned_count%` | How many enabled effects the player owns |
| `%snkilleffects_total_count%` | How many enabled effects the server ships |
| `%snkilleffects_has_<id>%` | `true` or `false`, whether the player owns that effect |
| `%snkilleffects_expires_<id>%` | Remaining time of a timed grant, empty when permanent or not owned |

For example, `%snkilleffects_has_rocket%` and `%snkilleffects_expires_rocket%` answer for the `rocket` effect.

{% hint style="info" %}
These read from the in-memory player cache, so they only answer for players who are online. An offline player reads as owning nothing.
{% endhint %}
