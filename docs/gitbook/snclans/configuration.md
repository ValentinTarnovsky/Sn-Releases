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
  # /clan roster
  roster: gui
  # /clan (bare command with no args): open the main menu, or show chat help
  main: gui

# ------------------------------------------------------------
#  Clan creation limits and validation
# ------------------------------------------------------------
clan:
  # Maximum members a clan can hold (leader included).
  max-members: 10
  name:
    # Min/max length of a clan name (stripped of colors).
    min-length: 3
    max-length: 16
    # Characters allowed in a raw (stripped) clan name.
    regex: "^[A-Za-z0-9_]+$"
    # Names that cannot be used (case-insensitive, matched against stripped name).
    blacklist: [admin, staff, owner, server, mod, sn, snclans]
  tag:
    # Whether clans have a short tag/prefix.
    enabled: true
    # Min/max length of a clan tag.
    min-length: 2
    max-length: 5
    regex: "^[A-Za-z0-9]+$"
  # Default join policy for newly created clans: open | request | invite-only
  default-join-policy: invite-only

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

# Minimum role required to perform each managed action.
# (disband and transfer are always leader-only and are not configurable here.)
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
  settings: co-leader
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
  warmup-seconds: 5
  # Cancel the warmup if the player moves to another block.
  cancel-on-move: true
  # Cancel the warmup if the player takes damage.
  cancel-on-damage: true

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
  # How long a clan must wait between rallies.
  cooldown-seconds: 600
  # Teleport warmup for clanmates joining the rally.
  teleport-warmup-seconds: 0

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
#  grants points manually. triggers award points automatically:
#    player-kill  = points to the killer's clan per player kill
#    mob-kill     = points to the killer's clan per mob kill
#    death        = points removed from the victim's clan per death
# ------------------------------------------------------------
points:
  kills:
    display: "&#8354f2Kills"
    triggers:
      player-kill: 1
  mobkills:
    display: "&#8354f2Mob Kills"
    triggers:
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
  # Notify staff holding snclans.notify of admin-relevant clan events.
  notify-staff: true
```

## Other managed YAML

These files are also auto-merged on boot, so your edits and comments survive updates.

- `lang/messages_en.yml`: all player-facing messages. Copy it to `messages_<code>.yml` and set `lang` in `config.yml` to add a language.
- `guis/main.yml`, `guis/info.yml`, `guis/list.yml`, `guis/top.yml`, `guis/roster.yml`, `guis/settings.yml`, `guis/confirm.yml`: layout, items, and titles for each menu.
