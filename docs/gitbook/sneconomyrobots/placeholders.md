# Placeholders

All placeholders require [PlaceholderAPI](https://www.spigotmc.org/resources/placeholderapi.6245/)
and use the `%sneconomyrobots_` prefix. Slot numbers are 1-based, matching what the player sees in
the menu, and economy ids are matched case-insensitively.

A placeholder whose player is offline, or whose data has not finished loading, returns the
`status.unknown` label. One that resolves to nothing, such as an empty bag or an uncapped robot,
returns `status.none`. Both words are configurable in the language file.

## Counters

| Placeholder | Description |
|-------------|-------------|
| `%sneconomyrobots_equipped%` | Robots currently in active slots |
| `%sneconomyrobots_slots_used%` | The same number, under the slot vocabulary |
| `%sneconomyrobots_slots_unlocked%` | Active slots the player owns |
| `%sneconomyrobots_slots_free%` | Unlocked minus used, never negative |
| `%sneconomyrobots_slots_max%` | The configured `slots.max` ceiling |
| `%sneconomyrobots_storage_used%` | Robots in storage |
| `%sneconomyrobots_storage_max%` | Storage capacity from the player's permissions, refreshed every 5 seconds (a rank granted mid-session shows up within that window) |
| `%sneconomyrobots_storage_free%` | Capacity minus used, never negative |

## Claim bag

| Placeholder | Description |
|-------------|-------------|
| `%sneconomyrobots_bag_pending%` | The whole pending listing |
| `%sneconomyrobots_bag_amount_<economy>%` | That economy's pending balance, abbreviated |
| `%sneconomyrobots_bag_raw_<economy>%` | The same balance unabbreviated, for numeric comparison |
| `%sneconomyrobots_bag_status_<economy>%` | The bag state word |
| `%sneconomyrobots_bag_name_<economy>%` | The economy's display name |

## Accrual rate

Summed over the player's equipped robots, per hour, and it mirrors what production actually pays:
a robot contributes nothing for an economy it has not unlocked or once it has hit its production
cap, the rate reads `0` while that economy's claim bag sits at `bag.max-per-economy` or while the
player is outside `production.active-worlds`, and it reads `status.unknown` while the bag is still
loading.

| Placeholder | Description |
|-------------|-------------|
| `%sneconomyrobots_rate_<economy>%` | Accrual per hour, abbreviated |
| `%sneconomyrobots_rate_raw_<economy>%` | The same figure unabbreviated |

## Per active slot

| Placeholder | Description |
|-------------|-------------|
| `%sneconomyrobots_slot_<n>_type%` | The robot id |
| `%sneconomyrobots_slot_<n>_tier%` | The tier's display name |
| `%sneconomyrobots_slot_<n>_tier_id%` | The integer tier id |
| `%sneconomyrobots_slot_<n>_rarity%` | The tier's rarity word |
| `%sneconomyrobots_slot_<n>_multiplier%` | The instance multiplier |
| `%sneconomyrobots_slot_<n>_produced%` | The production counter |
| `%sneconomyrobots_slot_<n>_limit%` | The production cap, or `status.none` if it never stops |
| `%sneconomyrobots_slot_<n>_interval%` | Seconds between payouts, after every upgrade |
| `%sneconomyrobots_slot_<n>_status%` | The robot's state word |

A slot the player has not unlocked answers `status.slot-locked` for `status`, and `status.none` for
every other field. An unlocked but empty slot answers `status.slot-empty` and `status.none`.

{% hint style="warning" %}
An EdTools currency whose id begins with `raw_`, `amount_`, `status_` or `name_` can collide with
the prefixes above. For example `rate_raw_gold` reads as `rate_raw_` plus `gold`. Rename the
currency if you hit this, because PlaceholderAPI has no escaping syntax.
{% endhint %}
