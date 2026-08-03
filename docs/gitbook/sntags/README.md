# SnTags

SnTags is a chat tag system for Paper servers. Staff create tags, hand them out to players, and each player picks the one they wear from a paginated selector menu. The active tag is exported through PlaceholderAPI, so chat plugins, TAB and scoreboards render it wherever you want it.

## Features

- A global tag catalogue kept in `tags.yml`, editable by hand or through `/tagadmin create` and `/tagadmin delete`. The order of the entries in the file is the order they appear in the menu.
- Per-player ownership: a player only sees the tags they have been given, and picks one from `/tags`.
- Personal tags with `/tagadmin custom`, minted for a single player and identified by a numeric id.
- A fully configurable selector menu - slots, materials, lore, pagination and sounds all live in `guis/tag-selector.yml`, with nothing hardcoded in the plugin.
- `sntags.admin.viewall` lets staff see and try every tag in the catalogue, owned or not.
- Two placeholders, `%sntags_tag%` and `%sntags_has_tag%`, with raw uncolorized output so the consuming plugin controls the rendering.
- Display text is validated on input: colours, decorations, `<gradient>` and `<rainbow>` are accepted, and interactive MiniMessage is refused, because tag text reaches other players' chat.
- SQLite or MySQL, with every write off the main thread.

## Optional integrations

SnTags runs on **SnLib** alone. One optional soft dependency:

- **PlaceholderAPI**: registers the `%sntags_*%` placeholders. Without it the plugin runs normally and the expansion is simply not registered.

## Version 2.0.0

2.0.0 is a full rewrite on SnLib. If you are coming from 1.x, read [Installation](installation.md) before putting it on a live server - it is a clean break rather than a drop-in update, and skipping the upgrade steps leaves a MySQL server on an empty database.

## Links

- Releases: [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases/releases?q=sntags-)
