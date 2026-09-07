# Placeholders

All placeholders require [PlaceholderAPI](https://www.spigotmc.org/resources/placeholderapi.6245/) and use the `%sndiscordlink_` prefix.

| Placeholder | Description |
|-------------|-------------|
| `%sndiscordlink_linked%` | Whether the player is linked, rendered with the `status.linked-yes` and `status.linked-no` words from the language file |
| `%sndiscordlink_discord_id%` | The linked Discord id, or the `status.none` word when the player is not linked |
| `%sndiscordlink_discord_name%` | The same id as above, so you can render it as a mention. No display name is stored |
| `%sndiscordlink_boosting%` | Whether the linked member boosts the guild, using `status.boosting-yes` and `status.boosting-no` |
| `%sndiscordlink_linked_count%` | The number of linked accounts across the whole network |

{% hint style="info" %}
The words these placeholders return live in the `status:` block of `lang/messages_en.yml`. Edit them once there and every command, message and placeholder follows.
{% endhint %}

{% hint style="info" %}
`%sndiscordlink_linked_count%` is served from a cache refreshed every `sync.count-seconds`, so it never queries the database from the main thread.
{% endhint %}
