# Placeholders

All placeholders require [PlaceholderAPI](https://www.spigotmc.org/resources/placeholderapi.6245/) and use the `%battlepass_` prefix.

## Progress

| Placeholder | Description |
|-------------|-------------|
| `%battlepass_tier%` | The player's current tier |
| `%battlepass_max_tier%` | The highest tier configured |
| `%battlepass_xp%` | XP earned toward the next tier |
| `%battlepass_xp_required%` | XP the next tier needs |
| `%battlepass_progress%` | Progress toward the next tier, as a percentage |
| `%battlepass_pass%` | The player's pass, as a display name |
| `%battlepass_pass_id%` | The player's pass, as an id for comparisons |
| `%battlepass_playtime%` | Tracked playtime, in raw form |
| `%battlepass_playtime_formatted%` | Tracked playtime, formatted for display |

## Challenge slots

`%battlepass_challenge_<n>_<field>%` reads one challenge slot, where `<n>` is the slot number starting at 1.

| Field | Description |
|-------|-------------|
| `state` | Whether the slot is empty, running, finished or cooling down |
| `type` | The challenge type in the slot |
| `name` | The challenge display name |
| `progress` | How far the player has come |
| `target` | What the challenge asks for |
| `reward` | The XP the challenge pays |
| `percent` | Progress as a percentage |
| `expires` | Time left before the challenge expires |
| `cooldown` | Time left on the slot's cooldown |

For example, `%battlepass_challenge_1_percent%` gives the completion percentage of the first slot.

{% hint style="info" %}
Field names are not case sensitive, so `%battlepass_challenge_1_STATE%` and `%battlepass_challenge_1_state%` resolve the same way.
{% endhint %}
