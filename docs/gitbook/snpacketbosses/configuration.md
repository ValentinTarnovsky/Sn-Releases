# Configuration

SnPacketBosses ships with the following YAML files. New keys are auto-merged on boot;
your edits and comments are preserved.

The one exception is the `bosses/` folder. It is seeded once, on a fresh install, and
never written to again: a boss file you delete stays deleted, and a file you add is
picked up on the next reload. One file is one boss, and the file name is the boss id.

## config.yml

```yaml
# ============================================================
#  SnPacketBosses - configuration
#  Managed by SnLib: new keys are auto-merged on boot; your values and
#  comments are preserved. Do NOT add a config-version key (retired).
#  Set update-configs: false to freeze this file (SnLib only warns about
#  missing keys instead of inserting them).
#  Sections marked "# sn:extensible" are yours: entries you delete there
#  stay deleted.
# ============================================================

# Active language code; loads lang/messages_<code>.yml (falls back to en).
lang: en

# Master switch of the SnLib auto-updater for this plugin's managed files.
update-configs: true

# Runtime debug output (also toggleable live via /packetbosses debug).
debug:
  # Master toggle of the debug output.
  enabled: false
  # Verbosity threshold: OFF, INFO, DEBUG or TRACE.
  level: DEBUG
  # Category filter; an empty list lets every category through.
  categories: []

# ------------------------------------------------------------
#  Main command.
# ------------------------------------------------------------
command:
  # Aliases of /packetbosses. Re-read on /packetbosses reload.
  aliases: [pbosses]

# ------------------------------------------------------------
#  Database. type=sqlite needs nothing else; type=mysql reads host/port/etc.
# ------------------------------------------------------------
database:
  # sqlite or mysql
  type: sqlite
  # MySQL connection (ignored when type is sqlite).
  host: localhost
  port: 3306
  database: snpacketbosses
  username: root
  password: ""

# ============================================================
#  BAND 1 - MODEL: how a packet boss exists and ticks.
# ============================================================

# Runtime session settings.
sessions:
  # Interval in ticks for boss timers and ability checks. 20 ticks = 1 second.
  # Clamped to a minimum of 20 AND snapped down to a whole multiple of 20: the timer
  # counts WHOLE seconds, derived as tick-interval / 20, so anything in between would
  # stretch every configured time limit and ability cooldown. A value that had to be
  # snapped is reported in the console every time this file is read, i.e. on boot and on
  # every /packetbosses reload.
  tick-interval: 20
  # Interval in seconds between asynchronous saves of every active session.
  # Clamped to a minimum of 5.
  autosave-interval-seconds: 10

# Packet settings for the fake boss entities.
packets:
  # First fake entity ID used by bosses, holograms and fireballs. Must stay above the
  # server's real entity id range. Read once at startup: changing it needs a restart,
  # not a reload.
  fake-entity-id-start: 300000
  # Delay before the spawn packet burst is sent a second time, to beat client timing
  # misses. 0 disables the resend entirely.
  spawn-resend-delay-ticks: 2
  # Optional periodic full resync, for STATIONARY bosses only. 0 disables it.
  # Moving bosses are never periodically respawned: it reads as a visual snap-back.
  full-resync-interval-ticks: 0

# Packet movement loop.
movement:
  # Period of the movement loop in ticks. Clamped to a minimum of 2.
  tick-interval-ticks: 2

# ============================================================
#  BAND 3 - LIMITS: what a player cannot do during a fight.
# ============================================================

# Optional fight lockdown while a player has an active boss.
# Holders of snpacketbosses.admin.lock-bypass are exempt from all of it.
locks:
  # Master switch of the whole lockdown.
  enabled: true
  # Block commands that are not in allowed-commands below.
  block-commands: true
  # Block every teleport while the fight is active.
  block-teleports: true
  # Master gate of the per-world flight block below.
  block-flight: true
  # Worlds where flight is blocked during a fight (only used when block-flight is true).
  # Use ["*"] to block flight in every world.
  # List specific world names to block flight ONLY in those worlds.
  # Leave the list empty to never block flight.
  block-flight-worlds:
    - "*"
  # Command roots a locked player may still run. Matched against the command root with
  # the leading slash, anything after the first space and any "namespace:" prefix
  # stripped, so "/minecraft:tp" is matched as "tp".
  # The root "packetbosses" is ALWAYS allowed, whatever this list says: it is the admin
  # recovery path out of a stuck fight, and trimming it out of here must not be able to
  # close it. The aliases set in command.aliases above are folded in the same way. (An alias
  # that comes from plugin.yml because command.aliases was deleted entirely is NOT, so keep
  # that key set.) They are listed here anyway so the allowlist reads as the complete
  # picture.
  allowed-commands:
    - "packetbosses"
    - "pbosses"
```

## lang/messages_en.yml

```yaml
# ============================================================
#  SnPacketBosses - language file (English)
#  Managed by SnLib: new keys merge in on boot, your values are kept.
#  Restyle any line freely; your edits survive updates.
# ============================================================

# Prefix prepended to every single-line message sent via sn.lang().send(...)
prefix: "&#33CCFF&lSnPacketBosses &8| &7"

# snlib.* is SnLib's shared command contract (11 keys). Every Sn plugin ships
# this block UNCHANGED: SnLib bundles NEUTRAL defaults for any key you omit and
# merges them in on boot, so an incomplete block leaks unbranded lines. Shipping
# the FULL block is what keeps the whole fleet visually identical. Placeholders
# {plugin} {usage} {description} {page} {total} {min} {max} {value} {command}
# are filled by SnLib. NEVER put {prefix} here (it is auto-prepended).
snlib:
  no-permission: "&cYou do not have permission to use this command."
  usage: "&cUsage: &e{usage}"
  invalid-number: "&cInvalid number: &e{value}"
  invalid-value: "&cInvalid value: &e{value}"
  out-of-range: "&cValue must be between &e{min} &cand &e{max}&c: &e{value}"
  number-too-small: "&cValue must be at least &e{min}&c: &e{value}"
  player-not-found: "&cPlayer not found: &e{value}"
  unknown-subcommand: "&cUnknown subcommand: &e{value}"
  reload-done: "&aConfiguration reloaded."
  help:
    header: "&8&m----------&r &#33CCFF&l{plugin} &8&m----------"
    entry: "&#33CCFF{usage} &7{description}"
    footer: "&7Page &f{page}&7/&f{total} &8- &#33CCFF/{command} help <page>"

# Your own plugin messages. Do NOT write {prefix} in any value: SnLib
# auto-prepends the prefix above to every single-line message sent via
# sn.lang().send(...).
messages:

  # --- command feedback -------------------------------------------------
  boss-not-found: "&cUnknown boss: &f{boss}&c."
  invalid-egg: "&cThis boss egg is invalid."
  boss-disabled: "&cThis boss is disabled."
  already-active: "&cYou already have an active boss."
  wrong-world: "&cThis egg can only be used in &f{world}&c."
  gave-egg: "&aGave &f{amount}x {boss} &aegg(s) to &f{player}&a."
  killed-boss: "&aKilled &f{player}&a's active boss."
  killed-stored-boss: "&aDeleted &f{player}&a's stored boss session."
  killed-all: "&aKilled &f{amount}&a active boss(es) and wiped every stored session."
  no-active-boss-target: "&cThat player does not have an active boss."
  list-header: "&bConfigured bosses:"
  list-entry: "&7- &f{id} &8(&7{status}&8)"
  list-empty: "&7No bosses are configured."

  # --- spectator --------------------------------------------------------
  view-started: "&aNow spectating &f{player}&a's boss."
  view-stopped: "&aStopped spectating."
  view-self: "&cYou cannot spectate your own boss."
  view-none: "&cYou are not spectating anyone."
  view-player-only: "&cOnly a player can spectate a boss fight."

  # --- fight lifecycle --------------------------------------------------
  boss-spawned: "&aYou summoned &f{boss}&a. Farm with EdTools to defeat it."
  boss-restored: "&aYour paused boss session has been restored."
  # Shown when an egg is used in the moment after joining, before the stored fight has
  # been read back. It lasts about a second.
  restore-pending: "&cYour boss data is still loading. Try again in a moment."
  # Shown when a paused fight cannot resume because its boss is pinned to another
  # world: on the join that found it, and again if an egg is used meanwhile.
  restore-held: "&cYou have a paused boss fight waiting in &f{world}&c. Go there to resume it; you cannot summon a new boss until then. If that world is gone, ask an admin to run /packetbosses kill on you."
  boss-defeated: "&aYou defeated &f{boss}&a."
  boss-timeout: "&c{boss} escaped. You have no attempts left."
  # Creative only: the egg was never taken, so nothing is spent and nothing returns.
  boss-timeout-unspent: "&c{boss} escaped. Your egg was not spent."
  boss-failed-attempt: "&c{boss} escaped. The egg returns with &f{attempts}/{max_attempts}&c attempts."
  boss-admin-killed: "&cYour active boss was killed by an admin."
  boss-removed: "&cYour active boss was removed because it no longer exists."

  # --- abilities --------------------------------------------------------
  ability-warning: "&e&l⚠ &e{boss} is preparing &6{ability}&e in &65s&e!"
  ability-hit: "&c{boss} hit you and healed &f{heal} HP&c."
  fireball-deflected: "&aYou deflected the fireball! &f{boss}&a took &c{damage} HP&a damage."
  armored-activated: "&b{boss} became armored. Hit it &f{hits_required} &btimes to break the armor."
  armored-hit: "&bArmor hits: &f{hits}&7/&f{hits_required}&b."
  armored-broken: "&aYou broke {boss}'s armor."

  # --- fight lock -------------------------------------------------------
  command-locked: "&cYou cannot use commands while your boss is active."
  teleport-locked: "&cYou cannot teleport while your boss is active."
  flight-locked: "&cYou cannot fly while your boss is active."

# Short state words. Each is substituted into a {status} or {armor_state} slot of the
# messages above, of the boss bar title and of the hologram text, so a restyle here
# applies everywhere at once.
status:
  # Whether a configured boss can be spawned. Shown in the /packetbosses list rows.
  enabled: "&aEnabled"
  disabled: "&cDisabled"
  # The {armor_state} slot of bossbar.title and hologram.text while the boss IS armored.
  # Supports {armor_hits}, {armor_required_hits} and {armor_time}.
  armored-active: "&bArmor {armor_hits}/{armor_required_hits} &8({armor_time}s)"
  # The same slot while the boss is NOT armored. Empty by default.
  armored-inactive: ""

# Player-visible names of the boss abilities, substituted into the {ability} slot of
# messages.ability-warning. Potion entries are keyed by the effect type in lowercase.
# Every label has a code-side fallback (the title-cased effect id), so an entry you
# delete simply falls back instead of breaking.
# sn:extensible
abilities:
  fireball: "Fireball"
  armored: "Armored"
  # Fallback when an effect has no id at all.
  unknown: "Effect"
  nausea: "Nausea"
  blindness: "Blindness"
  slowness: "Slowness"
```

## bosses/guardian.yml

```yaml
# ============================================================
#  SnPacketBosses - boss definition
#  This folder is YOURS. It is seeded once, on a fresh install, and the plugin
#  never writes into it again: a file you delete stays deleted.
#  Add a boss by adding a file. The boss ID is the file name without .yml,
#  lowercased, so this file defines the boss "guardian".
#  Files whose name starts with "old-" are ignored.
# ============================================================

# Whether this boss can be spawned.
enabled: true

# Display name used in boss bars, messages, the egg and placeholders.
display-name: "&#33CCFF&lAbyss Guardian"

# PacketEvents entity type. Common examples: GUARDIAN, ELDER_GUARDIAN, BLAZE, ZOMBIE.
# Accepts the plain or the "minecraft:"-prefixed spelling; an unknown name falls back
# to GUARDIAN with a console warning.
entity-type: "GUARDIAN"

# Visual entity scale, sent as an attribute packet. Minimum 0.0625.
scale: 3.0

# Maximum HP for this boss. Minimum 1.
max-health: 5000.0

# Time limit in seconds. The timer pauses while the owner is offline. Minimum 1.
time-limit-seconds: 600

# HP removed per valid EdTools break event from the owner, before ability modifiers.
damage-per-break: 5.0

# World where the egg can be used. Accepts a single name or a list:
# allowed-egg-world: ["world", "pico"]
allowed-egg-world: "world"

# Boss bar shown to the owner only.
bossbar:
  # Adventure BossBar.Color: PINK, BLUE, RED, GREEN, YELLOW, PURPLE, WHITE.
  color: "BLUE"
  # Adventure BossBar.Overlay: PROGRESS, NOTCHED_6, NOTCHED_10, NOTCHED_12, NOTCHED_20.
  # This is the Adventure vocabulary, so a 20-segment bar is NOTCHED_20; other plugins
  # may call the same bar SEGMENTED_20, which is not accepted here.
  overlay: "NOTCHED_20"
  # Bar text. Placeholders: {boss} {boss_id} {hp} {max_hp} {time} {world}
  # {armor_state} {armor_hits} {armor_required_hits} {armor_time}.
  title: "{boss} &7HP: &f{hp}&7/&f{max_hp} &8| &7Time: &f{time}s &8| {armor_state}"

# Private packet hologram rendered above the boss, for the owner only.
hologram:
  # Whether the hologram is rendered at all.
  enabled: true
  # Height of the hologram above the boss.
  y-offset: 3.8
  # Hologram text. Same placeholders as the bar title.
  text: "{boss} &8| &f{hp}&7/&f{max_hp} HP &8| &f{time}s"

# Packet-only movement. When enabled, no fixed location is needed and the boss
# follows its owner across worlds.
movement:
  enabled: true
  # Normal orbit distance from the owner.
  orbit-radius: 2.8
  # Orbit distance while armored. What actually brings the boss into reach during an
  # armored phase is armored-vertical-offset below, which drops it from 4.4 to head height;
  # this radius is wider than the normal one, not narrower. Lower it if your players
  # struggle to land the hits that break the armor. (The code default is 1.7.)
  armored-orbit-radius: 3.7
  # Normal height above the owner.
  vertical-offset: 4.4
  # Height while armored, i.e. the boss drops to melee height.
  armored-vertical-offset: 0.15
  # Orbit speed around the owner. Clamped to 0 - 3600 (ten full turns a second); anything
  # higher is already a blur and an unbounded value is arithmetic the movement tick cannot
  # recover from.
  orbit-speed-degrees-per-second: 28.0
  # Fraction of the remaining delta applied per movement tick. Clamped to 0.05 - 1.0.
  smoothing: 0.22
  # The REAL follow speed cap. Keep it above a Speed-buffed player's sprint speed or
  # the boss trails behind and starts teleport-snapping.
  movement-speed-blocks-per-second: 24.0
  # Per-packet step floor. The effective cap is the HIGHER of this and the speed above,
  # so this can only raise the cap, never lower it.
  max-step-per-move: 0.8
  # If the visual position drifts at least this far, the boss is repositioned with an
  # absolute position sync instead of stepped. No respawn, so no visual reset.
  teleport-distance: 64.0
  # If the VERTICAL drift alone reaches this, only Y is snapped. 0 disables it.
  vertical-snap-distance: 2.0

# Fixed spawn point, read ONLY when movement.enabled is false. Uncomment it to pin the
# boss to one spot; the owner has to be in that world for the boss to be rendered at all.
# world defaults to the first allowed-egg-world above.
#location:
#  world: "world"
#  x: 0.5
#  y: 80.0
#  z: 0.5
#  yaw: 0.0
#  pitch: 0.0

# Egg item given by /packetbosses give.
egg:
  # Item material. An unknown or air material falls back to GUARDIAN_SPAWN_EGG.
  material: "GUARDIAN_SPAWN_EGG"
  # Item name. Same placeholders as the lore below.
  display-name: "&#33CCFF&lAbyss Guardian Egg"
  # Item lore. Placeholders: {boss} {id} {world} {attempts} {max_attempts}.
  lore:
    - "&7Use in &f{world}&7 to summon:"
    - "&#33CCFF{boss}"
    - "&7Defeat it by farming with EdTools."
    - ""
    - "&eAttempts: &f{attempts}&7/&f{max_attempts}"
  # Enchantment glint on the egg.
  glow: true
  # Custom model data of the item. 0 leaves it unset.
  custom-model-data: 0
  # Tries the player gets. On time-out the egg returns with one attempt fewer; once it
  # reaches 0 the egg is gone.
  max-attempts: 3

# Reward commands executed from the console when the owner kills the boss.
# Not run on time-out and not run on an admin kill.
rewards:
  # One command per line, without the leading slash. Placeholders: {player} {boss}.
  commands:
    - "say {player} defeated {boss}"

# Configurable boss abilities. Each one telegraphs 5 seconds before it fires.
abilities:
  potion-effects:
    # Whether this boss applies potion effects at all.
    enabled: true
    # One entry per effect, each keeping its OWN cooldown, tracked by its position in this
    # list. Fields: type (potion effect id), amplifier (0 is level I), duration-seconds,
    # cooldown-seconds, chance (0.0 - 1.0, rolled while off cooldown) and range in blocks,
    # which the owner must be within. A type that is unknown OR malformed - a space in it,
    # an empty value - skips that entry with a warning and leaves the rest of the boss
    # loading. The "minecraft:" prefix is accepted here, as it is for entity-type.
    effects:
      - type: "NAUSEA"
        amplifier: 0
        duration-seconds: 4
        cooldown-seconds: 30
        chance: 0.25
        range: 24.0
      - type: "BLINDNESS"
        amplifier: 0
        duration-seconds: 3
        cooldown-seconds: 45
        chance: 0.15
        range: 20.0
      - type: "SLOWNESS"
        amplifier: 1
        duration-seconds: 5
        cooldown-seconds: 35
        chance: 0.20
        range: 24.0
  fireball:
    # Whether this boss throws fireballs.
    enabled: true
    # Seconds before the boss can throw again.
    cooldown-seconds: 20
    # Chance rolled while the ability is off cooldown. 0.0 - 1.0.
    chance: 0.30
    # The owner must be within this range in blocks at roll, launch and impact time.
    range: 28.0
    # Magnitude of the velocity on the fireball spawn packet. Visual speed only.
    speed: 1.8
    # Ticks between launch and impact, so also the window the owner has to deflect.
    travel-ticks: 20
    # HP the owner loses on an undeflected hit.
    hit-damage: 4.0
    # HP the boss heals on an undeflected hit. A DEFLECTED fireball instead damages the
    # boss for 25% of this value, so setting it to 0 also removes the deflection reward.
    heal-on-hit: 100.0
  armored:
    # Whether this boss can enter the armored phase.
    enabled: true
    # Seconds before the boss can become armored again, counted from when the phase ENDS.
    cooldown-seconds: 60
    # Seconds the armored phase lasts unless broken by physical hits.
    duration-seconds: 12
    # Chance checked when the ability is off cooldown.
    chance: 0.35
    # The owner must be within this range for the boss to activate armor.
    range: 24.0
    # EdTools damage multiplier while armored. 0.25 means 75% damage reduction.
    damage-multiplier: 0.25
    # Physical attacks on the boss needed to break the armor early.
    required-hits: 5
```
