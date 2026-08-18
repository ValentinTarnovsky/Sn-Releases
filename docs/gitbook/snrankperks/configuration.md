# Configuration

SnRankPerks ships with the following YAML files. New keys are auto-merged on boot; your edits
and comments are preserved.

## config.yml

```yaml
# ============================================================
#  SnRankPerks - configuration
#  Managed by SnLib: new keys are auto-merged on boot; your values and
#  comments are preserved. Do NOT add a config-version key.
#  Set update-configs: false to freeze this file (SnLib only warns about
#  missing keys instead of inserting them).
# ============================================================

# Active language code; loads lang/messages_<code>.yml (falls back to en).
lang: en

# Master switch of the SnLib auto-updater for this plugin's managed files.
update-configs: true

# Runtime debug output.
debug:
  enabled: false
  level: DEBUG
  categories: []

# ------------------------------------------------------------
#  Main command.
# ------------------------------------------------------------
command:
  # Aliases of /rankperks. Re-read on /rankperks reload.
  aliases: [perks]

# ------------------------------------------------------------
#  Database. type=sqlite needs nothing else; type=mysql reads host/port/etc.
# ------------------------------------------------------------
database:
  type: sqlite
  host: localhost
  port: 3306
  database: snrankperks
  username: root
  password: ""

# ------------------------------------------------------------
#  1. MODEL - who gets access to every feature below.
# ------------------------------------------------------------
access:
  # GROUP: LuckPerms group membership (+ the snrankperks.bypass permission for
  #        glow/chat-color-only access). Live-synced, no relog needed.
  # PERMISSION: a single permission node, no LuckPerms dependency for the
  #        access check itself (LuckPerms stays optional, only for the prefix
  #        fallback below). Live-synced on /rankperks reload; a grant/revoke made
  #        through a different permissions plugin with no reload in between only
  #        takes effect on the player's next relog.
  mode: GROUP
  # LuckPerms group name to check for access. Only read when mode is GROUP.
  group-name: "okiplus"
  # Permission node granting full access. Only read when mode is PERMISSION.
  permission: "snrankperks.use"

# ------------------------------------------------------------
#  2. FEATURES - join flow, rainbow glow speed, chat emoji passthrough.
# ------------------------------------------------------------
join:
  # Announce a full-access player's connection to all players.
  announcement-enabled: true
  # Give the custom interactive item on join.
  item-enabled: true
  # Hotbar slot for the join item (0-8).
  item-slot: 4
  # Delay in ticks before giving the item (join-handling plugins may still be
  # populating the inventory).
  item-delay-ticks: 5

rainbow:
  # Speed of rainbow glow color cycling in ticks (20 ticks = 1 second).
  speed-ticks: 10

# Emoji passthrough: tokens matching `pattern` are skipped by the per-letter
# chat colorizer so texture-pack glyphs (e.g. :amongus:) render intact.
emoji-passthrough:
  enabled: true
  pattern: ":[A-Za-z0-9_]+:"
```

## items.yml

```yaml
# ============================================================
#  SnRankPerks - physical items
#  Managed by SnLib: new keys are auto-merged on boot; your values and
#  comments are preserved.
# ============================================================

# The join item: given to every player on connect (regardless of access),
# right-click opens the main menu.
join-item:
  material: DIAMOND
  display-name: "&d&lOki&f&lPlus"
  lore:
    - ""
    - "&7Click derecho para abrir"
    - "&7el menu &dOki&fPlus"
    - ""
  right-click-actions:
    - "[open-rankperks-menu]"
```

## Catalog files (chatcolors.yml, glows.yml, prefixes.yml)

These three files are the actual cosmetics you and your players pick from. They are seeded once
from the jar and never auto-updated afterward: add, remove or edit entries freely, and your
changes are never touched again.

Each entry needs a display name, the value it produces, a GUI icon and a GUI slot. A chat color
entry looks like this (from `chatcolors.yml`):

```yaml
chatcolors:
  red:
    display-name: "&cRojo"
    color-code: "&c"
    material: RED_CONCRETE
    slot: 25
    menu: "main"
```

`color-code` accepts a legacy code (`&0`-`&f`), a hex color (`&#RRGGBB`), a gradient
(`&g[#RRGGBB,#RRGGBB,...]`) or a uniform-cycling color (`&u[...]`, same bracket syntax). `menu`
picks which of the three chat-color GUI pages the entry appears on (`main`, `hex` or `custom`).
An entry can also set `base64-texture` to render a custom player-head icon instead of a plain
block.

`glows.yml` entries take a `color` (any `NamedTextColor` name, or `RAINBOW` for the animated
glow) instead of `color-code`. `prefixes.yml` entries take a `value` (the prefix text itself,
any color codes included) instead of `color-code`.

## Language file

`lang/messages_en.yml` holds every player-facing message plus the shared SnLib command
contract. It is managed the same way as `config.yml`: new keys merge in on boot, your edits
survive updates.

## GUI layouts

The six files under `guis/` (`main-menu.yml`, `glow-menu.yml`, `prefix-menu.yml`,
`chat-color-menu.yml`, `chat-color-hex-menu.yml`, `chat-color-custom-menu.yml`) define the
menus themselves: title, size, static buttons and the templates the catalog entries above
render into. Edit them to restyle a menu; the catalog files control what shows up inside it.
