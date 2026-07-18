# Configuration

SnClans ships with the following YAML files. New keys are auto-merged on boot; your edits and comments are preserved.

## config.yml

```yaml
# ============================================================
#  SnClans - configuration
#  Managed by SnLib: new keys are auto-merged on boot; your values and
#  comments are preserved. Do NOT add a config-version key (retired).
#  Set update-configs: false to freeze this file (SnLib only warns about
#  missing keys instead of inserting them).
# ============================================================

# Active language code; loads lang/messages_<code>.yml (falls back to en).
lang: en

# Master switch of the SnLib auto-updater for this plugin's managed files.
update-configs: true

# Runtime debug output (also toggleable live via /clan debug).
debug:
  enabled: false

# ------------------------------------------------------------
#  Main command. The alias list is re-read on /clan reload.
# ------------------------------------------------------------
command:
  # Aliases of /clan. Re-read on /clan reload.
  aliases: [c]

# ------------------------------------------------------------
#  Database. type=sqlite needs nothing else; type=mysql reads host/port/etc.
# ------------------------------------------------------------
database:
  # sqlite or mysql
  type: sqlite
  # MySQL connection (ignored when type is sqlite).
  host: localhost
  port: 3306
  database: snclans
  username: root
  password: ""

# ------------------------------------------------------------
#  Presentation: how each interactive feature is shown.
#  gui  = open a menu   |   chat = print to chat
# ------------------------------------------------------------
presentation:
  # /clan info [clan]
  info: gui
  # /clan list
  list: gui
  # /clan top <point>
  top: gui
  # /clan stats
  stats: gui
  # /clan members
  members: gui
  # Bare /clan and /clan menu. gui = open the main menu; chat = show command help.
  main: gui

# ------------------------------------------------------------
#  Clan creation limits and validation
# ------------------------------------------------------------
clan:
  # Maximum members a clan can hold (leader included).
  max-members: 10
  name:
    # Min/max visible length of a clan name, styling excluded.
    min-length: 3
    max-length: 16
    # Characters allowed in the plain (styling-stripped) clan name.
    regex: "^[A-Za-z0-9_]+$"
    # Names that cannot be used, matched case-insensitively against the plain name.
    blacklist: [admin, staff, owner, server, mod, sn, snclans]
    # Styling a player may put in a clan name. enabled: false keeps names plain.
    style:
      enabled: false
      allow-legacy-colors: true
      allow-hex: true
      allow-bold: true
      allow-italic: true
      allow-underline: true
      allow-strikethrough: false
      allow-obfuscated: false
      allow-minimessage: false
      allow-gradient: false
      # reject = drop all styling to plain text; strip = remove only disallowed styling.
      on-disallowed: reject
  tag:
    # Whether clans have a short tag/prefix.
    enabled: true
    # Min/max visible length of a clan tag, styling excluded.
    min-length: 2
    max-length: 5
    # Characters allowed in the plain (styling-stripped) clan tag.
    regex: "^[A-Za-z0-9]+$"
    # Styling a player may put in a clan tag. enabled: false keeps tags plain.
    style:
      enabled: false
      allow-legacy-colors: true
      allow-hex: true
      allow-bold: true
      allow-italic: true
      allow-underline: true
      allow-strikethrough: false
      allow-obfuscated: false
      allow-minimessage: false
      allow-gradient: false
      # reject = drop all styling to plain text; strip = remove only disallowed styling.
      on-disallowed: reject
  allies:
    # Master switch of the ally system. false = ally commands, ally chat and
    # ally friendly-fire are disabled (stored alliances are kept but inert).
    enabled: true
    # Maximum simultaneous alliances a clan can hold. 0 = unlimited.
    max: 1

# ------------------------------------------------------------
#  Role display names. The ladder (Leader > Co-Leader > Officer > Member)
#  is fixed; only the display strings below are configurable.
# ------------------------------------------------------------
roles:
  leader:
    display: "&#8354f2Leader"
  co-leader:
    display: "&#8354f2Co-Leader"
  officer:
    display: "&#8354f2Officer"
  member:
    display: "&7Member"

# Minimum role required to perform each managed action. These thresholds seed a
# clan's editable permission matrix on creation. disband and transfer are always
# leader-only and are not part of the matrix.
action-roles:
  invite: officer
  kick: officer
  ban: co-leader
  unban: co-leader
  promote: co-leader
  demote: co-leader
  rename: co-leader
  sethome: officer
  banner: officer
  pvp: co-leader
  description: co-leader
  open-close: co-leader
  ally: co-leader

# ------------------------------------------------------------
#  Cooldowns (seconds). 0 disables a cooldown.
# ------------------------------------------------------------
cooldowns:
  # Between clan-home teleports per player.
  home: 60
  # Between clan renames per player.
  rename: 300

# ------------------------------------------------------------
#  Clan home teleport
# ------------------------------------------------------------
home:
  # Seconds the player must stand still before the teleport fires (0 = instant).
  # Moving to another block or taking damage cancels the warmup.
  warmup-seconds: 5

# ------------------------------------------------------------
#  Rally banner. /clan banner spawns a banner at the leader's location;
#  clanmates run /clan banner to teleport to it. It despawns after the
#  duration below.
# ------------------------------------------------------------
banner:
  # Banner block material placed on rally.
  material: WHITE_BANNER
  # How long the rally banner stays active before despawning.
  duration-seconds: 45
  # How long a clan must wait between placing rally banners.
  cooldown-seconds: 600
  # Teleport warmup for clanmates joining the rally.
  teleport-warmup-seconds: 0
  # Per-player cooldown between rally teleports to the banner.
  rally-cooldown-seconds: 3
  # Floating countdown hologram above the rally banner.
  hologram:
    # Whether to show the hologram.
    enabled: true
    # Hologram backend: snlib (built-in, no extra plugin) or decentholograms
    # (requires the DecentHolograms plugin; falls back to snlib if missing).
    # Line spacing under decentholograms follows DecentHolograms' own config.
    provider: snlib
    # Height above the banner block the hologram floats at.
    y-offset: 1.8
    # Hologram lines. Placeholders: {clan} = clan name, {time} = remaining rally time.
    lines:
      - "&#8354f2&l{clan}"
      - "&7Rally &8- &e{time}"

# ------------------------------------------------------------
#  Invites
# ------------------------------------------------------------
invite:
  # How long a pending invite lasts before it expires.
  expiry-seconds: 60

# ------------------------------------------------------------
#  PvP / friendly fire
# ------------------------------------------------------------
pvp:
  # Default friendly-fire state for new clans (false = clanmates cannot hurt each other).
  default-friendly-fire: false
  # Whether /clan pvp can toggle the clan friendly-fire flag.
  allow-toggle: true
  # Whether allied clans can damage each other (false = protected).
  ally-friendly-fire: false

# ------------------------------------------------------------
#  Chat channels. {role} {player} {clan} {message} {tag} are substituted, and
#  %placeholderapi% tokens in the format are resolved. Players need the
#  snclans.chat.color permission to use color codes in their own message.
# ------------------------------------------------------------
chat:
  clan-format: "&8[&#8354f2Clan&8] &7{role} &f{player}&8: &7{message}"
  ally-format: "&8[&#8354f2Ally&8] &7{clan} &f{player}&8: &7{message}"

# ------------------------------------------------------------
#  Custom points. Each entry is a point type clans accumulate.
#  /clan top <id> ranks clans by that type. /clan givepoint <id> <n>
#  grants points manually. The info menu shows one {points_<id>}
#  placeholder per entry. Trigger types:
#    player-kill  = points to the killer's clan per player kill
#    mob-kill     = points to the killer's clan per mob kill
#    death        = points removed from the victim's clan per death
# ------------------------------------------------------------
points:
  kills:
    # Name shown in menus and leaderboards.
    display: "&#8354f2Kills"
    triggers:
      # Award to the killer's clan on a player kill.
      player-kill: 1
      # Remove from the victim's clan on death.
      death: 1
  mobkills:
    # Name shown in menus and leaderboards.
    display: "&#8354f2Mob Kills"
    triggers:
      # Award to the killer's clan on a mob kill.
      mob-kill: 1

# ------------------------------------------------------------
#  Region / world restrictions. WorldGuard is a soft dependency; if it is
#  absent, region rules are ignored and only world lists apply.
#  worlds-mode / regions-mode: blacklist (deny listed) or whitelist (allow only listed)
# ------------------------------------------------------------
restrictions:
  home:
    worlds-mode: blacklist
    worlds: []
    regions-mode: blacklist
    regions: []
  banner:
    worlds-mode: blacklist
    worlds: []
    regions-mode: blacklist
    regions: []
  pvp:
    worlds-mode: blacklist
    worlds: []
    regions-mode: blacklist
    regions: []

# ------------------------------------------------------------
#  Notifications
# ------------------------------------------------------------
notifications:
  # Notify online clan members when their clan is affected (invite, kick, promote...).
  notify-clan: true
  # Notify online clan members when a clanmate connects or disconnects.
  connection-events: true
  # Notify staff holding snclans.notify of admin-relevant clan events.
  notify-staff: true

# ------------------------------------------------------------
#  Server-wide broadcasts for clan lifecycle events.
# ------------------------------------------------------------
broadcasts:
  # Announce to the whole server when a clan is created.
  create: true
  # Announce to the whole server when a clan is disbanded.
  disband: true
  # Announce to the whole server when a clan is renamed.
  rename: true
```

## Notable settings

A few keys deserve a closer note on how they behave at runtime.

### notifications.connection-events

When `true`, online clan members see a short line as a clanmate connects or disconnects. Set it to `false` to silence those join and quit notices.

### clan.allies.enabled

Master switch of the ally system. When `false`, the ally commands are disabled, ally chat is rerouted to clan chat, and ally friendly-fire is ignored. Stored alliances are kept but stay inert until you switch it back on.

### clan.allies.max

Maximum simultaneous alliances a clan can hold, where `0` means unlimited. The shipped default is now `1`. Existing installs keep their current value: SnLib only inserts missing keys and never overwrites yours.

### banner.hologram.provider

Chooses the rally hologram backend: `snlib` (built-in, no extra plugin) or `decentholograms`. When you pick `decentholograms` but the plugin is absent, SnClans falls back to `snlib` and logs a console warning. Under `decentholograms`, line spacing follows DecentHolograms' own config.

### banner.hologram.lines

The hologram text, one entry per line. Two placeholders are available: `{clan}` for the clan name and `{time}` for the remaining rally time. The default shows the clan name above a `Rally` countdown line.

### banner.cooldown-seconds

The remaining banner cooldown is recomputed against the live value on every check. Editing it and running `/clan reload` retimes cooldowns that are already running, with no restart needed.

## Other managed YAML

These files are also auto-merged on boot, so your edits and comments survive updates.

- `lang/messages_en.yml`: all player-facing messages. Copy it to `messages_<code>.yml` and set `lang` in `config.yml` to add a language.
- `guis/main.yml`, `guis/info.yml`, `guis/list.yml`, `guis/top.yml`, `guis/members.yml`, `guis/permissions.yml`, `guis/confirm.yml`: layout, items, and titles for each menu.

The `guis/` files are seeded on first boot, so every menu opens correctly the first time you start the plugin.

## Language file

`lang/messages_en.yml` holds the message prefix, the shared `snlib` command contract, your own `messages`, and the chat `lists`.

The `prefix` value at the top of the file is prepended automatically by SnLib to every single-line message sent through it. Do not write a literal prefix token inside any message value: SnLib adds the configured prefix for you, and a hardcoded one would render twice.

The `snlib` block is SnLib's shared command contract: 11 keys covering permission errors, usage, number and value validation, unknown subcommands, reload confirmation, and the help header, entry, and footer. Ship the full block so SnClans matches the rest of the Sn fleet. SnLib fills any omitted key with a neutral default, which leaves an unbranded line. Placeholders such as `{plugin}`, `{usage}`, `{value}`, and `{command}` are substituted by SnLib.
