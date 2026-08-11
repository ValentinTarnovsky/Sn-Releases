# Placeholders

All placeholders below require [PlaceholderAPI](https://www.spigotmc.org/resources/placeholderapi.6245/)
and use the `%snkits_` prefix. SnKits detects PlaceholderAPI at startup and registers the expansion
only when it is present. Without it the plugin is fully functional.

Each one takes a kit id as the last part of the identifier.

| Placeholder | Description |
|-------------|-------------|
| `%snkits_cooldown_<id>%` | Time left on the oldest recharging copy, formatted, or the ready label |
| `%snkits_available_<id>%` | Whether the player can claim the kit right now |
| `%snkits_onetime_used_<id>%` | Whether the player already spent a one-time kit |
| `%snkits_uses_left_<id>%` | Claims of the kit still available right now |
| `%snkits_uses_max_<id>%` | Claims of the kit the player owns in total |

```
%snkits_cooldown_default%
%snkits_uses_left_ninja%
```

Every one reads from memory, so putting them on a scoreboard that refreshes every tick costs
nothing.

## Inside SnKits itself

The plugin's own menus and messages carry four tokens that need no PlaceholderAPI at all:

| Token | Description |
|-------|-------------|
| `{kit}` | The kit id |
| `{cooldown}` | Time left on the oldest recharging copy, or the ready label |
| `{uses_left}` | Copies of this kit the viewer can still claim |
| `{uses_max}` | Copies of this kit the viewer owns in total |

They resolve in `display-name` and `lore` in all four kit states.

{% hint style="info" %}
Kit item names and lore resolve PlaceholderAPI tokens too, per player, both in the preview and on
the item that lands in the inventory.
{% endhint %}

{% hint style="warning" %}
The identifier strings are a public contract and do not change between versions. A placeholder
typed into a scoreboard or tab plugin keeps working across updates.
{% endhint %}
