# SnKills

SnKills replaces the vanilla death message with one you write yourself, per damage cause, and
puts the killer's weapon in chat with its real in-game tooltip attached.

## Features

- One message template per Bukkit damage cause: fall, lava, void, drowning, the lot. Leave one
  empty and that death is silent; fill `FALLBACK` and everything else gets a default line.
- `PLAYER_KILL` fires whenever a player did the killing, whatever the damage cause was, so a
  PvP message never has to be duplicated across `ENTITY_ATTACK`, `PROJECTILE` and the rest.
- `%item%` renders the killer's weapon as a hoverable component showing the item's **real**
  tooltip - name, enchantments, lore and attribute lines - exactly what you see hovering it in
  your inventory. Not an approximation built from lore lines.
- Full PlaceholderAPI support in every template, resolved against the victim.
- Colour codes (`&a`), hex (`&#RRGGBB`) and the MiniMessage colour tags (`<gradient>`,
  `<rainbow>`) all render.
- New damage causes added by a Minecraft update are inserted into your `config.yml`
  automatically, keeping your values and your comments.

## What it deliberately does not do

- Interactive MiniMessage tags (`<click>`, `<hover>`, `<font>`) render as literal text. A
  placeholder can resolve to text a player chose, and a death message goes to every online
  player, so live click actions are not worth the exposure.
- The vanilla death message is always suppressed, for every death, whether or not SnKills
  prints one of its own. There is no opt-out key.

## Optional integrations

- **PlaceholderAPI**: any placeholder works inside a death message template, resolved against
  the victim. Without it the tokens are left untouched and the message still sends.

## Links

- Source and releases: part of the shared [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases) repo
