# Configuration

SnRTP ships with the following YAML files. New keys are auto-merged on boot; your edits and comments are preserved.

## config.yml

```yaml
# ============================================================
#  SnRTP - configuration
#  Managed by SnLib: new keys are auto-merged on boot; your values and
#  comments are preserved. Do NOT add a config-version key (retired).
#  Set update-configs: false to freeze this file (SnLib only warns about
#  missing keys instead of inserting them).
# ============================================================

# Active language code; loads lang/messages_<code>.yml (falls back to en).
lang: en

# Master switch of the SnLib auto-updater for this plugin's managed files.
update-configs: true

# Runtime debug output (also toggleable live via /rtp debug).
debug:
  enabled: false

# ------------------------------------------------------------
#  Main command.
# ------------------------------------------------------------
command:
  # Aliases of /rtp. Re-read on /rtp reload.
  aliases: [wild]

# ------------------------------------------------------------
#  Safe-location search. The search is fully async: each attempt loads its
#  chunk off the main thread (Paper getChunkAtAsync), then scans for a safe
#  spot on the main thread with the chunk already loaded.
# ------------------------------------------------------------
search:
  # Max candidate points tried per random teleport before giving up.
  max-attempts: 30
  # Blocks a player is never allowed to land ON (unsafe footing) or inside.
  # Case-insensitive Bukkit material names; unknown names are ignored with a warning.
  unsafe-blocks:
    - LAVA
    - WATER
    - FIRE
    - CACTUS
    - MAGMA_BLOCK
    - CAMPFIRE
    - SOUL_CAMPFIRE
    - SWEET_BERRY_BUSH
    - POWDER_SNOW
    - WITHER_ROSE

# ------------------------------------------------------------
#  Warmup / cooldown defaults. Each world may override these below.
# ------------------------------------------------------------
defaults:
  # Seconds of warmup before the teleport fires (cancels on move/damage). 0 = instant.
  warmup-seconds: 3
  # Seconds of cooldown between random teleports of the SAME world. 0 = no cooldown.
  # A failed search (no safe spot found) never consumes the cooldown.
  cooldown-seconds: 60

# ------------------------------------------------------------
#  Effects. Toggleable, admin-global (no per-player state). enabled: false
#  turns off every particle and sound.
#
#  Only some particles carry a color. Set warmup-particle / arrival-particle to
#  DUST (a plain colored dot, best all-rounder), DUST_COLOR_TRANSITION (fades
#  from the color into the fade-color) or ENTITY_EFFECT (color only, no size) to
#  use the color keys below. Every other particle keeps its own vanilla look and
#  ignores color / fade-color / size.
#  Unknown particles, and particles needing data this plugin cannot build
#  (BLOCK, ITEM, ...), fall back to PORTAL / END_ROD with a console warning.
# ------------------------------------------------------------
effects:
  # Master switch for all animations and sounds.
  enabled: true
  # Warmup animation drawn around the player: portal | spiral | beam.
  style: portal
  # Particle used by the warmup animation (any Bukkit Particle name).
  warmup-particle: PORTAL
  # Warmup particle color, "#RRGGBB" or "R,G,B". Color-capable particles only.
  warmup-color: "#8354f2"
  # Color DUST_COLOR_TRANSITION fades into. "" = no fade (stays warmup-color).
  warmup-fade-color: ""
  # Warmup particle scale, 0.1 - 4.0. DUST and DUST_COLOR_TRANSITION only.
  warmup-size: 1.0
  # Sound when the search succeeds and the warmup starts ("SOUND vol pitch"). "" = silent.
  warmup-sound: "BLOCK_PORTAL_TRIGGER 0.6 1.4"
  # Particle burst played at the destination on arrival.
  arrival-particle: END_ROD
  # Arrival particle color, "#RRGGBB" or "R,G,B". Color-capable particles only.
  arrival-color: "#8354f2"
  # Color DUST_COLOR_TRANSITION fades into. "" = no fade (stays arrival-color).
  arrival-fade-color: ""
  # Arrival particle scale, 0.1 - 4.0. DUST and DUST_COLOR_TRANSITION only.
  arrival-size: 1.0
  # Sound played at the destination on arrival. "" = silent.
  arrival-sound: "ENTITY_ENDERMAN_TELEPORT 1.0 1.0"

# ------------------------------------------------------------
#  Worlds. One block-icon per world is drawn in the /rtp menu (guis/menu.yml);
#  the MECHANICS of each world's RTP live here, keyed by the SAME id used in that
#  menu button's "[custom] <key>" action.
#
#  enabled      : false disables this world's RTP (menu button and /rtp <key>).
#  world        : the Bukkit world name (as in the server's level folders).
#  center       : SPAWN uses the world spawn as the ring centre; ORIGIN uses 0,0;
#                 FIXED uses center-x / center-z below.
#  min/max-radius: a SQUARE ring around the centre, in blocks.
#  scan-mode    : AUTO (default) picks SURFACE for open-sky worlds and ROOFED for the
#                 Nether. SURFACE stands on the highest block; ROOFED scans DOWN for an
#                 air pocket under a ceiling (use it for custom roofed/void worlds).
#  scan-min-y / scan-max-y: bound the safe-Y search. SURFACE uses the highest block and
#                 rejects spots outside this range; ROOFED scans DOWN from scan-max-y.
#  permission   : "" = anyone with snrtp.use; otherwise the player must also hold it.
#
#  To add a world: add a block here AND a button in guis/menu.yml with the same key.
# ------------------------------------------------------------
worlds:
  overworld:
    enabled: true
    world: world
    center: SPAWN
    center-x: 0
    center-z: 0
    min-radius: 200
    max-radius: 5000
    scan-mode: AUTO
    scan-min-y: 62
    scan-max-y: 320
    warmup-seconds: 3
    cooldown-seconds: 60
    permission: ""
  nether:
    enabled: true
    world: world_nether
    center: SPAWN
    center-x: 0
    center-z: 0
    min-radius: 100
    max-radius: 3000
    scan-mode: AUTO
    scan-min-y: 32
    scan-max-y: 120
    warmup-seconds: 3
    cooldown-seconds: 60
    permission: ""
  end:
    enabled: true
    world: world_the_end
    center: SPAWN
    center-x: 0
    center-z: 0
    min-radius: 100
    max-radius: 3000
    scan-mode: AUTO
    scan-min-y: 40
    scan-max-y: 120
    warmup-seconds: 3
    cooldown-seconds: 60
    permission: ""
```

### Colored particles

Minecraft only lets some particles be tinted. The rest have a fixed vanilla look, so a color set on them does nothing.

| Particle | Reads | Result |
|----------|-------|--------|
| `DUST` | color, size | A solid dot in your color. The one to pick for a custom-colored animation. |
| `DUST_COLOR_TRANSITION` | color, fade-color, size | Fades from color into fade-color. Leave `fade-color: ""` to stay on one color. |
| `ENTITY_EFFECT` | color | Tinted potion swirl. Ignores size. |
| anything else (`PORTAL`, `END_ROD`, `FLAME`, ...) | nothing | Keeps its own vanilla colors. |

Colors accept `"#RRGGBB"` or `"R,G,B"`. A malformed value logs a warning and keeps the default `#8354f2`.

A purple warmup spiral and a matching arrival burst:

```yaml
effects:
  enabled: true
  style: spiral
  warmup-particle: DUST
  warmup-color: "#8354f2"
  warmup-size: 1.0
  arrival-particle: DUST_COLOR_TRANSITION
  arrival-color: "#8354f2"
  arrival-fade-color: "#ffffff"
  arrival-size: 1.4
```

Particle names are matched leniently: case does not matter, a `minecraft:` prefix and `.`/`-` separators are accepted, and `DUST` / `REDSTONE` resolve to each other so one config works on both 1.20.x and 1.21.x. An unknown particle, or one needing data SnRTP cannot build (`BLOCK`, `ITEM`, ...), falls back to `PORTAL` / `END_ROD` and warns once in console.

## guis/menu.yml

The single random-teleport menu. Each world button routes through a `[custom] <key>` action whose key matches a `worlds.<key>` entry in `config.yml`. Edit icons, slots and the layout mask here; add a button with a matching key to add a world.

## lang/messages_en.yml

All player-facing messages. Restyle any value freely; keys auto-merge on boot and your edits are kept.
