# Configuration

SnTempBlocks ships two YAML files, and they are managed differently on purpose.

`config.yml` is managed by SnLib: new keys are auto-merged on boot and your edits and comments
are preserved. `zones.yml` is yours. It is seeded once on first boot and never touched again, so
an update never adds, renames or removes anything you wrote there.

Every setting is re-read by `/tempblocks reload`. Reloading never clears what is already tracked:
tracked blocks are runtime state, not something a file can rebuild.

## config.yml

```yaml
# ============================================================
#  SnTempBlocks - configuration
#  Managed by SnLib: new keys are auto-merged on boot; your values and
#  comments are preserved. Do NOT add a config-version key (retired).
#  Set update-configs: false to freeze this file (SnLib only warns about
#  missing keys instead of inserting them).
#  Sections marked "# sn:extensible" are yours: entries you delete there
#  stay deleted.
#
#  The zones themselves are NOT here: they live in zones.yml, which is seeded
#  once and then fully yours.
# ============================================================

# Active language code; loads lang/messages_<code>.yml (falls back to en).
lang: en

# Master switch of the SnLib auto-updater for this plugin's managed files.
update-configs: true

# Runtime debug output (also toggleable live via /tempblocks debug).
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
  # Aliases of /tempblocks. Re-read on /tempblocks reload.
  aliases: [tb, stb]

# ------------------------------------------------------------
#  Database. type=sqlite needs nothing else; type=mysql reads host/port/etc.
#  This is where the tracking index is mirrored, so a restart never leaves
#  orphan blocks behind. SQLite is the right choice unless you run a network.
# ------------------------------------------------------------
database:
  # sqlite or mysql
  type: sqlite
  # MySQL connection (ignored when type is sqlite).
  host: localhost
  port: 3306
  database: sntempblocks
  username: root
  password: ""

# ============================================================
#  TRACKING - what the plugin considers a temporary block.
# ============================================================
tracking:
  # Track blocks placed by players inside a zone (BlockPlaceEvent).
  blocks: true

  # Track water and lava sources emptied from a bucket inside a zone
  # (PlayerBucketEmptyEvent). Bucket placement does NOT fire BlockPlaceEvent,
  # which is why it is a separate switch.
  liquids: true

  # Track every block a tracked liquid flows into (BlockFromToEvent) and remove
  # the whole body together with its source. Turn this OFF only if you would
  # rather cancel liquid spread inside zones entirely (see limits.max-flow-blocks
  # for the cap that applies while it is on).
  liquid-flow: true

  # Cancel liquid spread that would leave the zone. Keep this true: a flow block
  # outside the zone is never tracked, so it would stay in the world forever.
  contain-flow-in-zone: true

  # What happens to blocks whose zone was deleted from zones.yml on a reload.
  #   EXPIRE_NOW - remove them on the next sweep (recommended, leaves no orphans)
  #   KEEP       - stop tracking them and leave them permanently in the world
  on-zone-removed: EXPIRE_NOW

# ============================================================
#  LIMITS - the caps that keep a wipe from freezing the server.
# ============================================================
limits:
  # Ticks between expiry sweeps. Lower is more precise, and costs more CPU.
  check-interval-ticks: 10

  # Blocks removed per tick. A wipe larger than this is spread across several
  # ticks instead of stalling the server in one.
  max-removals-per-tick: 500

  # Blocks a single liquid source may flow into before further spread from that
  # source is cancelled. Stops one bucket in a pit from creating thousands of
  # tracked records.
  max-flow-blocks: 400

  # Seconds between async flushes of the tracking index to the database. A full
  # blocking flush also runs on shutdown, so this only bounds crash loss.
  save-interval-seconds: 30

# ============================================================
#  FEEDBACK - what a player sees when a block expires.
# ============================================================
effects:
  # Particle spawned where a block expires; empty disables it. Minecraft renamed
  # several particles in 1.20.5 (BLOCK_CRACK -> BLOCK, SMOKE_NORMAL -> SMOKE,
  # EXPLOSION_NORMAL -> POOF); either spelling of those three works here. An
  # unknown name warns once on boot and simply shows no particle.
  particle: BLOCK_CRACK

  # Sound played where a block expires; empty disables it. Written as the sound
  # key ("block.stone.break", "entity.item.break"). The old enum spelling
  # (BLOCK_STONE_BREAK) is accepted too, but a few sounds do not translate
  # cleanly from it, so prefer the key.
  sound: block.stone.break

  # Volume and pitch of that sound.
  volume: 0.4
  pitch: 1.2

  # Play the effects during a bulk INTERVAL wipe too. Off by default on purpose:
  # a wipe of thousands of blocks would be a particle and sound storm.
  on-bulk-wipe: false
```

## zones.yml

This is where you define your zones. A zone is a WorldGuard region, and blocks are only ever
tracked and removed inside one.

```yaml
# ============================================================
#  SnTempBlocks - zones
#
#  THIS FILE IS YOURS. It is seeded once on first boot and never touched again:
#  SnTempBlocks will not add, rename or remove anything here on an update, and
#  every field is read with a default, so a field you delete simply falls back.
#  Delete the two examples below once you have written your own zones.
#
#  A zone is a WorldGuard region. Blocks are only ever tracked and removed
#  inside a zone - never anywhere else on the server.
#
#    world:  the world the region belongs to
#    region: the WorldGuard region id, or __global__ for the WHOLE world
#            (__global__ is WorldGuard's own name for the world-wide region)
#    mode:   PER_BLOCK or INTERVAL
#
#  PER_BLOCK - every block expires on its own timer, counted from the moment it
#              was placed. Use it when players should be able to build and have
#              their work fade behind them.
#
#  INTERVAL  - one timer for the whole zone. When it fires, EVERYTHING tracked
#              in that zone disappears at once, no matter how old it is. A block
#              placed one second before the wipe goes with the rest.
#
#  Durations accept 10s, 5m, 2h, 1d, or a plain number of seconds.
# ============================================================

zones:

  # ----------------------------------------------------------
  #  Example 1 - PER_BLOCK.
  #  A build zone where each material has its own lifetime.
  # ----------------------------------------------------------
  build-arena:
    world: world
    region: build_arena
    mode: PER_BLOCK

    per-block:
      # Lifetime of any placed block not listed under "blocks" below.
      # Set it to 0 to track ONLY the materials you list explicitly, and leave
      # everything else in the world permanently.
      default: 30s

      # Per-material lifetimes. The material name is the Bukkit name.
      blocks:
        COBBLESTONE: 30s
        WHITE_WOOL: 10s
        COBWEB: 5s

      # Lifetime of a bucket-placed water or lava source. The whole body of
      # liquid that flowed from it is removed at the same instant.
      # Set to 0 to leave placed liquids alone in this zone.
      liquid: 15s

  # ----------------------------------------------------------
  #  Example 2 - INTERVAL.
  #  A PVP arena that resets itself whole, every 5 minutes.
  #  This example covers the ENTIRE world via the __global__ region.
  # ----------------------------------------------------------
  pvp-arena:
    world: world_pvp
    region: __global__
    mode: INTERVAL

    interval:
      # How often everything tracked in this zone is wiped.
      every: 5m

      # Warn players standing in the zone this long before the wipe.
      # An empty list disables the warnings.
      warn-at: [60s, 30s, 10s]

      # Track bucket-placed liquids here too. They are wiped with everything
      # else when the timer fires, so they need no lifetime of their own.
      liquids: true
```

### Choosing an engine

| Mode | Use it when |
|------|-------------|
| `PER_BLOCK` | Players should build and have their work fade behind them. Each block expires on its own timer, counted from when it was placed. |
| `INTERVAL` | The whole area should reset as one, like a PVP arena. Everything tracked disappears together, however old it is. |

"Everything disappears in X time" is `PER_BLOCK` with only `default` set and no material list.
There is no third mode for it.

{% hint style="info" %}
An `INTERVAL` cycle announces every time, including when the zone was already empty. The
warnings in `warn-at` are sent to whoever is standing in the zone, and the wipe itself sends
`messages.wipe-done` with the count, or `messages.wipe-done-empty` when there was nothing to
remove. Before 1.1.1 an empty zone stayed silent, which made the cycle seem to come and go
depending on whether somebody had placed a block.
{% endhint %}

{% hint style="info" %}
Set `region: __global__` to cover an entire world. That is WorldGuard's own name for the
world-wide region, so you do not need to create a region first.
{% endhint %}

{% hint style="warning" %}
A zone with a mistake in it is skipped, not fatal. The console names the zone and the reason, the
other zones keep working, and the plugin stays up. Check your console after editing this file.
{% endhint %}

## lang/messages_en.yml

Every message, including the short state words the placeholders substitute, lives here. Change
`lang` in `config.yml` to load a different file from the `lang/` folder.

```yaml
# ============================================================
#  SnTempBlocks - language file (English)
#  Managed by SnLib: new keys merge in on boot, your values are kept.
#  Restyle any line freely; your edits survive updates.
# ============================================================

# Prefix prepended to every single-line message sent via sn.lang().send(...)
prefix: "&#8354f2&lSnTempBlocks &8| &7"

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
    header: "&8&m----------&r &#8354f2&l{plugin} &8&m----------"
    # Branded no-separator form. SnLib's neutral default is "&e{usage} &7{description}".
    entry: "&#8354f2{usage} &7{description}"
    footer: "&7Page &f{page}&7/&f{total} &8- &#8354f2/{command} help <page>"

# Short state words. Each is substituted into a {status}, {mode} or {value} slot
# of the messages below and of the PlaceholderAPI output, so a restyle here
# applies everywhere at once.
status:
  # Engine of a zone, shown by /tempblocks list and %sntempblocks_mode%.
  mode-per-block: "&#8354f2Per block"
  mode-interval: "&#8354f2Interval"
  # Shown where the player is standing in no zone at all.
  none: "None"
  # Shown for a zone whose WorldGuard region no longer resolves.
  unknown: "Unknown"
  # Shown by the next-wipe placeholders on a PER_BLOCK zone, which has no single
  # countdown to report. Never a fake number.
  not-applicable: "&7n/a"

# Your own plugin messages. Do NOT write {prefix} in any value: SnLib
# auto-prepends the prefix above to every single-line message sent via
# sn.lang().send(...). A literal {prefix} renders verbatim, and SnLib
# logs a one-time WARN when the active lang file contains one.
messages:
  # ---- warnings broadcast inside a zone before an INTERVAL wipe ----
  wipe-warning: "&eEverything you placed here vanishes in &f{time}&e."
  wipe-done: "&7The zone was wiped: &f{count} &7blocks removed."
  # Sent instead of wipe-done when the cycle found nothing to remove. An empty
  # cycle still announces: silence there reads as a broken plugin to the players
  # who just watched the countdown reach zero.
  wipe-done-empty: "&7The zone was wiped."

  # ---- command feedback ----
  zone-not-found: "&cNo zone named &e{zone}&c. Use &e/{label} list&c."
  here-in-zone: "&7You are in zone &f{zone} &8({mode}) &7- next wipe: &f{time}"
  here-no-zone: "&7You are not standing in any SnTempBlocks zone."
  wipe-forced: "&aWiped &f{count} &ablocks from &f{zone}&a."
  wipe-forced-all: "&aWiped &f{count} &ablocks from &f{zones} &azones."
  purge-done: "&aStopped tracking &f{count} &ablocks in &f{zone}&a. They are now permanent."

  # ---- startup refusal, printed to the console ----
  # SnTempBlocks disables itself when this is sent: zones ARE WorldGuard regions,
  # so without WorldGuard answering region queries there is nothing to resolve.
  # A zone whose own region is missing is a different, much smaller problem: it
  # is disabled on its own and reported as a console WARNING naming the region,
  # which says more than a chat line could and reaches the person who edits the file.
  worldguard-missing: "&cWorldGuard is unavailable. SnTempBlocks cannot resolve any zone and will not enable."

# Multi-line views printed by the admin commands. Rendered via getList, so no
# prefix is added and every line is styled on its own.
lists:
  list-header:
    - "&8&m----------&r &#8354f2&lTemp Block Zones &8&m----------"
    - "&7Zones: &f{count}&7, tracked blocks: &f{tracked}"
  list-entry: "&#8354f2{zone} &8| &7{world}&8/&7{region} &8| {mode} &8| &7{tracked} blocks &8| &7{time}"
  info:
    - "&8&m----------&r &#8354f2&l{zone} &8&m----------"
    - "&7World: &f{world}"
    - "&7Region: &f{region}"
    - "&7Mode: {mode}"
    - "&7Tracked blocks: &f{tracked}"
    - "&7Pending in unloaded chunks: &f{pending}"
    - "&7Next wipe: &f{time}"
```
