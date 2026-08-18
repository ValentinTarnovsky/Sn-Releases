# Configuration

SnEnvoys ships with the following YAML files. New keys are auto-merged on boot; your edits and
comments are preserved.

## config.yml

```yaml
# ============================================================
#  SnEnvoys - configuration
#  Managed by SnLib: new keys are auto-merged on boot; your values and
#  comments are preserved. SnLib owns this file's structure; it carries no
#  version key of its own.
#  Set update-configs: false to freeze this file (SnLib only warns about
#  missing keys instead of inserting them).
#  Sections marked "# sn:extensible" are yours: entries you delete there
#  stay deleted.
# ============================================================

# Active language code; loads lang/messages_<code>.yml (falls back to en).
lang: en

# Master switch of the SnLib auto-updater for this plugin's managed files.
update-configs: true

# Runtime debug output (also toggleable live via /envoy debug).
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
  # Aliases of /envoy. Re-read on /envoy reload.
  aliases: [envoys]

# ------------------------------------------------------------
#  Public developer API.
# ------------------------------------------------------------
# Public developer API events. When false, no API event is dispatched (zero
# cost) and cancellable hooks report "not cancelled" so gameplay proceeds.
# The query facade stays available either way.
api-events:
  enabled: true

# ------------------------------------------------------------
#  Claiming rules shared by envoys and Supply Drops.
# ------------------------------------------------------------
claims:
  # When true, players in Creative mode cannot claim an envoy or a Supply Drop
  # (the reward commands run regardless of mode, so this closes a free-loot
  # exploit on servers where Creative access is common).
  deny-creative: true

# ------------------------------------------------------------
#  Envoy: the block dropped during a timed event.
# ------------------------------------------------------------
envoy:
  # Block that represents an envoy. Two accepted formats:
  #   - A Bukkit Material name, e.g. CHEST, ENDER_CHEST, BEACON
  #   - A custom head: "basehead-<base64>" (paste the Base64 value from
  #     minecraft-heads.com "Other / Value" field after the basehead- prefix)
  material: CHEST
  # Minutes between envoy events. The countdown restarts when an event ends.
  interval-minutes: 120
  # How many envoys spawn per event. Locations are picked randomly without
  # repeats from the pool registered with /envoy editor. If this number is
  # higher than the number of registered locations, it is capped to the pool size.
  envoys-per-event: 5
  # Minutes the event lasts. When time runs out, unclaimed envoys despawn.
  duration-minutes: 10
  # Highlight the unclaimed envoys with a glowing, blinking outline during the
  # final seconds of an event so players can rush to grab whatever is left.
  # Blocks cannot glow natively, so each remaining envoy is shadowed by a
  # client-side glowing block display at the same spot. Custom heads glow with
  # a generic head outline (a block display mirrors block state, not the skin).
  glow:
    # Master switch for the end-of-event glow highlight.
    enabled: true
    # Start glowing when this many seconds remain in the event.
    start-seconds: 30
    # Glow outline color. A named color (RED, ORANGE, YELLOW, LIME, GREEN,
    # AQUA, BLUE, PURPLE, PINK, WHITE, BLACK) or a hex value like "#FF00AA".
    color: YELLOW

# ------------------------------------------------------------
#  Location editor (/envoy editor).
# ------------------------------------------------------------
editor:
  # Diamond blocks handed to an admin entering /envoy editor.
  give-amount: 64

# ------------------------------------------------------------
#  Announcements around an envoy event.
# ------------------------------------------------------------
announcements:
  pre-event:
    # How many warning broadcasts before the event starts.
    count: 3
    # Seconds between each warning. With count 3 and interval 60 the warnings
    # fire at 180s, 120s and 60s before the event.
    interval-seconds: 60
  during-event:
    # Broadcast periodic reminders while the event is active.
    enabled: true
    # Seconds between reminder broadcasts.
    interval-seconds: 60

# ------------------------------------------------------------
#  Boss bar shown to everyone while an envoy event is active. Its title
#  reports how many envoys are still unclaimed and the bar empties as they
#  are claimed. Removed automatically when the last envoy is claimed or the
#  event ends. Edit the title text in lang/messages_en.yml
#  (messages.envoy.bossbar-title).
# ------------------------------------------------------------
bossbar:
  # Master switch for the boss bar.
  enabled: true
  # Bar color: PINK, BLUE, RED, GREEN, YELLOW, PURPLE or WHITE.
  color: YELLOW
  # Bar style: SOLID, SEGMENTED_6, SEGMENTED_10, SEGMENTED_12 or SEGMENTED_20.
  style: SEGMENTED_10

# ------------------------------------------------------------
#  Sounds.
# ------------------------------------------------------------
sounds:
  # Namespaced sound keys (https://minecraft.wiki -> sounds.json). Empty string disables.
  # Played to the player that claims an envoy.
  claim: "entity.player.levelup"
  # Played to everyone when the event starts.
  start: "entity.ender_dragon.growl"
  # Played to everyone when the event ends.
  end: ""
  # Volume and pitch applied to all plugin sounds.
  volume: 1.0
  pitch: 1.0

# ------------------------------------------------------------
#  Rewards. One reward is picked per claimed envoy, weighted by chance.
#  This section is owner-extensible: add, rename or delete rewards freely;
#  entries you delete are never re-added on boot.
# ------------------------------------------------------------
# sn:extensible
rewards:
  diamonds:
    # Relative weight; weights do not need to add up to 100 across the pool.
    chance: 70
    # Console commands; %player% is replaced with the claimer's name.
    commands:
      - "give %player% diamond 3"
    # Server-wide message when this reward is claimed; empty sends nothing.
    # Supports {prefix}, {player}, HEX colors and [center].
    broadcast: ""
  netherite:
    # Relative weight; weights do not need to add up to 100 across the pool.
    chance: 25
    # Console commands; %player% is replaced with the claimer's name.
    commands:
      - "give %player% netherite_ingot 1"
    # Server-wide message when this reward is claimed; empty sends nothing.
    # Supports {prefix}, {player}, HEX colors and [center].
    broadcast: "{prefix}&e{player} &7found &bNetherite &7in an envoy!"
  jackpot:
    # Relative weight; weights do not need to add up to 100 across the pool.
    chance: 5
    # Console commands; %player% is replaced with the claimer's name.
    commands:
      - "give %player% diamond_block 8"
      - "xp add %player% 100 levels"
    # Server-wide message when this reward is claimed; empty sends nothing.
    # Supports {prefix}, {player}, HEX colors and [center].
    broadcast: "{prefix}&6&l{player} &ewon the &6&lJACKPOT &efrom an envoy!"

# ------------------------------------------------------------
#  Supply Drop: an optional timed block that falls from the sky at a
#  random location and can be claimed by the first player to click it.
#  Off by default.
# ------------------------------------------------------------
supply-drop:
  # Master switch. The feature stays fully idle until this is true.
  enabled: false
  # World the automatic cycle drops into. Ignored by "/envoy drop here",
  # which always drops in the admin's own current world.
  world: world
  # Center of the drop ring.
  center:
    x: 0
    z: 0
  # Distance range (blocks) from the center where a drop can land.
  radius:
    min: 50
    max: 300
  # Minutes between automatic drops.
  interval-minutes: 15
  announcements:
    # How many countdown warnings are broadcast before a drop.
    count: 4
    # Seconds between each warning.
    interval-seconds: 180
  # Seconds an unclaimed drop stays on the ground before it despawns.
  despawn-seconds: 120
  # Height (blocks) above the target the falling visual starts from.
  fall-height: 30
  # Ticks the fixed-timer landing fallback waits, used only when the
  # falling visual entity fails to spawn.
  fall-ticks: 50
  # Block used for the falling visual. Minecraft cannot draw a falling
  # block that normally renders via a block-entity (chests, shulker
  # boxes, beds, banners, signs, heads/skulls, conduit, barrier,
  # structure void, light, moving piston), so those are auto-substituted
  # (logged) even if set here directly. "auto" follows "material" below
  # whenever it can be drawn while falling.
  fall-visual-material: BARREL
  beam:
    # Vertical particle beam marking a landed drop so it can be spotted.
    enabled: true
    particle: END_ROD
    height: 30
  # Block the landed drop is made of.
  material: CHEST
  sound:
    # Played to every online player when a drop lands. Empty disables it.
    land: "entity.generic.explode"
  protection:
    # Blocks the surrounding area from being placed/broken while a drop
    # is announced, falling or landed.
    enabled: true
    radius: 32
  region-avoidance:
    # Requires WorldGuard. Rejects landing spots that touch a region.
    enabled: true
    # Extra clearance (blocks) kept between a drop and any region.
    clearance: 32
    # Candidate coordinates tried before a cycle gives up on avoidance.
    max-attempts: 24
  # Rewards drawn when a Supply Drop is claimed, weighted by chance. This
  # section is owner-extensible: add, rename or delete rewards freely;
  # entries you delete are never re-added on boot.
  # sn:extensible
  rewards:
    common:
      # Relative weight; weights do not need to add up to 100 across the pool.
      chance: 70
      # Console commands; %player% is replaced with the claimer's name.
      commands:
        - "give %player% diamond 5"
      # Server-wide message when this reward is claimed; empty sends nothing.
      # Supports {prefix}, {player}, HEX colors and [center].
      broadcast: ""
    rare:
      # Relative weight; weights do not need to add up to 100 across the pool.
      chance: 30
      # Console commands; %player% is replaced with the claimer's name.
      commands:
        - "give %player% netherite_ingot 2"
      # Server-wide message when this reward is claimed; empty sends nothing.
      # Supports {prefix}, {player}, HEX colors and [center].
      broadcast: "{prefix}&e{player} &7opened a &bSupply Drop&7!"
```

## locations.yml

Not a configuration file: it holds the envoy spawn points registered with `/envoy editor`.
Never merged, never seeded and never touched by an update - editor changes always stick.
