# Placeholders

All placeholders require [PlaceholderAPI](https://www.spigotmc.org/resources/placeholderapi.6245/)
and use the `%sncompanions_` prefix. Without PlaceholderAPI the plugin runs normally and simply does
not register the expansion.

## Collection

| Placeholder | Description |
|-------------|-------------|
| `%sncompanions_companions_total%` | Companions owned in total, stored plus equipped |
| `%sncompanions_companions_stored%` | Companions currently in storage |
| `%sncompanions_companions_equipped%` | Companions currently equipped |

## Capacity

| Placeholder | Description |
|-------------|-------------|
| `%sncompanions_storage_used%` | Storage slots in use |
| `%sncompanions_storage_total%` | Total storage capacity, permissions and purchases included |
| `%sncompanions_storage_free%` | Storage capacity still free |
| `%sncompanions_slots_used%` | Equip slots in use |
| `%sncompanions_slots_total%` | Total equip slots, permissions and purchases included |
| `%sncompanions_slots_free%` | Equip slots still free |

## Buffs

| Placeholder | Description |
|-------------|-------------|
| `%sncompanions_buff_damage%` | Effective damage buff percentage, cap and world gate included |
| `%sncompanions_buff_resistance%` | Effective resistance buff percentage |
| `%sncompanions_buff_speed%` | Effective speed buff percentage |

A buff placeholder reports what the player actually receives right now. In a world where the
buff is gated off it reports zero, not the raw total.

## One equip slot

| Placeholder | Description |
|-------------|-------------|
| `%sncompanions_equipped_<n>%` | Display name of the companion in slot `<n>` |
| `%sncompanions_equipped_<n>_level%` | Level of the companion in slot `<n>` |
| `%sncompanions_equipped_<n>_exp%` | Experience of the companion in slot `<n>` |
| `%sncompanions_equipped_<n>_exp_next%` | Experience still needed for the next level |
| `%sncompanions_equipped_<n>_cap%` | Level cap of the companion in slot `<n>` |

Slot numbers start at 1. An empty slot answers the configured "none" word for the name and
`0` for a number, so a scoreboard row never renders blank. An unknown slot or field leaves the
token untouched instead of pretending it means something.

The occupied slots are always `1..n` with no gap: taking a companion out of the middle
slides the rest down, so the free slots are the last ones. A scoreboard listing slots 1 to 3
therefore fills from the top and never shows an empty row between two companions.
