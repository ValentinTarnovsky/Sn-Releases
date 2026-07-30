# SnEconomyRobots

SnEconomyRobots is an EdTools addon. Players equip robots into active slots, and those robots accrue
passive income across several economies at once. The income lands in a claim bag instead of being
paid out tick by tick, so a player claims it when they choose.

## Features

- Slot-based robots: each player owns a number of active slots and a separate storage for the rest.
- Multi-economy passive income, accrued in memory and paid in one aggregated call per economy.
- A claim bag, so income is collected on demand rather than trickling into the balance.
- Robot boxes that roll new robots straight into storage, with per-tier chance tables.
- Merging: combine robots into a higher tier through a weighted roll you configure.
- Upgrade tracks that raise a robot's production interval or its lifetime production cap.

## Optional integrations

- **PlaceholderAPI**: exposes the full `%sneconomyrobots_...%` placeholder set for scoreboards, tab
  lists and holograms. Without it the plugin runs normally and simply registers no placeholders.
- **Vault**: lets a robot pay into a Vault economy alongside EdTools currencies. Without it, only
  EdTools currencies are available.

## Links

- Source and releases: part of the shared [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases) repo
