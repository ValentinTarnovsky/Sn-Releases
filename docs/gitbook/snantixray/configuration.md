# Configuration

SnAntiXray ships two YAML files. New keys are merged in on boot, and your values and comments are
preserved. Both are re-read by `/antixray reload`, so no restart is needed after an edit.

{% hint style="warning" %}
`worlds.yml` is the file that decides whether cheat detection works at all. Decoy veins are the only
signal the detection layer scores, so a server with no enabled world and no usable rule has
detection switched off no matter what `fake-ores.enabled` says. Two defaults switch things off
silently: a world without `enabled: true`, and a rule without a `frequency` above zero. The startup
log tells you when it happens.
{% endhint %}

## config.yml

```yaml
# ============================================================
#  SnAntiXray - configuration
#  Managed by SnLib: new keys are auto-merged on boot; your values and
#  comments are preserved. Do NOT add a config-version key (retired).
#  Set update-configs: false to freeze this file (SnLib only warns about
#  missing keys instead of inserting them).
#  Entries removed from a list here stay removed; the updater only adds
#  missing keys, never missing list items.
# ============================================================

# Active language code; loads lang/messages_<code>.yml (falls back to en).
lang: en

# Master switch of the SnLib auto-updater for this plugin's managed files.
update-configs: true

# Runtime debug output (also toggleable live via /antixray debug).
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
  # Aliases of /antixray. Re-read on /antixray reload.
  aliases: [axr, snax]

# ------------------------------------------------------------
#  Database. type=sqlite needs nothing else; type=mysql reads host/port/etc.
# ------------------------------------------------------------
database:
  # sqlite or mysql
  type: sqlite
  # MySQL connection (ignored when type is sqlite).
  host: localhost
  port: 3306
  database: snantixray
  username: root
  password: ""

# ------------------------------------------------------------
#  Anti X-Ray. Ore blocks are sent as a filler block until they become
#  legitimately exposed. The real world is never modified.
# ------------------------------------------------------------
anti-xray:
  # Master toggle of the ore-hiding layer.
  enabled: true
  # Blocks sent as the filler block while they stay unexposed.
  ore-blocks:
    - COAL_ORE
    - DEEPSLATE_COAL_ORE
    - IRON_ORE
    - DEEPSLATE_IRON_ORE
    - COPPER_ORE
    - DEEPSLATE_COPPER_ORE
    - GOLD_ORE
    - DEEPSLATE_GOLD_ORE
    - REDSTONE_ORE
    - DEEPSLATE_REDSTONE_ORE
    - LAPIS_ORE
    - DEEPSLATE_LAPIS_ORE
    - DIAMOND_ORE
    - DEEPSLATE_DIAMOND_ORE
    - EMERALD_ORE
    - DEEPSLATE_EMERALD_ORE
    - NETHER_GOLD_ORE
    - NETHER_QUARTZ_ORE
    - ANCIENT_DEBRIS
  # Block sent in place of a hidden block, picked by dimension. Anti ESP uses it too.
  filler:
    # Overworld filler above deepslate-y.
    overworld: STONE
    # Overworld filler at or below deepslate-y.
    overworld-deepslate: DEEPSLATE
    # Y level at or below which the overworld switches to the deepslate filler.
    deepslate-y: 0
    # Nether filler.
    nether: NETHERRACK
    # End filler.
    end: END_STONE

# ------------------------------------------------------------
#  Anti ESP. Hides non-ore blocks with the same per-dimension filler,
#  so block-entity radars cannot map bases, containers or spawners.
# ------------------------------------------------------------
anti-esp:
  # Master toggle of the block-entity hiding layer.
  enabled: true
  # Blocks hidden until they become legitimately exposed. Any block id is accepted.
  hidden-blocks:
    - CHEST
    - TRAPPED_CHEST
    - BARREL
    - ENDER_CHEST
    - SHULKER_BOX
    - WHITE_SHULKER_BOX
    - ORANGE_SHULKER_BOX
    - MAGENTA_SHULKER_BOX
    - LIGHT_BLUE_SHULKER_BOX
    - YELLOW_SHULKER_BOX
    - LIME_SHULKER_BOX
    - PINK_SHULKER_BOX
    - GRAY_SHULKER_BOX
    - LIGHT_GRAY_SHULKER_BOX
    - CYAN_SHULKER_BOX
    - PURPLE_SHULKER_BOX
    - BLUE_SHULKER_BOX
    - BROWN_SHULKER_BOX
    - GREEN_SHULKER_BOX
    - RED_SHULKER_BOX
    - BLACK_SHULKER_BOX
    - FURNACE
    - BLAST_FURNACE
    - SMOKER
    - HOPPER
    - DISPENSER
    - DROPPER
    - SPAWNER
    - TRIAL_SPAWNER
    # VAULT is the trial-chamber vault and exists only on Paper 1.20.6 and newer. On an older
    # server the plugin logs one "Unknown block 'VAULT'" line at boot and carries on; that line
    # is expected there, not a fault. Delete this entry to silence it.
    - VAULT
    - BREWING_STAND
    - BEACON
    - LECTERN
    - END_PORTAL_FRAME
    - WHITE_BED
    - ORANGE_BED
    - MAGENTA_BED
    - LIGHT_BLUE_BED
    - YELLOW_BED
    - LIME_BED
    - PINK_BED
    - GRAY_BED
    - LIGHT_GRAY_BED
    - CYAN_BED
    - PURPLE_BED
    - BLUE_BED
    - BROWN_BED
    - GREEN_BED
    - RED_BED
    - BLACK_BED
  # Withhold the block-entity payload of a hidden block so container contents cannot leak.
  withhold-block-entities: true

# ------------------------------------------------------------
#  Fake ores. Decoy veins injected into the client view only; they exist
#  in no real world, so touching one is a cheat signal. The per-world vein
#  rules live in worlds.yml.
#  Setting enabled to false ALSO disables the whole detection layer below:
#  fake veins are the only signal detection scores.
# ------------------------------------------------------------
fake-ores:
  # Master toggle of the decoy veins and of the detection layer built on them.
  enabled: true

# ------------------------------------------------------------
#  Detection. Scores fake-ore interactions and alerts staff. A periodic
#  sweep evaluates the scores; nothing is alerted at interaction time.
# ------------------------------------------------------------
detection:
  # Score added per fake-ore interaction. Right-clicking a fake ore is never scored.
  weights:
    # Breaking a fake ore: strong signal.
    break: 10.0
    # Starting to damage a fake ore: medium signal.
    damage: 4.0
  # Score levels that raise an alert.
  thresholds:
    # Raises the "possible cheater" alert.
    low: 25.0
    # Raises the "likely cheater" alert.
    high: 60.0
  # Seconds between two detection sweeps.
  sweep-interval: 60
  # Score removed per elapsed HOUR of wall-clock time, applied by the sweep before it
  # compares thresholds. Decay is anchored to a stored timestamp, so time a player spends
  # offline counts too: a score earned today keeps fading while they are away.
  decay-per-hour: 5.0
  # Seconds before the same player can raise another alert at the same level. Without this
  # a player parked above a threshold would re-alert on every single sweep.
  alert-cooldown: 900

# ------------------------------------------------------------
#  Ghost blocks. Budgets for the block updates the plugin sends when a
#  hidden block becomes exposed, so mass block changes cannot spike a tick.
# ------------------------------------------------------------
ghost-blocks:
  # Block updates sent to one player per tick; the excess defers to the next tick.
  per-player-per-tick: 64
  # Cap on one player's pending block-update queue. The REVEAL itself is never lost on
  # overflow - the position is recorded before its update is queued, so the next chunk
  # send carries the real block either way. What overflow costs is the fast path: the
  # fallback resends the whole chunk, but at most once per queue-drain window per player,
  # so an overflow inside that window leaves the client on filler until an ordinary chunk
  # send corrects it. Raise this, or per-player-per-tick, if /antixray stats shows the
  # overflow counter climbing.
  max-queued-per-player: 4096

# ------------------------------------------------------------
#  Threads. Each pool is sized as a fraction of the available CPU cores,
#  resolved to at least one thread.
#  On Folia the region scheduler replaces three of these four: only
#  outgoing-packet-handler still has an effect there.
# ------------------------------------------------------------
threads:
  # Cores warming decoy veins for the chunks around one just sent, so they are
  # ready before the player walks into them. A miss still generates on the spot,
  # it is just slower. Ignored on Folia.
  fake-ore-generator: 0.25
  # Cores flushing queued block updates. Ignored on Folia.
  ghost-block-ticker: 0.25
  # Cores sending the block updates that reveal a block. This does NOT size the
  # chunk rewrite: that runs inline on the network thread that carries the chunk,
  # by design, so it never queues behind anything. Applies on Folia too.
  outgoing-packet-handler: 0.5
  # Flush queued block updates on a pool instead of the server tick. This does NOT
  # move the exposure CHECKS off the main thread - those always run on the thread
  # that delivered the block event. Ignored on Folia.
  async-block-checks: true
```

## worlds.yml

Unlike `config.yml`, this file is seeded once and never merged again. The worlds and rules you write
here are kept exactly as they are across plugin updates, so nothing you add is ever touched by an
upgrade.

```yaml
# ============================================================
#  SnAntiXray - per-world fake ore rules
#  Seeded once by SnLib and never merged afterwards: the worlds and rules
#  you write here are kept exactly as they are across plugin updates.
#
#  TIP - MIRROR VANILLA ORE DISTRIBUTION IN EVERY HEIGHT YOU SET.
#  A decoy vein only works while it passes for real terrain. Diamonds at
#  Y=-50 read as natural; diamonds at Y=80 tell even a legitimate player
#  who stumbles on one that the ore placement is wrong, and that blows the
#  cover of the whole decoy layer for everyone.
#
#  A world that is not listed here, or listed with enabled: false, receives
#  no fake veins. World names are matched case-insensitively.
#  Fake ores also need fake-ores.enabled: true in config.yml.
#
#  ----- VEIN RULE KEYS -----
#  Documented here rather than on one example vein, so the reference
#  survives you deleting the examples. Every key has a default, and two of
#  those defaults switch things OFF - write a rule by hand and you want the
#  "default" column in front of you.
#    material   Ore block the decoy vein is made of. No default: a rule
#               without it is rejected. A block this server does not know is
#               rejected with a warning at load, because the client would
#               silently drop it and the vein would simply not exist.
#    min-y      Lowest Y a vein of this rule can be placed at. Default -64.
#    max-y      Highest Y a vein of this rule can be placed at. Defaults to
#               min-y, which is a one-block band - set it explicitly. Never
#               below min-y; if you invert them the plugin clamps to min-y.
#    size       Blocks per generated vein. Default 4, clamped to 128.
#    frequency  Average veins of this rule per chunk. Default 0, and 0
#               DISABLES the rule - a rule written without this line
#               generates nothing. Clamped to 64.
#  The world-level "enabled" key defaults to FALSE, so a world block written
#  without it receives no veins either.
#  Both defaults matter more than they look: decoy veins are the only signal
#  the detection layer scores, so a world whose rules all resolve to nothing
#  leaves cheat detection inert. The plugin says so on startup when it
#  happens - read the boot log after editing this file.
#  The key naming each rule (diamond, gold, ...) is yours; it only appears
#  in warnings about that rule.
#  Edits here are applied by /antixray reload; no restart is needed.
# ============================================================

worlds:
  # Example world. Rename this key to one of your own world folders.
  world:
    # Whether this world receives fake veins.
    enabled: true
    veins:
      diamond:
        material: DEEPSLATE_DIAMOND_ORE
        min-y: -59
        max-y: -50
        size: 4
        frequency: 0.35
      gold:
        material: DEEPSLATE_GOLD_ORE
        min-y: -48
        max-y: -16
        size: 5
        frequency: 0.5
      iron:
        material: IRON_ORE
        min-y: 8
        max-y: 54
        size: 6
        frequency: 0.8
```

## Tuning notes

### Choosing decoy heights

Mirror vanilla ore distribution. A decoy only works while it passes for real terrain. Diamonds at
Y=-50 read as natural, but diamonds at Y=80 tell even an honest player who stumbles on one that the
placement is wrong. That teaches cheaters which veins are bait and blows the cover of the whole
layer for everyone.

### Ghost block budgets

`ghost-blocks.per-player-per-tick` caps how many block updates one player receives per tick, so a
large explosion spreads its reveals over several ticks instead of spiking one. If `/antixray stats`
shows the overflow counter climbing, raise that value or raise `ghost-blocks.max-queued-per-player`.

### Threads

Each pool is sized as a fraction of your available CPU cores and always resolves to at least one
thread. On Folia the region scheduler replaces three of the four, so only
`threads.outgoing-packet-handler` has an effect there. The startup log names the ignored keys.

{% hint style="info" %}
Despite its name, `threads.async-block-checks` does not move exposure checks off the main thread.
Those always run on the thread that delivered the block event, because they read the live world. The
key only chooses whether the queued ghost-block flush runs on its own pool or on the server tick.
{% endhint %}
