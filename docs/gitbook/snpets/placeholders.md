# Placeholders

All placeholders require [PlaceholderAPI](https://www.spigotmc.org/resources/placeholderapi.6245/)
and use the `%snpets_` prefix. Without PlaceholderAPI the plugin runs normally and simply does
not register the expansion.

## Collection

| Placeholder | Description |
|-------------|-------------|
| `%snpets_pets_total%` | Pets owned in total, stored plus equipped |
| `%snpets_pets_stored%` | Pets currently in storage |
| `%snpets_pets_equipped%` | Pets currently equipped |

## Capacity

| Placeholder | Description |
|-------------|-------------|
| `%snpets_storage_used%` | Storage slots in use |
| `%snpets_storage_total%` | Total storage capacity, permissions and purchases included |
| `%snpets_storage_free%` | Storage capacity still free |
| `%snpets_slots_used%` | Equip slots in use |
| `%snpets_slots_total%` | Total equip slots, permissions and purchases included |
| `%snpets_slots_free%` | Equip slots still free |

## Buffs and currencies

| Placeholder | Description |
|-------------|-------------|
| `%snpets_buff_damage%` | Effective damage buff percentage, cap and world gate included |
| `%snpets_buff_resistance%` | Effective resistance buff percentage |
| `%snpets_buff_speed%` | Effective speed buff percentage |
| `%snpets_currency_trait-ticket%` | Trait tickets held |
| `%snpets_currency_dice-normal%` | Normal dice held |
| `%snpets_currency_dice-special%` | Special dice held |

A buff placeholder reports what the player actually receives right now. In a world where the
buff is gated off it reports zero, not the raw total.

## One equip slot

| Placeholder | Description |
|-------------|-------------|
| `%snpets_equipped_<n>%` | Display name of the pet in slot `<n>` |
| `%snpets_equipped_<n>_level%` | Level of the pet in slot `<n>` |
| `%snpets_equipped_<n>_exp%` | Experience of the pet in slot `<n>` |
| `%snpets_equipped_<n>_exp_next%` | Experience still needed for the next level |
| `%snpets_equipped_<n>_cap%` | Level cap of the pet in slot `<n>` |

Slot numbers start at 1. An empty slot answers the configured "none" word for the name and
`0` for a number, so a scoreboard row never renders blank. An unknown slot or field leaves the
token untouched instead of pretending it means something.

Since 1.3.0 the occupied slots are always `1..n` with no gap: taking a pet out of the middle
slides the rest down, so the free slots are the last ones. A scoreboard listing slots 1 to 3
therefore fills from the top and never shows an empty row between two pets.
