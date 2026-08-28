# Configuration

SnGens ships eleven configuration files plus one language file and eight menu layouts. Every
one of them is plain YAML, and every player facing string in them supports colour codes.

| File | What it holds | Updated on new versions |
|------|---------------|-------------------------|
| `config.yml` | Global behaviour: language, database, tick rate, limits, corruption, leaderboard, upgrade menu | Merged |
| `generators.yml` | Every generator: item, drops, sell values, upgrade chain | Seeded once |
| `wands.yml` | Sellwand, build wand, upgrade wands, admin region wand | Merged |
| `events.yml` | Timed server events and their rotation | Seeded once |
| `storages.yml` | Collector and infinite hopper | Merged |
| `armors.yml` | Armor sets and their full set bonuses | Seeded once |
| `offhands.yml` | Off-hand items and their bonuses | Seeded once |
| `gui/*.yml` | The eight menu layouts | Merged |
| `lang/messages_<code>.yml` | Every message the plugin sends | Merged |

**Merged** means the plugin inserts keys that a new version added into your existing file on
every boot, and on every `/gens reload`, while leaving your values, lists and comments alone.
Set `update-configs: false` at the top of `config.yml` to switch that off, in which case the
plugin only logs how many sections are missing.

**Seeded once** means the file is written on a fresh install and never touched again. Those
four files hold content you own: your generators, your events, your gear. A generator you
delete stays deleted, a generator you add is picked up on the next reload, and a later version
never pushes new example entries into them.

## Colour codes

Every string accepts legacy codes such as `&a`, `&l` and `&7`, and hex colours in the form
`&#RRGGBB`:

```yaml
display-name: "&#D8A073&lWheat Generator"
lore:
  - "&8Tier 1 Generator"
  - "&7&oPlace down to activate."
```

## Applying changes

`/gens reload` re-reads every file and re-applies the settings, including the collector and
hopper values. Two things still need a restart: the database connection block, since the pool
is built at startup, and anything already handed out as an item, since a generator or wand in a
player's inventory carries its own data.

---

## config.yml

The main file. If you only ever touch one file, this is it.

### The keys people change first

| Key | Default | What it does |
|-----|---------|--------------|
| `lang` | `en` | Which `lang/messages_<code>.yml` is loaded |
| `generator-tick-interval-seconds` | `20` | Seconds between drops, for every generator |
| `player-defaults.max-generators` | `10` | Slots a brand new player gets |
| `player-defaults.starting-generator` | `wheat_generator` x3 | What a player receives on first join |
| `corruption.enabled` | `true` | Whether generators break over time |
| `corruption.interval` | `300` | Minutes between corruption waves |
| `gens-top.mode` | `ISLAND` | Rank islands or individual players |
| `online-only` | `true` | Generators only tick while their owner is online |
| `player-generator-limit` | off | A hard ceiling on placed generators per player |
| `player-multiplier-limit` | off | A hard ceiling on the sell multiplier |

```yaml
# -----------------------------------------------------------------------------
# Auto-merge
# -----------------------------------------------------------------------------
# When true, on startup (and on /sngens reload) any new keys/sections that
# the plugin ships in this version are merged into your existing YML files
# without touching your existing values, sections, lists or comments.
# When false, your YMLs are left alone; the plugin logs a warning if any
# bundled file is missing keys but keeps running on safe defaults.
update-configs: true


# -----------------------------------------------------------------------------
# Debug
# -----------------------------------------------------------------------------
# When true, the plugin emits verbose [DEBUG] log lines for chunk loads, the
# generator tick loop, listener entries, equipment events, reload phases, and
# database connect/migration steps. Leave false in production - only flip on
# while diagnosing a specific issue.
debug: false

# -----------------------------------------------------------------------------
# Developer API
# -----------------------------------------------------------------------------
# Master switch for the public SnGens API event bus. When true, the plugin
# fires custom Bukkit events (place / upgrade / break / repair / sell / sellwand
# / collector-sell / corrupt / server-event / top-update / refund) that other
# plugins can listen to. Turn off to skip all API event dispatch with zero
# overhead. The query API (SnGensProvider#get) stays available either way.
api-events:
  enabled: true

# -----------------------------------------------------------------------------
# External equipment (compatibility with cosmetic / world-forced armor plugins)
# -----------------------------------------------------------------------------
# Some plugins force their own items into a player's armor or off-hand slots when
# the player enters a given world, and hold whatever the player was wearing until
# they leave. The player cannot take those items off. Without this section the
# SnGens armor set would simply read as "not worn" and its bonus would drop.
#
# When a slot is recognised as taken by such a plugin, SnGens treats it as still
# holding the piece the player really had on: the last armor set and off-hand the
# player genuinely equipped is remembered on the player itself, so the bonus
# survives the swap, a relog and a full server restart. Taking your own gear off
# normally still breaks the set, so this cannot be abused.
external-equipment:
  enabled: true

  # Persistent-data keys that identify an item belonging to the other plugin.
  # Use "namespace" to match every key that plugin writes, or "namespace:key" for
  # one exact key. A plugin's namespace is its plugin.yml name in lowercase.
  # Run with debug: true and walk into the affected world - SnGens then logs the
  # real keys of the item that took the slot, so you can confirm the value below.
  markers:
    - okimc-edtoolsarmors
    - okimc-edtoolsoffhands

  # Optional world filter. Empty = every world.
  # Leave 'markers' empty and fill this instead if the other plugin writes no
  # persistent data on its items: inside these worlds ANY foreign item is then
  # accepted as having taken the slot.
  # With both lists empty the feature stays off (it would be exploitable).
  worlds: []

  # Grace window, in ticks, before SnGens reads the armor slots after a join or a
  # world change (20 ticks = 1 second). The other plugin usually needs a moment to
  # put its gear on; reading too early would look like "the player took the set
  # off" and forget the remembered set. Raise it if the bonus is still lost.
  settle-delay-ticks: 40

# =============================================================================
#  SnGens - main configuration
# =============================================================================
#  Edit and run /sngens reload to apply.
#  Messages live in lang/messages_<code>.yml - see `lang` below.
# =============================================================================


# -----------------------------------------------------------------------------
# Localisation
# -----------------------------------------------------------------------------
# Active language file: lang/messages_<code>.yml. Missing keys fall back to
# messages_en.yml. Add a translated copy and switch the code to enable it.
lang: en


# -----------------------------------------------------------------------------
# Storage
# -----------------------------------------------------------------------------
# When mysql.enabled = false, SnGens uses a local SQLite file (SnGens.db)
# under the plugin data folder. No setup required.
mysql:
  enabled: false
  host: localhost
  port: 3306
  database: db_sngens
  user: root
  password: ''
  useSSL: false

# Periodic flush of dirty in-memory state (generators + users) to the database.
database:
  autosave-interval-minutes: 10

# -----------------------------------------------------------------------------
# Per-player defaults (applied on first join)
# -----------------------------------------------------------------------------
player-defaults:
  max-generators: 10
  multiplier: 1.0
  starting-generator:
    id: wheat_generator
    amount: 3


# -----------------------------------------------------------------------------
# Drop tick (global)
# -----------------------------------------------------------------------------
# How often, in seconds, every generator drops. Per-player accounting is
# preserved on top of this value. Speed events still apply on top.
generator-tick-interval-seconds: 20


# -----------------------------------------------------------------------------
# Performance tuning
# -----------------------------------------------------------------------------
performance:
  # When bulk-removing generators (admin /gens removegenerators, /gens pickup,
  # SSB2 island disband/kick/leave/ban), this many chunks are processed per tick.
  bulk-removal-chunks-per-tick: 4


# -----------------------------------------------------------------------------
# Item stacking
# -----------------------------------------------------------------------------
# Merges generator drops within the same chunk + Y band into a single stacked
# item entity. Cuts entity count and GC pressure when many gens tick at once.
item-stacking:
  enabled: true
  # Y band that counts as the same vertical layer.
  height-threshold: 5
  # Safety upper bound on Item entities spawned per region per tick. The actual
  # per-tick spawn rate is computed adaptively so that a full cycle's pending
  # drops finish spawning over `generator-tick-interval-seconds * 20` ticks,
  # which keeps Netty's outbound queue draining continuously and removes the
  # periodic ping spike. This cap only kicks in for unusually large bursts; if
  # the computed budget exceeds it, drains take longer than one cycle.
  max-entities-per-tick: 5000
  # Visual scatter: when many gens drop into the same Y band, distribute the
  # spawned stacks across the actual gen locations contributing to the bucket
  scatter: true
  # Drop the vanilla 64-stack split when spawning Item entities. Item entities
  # in Minecraft do not enforce maxStackSize; setting this true makes the
  # accumulator emit a single Item entity carrying the full amount, which
  # ticks once instead of N times. Hopper pickup still respects vanilla
  # stack limits when items enter inventories.
  unbounded-stack-size: false


# -----------------------------------------------------------------------------
# Generator behaviour
# -----------------------------------------------------------------------------
# Generators only tick while their owner is online.
online-only: true
# Block explosion damage on placed generators.
anti-explosion: true
# Block fire/burn damage on flammable generator blocks.
anti-burn: true
# Stop players from placing items dropped by generators.
disable-drop-place: true
# Stop crafting recipes that include any generator block or drop item.
disable-crafting: true
# Set the block back to its expected material if it goes out of sync.
force-update-blocks: true
# Drop the generator item when it is broken (otherwise it disappears).
drop-on-break: false
# Allow /gens pickup to recover corrupted generators.
broken-pickup: true
# True  = pickup requires breaking the block (vanilla animation).
# False = single tap on the configured interaction key picks up.
animation-break: false
# Restrict repairs of corrupted generators to the owner.
repair-owner-only: true
# Hand back generators, Collectors and Infinite Hoppers when their owner is
# removed from a SuperiorSkyblock2 island (kicked / left / banned / disbanded).
# Everything is stored in the player's vault, claimed with /gens recover.
# When false, the blocks are destroyed instead. What happens to the ITEMS held
# inside a Collector or Hopper is decided by island-removal.drop-contents in
# storages.yml.
island-pickup: true
# Restrict generator placement to islands the player is a member of.
# Requires SuperiorSkyblock2; if SSB2 is not loaded this is a no-op
# regardless of value. When true, placing in wilderness or on someone
# else's island is blocked with the `island-place-only` message.
island-place-only: false


# -----------------------------------------------------------------------------
# Generator shop
# -----------------------------------------------------------------------------
shop:
  # When true, clicking a generator in the shop opens a buy-quantity submenu
  # (amounts configured in gui/shop_buy_amount.yml) so players can buy several
  # at once. When false, clicking buys exactly one generator immediately.
  buy-amount-menu: true


# -----------------------------------------------------------------------------
# Interaction bindings
# -----------------------------------------------------------------------------
# Click types: LEFT, SHIFT_LEFT, RIGHT, SHIFT_RIGHT. Empty = feature disabled.
interaction:
  gens-pickup: LEFT
  gens-upgrade: SHIFT_RIGHT
  gens-fix: SHIFT_RIGHT
  hand-sell: SHIFT_RIGHT


# -----------------------------------------------------------------------------
# Corruption system
# -----------------------------------------------------------------------------
# Periodically marks a fraction of placed generators as broken. Owners must
# shift + right-click them to repair.
corruption:
  enabled: true
  # Wave interval, in minutes.
  interval: 300
  # Only corrupt generators whose owner is currently online.
  online-only: true
  # Percentage of placed generators considered each wave.
  percentage: 10
  # Reminder cadence sent privately to affected owners.
  notify:
    interval: 5
  # DecentHolograms hologram shown above broken generators.
  hologram:
    height: 2
    lines:
      - '&4&lBROKEN GENERATOR'
      - '&7(Shift + Right Click)'
  # Generator ids that can never be corrupted.
  blacklisted-generators:
    - disabled_generator
    - unbreakable_generator


# -----------------------------------------------------------------------------
# World & player limits
# -----------------------------------------------------------------------------
# Worlds where placement is denied and existing generators stop ticking.
blacklisted-worlds:
  - disablegen
  - notagenworld
# Hard cap on placed generators per player. enabled=false leaves it uncapped.
player-generator-limit:
  enabled: false
  limit: 50
# Hard cap on per-player sell multiplier.
player-multiplier-limit:
  enabled: false
  limit: 5.0


# -----------------------------------------------------------------------------
# Gens Top
# -----------------------------------------------------------------------------
# /gens top leaderboard. Ranks scopes (players or islands) by the total
# shop-cost value of all their currently-placed generators.
gens-top:
  enabled: true
  # PLAYER ranks individual players. ISLAND folds all SuperiorSkyblock2 island
  mode: ISLAND
  # How often to recompute the top, in seconds.
  refresh-interval-seconds: 300
  # Number of entries displayed (and persisted).
  size: 10
  # ISLAND mode only - when true, players without an island are ranked as
  # individuals instead of being skipped.
  include-islandless: false
  # ISLAND mode only. On a skyblock plugin that separates solo islands from team
  # islands, split /gens top into two boards in the same menu:
  # team islands on the left, solo islands on the right. Ignored on plain
  # SuperiorSkyblock2 or with no skyblock plugin - the single combined board is
  # shown. The two boards are held in memory only and are never persisted, so a
  # server without such a plugin is unaffected.
  split-solo-team: true
  # Generator id used as a top-counting cutoff. Generators that come AFTER this
  # one in the upgrade chain do NOT count toward /gens top (levels the board so
  # players keep competing). Must be a valid generator id from generators.yml.
  # Empty = every tier counts. Editable in-game with /gens toplimit <id>.
  count-limit: ""
  broadcast:
    # Master toggle. When false, no broadcasts are emitted.
    enabled: true
    # Ranks to watch for broadcasting.
    notify-ranks: [1, 2, 3]


# -----------------------------------------------------------------------------
# Upgrade GUI (/sngens upgrade)
# -----------------------------------------------------------------------------
# Layout, view modes, sort modes, and per-island usage limits live in
# gui/upgrade_gens_gui.yml. Limits below are stored in SQLite and persist
# across restarts.
upgrade-gui:
  # Default view mode when the menu opens. PERSONAL = only your generators.
  # ISLAND  = aggregate generators of every member of your SuperiorSkyblock2
  # island. ISLAND falls back to PERSONAL when the player has no island.
  default-view-mode: ISLAND
  # How many generators a Q-key (drop) click upgrades in a single interaction.
  batch-amount: 10
  # When the player triggers an "upgrade-all" or batch action, how many
  # block.setType updates the plugin applies per tick. Higher = the visual
  # block change is faster but the server may stutter on huge bulk runs.
  world-updates-per-tick: 200
  usage:
    # Master toggle. When false, every player can upgrade without limits and
    # the info-uses item is hidden.
    enabled: true
    # Permission that bypasses the island-owner check AND skips usage
    # consumption (intended for staff).
    staff-bypass-permission: "sngens.upgrade.bypass"
    # Daily uses granted to every island. Resets at reset-time. -1 = infinite.
    daily-base: 3
    # HH:mm in server timezone. When this clock-time passes the daily counter
    # is restored to daily-base. Bonus pool from /sngens addupgrades is not
    # touched on reset.
    reset-time: "00:00"
    # Label shown in the upgrade-uses info item (and command feedback) when
    # the daily pool or bonus pool is set to infinite (-1). Translate freely.
    infinite-placeholder: "Unlimited"
```

### Interaction bindings

The `interaction` block decides which click does what on a placed generator. Valid values are
`LEFT`, `SHIFT_LEFT`, `RIGHT`, `SHIFT_RIGHT`, and an empty value disables that action entirely.

```yaml
interaction:
  gens-pickup: LEFT          # pick the generator up
  gens-upgrade: SHIFT_RIGHT  # upgrade it one tier, paying the cost
  gens-fix: SHIFT_RIGHT      # repair it when corrupted
  hand-sell: SHIFT_RIGHT     # sell the drops you are holding
```

`gens-upgrade` and `gens-fix` can share a binding, since a generator is either corrupted or
not. Setting `animation-break: true` makes a pickup require actually breaking the block, with
the vanilla animation, instead of a single click.

```yaml
interaction:
  gens-pickup: ""            # players can no longer pick generators up at all
```

### Item stacking

This is the block that keeps a big farm playable. Instead of spawning one item entity per drop,
the plugin groups the drops of a chunk inside a vertical band and spawns one stacked entity for
the whole group.

```yaml
item-stacking:
  enabled: true
  height-threshold: 5          # blocks that count as the same layer
  max-entities-per-tick: 5000  # safety ceiling per region per tick
  scatter: true                # spread the stacks over the real generator positions
  unbounded-stack-size: false  # allow a single entity to carry more than 64
```

`scatter: false` looks tidier, since every stack appears in one place, but players lose the
visual feedback of drops appearing across their farm. `unbounded-stack-size: true` cuts entity
count further on very large farms, at the cost of stacks that look unusual in the world. Items
entering an inventory always respect vanilla stack limits either way.

### External equipment

Some plugins force their own items into the armor or off-hand slots when a player enters a
world, and hand the real gear back on the way out. Without help, SnGens would read those slots,
see no armor set, and quietly drop the set bonus.

The `external-equipment` block teaches it to recognise those items. When a slot is taken that
way, the plugin keeps applying the last set and off-hand the player genuinely equipped, and it
remembers that on the player, so it survives a relog and a restart.

```yaml
external-equipment:
  enabled: true
  markers:
    - okimc-edtoolsarmors     # a plugin namespace, lowercase, as in its plugin.yml name
    - some-plugin:some-key    # or one exact persistent-data key
  worlds: []                  # empty means every world
  settle-delay-ticks: 40      # grace period before reading the slots after a join
```

{% hint style="info" %}
Not sure which key the other plugin writes? Set `debug: true`, walk into the affected world,
and the console prints the real keys of the item that took the slot.
{% endhint %}

With both `markers` and `worlds` empty the feature turns itself off and logs a warning, because
accepting any foreign item in any world would be trivial to abuse. Taking off your own armor
still breaks the set normally.

### Corruption

A corruption wave marks a share of placed generators as broken. A broken generator stops
producing and grows a hologram over itself until the owner repairs it for the price set on that
generator in `generators.yml`.

```yaml
corruption:
  enabled: true
  interval: 300          # minutes between waves
  online-only: true      # only corrupt generators whose owner is connected
  percentage: 10         # share of placed generators considered each wave
  notify:
    interval: 5          # minutes between reminders to affected owners
  hologram:
    height: 2
    lines:
      - '&4&lBROKEN GENERATOR'
      - '&7(Shift + Right Click)'
  blacklisted-generators:
    - disabled_generator
```

`percentage` is how much of the placed population the wave looks at. Whether a given generator
inside that slice actually breaks is then weighted by its own `corrupted.chance`, so a cheap
tier can be fragile while an expensive one is nearly immune.

To disable corruption entirely, set `corruption.enabled: false`. Existing broken generators can
still be repaired.

### Leaderboard

```yaml
gens-top:
  enabled: true
  mode: ISLAND                    # ISLAND or PLAYER
  refresh-interval-seconds: 300
  size: 10
  include-islandless: false
  count-limit: ""
  broadcast:
    enabled: true
    notify-ranks: [1, 2, 3]
```

`mode: ISLAND` needs SuperiorSkyblock2. It sums every member's generators into one island
entry. Players without an island are skipped unless `include-islandless: true`, which ranks
them as individuals alongside the islands. With no skyblock plugin installed, use
`mode: PLAYER`.

The board is recomputed every `refresh-interval-seconds` and stored, so `/gens top` and the
leaderboard placeholders answer instantly and survive a restart. `broadcast.notify-ranks` picks
which positions announce a change to the whole server.

`count-limit` is the anti-runaway knob. Set it to a generator id, and every tier after that id
in the upgrade chain stops counting toward the board. The players who already maxed out cannot
grow their score further, so the race stays open. Change it in game with `/gens toplimit`.

### Upgrade menu limits

The bulk upgrade menu can be rationed, which stops one player from upgrading an entire island
in a single sitting.

```yaml
upgrade-gui:
  default-view-mode: ISLAND
  batch-amount: 10             # how many the drop key upgrades in one click
  world-updates-per-tick: 200  # block updates per tick during a bulk run
  usage:
    enabled: true
    staff-bypass-permission: "sngens.upgrade.bypass"
    daily-base: 3              # uses per island per day, -1 for unlimited
    reset-time: "00:00"        # server clock time when the daily pool refills
    infinite-placeholder: "Unlimited"
```

Each island has two pools: a daily one that refills at `reset-time`, and a bonus pool granted
with `/gens addupgrades` that the reset never touches. Set `usage.enabled: false` to remove
limits entirely and hide the counter item.

---

## generators.yml

One top-level key per generator, and that key is the id you use everywhere else: in
`player-defaults.starting-generator`, in `/gens give`, in `upgrade.next-generator`, in
`gens-top.count-limit`.

### The shape of a generator

```yaml
my_generator:
  display-name: "&aMy Generator"   # used in messages
  online-only: true                # optional, overrides the global setting

  corrupted:
    enabled: true                  # can this generator break at all
    cost: 100                      # price to repair it
    chance: 0.5                    # weight in the corruption roll

  upgrade:
    next-generator: better_gen     # the id it upgrades into, or null for the last tier
    upgrade-cost: 500              # price of that one step

  upgrade-requirements: []         # optional guards, see below

  item:                            # the item players hold and place
    material: EMERALD_BLOCK
    display-name: "&aMy Generator"
    lore: ["&7Tier 5"]

  drops:                           # what it produces each tick
    '0':
      chance: 70                   # relative weight, not a percentage
      sell-value: 100              # what one unit is worth to /sell
      item:
        material: EMERALD
        display-name: "&aEmerald"
```

### Drops and chances

`chance` values are weights, compared against each other. Two drops with `70` and `30` behave
exactly like two drops with `7` and `3`. You do not have to make them add up to 100.

`sell-value` is stamped onto the dropped item itself. That is the only source of worth in the
plugin: an item without it cannot be sold, and an item with it can be sold no matter where it
ends up, including inside a chest, a collector or a hopper.

A drop can also run console commands instead of, or in addition to, giving an item:

```yaml
  drops:
    '2':
      chance: 1
      commands:
        - "give %player% diamond 1"
        - "broadcast &e%player% &7found something rare"
```

### Upgrade requirements

An optional list of guards checked before an upgrade is allowed. Two types exist.

```yaml
  upgrade-requirements:
    needs_rank:
      type: PERMISSION
      permission: myserver.rank.vip
      message: "&cOnly VIPs may upgrade into this tier."
    needs_level:
      type: PLACEHOLDER
      placeholder: "%player_level%"
      value: "30"
      message: "&cYou need level 30 for this tier."
```

`message` is optional. Without it the plugin sends `requirement-fail-default` from the language
file.

### Where the shop price comes from

There is no `cost` key on a generator. The shop derives every price from the upgrade chain,
starting at `base-cost` in `gui/shop_gui.yml`:

```
price(first generator) = base-cost
price(next generator)  = price(previous) + upgrade-cost of previous
```

With the shipped example that reads:

| Generator | Upgrade cost | Shop price |
|-----------|--------------|------------|
| `wheat_generator` | 50 | 50 |
| `iron_generator` | 500 | 100 |
| `diamond_generator` | 5000 | 600 |
| `netherite_generator` | none | 5600 |

So one number, `upgrade.upgrade-cost`, sets both what an upgrade costs and what the next tier
sells for in the shop. Prices are recomputed on boot and on every `/gens reload`.

### Worked example: adding a fifth tier

Add the new generator, then point the previous last tier at it.

```yaml
# 1. netherite is no longer the end of the chain
netherite_generator:
  upgrade:
    next-generator: star_generator
    upgrade-cost: 50000

# 2. the new tier
star_generator:
  display-name: "&#FFD700&lStar Generator"
  corrupted:
    enabled: true
    cost: 10000
    chance: 2.0
  upgrade:
    next-generator: null
    upgrade-cost: 0
  item:
    material: BEACON
    display-name: "&#FFD700&lStar Generator"
    lore:
      - "&8Tier 5 Generator &8(final)"
      - "&7"
      - "&7&oPlace down to activate."
  drops:
    '0':
      chance: 80
      sell-value: 9000
      item:
        material: NETHER_STAR
        display-name: "&#FFD700Star Shard &7(/sell)"
    '1':
      chance: 20
      sell-value: 25000
      item:
        material: BEACON
        display-name: "&#FFD700Condensed Star &7(/sell)"
```

Run `/gens reload`. The new tier appears in the shop at 5600 + 50000 = 55600, the upgrade menu
offers netherite to star at 50000, and `/gens give <player> star_generator` works immediately.

```yaml
# =============================================================================
#  SnGens - Generators
# =============================================================================
#  Each top-level key is a generator id. Reference the same id from the shop,
#  starting-generator config, /sngens give, and PAPI placeholders.
# -----------------------------------------------------------------------------
#  Schema (per generator):
#
#    <generator_id>:
#      display-name: <string>           # used in messages and the placed block
#      online-only: <bool>              # OPTIONAL, overrides config.yml default
#
#      corrupted:
#        enabled: <bool>                # opt-in to corruption
#        cost: <double>                 # money to repair
#        chance: <double>               # weight in the corruption roll
#
#      upgrade:
#        next-generator: <id|null>      # id to upgrade into, or null
#        upgrade-cost: <double>         # cost to upgrade one tier
#
#      upgrade-requirements:            # OPTIONAL list of guards
#        <key>:
#          type: PERMISSION|PLACEHOLDER
#          message: "<chat>"            # falls back to messages.requirement-fail-default
#          permission: <node>           # PERMISSION only
#          placeholder: "%expansion_x%" # PLACEHOLDER only
#          value: "<expected>"          # PLACEHOLDER only
#
#      item:                            # the held / placed generator item
#        material: <Material|head;<base64>>
#        display-name: <string>
#        item-model: "<namespace:id>"   # OPTIONAL custom model
#        enchantments: ["ENCHANT;LEVEL"]
#        flags: ["HIDE_ATTRIBUTES", ...]
#        custom-model-data: <int>
#        lore: ["..."]
#
#      drops:                           # what the generator emits each tick
#        <key>:
#          chance: <double>             # relative weight
#          sell-value: <double>         # OPTIONAL - needed for /sell
#          commands: ["..."]            # OPTIONAL - run as console on drop
#          item:                        # same item schema as above
#            material: <Material>
#            display-name: <string>
#            lore: ["..."]
# =============================================================================


# -----------------------------------------------------------------------------
# Tier 1 - wheat (starter, handed out on first join)
# -----------------------------------------------------------------------------
wheat_generator:
  display-name: "&#D8A073&lWheat Generator"
  corrupted:
    enabled: true
    cost: 12.5
    chance: 0.25
  upgrade:
    next-generator: iron_generator
    upgrade-cost: 50
  upgrade-requirements: []
  item:
    material: HAY_BLOCK
    display-name: "&#D8A073&lWheat Generator"
    enchantments: []
    flags: []
    custom-model-data: 0
    lore:
      - "&8Tier 1 Generator"
      - "&7"
      - "&8Corruption: &f0.25%"
      - "&8Repair: &c$12.5"
      - "&8Upgrade: &a$50 &7→ Iron"
      - "&7"
      - "&7&oPlace down to activate."
  drops:
    '0':
      chance: 70
      sell-value: 8
      item:
        material: WHEAT
        display-name: "&eWheat &7(/sell)"
        lore:
          - "&7Sell value: &a$8"
    '1':
      chance: 30
      sell-value: 16
      item:
        material: BREAD
        display-name: "&eCondensed Wheat &7(/sell)"
        lore:
          - "&7Sell value: &a$16"


# -----------------------------------------------------------------------------
# Tier 2 - iron
# -----------------------------------------------------------------------------
iron_generator:
  display-name: "&#B0B0B0&lIron Generator"
  corrupted:
    enabled: true
    cost: 60
    chance: 0.5
  upgrade:
    next-generator: diamond_generator
    upgrade-cost: 500
  item:
    material: IRON_BLOCK
    display-name: "&#B0B0B0&lIron Generator"
    enchantments: []
    flags: []
    custom-model-data: 0
    lore:
      - "&8Tier 2 Generator"
      - "&7"
      - "&8Corruption: &f0.5%"
      - "&8Repair: &c$60"
      - "&8Upgrade: &a$500 &7→ Diamond"
      - "&7"
      - "&7&oPlace down to activate."
  drops:
    '0':
      chance: 70
      sell-value: 40
      item:
        material: IRON_INGOT
        display-name: "&fIron Ingot &7(/sell)"
        lore:
          - "&7Sell value: &a$40"
    '1':
      chance: 30
      sell-value: 90
      item:
        material: IRON_BLOCK
        display-name: "&fIron Block &7(/sell)"
        lore:
          - "&7Sell value: &a$90"


# -----------------------------------------------------------------------------
# Tier 3 - diamond
# -----------------------------------------------------------------------------
diamond_generator:
  display-name: "&bDiamond Generator"
  corrupted:
    enabled: true
    cost: 250
    chance: 1.0
  upgrade:
    next-generator: netherite_generator
    upgrade-cost: 5000
  item:
    material: DIAMOND_BLOCK
    display-name: "&bDiamond Generator"
    enchantments: []
    flags: []
    custom-model-data: 0
    lore:
      - "&8Tier 3 Generator"
      - "&7"
      - "&8Corruption: &f1.0%"
      - "&8Repair: &c$250"
      - "&8Upgrade: &a$5,000 &7→ Netherite"
      - "&7"
      - "&7&oPlace down to activate."
  drops:
    '0':
      chance: 70
      sell-value: 220
      item:
        material: DIAMOND
        display-name: "&bDiamond &7(/sell)"
        lore:
          - "&7Sell value: &a$220"
    '1':
      chance: 30
      sell-value: 500
      item:
        material: DIAMOND_BLOCK
        display-name: "&bDiamond Block &7(/sell)"
        lore:
          - "&7Sell value: &a$500"


# -----------------------------------------------------------------------------
# Tier 4 - netherite (final, no upgrade)
# -----------------------------------------------------------------------------
netherite_generator:
  display-name: "&8&lNetherite Generator"
  corrupted:
    enabled: true
    cost: 1500
    chance: 1.5
  upgrade:
    next-generator: null
    upgrade-cost: 0
  item:
    material: NETHERITE_BLOCK
    display-name: "&8&lNetherite Generator"
    enchantments:
      - "DURABILITY;1"
    flags:
      - "HIDE_ENCHANTS"
    custom-model-data: 0
    lore:
      - "&8Tier 4 Generator &8(final)"
      - "&7"
      - "&8Corruption: &f1.5%"
      - "&8Repair: &c$1,500"
      - "&8&oFully upgraded."
      - "&7"
      - "&7&oPlace down to activate."
  drops:
    '0':
      chance: 70
      sell-value: 1200
      item:
        material: NETHERITE_INGOT
        display-name: "&8&lNetherite Ingot &7(/sell)"
        lore:
          - "&7Sell value: &a$1,200"
    '1':
      chance: 30
      sell-value: 2800
      item:
        material: NETHERITE_BLOCK
        display-name: "&8&lNetherite Block &7(/sell)"
        lore:
          - "&7Sell value: &a$2,800"
```

---

## wands.yml

Four tools live here. All of them are handed out by an admin command, and all of them carry
their settings on the item, so a wand already in a player's inventory keeps working exactly as
it was created.

| Wand | Given with | What it does |
|------|-----------|--------------|
| Sellwand | `/gens sellwand <player> <multiplier> <uses>` | Right click a container to sell everything sellable inside it |
| Build wand | `/gens buildwand <player> <distance> <uses>` | Clone a placed generator down a line |
| Upgrade wand | `/gens upgradewand <player> free\|radius <uses> [radius]` | Upgrade for free, or upgrade an area at once |
| Admin region wand | `/gens wand give` | Select a cuboid and fill it with a generator |

`island-use-only: true` at the top of the file restricts all three player wands to islands the
holder belongs to. It needs SuperiorSkyblock2, and `sngens.admin` bypasses it.

### Sellwand

The item's name and lore accept `{uses}`, `{multiplier}`, `{total_sold}` and `{total_items}`,
and the last two are updated on the item as the player uses it, so the wand doubles as a
personal ledger.

`force-same-chunk: true` limits a swing to containers in the chunk the player is standing in.
A wand created with `-1` uses is unlimited and renders `unlimited-placeholder` instead of a
number.

### Build wand

Click a placed generator to preview a line of new generators in the direction you are facing.
Click again on the same generator or on air to confirm. Clicking a different generator restarts
the preview, and clicking any other block cancels it.

The cost is the source generator's shop price multiplied by the distance, and the holder needs
enough free generator slots for all of them, unless they have `sngens.admin`. Only air blocks
are filled. An unconfirmed preview expires after `preview-timeout-seconds`.

### Upgrade wands

`free` upgrades one clicked generator to the next tier at no cost. `radius` upgrades every
generator in an odd sized square around the click, and charges the sum of their upgrade costs,
so a wielder who cannot afford the whole area is not charged and loses no use. Either way one
click costs one use.

### Admin region wand

Left click sets the first corner, right click the second, and the selection is drawn with
particles while you work. `particle.type` accepts `WAX_OFF`, `END_ROD` or `DUST`, and only
`DUST` reads `dust-color` and `dust-size`. Very large selections render only their corners, and
`fill.max-volume` caps a single fill.

```yaml
# =============================================================================
#  SnGens - Wands
# =============================================================================
#  Top-level keys are wand ids. Currently only `sellwand` is defined; future
#  wand types are added as siblings.
# -----------------------------------------------------------------------------
#  Sellwand placeholders (used in display-name and lore):
#    {uses}   {multiplier}   {total_sold}   {total_items}
# =============================================================================

# Restrict every wand (sellwand, buildwand, upgradewand) to islands the
# player is a member of. Requires SuperiorSkyblock2; if SSB2 is not loaded
# this is a no-op regardless of value. When true, using a wand in
# wilderness or on someone else's island is blocked with the
# `wand-island-only` message. Players with `sngens.admin` bypass.
island-use-only: false

sellwand:
  # Restrict the wand to containers in the player's current chunk.
  force-same-chunk: false
  # Text rendered when {uses} is unlimited.
  unlimited-placeholder: "Unlimited"
  item:
    material: BLAZE_ROD
    display-name: '&a&lSell Wand &7· &f{uses}'
    custom-model-data: 0
    flags:
      - HIDE_ATTRIBUTES
      - HIDE_ENCHANTS
    enchantments:
      - DURABILITY;1
    lore:
      - '&7'
      - '&7Right Click &8→ &asell container'
      - '&7'
      - '&a&lWORKS ON'
      - '&8▸ &fChests, Barrels'
      - '&8▸ &fHoppers, Shulkers'
      - '&7'
      - '&a&lDETAILS'
      - '&8▸ &7Multiplier &f{multiplier}x'
      - '&8▸ &7Charges &f{uses}'
      - '&7'
      - '&a&lLEDGER'
      - '&8▸ &7Earned &f${total_sold}'
      - '&8▸ &7Sold &f{total_items}'
      - '&7'

# =============================================================================
#  Admin Region Wand (/gens wand)
# -----------------------------------------------------------------------------
#  WorldEdit-style wand for staff. Left-click sets pos1, right-click sets pos2.
#  /gens wand pick   opens a picker GUI to choose the generator type.
#  /gens wand fill   fills the cuboid with the chosen generator.
#  /gens wand clear  clears the selection + cancels particle visualisation.
# =============================================================================
adminwand:
  item:
    material: GOLDEN_AXE
    display-name: '&c&lGens Admin Wand'
    custom-model-data: 0
    flags:
      - HIDE_ATTRIBUTES
      - HIDE_ENCHANTS
    lore:
      - '&7'
      - '&7Left Click &8→ &cset pos 1'
      - '&7Right Click &8→ &cset pos 2'
      - '&7'
      - '&c&lCOMMANDS'
      - '&8▸ &f/gens wand pick &8- &7choose generator'
      - '&8▸ &f/gens wand fill &8- &7fill region'
      - '&8▸ &f/gens wand clear &8- &7clear selection'
      - '&7'
  particle:
    # WAX_OFF (orange 4-point sparkle, recommended), END_ROD, or DUST.
    # Only DUST honours dust-color and dust-size.
    type: WAX_OFF
    # Used when type=DUST. RGB triplet, 0-255.
    dust-color: "255,140,0"
    dust-size: 1.2
    # Distance between particle samples on each edge, in blocks.
    step: 0.5
    # How often particles redraw, in ticks. 6 ≈ 3.3 redraws/sec.
    period-ticks: 6
    # Selections larger than this volume only render their 8 corners
    # (avoids spamming particles for huge regions).
    max-render-volume: 250000
  fill:
    # Hard cap on the volume of a single /gens wand fill execution.
    max-volume: 50000
    # Block types that the fill replaces. Anything else is left alone.
    replaceable:
      - AIR
      - CAVE_AIR
      - VOID_AIR

# =============================================================================
#  Upgrade Wands (/gens upgradewand <player> <free|radius> <uses> [radius])
# -----------------------------------------------------------------------------
#  free   - right-click a placed generator to upgrade it to the next tier
#           skipping the money cost. 1 use per click.
#  radius - right-click a placed generator to upgrade every placed generator
#           inside an N x N horizontal slab around the click (N stored on the
#           item, must be odd). Total Vault cost is the sum of each upgraded
#           generator's cost; if the wielder cannot afford it, no use is
#           consumed. 1 use per click regardless of generators upgraded.
#
#  Wands of the same type stack: clicking one onto another in the inventory
#  merges their {uses}. Radius wands additionally require equal radius to
#  stack.
#
#  Placeholders: {uses} {radius}
# =============================================================================
upgradewand:
  # Text rendered when {uses} is -1 (unlimited).
  unlimited-placeholder: "Unlimited"
  free:
    item:
      material: IRON_SHOVEL
      display-name: '&d&lFree Upgrade &7· &f{uses}'
      custom-model-data: 0
      flags:
        - HIDE_ATTRIBUTES
        - HIDE_ENCHANTS
      lore:
        - '&7'
        - '&7Right Click &8→ &dfree upgrade'
        - '&7'
        - '&d&lDETAILS'
        - '&8▸ &7Charges &f{uses}'
        - '&8▸ &7Cost &fFree'
        - '&7'
  radius:
    item:
      material: DIAMOND_SHOVEL
      display-name: '&6&lRadius Upgrade &7({radius}x{radius}) &7· &f{uses}'
      custom-model-data: 0
      flags:
        - HIDE_ATTRIBUTES
        - HIDE_ENCHANTS
      lore:
        - '&7'
        - '&7Right Click &8→ &6upgrade {radius}x{radius} area'
        - '&7'
        - '&6&lDETAILS'
        - '&8▸ &7Charges &f{uses}'
        - '&8▸ &7Radius &f{radius}x{radius}'
        - '&8▸ &7Cost &fsum of generators'
        - '&7'

# =============================================================================
#  Build Wand (/gens buildwand <player> <distance> <uses>)
# -----------------------------------------------------------------------------
#  Click 1 on a placed generator → preview particles where the new generators
#  would be placed (line of <distance> blocks in the cardinal direction the
#  player is facing).
#  Click 2 on the same generator OR on air → confirm: deduct Vault cost
#  (source.cost x distance), place clones of the source generator, consume
#  one use.
#  Click 2 on a different generator → abort current preview, start a new one
#  for that generator.
#  Click 2 on any other block → cancel preview without placing.
#
#  Obstruction: only AIR / CAVE_AIR / VOID_AIR may receive a new gen.
#  Slot limit: the wielder's max-generators must accommodate <distance> more
#  gens, unless the wielder has the sngens.admin permission (bypass).
#
#  Stacking: two build wands stack into one if and only if they share the
#  same {distance}; their {uses} are added.
#
#  Placeholders: {uses} {distance}
# =============================================================================
buildwand:
  # Text rendered when {uses} is -1 (unlimited).
  unlimited-placeholder: "Unlimited"
  # Seconds an unconfirmed preview lives before it auto-cancels silently.
  # 0 disables the timeout (preview only ends on confirm/cancel/quit).
  preview-timeout-seconds: 15
  item:
    material: GOLDEN_HOE
    display-name: '&b&lBuild Wand &7(+{distance}) &7· &f{uses}'
    custom-model-data: 0
    flags:
      - HIDE_ATTRIBUTES
      - HIDE_ENCHANTS
    lore:
      - '&7'
      - '&7Click 1 &8→ &bpreview line'
      - '&7Click 2 same / air &8→ &bconfirm'
      - '&7Click 2 other gen &8→ &brestart'
      - '&7Click 2 other block &8→ &bcancel'
      - '&7'
      - '&b&lDETAILS'
      - '&8▸ &7Distance &f{distance} blocks'
      - '&8▸ &7Charges &f{uses}'
      - '&8▸ &7Cost &fbase cost × {distance}'
      - '&7'
```

---

## events.yml

A server event is a global modifier that runs for a while, announces itself, then ends. Between
events the plugin waits `wait-time` seconds and starts the next one.

```yaml
events:
  enabled: true
  random: false      # false cycles in declaration order, true picks by weight
  wait-time: 900     # seconds between events
```

### Event types

| `type` | Extra field | Effect while active |
|--------|-------------|---------------------|
| `mixed_up` | none | Every generator drops items from a random other generator |
| `generator_speed` | `percentage` | The drop interval is shortened by that percentage |
| `drop_multiplier` | `drop-amount` | That many drops per tick instead of one |
| `generator_upgrade` | `tier-amount` | Generators drop as if they were that many tiers higher |
| `sell_multiplier` | `multiplier` | Drop value is multiplied for every sale |

Common fields on every event: `display-name`, `duration` in seconds, `chance` as its weight in
random mode, `only-by-command`, the `start-message` and `end-message` lists, and
`blacklisted_generators`.

`{name}`, `{duration}` and `{next_duration}` are available in the two message lists.

### Worked example: a weekend double sell event

```yaml
  events:
    weekend_boost:
      type: sell_multiplier
      display-name: '&#ffd600&lWeekend Boost'
      multiplier: 3.0
      duration: 600
      chance: 0
      only-by-command: true
      start-message:
        - '&7'
        - '  &#ffd600&lWeekend Boost'
        - '  &7Everything sells for &e3x &7for the next &f{duration}&7.'
        - '&7'
      end-message:
        - '  &#ffd600&lWeekend Boost is over.'
      blacklisted_generators: []
```

`only-by-command: true` keeps it out of the automatic rotation. Start it yourself with
`/gens startevent weekend_boost`.

{% hint style="info" %}
`blacklisted_generators` lists generator ids the event does not touch. Use it to keep a special
or reward generator out of a tier boost or a mixed up event.
{% endhint %}

```yaml
# =============================================================================
#  SnGens - Events
# =============================================================================
#  Periodic global modifiers. When `random: true` events are picked weighted
#  by `chance`; otherwise they cycle in declaration order. Wait time runs
#  between events.
# -----------------------------------------------------------------------------
#  Types and their unique fields:
#    mixed_up          → no extra field
#    generator_speed   → percentage (int, drop-interval reduction)
#    drop_multiplier   → drop-amount (int, drops per tick)
#    generator_upgrade → tier-amount (int)
#    sell_multiplier   → multiplier (double)
#
#  Common fields:
#    display-name, duration (sec), chance, only-by-command,
#    start-message, end-message, blacklisted_generators
#
#  Placeholders in start/end messages: {name} {duration} {next_duration}
# =============================================================================

events:
  enabled: true
  random: false
  wait-time: 900

  placeholders:
    no-event: '&7Waiting Event...'
    no-event-timer: '&7Event in: &f{timer}'
    active-event: '{display_name}'
    active-event-timer: '&7Time left: &f{timer}'

  # ---------------------------------------------------------------------------
  # Event definitions
  # ---------------------------------------------------------------------------
  events:

    chaos_drops:
      type: mixed_up
      display-name: '&#ff00ff&lChaos Drops'
      duration: 180
      chance: 75
      only-by-command: false
      start-message:
        - '&7'
        - '  &#ff00ff&lEvent Started'
        - '  &#ff00ffType: &f{name}'
        - '  &#ff00ffDuration: &f{duration}'
        - '&7'
        - '  &7Each generator emits drops from a &drandom &7generator.'
        - '&7'
      end-message:
        - '&7'
        - '  &#ff00ff&lEvent Ended'
        - '  &#ff00ffNext event in: &f{next_duration}'
        - '&7'
      blacklisted_generators:
        - disabled_generator_id

    turbo_speed:
      type: generator_speed
      display-name: '&#ff1744&lTurbo Speed'
      percentage: 50
      duration: 180
      chance: 75
      only-by-command: false
      start-message:
        - '&7'
        - '  &#ff1744&lEvent Started'
        - '  &#ff1744Type: &f{name}'
        - '  &#ff1744Duration: &f{duration}'
        - '&7'
        - '  &7Generators tick &c2x faster&7.'
        - '&7'
      end-message:
        - '&7'
        - '  &#ff1744&lEvent Ended'
        - '  &#ff1744Next event in: &f{next_duration}'
        - '&7'
      blacklisted_generators:
        - disabled_generator_id

    drop_rain:
      type: drop_multiplier
      display-name: '&#00b0ff&lDrop Rain'
      drop-amount: 2
      duration: 180
      chance: 75
      only-by-command: false
      start-message:
        - '&7'
        - '  &#00b0ff&lEvent Started'
        - '  &#00b0ffType: &f{name}'
        - '  &#00b0ffDuration: &f{duration}'
        - '&7'
        - '  &7Generators emit &b2x &7drops per tick.'
        - '&7'
      end-message:
        - '&7'
        - '  &#00b0ff&lEvent Ended'
        - '  &#00b0ffNext event in: &f{next_duration}'
        - '&7'
      blacklisted_generators:
        - disabled_generator_id

    tier_boost:
      type: generator_upgrade
      display-name: '&#00e676&lTier Boost'
      tier-amount: 1
      duration: 180
      chance: 75
      only-by-command: false
      start-message:
        - '&7'
        - '  &#00e676&lEvent Started'
        - '  &#00e676Type: &f{name}'
        - '  &#00e676Duration: &f{duration}'
        - '&7'
        - '  &7Generators drop &a1 tier above &7their current tier.'
        - '&7'
      end-message:
        - '&7'
        - '  &#00e676&lEvent Ended'
        - '  &#00e676Next event in: &f{next_duration}'
        - '&7'
      blacklisted_generators:
        - disabled_generator_id

    gold_rush:
      type: sell_multiplier
      display-name: '&#ffd600&lGold Rush'
      multiplier: 2.0
      duration: 120
      chance: 75
      only-by-command: false
      start-message:
        - '&7'
        - '  &#ffd600&lEvent Started'
        - '  &#ffd600Type: &f{name}'
        - '  &#ffd600Duration: &f{duration}'
        - '&7'
        - '  &7Generator drop value increased by &e2x &7(/sell).'
        - '&7'
      end-message:
        - '&7'
        - '  &#ffd600&lEvent Ended'
        - '  &#ffd600Next event in: &f{next_duration}'
        - '&7'
      blacklisted_generators:
        - disabled_generator_id
```

---

## storages.yml

Two blocks that capture generator drops, deliberately different from each other.

| | Collector | Infinite hopper |
|---|-----------|-----------------|
| Scope | Every drop in its chunk | Items in a small box above itself |
| When it captures | Before the item entity exists | After the item exists, by scanning |
| Item types | Unlimited | The first `max-types` types lock in |
| Selling | Sell all button, sell on break, sale logs | Sellwand only |
| Best at | Absorbing a whole farm | Specialising on a few valuable drops |

The collector never lets the drop become an entity, so it is the option that removes the most
work from the server. The hopper lets items exist for a moment first, which is what produces
the visible sucked up effect players like.

Both blocks cap how many one player may place, both can be restricted to islands, and both draw
a hologram with `{owner}` and their contents.

Both also follow the generators when their owner leaves an island. A kick, leave, ban or disband
removes them from the island and stores the blocks in the owner's vault, claimed with
`/gens recover`. The `island-pickup` key in `config.yml` decides whether they are refunded or
destroyed, and `island-removal` here decides what happens to the items they were holding.

```yaml
island-removal:
  drop-contents: false   # true drops the stored items on the ground instead of voiding them
```

{% hint style="info" %}
The default voids the contents because the block is being removed from an island that is about
to be reset, or that now belongs to someone else. Dropping them there would either let them
despawn or hand them to the island owner. Set `drop-contents: true` if you would rather they
fall on the ground exactly as if a player had broken the block.
{% endhint %}

```yaml
collector:
  enabled: true
  max-per-player: 6
  capacity:
    infinite: true      # false enforces max-items
    max-items: 1000000
    void-excess: true   # delete overflow instead of dropping it
  break:
    sell-on-break: true  # pay the owner when the block is broken
    purge-on-sell: true
```

```yaml
hopper:
  enabled: true
  max-per-player: 12
  max-types: 5          # how many distinct item types lock in
  void-excess: false    # what happens to types that arrive after the slots are full
  break-drop-cap: 256   # stacks dropped on break before the rest is voided
```

{% hint style="warning" %}
`break-drop-cap` exists to protect the server. A hopper that stored a billion items would
otherwise try to spawn millions of entities in a single tick when broken. The player is told
how many stacks dropped and that the rest was voided.
{% endhint %}

The hopper's `scan` block controls how it absorbs. It scans its own box on a timer, with
adaptive backoff: an idle hopper doubles its interval up to `idle-backoff-ticks` and costs
almost nothing, while a productive one stays at `base-interval-ticks`. The box is slightly
wider than a vanilla hopper cup so items arriving on a water stream are caught reliably.

```yaml
# =============================================================================
#  SnGens - Storage utilities (Collector, future Infinite Hopper)
# =============================================================================
#  This file groups every "absorb-and-store generator drops" feature so they
#  share one config surface. Both behaviour AND cosmetics live here - the
#  plugin reads it via STORAGES_CONFIG and Settings caches the values.
# =============================================================================

# -----------------------------------------------------------------------------
# Island removal (shared by Collector and Infinite Hopper)
# -----------------------------------------------------------------------------
# When a player is kicked from, leaves, is banned from, or disbands a
# SuperiorSkyblock2 island, their Collectors and Hoppers are removed from the
# island and the blocks are returned through the refund vault, exactly like
# generators. The `island-pickup` flag in config.yml governs whether they are
# refunded or simply destroyed.
island-removal:
  # What happens to the ITEMS stored inside those Collectors and Hoppers.
  # false = the contents are voided with the block (default). The island is
  #         about to be reset or belongs to someone else, so dropped items
  #         would either despawn or be handed to the island owner.
  # true  = the contents drop on the ground where the block stood, exactly as
  #         if a player had broken it. Hoppers respect hopper.break-drop-cap.
  drop-contents: false

# -----------------------------------------------------------------------------
# Collector
# -----------------------------------------------------------------------------
# Chunk-bound block that intercepts generator drops BEFORE they spawn as
# entities. Items go straight into a virtual storage the owner browses
# through a GUI (right-click). Massive entity-count reduction on large farms.
# Selling routes through the same SellManager path as /sell - players see
# the configured sell.title / sell.action-bar / sell.chat visuals from
# lang/messages_<code>.yml. No separate collector-sold message.
collector:
  enabled: true
  # Restrict Collector placement to islands the player is a member of.
  # Requires SuperiorSkyblock2; no-op when SSB2 is not loaded.
  # Admins (sngens.collector.admin) bypass.
  island-place-only: false
  # Hard cap on placed Collectors per player. 0 disables the cap.
  # Admins (sngens.collector.admin) bypass.
  max-per-player: 6
  capacity:
    # Unlimited storage (default). Set to false to enforce max-items.
    infinite: true
    # Only honoured when infinite=false. Total summed item amount.
    max-items: 1000000
    # Only honoured when infinite=false. true = silently delete excess once
    # the Collector is full (no entity, no item). false = let the leftover
    # spawn normally on the ground.
    void-excess: true
  break:
    # When true, breaking a Collector also auto-sells its contents to the
    # owner via Vault. Falls back to "drop on ground" if Vault is unavailable.
    sell-on-break: true
    # When true, sell-on-break wipes the storage after selling. When false,
    # any items that didn't have a sell value drop on the ground.
    purge-on-sell: true
  item:
    material: ENDER_CHEST
    name: "&aChunk Collector"
    lore:
      - "&7Place me in a chunk to absorb"
      - "&7all generator drops in that chunk."
      - ""
      - "&eRight-click placed: open menu"
      - "&eBreak: pick up + sell contents"
  hologram:
    # Block-relative Y offset for the floating "owner / count / capacity" tag.
    y-offset: 2
    # Refresh cadence. Hologram is repainted only when items have actually
    # changed (debounced) - this is just the polling interval.
    refresh-ticks: 20
    lines:
      - "&b&lCOLLECTOR"
      - "&fOwner: &e{owner}"
      - "&fItems: &a{items}&7/&a{capacity}"
# GUI titles live in each gui/collector_*.yml under the `title:` key
# (mirrors gui/shop_gui.yml, gui/top_gui.yml, etc.).

# -----------------------------------------------------------------------------
# Infinite Hopper
# -----------------------------------------------------------------------------
# Block-precise vanilla HOPPER that absorbs ANY ItemSpawnEvent in its column
# (same X,Z, hopper Y < item Y). Each hopper has a small fixed number of
# "type slots" (default 5). The first dropped item type to land claims a
# slot and stacks infinitely; once all slots are claimed, drops of new
# types fall to the ground (or void, see below). Items are visible as
# entities for ~flush-interval-ticks before being absorbed (the "sucked-up"
# effect). Vanilla pickup + transfer are blocked so the hopper's vanilla
# inventory stays empty and our virtual storage is the single source of truth.
#
# Selling: sellwand only - right-click the hopper with a sellwand. There is
# NO Sell-All button, NO sell-on-break, NO sale logs. Differentiates the
# Hopper from the Collector as a gameplay choice (specialise vs absorb-all).
hopper:
  enabled: true
  # Restrict Hopper placement to islands the player is a member of.
  # Requires SuperiorSkyblock2; no-op when SSB2 is not loaded.
  # Admins (sngens.hopper.admin) bypass.
  island-place-only: false
  # Hard cap on placed Hoppers per player. 0 disables the cap.
  # Admins (sngens.hopper.admin) bypass.
  max-per-player: 12
  # Number of distinct item types a single Hopper can hold. The first N
  # types dropped into the hopper claim slots permanently; dropping a new
  # type after all N slots are full falls through (see void-excess).
  max-types: 5
  # When all type slots are full and a new type tries to enter:
  # true  = item is silently removed (no entity, no money).
  # false = item falls to the ground naturally for the player to deal with.
  void-excess: false
  # Flush cadence - how often (ticks) the manager drains absorbed items
  # from its per-region buffer. Lower = faster suction, higher = fewer
  # scheduler dispatches. 5 ticks (~0.25s) is the visual sweet spot.
  flush-interval-ticks: 5
  # Persistence cadence (minutes). Re-uses the same async writer as the
  # Collector. 0 = autosave disabled (only flush on shutdown).
  autosave-interval-minutes: 5
  # Maximum number of physical item stacks a Hopper drops on break before
  # voiding the rest. Prevents a 1B-item hopper from spawning millions of
  # entities in one tick. The player gets a "X stacks dropped, rest voided"
  # message when this trips.
  break-drop-cap: 256
  item:
    material: HOPPER
    name: "&6Infinite Hopper"
    lore:
      - "&7Place me - drops above me"
      - "&7are absorbed and stacked forever."
      - ""
      - "&eFirst &6{max-types}&e item types lock in."
      - "&eRight-click placed: open menu"
      - "&eSellwand: sell entire contents"
      - "&eBreak: pick up + drop contents"
  # Item-absorption scan. The Hopper does NOT use vanilla pickup or
  # ItemSpawnEvent - it scans its own bbox above the block on a global
  # timer, with adaptive backoff so idle hoppers cost almost nothing.
  # Bulk absorbs every eligible Item entity in the bbox in one go (vs
  # vanilla's 1-item-per-tick goteo).
  scan:
    # Base scan period in ticks while the hopper is actively absorbing.
    base-interval-ticks: 2
    # Maximum scan period in ticks once the hopper has been idle. Adaptive
    # backoff doubles the interval each idle scan up to this cap.
    idle-backoff-ticks: 40
    # Bbox above the hopper block. Centered on the block's top face (X+0.5,
    # Z+0.5). Items inside the bbox are absorbed; items outside (e.g. on
    # neighbour blocks, falling far off-axis) are ignored. Defaults are
    # slightly wider than vanilla cup so water-stream items in mid-fall
    # are caught reliably.
    bbox-x-radius: 0.8
    bbox-z-radius: 0.8
    bbox-y-up: 2.5
    # How far DOWN from the hopper's top face the bbox extends. The hopper
    # block has an interior cavity; items pushed in horizontally by ice or
    # water can end up trapped inside it with foot Y < top face. 1.0 covers
    # the entire hopper block so trapped items are absorbed on the next scan.
    bbox-y-down: 1.0
  hologram:
    enabled: true
    # Block-relative Y offset for the floating "owner / items / types" tag.
    y-offset: 2.2
    # Refresh cadence. Hologram is repainted only when the hopper's
    # contents have actually changed (debounced via dirty flag); this is
    # the polling interval.
    refresh-ticks: 20
    lines:
      - "&6&lHOPPER"
      - "&fOwner: &e{owner}"
      - "&fItems: &a{items}"
      - "&fTypes: &a{slots-used}&7/&a{slots-max}"
```

---

## armors.yml and offhands.yml

Both files define gear that boosts generator production. Five bonus keys exist, and they mean
the same thing in both files.

| Bonus key | Effect |
|-----------|--------|
| `speed` | Percentage faster generator interval |
| `tier` | Flat tier offset applied to the drop pool |
| `sell-multiplier` | Percentage bonus on every sale |
| `drop-multiplier` | Percentage more drops per generation |
| `lucky-chance` | Percentage chance to double the drop count |

Lore in both files accepts `{speed}`, `{tier}`, `{sell_multiplier}`, `{drop_multiplier}` and
`{lucky_chance}`, so the numbers on the item always match the numbers in the config.

An off-hand item works on its own, in the off-hand slot. An armor set only works as a complete
four piece set of the same set id: mixing a helmet from one set with the boots of another gives
no bonus from either. Each set defines its own `full` and `broken` messages, sent when the set
is completed or broken.

```yaml
armors:
  my_set:
    enabled: true
    full-set-bonuses:
      sell-multiplier: 20
      drop-multiplier: 10
    pieces:
      helmet: { material: LEATHER_HELMET, display-name: "&6My Helmet" }
      # chestplate, leggings, boots
    messages:
      full: "&aMy Set complete! &7(+{sell_multiplier}% sell)"
      broken: "&7My Set broken."
```

{% hint style="info" %}
The sell bonus from gear is multiplicative. It is applied as a percentage of the multiplier the
player already has from permissions, their stored multiplier and the active event, rather than
being added on top as a flat value. A player with no other multiplier therefore gains nothing
from a sell-multiplier piece.
{% endhint %}

Hand pieces out with `/gens armor give`, `/gens armor giveset` and `/gens offhand give`.

```yaml
# =============================================================================
#  SnGens - Armor sets
# =============================================================================
#  Each set has 4 pieces: helmet, chestplate, leggings, boots.
#  Bonuses activate ONLY when a player wears the full set (4/4 same set).
#  Mixing pieces from different sets = no bonus from any of them.
#
#  Recognised bonus keys: same as offhands.yml
#    speed | tier | sell-multiplier | drop-multiplier | lucky-chance
#
#  Lore placeholders: {speed} {tier} {sell_multiplier} {drop_multiplier} {lucky_chance}
# =============================================================================

armors:

  swift_set:
    enabled: true
    full-set-bonuses:
      speed: 25
    pieces:
      helmet:
        material: LEATHER_HELMET
        color: { r: 100, g: 220, b: 255 }
        display-name: "&b&lSwift Helmet"
        flags:
          - HIDE_ATTRIBUTES
          - HIDE_UNBREAKABLE
        lore:
          - "&7Part of the &bSwift &7set."
          - "&8» Full set: &b+{speed}% &7speed"
      chestplate:
        material: LEATHER_CHESTPLATE
        color: { r: 100, g: 220, b: 255 }
        display-name: "&b&lSwift Chestplate"
        flags:
          - HIDE_ATTRIBUTES
          - HIDE_UNBREAKABLE
        lore:
          - "&7Part of the &bSwift &7set."
          - "&8» Full set: &b+{speed}% &7speed"
      leggings:
        material: LEATHER_LEGGINGS
        color: { r: 100, g: 220, b: 255 }
        display-name: "&b&lSwift Leggings"
        flags:
          - HIDE_ATTRIBUTES
          - HIDE_UNBREAKABLE
        lore:
          - "&7Part of the &bSwift &7set."
          - "&8» Full set: &b+{speed}% &7speed"
      boots:
        material: LEATHER_BOOTS
        color: { r: 100, g: 220, b: 255 }
        display-name: "&b&lSwift Boots"
        flags:
          - HIDE_ATTRIBUTES
          - HIDE_UNBREAKABLE
        lore:
          - "&7Part of the &bSwift &7set."
          - "&8» Full set: &b+{speed}% &7speed"
    messages:
      full: "&aSwift Set complete! &7(+{speed}% speed)"
      broken: "&7Swift Set broken."

  prosper_set:
    enabled: true
    full-set-bonuses:
      tier: 3
    pieces:
      helmet:
        material: LEATHER_HELMET
        color: { r: 230, g: 200, b: 80 }
        display-name: "&e&lProsper Helmet"
        flags:
          - HIDE_ATTRIBUTES
          - HIDE_UNBREAKABLE
        lore:
          - "&7Part of the &eProsper &7set."
          - "&8» Full set: &e+{tier} &7tier"
      chestplate:
        material: LEATHER_CHESTPLATE
        color: { r: 230, g: 200, b: 80 }
        display-name: "&e&lProsper Chestplate"
        flags:
          - HIDE_ATTRIBUTES
          - HIDE_UNBREAKABLE
        lore:
          - "&7Part of the &eProsper &7set."
          - "&8» Full set: &e+{tier} &7tier"
      leggings:
        material: LEATHER_LEGGINGS
        color: { r: 230, g: 200, b: 80 }
        display-name: "&e&lProsper Leggings"
        flags:
          - HIDE_ATTRIBUTES
          - HIDE_UNBREAKABLE
        lore:
          - "&7Part of the &eProsper &7set."
          - "&8» Full set: &e+{tier} &7tier"
      boots:
        material: LEATHER_BOOTS
        color: { r: 230, g: 200, b: 80 }
        display-name: "&e&lProsper Boots"
        flags:
          - HIDE_ATTRIBUTES
          - HIDE_UNBREAKABLE
        lore:
          - "&7Part of the &eProsper &7set."
          - "&8» Full set: &e+{tier} &7tier"
    messages:
      full: "&aProsper Set complete! &7(+{tier} tier)"
      broken: "&7Prosper Set broken."

  greed_set:
    enabled: true
    full-set-bonuses:
      sell-multiplier: 20
      drop-multiplier: 10
    pieces:
      helmet:
        material: LEATHER_HELMET
        color: { r: 255, g: 180, b: 0 }
        display-name: "&6&lGreed Helmet"
        flags:
          - HIDE_ATTRIBUTES
          - HIDE_UNBREAKABLE
        lore:
          - "&7Part of the &6Greed &7set."
          - "&8» Full set: &6+{sell_multiplier}% &7sell"
          - "&8» Full set: &6+{drop_multiplier}% &7drops"
      chestplate:
        material: LEATHER_CHESTPLATE
        color: { r: 255, g: 180, b: 0 }
        display-name: "&6&lGreed Chestplate"
        flags:
          - HIDE_ATTRIBUTES
          - HIDE_UNBREAKABLE
        lore:
          - "&7Part of the &6Greed &7set."
          - "&8» Full set: &6+{sell_multiplier}% &7sell"
          - "&8» Full set: &6+{drop_multiplier}% &7drops"
      leggings:
        material: LEATHER_LEGGINGS
        color: { r: 255, g: 180, b: 0 }
        display-name: "&6&lGreed Leggings"
        flags:
          - HIDE_ATTRIBUTES
          - HIDE_UNBREAKABLE
        lore:
          - "&7Part of the &6Greed &7set."
          - "&8» Full set: &6+{sell_multiplier}% &7sell"
          - "&8» Full set: &6+{drop_multiplier}% &7drops"
      boots:
        material: LEATHER_BOOTS
        color: { r: 255, g: 180, b: 0 }
        display-name: "&6&lGreed Boots"
        flags:
          - HIDE_ATTRIBUTES
          - HIDE_UNBREAKABLE
        lore:
          - "&7Part of the &6Greed &7set."
          - "&8» Full set: &6+{sell_multiplier}% &7sell"
          - "&8» Full set: &6+{drop_multiplier}% &7drops"
    messages:
      full: "&aGreed Set complete! &7(+{sell_multiplier}% sell, +{drop_multiplier}% drops)"
      broken: "&7Greed Set broken."

  lucky_set:
    enabled: true
    full-set-bonuses:
      lucky-chance: 12
      drop-multiplier: 5
    pieces:
      helmet:
        material: LEATHER_HELMET
        color: { r: 200, g: 100, b: 220 }
        display-name: "&d&lLucky Helmet"
        flags:
          - HIDE_ATTRIBUTES
          - HIDE_UNBREAKABLE
        lore:
          - "&7Part of the &dLucky &7set."
          - "&8» Full set: &d{lucky_chance}% &7double drop"
          - "&8» Full set: &d+{drop_multiplier}% &7drops"
      chestplate:
        material: LEATHER_CHESTPLATE
        color: { r: 200, g: 100, b: 220 }
        display-name: "&d&lLucky Chestplate"
        flags:
          - HIDE_ATTRIBUTES
          - HIDE_UNBREAKABLE
        lore:
          - "&7Part of the &dLucky &7set."
          - "&8» Full set: &d{lucky_chance}% &7double drop"
          - "&8» Full set: &d+{drop_multiplier}% &7drops"
      leggings:
        material: LEATHER_LEGGINGS
        color: { r: 200, g: 100, b: 220 }
        display-name: "&d&lLucky Leggings"
        flags:
          - HIDE_ATTRIBUTES
          - HIDE_UNBREAKABLE
        lore:
          - "&7Part of the &dLucky &7set."
          - "&8» Full set: &d{lucky_chance}% &7double drop"
          - "&8» Full set: &d+{drop_multiplier}% &7drops"
      boots:
        material: LEATHER_BOOTS
        color: { r: 200, g: 100, b: 220 }
        display-name: "&d&lLucky Boots"
        flags:
          - HIDE_ATTRIBUTES
          - HIDE_UNBREAKABLE
        lore:
          - "&7Part of the &dLucky &7set."
          - "&8» Full set: &d{lucky_chance}% &7double drop"
          - "&8» Full set: &d+{drop_multiplier}% &7drops"
    messages:
      full: "&aLucky Set complete! &7({lucky_chance}% double drop, +{drop_multiplier}% drops)"
      broken: "&7Lucky Set broken."

  mythic_set:
    enabled: true
    full-set-bonuses:
      speed: 10
      tier: 1
      sell-multiplier: 10
      drop-multiplier: 5
      lucky-chance: 5
    pieces:
      helmet:
        material: LEATHER_HELMET
        color: { r: 90, g: 0, b: 130 }
        display-name: "&5&lMythic Helmet"
        flags:
          - HIDE_ATTRIBUTES
          - HIDE_UNBREAKABLE
        lore:
          - "&7Part of the &5Mythic &7set."
          - "&8» &5+{speed}% &7speed &8| &5+{tier} &7tier"
          - "&8» &5+{sell_multiplier}% &7sell &8| &5+{drop_multiplier}% &7drops"
          - "&8» &5{lucky_chance}% &7double drop"
      chestplate:
        material: LEATHER_CHESTPLATE
        color: { r: 90, g: 0, b: 130 }
        display-name: "&5&lMythic Chestplate"
        flags:
          - HIDE_ATTRIBUTES
          - HIDE_UNBREAKABLE
        lore:
          - "&7Part of the &5Mythic &7set."
          - "&8» &5+{speed}% &7speed &8| &5+{tier} &7tier"
          - "&8» &5+{sell_multiplier}% &7sell &8| &5+{drop_multiplier}% &7drops"
          - "&8» &5{lucky_chance}% &7double drop"
      leggings:
        material: LEATHER_LEGGINGS
        color: { r: 90, g: 0, b: 130 }
        display-name: "&5&lMythic Leggings"
        flags:
          - HIDE_ATTRIBUTES
          - HIDE_UNBREAKABLE
        lore:
          - "&7Part of the &5Mythic &7set."
          - "&8» &5+{speed}% &7speed &8| &5+{tier} &7tier"
          - "&8» &5+{sell_multiplier}% &7sell &8| &5+{drop_multiplier}% &7drops"
          - "&8» &5{lucky_chance}% &7double drop"
      boots:
        material: LEATHER_BOOTS
        color: { r: 90, g: 0, b: 130 }
        display-name: "&5&lMythic Boots"
        flags:
          - HIDE_ATTRIBUTES
          - HIDE_UNBREAKABLE
        lore:
          - "&7Part of the &5Mythic &7set."
          - "&8» &5+{speed}% &7speed &8| &5+{tier} &7tier"
          - "&8» &5+{sell_multiplier}% &7sell &8| &5+{drop_multiplier}% &7drops"
          - "&8» &5{lucky_chance}% &7double drop"
    messages:
      full: "&aMythic Set complete!"
      broken: "&7Mythic Set broken."
```

```yaml
# =============================================================================
#  SnGens - Offhands
# =============================================================================
#  Each offhand is held in the off-hand slot. While equipped, its bonuses are
#  added to the player's snapshot every time it changes (event-driven, not
#  polled). Recognised bonus keys:
#
#    speed             - % faster generator interval
#    tier              - flat tier offset on the drop pool
#    sell-multiplier   - % bonus on every sell (stacks with sellwand/perms)
#    drop-multiplier   - % extra drops per generation
#    lucky-chance      - % chance to double the drop count
#
#  Lore placeholders: {speed} {tier} {sell_multiplier} {drop_multiplier} {lucky_chance}
# =============================================================================

offhands:

  swift:
    enabled: true
    bonuses:
      speed: 10
    item:
      material: NETHERITE_UPGRADE_SMITHING_TEMPLATE
      display-name: "&b&lSwift Offhand"
      flags:
        - HIDE_ATTRIBUTES
        - HIDE_UNBREAKABLE
        - HIDE_ARMOR_TRIM
      lore:
        - "&7Held in the off-hand slot."
        - ""
        - "&8» &b+{speed}% &7generator speed"
    messages:
      equip: "&aSwift Offhand equipped &7(+{speed}% speed)"
      unequip: "&7Swift Offhand removed."

  prosper:
    enabled: true
    bonuses:
      tier: 1
    item:
      material: NETHERITE_UPGRADE_SMITHING_TEMPLATE
      display-name: "&e&lProsper Offhand"
      flags:
        - HIDE_ATTRIBUTES
        - HIDE_UNBREAKABLE
        - HIDE_ARMOR_TRIM
      lore:
        - "&7Held in the off-hand slot."
        - ""
        - "&8» &e+{tier} &7generator tier"
    messages:
      equip: "&aProsper Offhand equipped &7(+{tier} tier)"
      unequip: "&7Prosper Offhand removed."

  goldrush:
    enabled: true
    bonuses:
      sell-multiplier: 15
    item:
      material: NETHERITE_UPGRADE_SMITHING_TEMPLATE
      display-name: "&6&lGoldrush Offhand"
      flags:
        - HIDE_ATTRIBUTES
        - HIDE_UNBREAKABLE
        - HIDE_ARMOR_TRIM
      lore:
        - "&7Held in the off-hand slot."
        - ""
        - "&8» &6+{sell_multiplier}% &7sell multiplier"
    messages:
      equip: "&aGoldrush Offhand equipped &7(+{sell_multiplier}% sell)"
      unequip: "&7Goldrush Offhand removed."

  fortune:
    enabled: true
    bonuses:
      lucky-chance: 5
    item:
      material: NETHERITE_UPGRADE_SMITHING_TEMPLATE
      display-name: "&d&lFortune Offhand"
      flags:
        - HIDE_ATTRIBUTES
        - HIDE_UNBREAKABLE
        - HIDE_ARMOR_TRIM
      lore:
        - "&7Held in the off-hand slot."
        - ""
        - "&8» &d{lucky_chance}% &7chance for double drop"
    messages:
      equip: "&aFortune Offhand equipped &7({lucky_chance}% double drop)"
      unequip: "&7Fortune Offhand removed."

  bountiful:
    enabled: true
    bonuses:
      drop-multiplier: 10
    item:
      material: NETHERITE_UPGRADE_SMITHING_TEMPLATE
      display-name: "&a&lBountiful Offhand"
      flags:
        - HIDE_ATTRIBUTES
        - HIDE_UNBREAKABLE
        - HIDE_ARMOR_TRIM
      lore:
        - "&7Held in the off-hand slot."
        - ""
        - "&8» &a+{drop_multiplier}% &7drops per generation"
    messages:
      equip: "&aBountiful Offhand equipped &7(+{drop_multiplier}% drops)"
      unequip: "&7Bountiful Offhand removed."
```

---

## gui/

Eight menu layouts. They share the same conventions:

- `title` is the menu name, `size` is the slot count, and slots are numbered from 0.
- `item-slots` or `entry-slots` list where the dynamic content goes. Content is paginated over
  them automatically.
- The `items` section holds static decoration and navigation. An entry with `type: NEXT_PAGE`
  or `type: PREVIOUS_PAGE` only renders when another page exists.
- `slots` on a static item accepts a list, so one definition can fill a whole border.

Removing a static item from `items` simply removes it from the menu. Changing `slots` moves it.

### gui/shop_gui.yml

The generator shop. Its `base-cost` key is the price of the first generator in the upgrade
chain, and every other price is derived from there, as explained under `generators.yml`.
`extra-lore` is appended to each generator's own lore, and accepts `{gen}`, `{gen_id}`,
`{cost}` and `{cost_short}`.

```yaml
# =============================================================================
#  SnGens - Generator Shop GUI
# =============================================================================
#  Items are auto-paginated from generators.yml. Cost is computed on the fly:
#    cost(gen_0) = base-cost
#    cost(gen_N) = cost(gen_{N-1}) + upgrade.upgrade-cost of gen_{N-1}
#  Prices refresh on /sngens reload and onEnable.
#
#  Lore placeholders:  {gen}  {gen_id}  {cost}  {cost_short}
# =============================================================================

title: "&8Generator Shop"
size: 54

# Slots used to display generators (paginated automatically).
item-slots: [10,11,12,13,14,15,16,19,20,21,22,23,24,25,28,29,30,31,32,33,34,37,38,39,40,41,42,43]

# Buy cost of the very first generator in the upgrade chain.
base-cost: 50.0

# Lines appended to each generator's existing lore.
extra-lore:
  - "&7"
  - "&fCost: &2&l$&a{cost_short}"
  - "&7"
  - "&eClick to purchase"


# -----------------------------------------------------------------------------
# Static / nav items. NEXT_PAGE and PREVIOUS_PAGE only render when more pages
# exist. Tipped-arrow colour follows `potion-type`; override with `color: {r,g,b}`.
# -----------------------------------------------------------------------------
items:

  next-page:
    type: NEXT_PAGE
    material: TIPPED_ARROW
    display-name: "&aNext Page"
    slots: [50]
    lore:
      - "&7Click to go to the next page!"
    potion-type: LUCK

  previous-page:
    type: PREVIOUS_PAGE
    material: TIPPED_ARROW
    display-name: "&cPrevious Page"
    slots: [48]
    lore:
      - "&7Click to go to the previous page!"
    potion-type: HEALING

  filler:
    material: GRAY_STAINED_GLASS_PANE
    display-name: "&7"
    slots: [0,1,2,3,4,5,6,7,8,9,17,18,26,27,35,36,44,45,46,47,48,49,50,51,52,53]
    lore: []
```

### gui/shop_buy_amount.yml

The buy quantity submenu, used when `shop.buy-amount-menu` is `true` in `config.yml`. Its
`quantities` map is a slot to amount mapping, so you decide both which amounts are offered and
where the buttons sit. Each button buys as many as the player can afford, up to the amount on
the button, and charges only for what was bought.

```yaml
# =============================================================================
#  SnGens - Buy Quantity submenu
# =============================================================================
#  Opened from the shop when `shop.buy-amount-menu` is true (config.yml).
#  Clicking a generator in the shop opens this menu; each button buys the
#  configured amount at once (buy-as-many-as-affordable if funds are short).
#
#  The icon of every buy button is ALWAYS the generator's block, and its stack
#  size mirrors the quantity (so it reads 1 / 16 / 32 / 64 in the item corner).
#
#  Title placeholder:            {gen}
#  Buy name/lore placeholders:   {gen} {amount} {cost} {cost_short} {total} {total_short}
#    {cost}/{cost_short}  = unit price of the generator
#    {total}/{total_short}= unit price * amount
# =============================================================================

title: "&8Buy - {gen}"
size: 27

# slot -> quantity purchased (and the visual stack size of the icon).
# Fully editable: add / remove / move any slot, set any amount. Amounts above
# 64 still work (the granted item is split into stacks), the icon just caps at 64.
quantities:
  10: 1
  12: 16
  14: 32
  16: 64

# Shared name / lore applied to every buy button above.
buy-name: "&aBuy &e{amount}&7x"
buy-lore:
  - "&7"
  - "&fUnit price: &2$&a{cost_short}"
  - "&fTotal: &2&l$&a{total_short}"
  - "&7"
  - "&eClick to purchase"

# -----------------------------------------------------------------------------
# Static items. BACK returns to the generator shop. Filler paints empty slots
# (it must not cover the buy slots or the back slot).
# -----------------------------------------------------------------------------
items:

  back:
    type: BACK
    material: ARROW
    display-name: "&cBack"
    slots: [22]
    lore:
      - "&7Return to the shop."

  filler:
    type: DUMMY
    material: GRAY_STAINED_GLASS_PANE
    display-name: "&7"
    slots: [0,1,2,3,4,5,6,7,8,9,11,13,15,17,18,19,20,21,23,24,25,26]
    lore: []
```

### gui/upgrade_gens_gui.yml

The bulk upgrade menu. It groups a player's generators by tier, one entry per tier, and offers
a single upgrade, a batch of `upgrade-gui.batch-amount`, or all of them.

Two toggles live in the menu itself: the view mode switches between your own generators and
every generator on your island, and the sort mode switches between most owned first and lowest
tier first. `title` and `title-island` let the two view modes carry different names.

Entry lore accepts `{amount}`, `{cost}`, `{total_cost}`, `{batch_amount}`, `{cost_batch}` and
the short variants of each.

```yaml
# =============================================================================
#  SnGens - Upgrade Generators GUI
# =============================================================================
#  Bulk-upgrade GUI opened via /sngens upgrade. Listed slots show how many
#  generators of each tier the player owns, with one-click / right-click bulk
#  upgrade.
#
#  View modes (toggle in slot 50):
#    PERSONAL - only your generators
#    ISLAND   - all generators of every member of your SuperiorSkyblock2 island
#
#  Sort modes (toggle in slot 52):
#    QUANTITY - most-owned first
#    TIER     - lowest tier first
#
#  Per-island usage limits + bonus pool come from `upgrade-gui.usage` in
#  config.yml. /sngens addupgrades grants bonus uses.
#
#  Lore placeholders for generator items:
#    {amount}  {amount_short}
#    {cost}  {cost_short}                  - cost of upgrading 1 gen
#    {total_cost}  {total_cost_short}      - cost of upgrading all
#    {batch_amount}                        - value of upgrade-gui.batch-amount
#    {cost_batch}  {cost_batch_short}      - cost of upgrading {batch_amount}
# =============================================================================

title: "Upgrade Generators"
title-island: "Upgrade Generators (Island)"
size: 54

# Slots that hold the per-tier upgrade entries (top 5 rows = 0..44).
item-slots: [0, 1, 2, 3, 4, 5, 6, 7, 8,
             9, 10, 11, 12, 13, 14, 15, 16, 17,
             18, 19, 20, 21, 22, 23, 24, 25, 26,
             27, 28, 29, 30, 31, 32, 33, 34, 35,
             36, 37, 38, 39, 40, 41, 42, 43, 44]

# Ticks between bulk-upgrade steps.
upgrade-speed: 5

# Stop after one full pass over the list?
stop-when-finished: true


# Lore for each generator entry.
lore:
  - "&7{amount} gens &8({amount_short} gens)"
  - "&7"
  - "&fOne gen: &2&l$&a{cost_short}"
  - "&fBatch ({batch_amount}): &2&l$&a{cost_batch_short}"
  - "&fTotal: &2&l$&a{total_cost_short}"
  - "&7"
  - "&fLeft Click: &eUpgrade one"
  - "&fRight Click: &eUpgrade all"
  - "&f[Q] Drop: &eUpgrade {batch_amount}"


# View / sort labels and option formatting.
view-modes:
  personal: "Personal"
  island: "Island"

sort-modes:
  quantity: "By Quantity"
  tier: "By Tier"

view-option-selected: " &a❯ &f{mode}"
view-option-unselected: " &7  {mode}"
sort-option-selected: " &a❯ &f{mode}"
sort-option-unselected: " &7  {mode}"


# -----------------------------------------------------------------------------
# Static / nav items.
# -----------------------------------------------------------------------------
items:

  previous-page:
    type: PREVIOUS_PAGE
    material: TIPPED_ARROW
    display-name: "&cPrevious Page"
    slots: [45]
    lore:
      - "&7Click to go to the previous page!"
    color: { r: 255, g: 0, b: 0 }

  next-page:
    type: NEXT_PAGE
    material: TIPPED_ARROW
    display-name: "&aNext Page"
    slots: [53]
    lore:
      - "&7Click to go to the next page!"
    color: { r: 0, g: 255, b: 0 }

  info-uses:
    material: PAPER
    display-name: "&eUpgrade Uses"
    slots: [48]
    lore:
      - "&7Remaining uses today:"
      - "&7"
      - " &7Daily: &f{daily_remaining}/{daily_max}"
      - " &7Bonus: &f{bonus_remaining}"
      - " &7Total: &a{total_remaining}"

  view-toggle:
    material: OAK_SIGN
    display-name: "&eView: &f{view_mode}"
    slots: [50]
    lore:
      - "&7Click to switch which generators"
      - "&7this menu shows."
      - "&7"
      - "{view_options}"
      - "&7"
      - "&eClick to cycle."

  sort-toggle:
    material: HOPPER
    display-name: "&eSort: &f{sort_mode}"
    slots: [52]
    lore:
      - "&7Click to switch the sort order"
      - "&7of generators in this menu."
      - "&7"
      - "{sort_options}"
      - "&7"
      - "&eClick to cycle."

  filler:
    material: GRAY_STAINED_GLASS_PANE
    display-name: "&7"
    slots: [46, 47, 49, 51]
    lore: []
```

### gui/top_gui.yml

The leaderboard menu. `entry-slots` decides how many ranks are visible and in which order they
are laid out. `entry-item` is the template for a populated rank, and the head is always the
owner's skull. `empty-entry` fills the ranks that nobody holds yet.

Entry placeholders: `{rank}`, `{name}`, `{value}`, `{value_short}` and
`{updated_seconds_ago}`.

```yaml
# =============================================================================
#  SnGens - Gens Top GUI
# =============================================================================
#  Renders the gens-top leaderboard. Entries are paginated to the slots below.
#  The skull head is set to the entry's owner (player) or the island's leader.
#
#  Entry placeholders:
#    {rank} {name} {value} {value_short} {updated_seconds_ago}
# =============================================================================

title: "&8Gens Top"
size: 54

# Slots filled with leaderboard entries (rank 1..N in order). When the
# snapshot has fewer entries than slots, remaining slots get the empty filler.
entry-slots: [10, 11, 12, 13, 14, 15, 16, 19, 20, 21, 22, 23, 24, 25]

# Item template for a populated entry. Material is forced to PLAYER_HEAD;
# only display-name and lore are read here.
entry-item:
  display-name: "&e#{rank} &f{name}"
  lore:
    - "&7"
    - "&fValue: &2$&a{value_short}"
    - "&7Raw: &f${value}"
    - "&7"
    - "&8Updated &f{updated_seconds_ago}s &8ago"

# Shown in entry-slots that have no corresponding ranked entry.
empty-entry:
  material: GRAY_STAINED_GLASS_PANE
  display-name: "&7Empty slot"
  lore:
    - "&8No entry at this rank yet."

# Static decoration / nav.
items:

  filler:
    material: BLACK_STAINED_GLASS_PANE
    display-name: "&7"
    slots: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 17, 18, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 50, 51, 52, 53]
    lore: []

  close:
    material: BARRIER
    display-name: "&cClose"
    slots: [49]
    lore:
      - "&7Click to close."

# =============================================================================
#  Split layout - solo/team island servers ONLY
# =============================================================================
#  Used instead of the single board above ONLY when a skyblock plugin that
#  separates solo from team islands is installed AND gens-top.mode = ISLAND AND
#  gens-top.split-solo-team = true. On plain SuperiorSkyblock2 / no skyblock,
#  these keys are ignored and the single board above is shown.
#
#  The same 54-slot menu then shows two boards side by side: team islands on the
#  left, solo islands on the right, separated by a divider column. Both boards
#  reuse the `entry-item` and `empty-entry` templates defined above.
# =============================================================================
split:
  # Menu title when the split layout is active.
  title: "&8Gens Top - &bTeam &7vs &eSolo"

  team:
    # Left columns, rows 1-4 (rank 1..N top-to-bottom, left-to-right).
    entry-slots: [10, 11, 12, 19, 20, 21, 28, 29, 30, 37, 38, 39]
    header:
      material: CYAN_BANNER
      display-name: "&b&lTeam Islands"
      slots: [1, 2, 3]
      lore:
        - "&7Top team islands by gens value."

  solo:
    # Right columns, rows 1-4.
    entry-slots: [14, 15, 16, 23, 24, 25, 32, 33, 34, 41, 42, 43]
    header:
      material: YELLOW_BANNER
      display-name: "&e&lSolo Islands"
      slots: [5, 6, 7]
      lore:
        - "&7Top solo islands by gens value."

  # Center column separating the two boards.
  divider:
    material: BLUE_STAINED_GLASS_PANE
    display-name: "&7"
    slots: [4, 13, 22, 31, 40]
    lore: []

  # Background panes for every remaining empty slot.
  filler:
    material: BLACK_STAINED_GLASS_PANE
    display-name: "&7"
    slots: [0, 8, 9, 17, 18, 26, 27, 35, 36, 44, 45, 46, 47, 48, 50, 51, 52, 53]
    lore: []

  close:
    material: BARRIER
    display-name: "&cClose"
    slots: [49]
    lore:
      - "&7Click to close."
```

### gui/collector_main.yml, collector_storage.yml, collector_logs.yml

The three collector menus: the main panel a player sees when they right click the block, the
paginated storage list of what it holds, and the paginated sale log.

Every button in the main panel declares what it does through its `type`: `INFO` shows the owner
and the contents, `STORAGE` opens the storage list, `SELL_ALL` sells everything at once, `LOGS`
opens the sale log, `REMOVE` breaks the block from inside the menu, and `DUMMY` is decoration.
Delete a button you do not want, or move it by changing its `slots`.

```yaml
# Collector main menu - opened on right-click on the Collector block.
# Placeholders: {items} {capacity} {owner}
title: "&8Collector"
size: 27

items:
  filler:
    material: GRAY_STAINED_GLASS_PANE
    display-name: "&7"
    type: DUMMY
    slots: [0, 1, 2, 3, 5, 6, 7, 8, 9, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26]
    lore: []

  info:
    material: ENDER_CHEST
    display-name: "&b&lCollector"
    type: INFO
    slots: [4]
    lore:
      - "&7Owner: &e{owner}"
      - "&7Stored: &a{items}&7/&a{capacity}"

  storage:
    material: CHEST
    display-name: "&aOpen Storage"
    type: STORAGE
    slots: [10]
    lore:
      - "&7View, withdraw, or sell stored items."

  sell-all:
    material: GOLD_INGOT
    display-name: "&6Sell All"
    type: SELL_ALL
    slots: [12]
    lore:
      - "&7Sell every item stored at once."
      - "&7Multiplier from your &eSnGens user&7 applies."

  logs:
    material: BOOK
    display-name: "&fSale Logs"
    type: LOGS
    slots: [14]
    lore:
      - "&7Last 50 sales - when, how much, what."

  remove:
    material: BARRIER
    display-name: "&cRemove Collector"
    type: REMOVE
    slots: [16]
    lore:
      - "&7Auto-sells contents (if Vault is up)"
      - "&7and returns the Collector item."
```

```yaml
# Collector storage menu - paginated list of stored item types.
# Click handlers (per item):
#   Left:        withdraw 1 stack (max-stack-sized)
#   Shift-Left:  withdraw all of that type
#   Right:       sell that type only
title: "&8Collector - Storage"
size: 54

# Slots used to render content items (paginated).
content-slots: [10, 11, 12, 13, 14, 15, 16, 19, 20, 21, 22, 23, 24, 25, 28, 29, 30, 31, 32, 33, 34, 37, 38, 39, 40, 41, 42, 43]

# Extra lore appended to each stored-item entry. Placeholders: {stored} {unit} {total}.
# entry-extra-lore is used when the item has a known unit value (sellable),
# entry-extra-lore-no-value is used when no unit value is known.
entry-extra-lore:
  - ""
  - "&7Stored: &a{stored}"
  - "&7Value/each: &2$&a{unit}"
  - "&7Total value: &2$&a{total}"
  - ""
  - "&eLeft-click&7: take 1 stack"
  - "&eShift-Left&7: take all"
  - "&eRight-click&7: sell this type"

entry-extra-lore-no-value:
  - ""
  - "&7Stored: &a{stored}"
  - ""
  - "&eLeft-click&7: take 1 stack"
  - "&eShift-Left&7: take all"
  - "&eRight-click&7: sell this type"

items:
  filler:
    material: GRAY_STAINED_GLASS_PANE
    display-name: "&7"
    type: DUMMY
    slots: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 17, 18, 26, 27, 35, 36, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53]
    lore: []

  back:
    material: ARROW
    display-name: "&cBack"
    type: BACK
    slots: [49]
    lore:
      - "&7Return to main menu."

  previous-page:
    material: TIPPED_ARROW
    display-name: "&cPrevious Page"
    type: PREVIOUS_PAGE
    slots: [45]
    lore:
      - "&7Click to go to the previous page."
    potion-type: HEALING

  next-page:
    material: TIPPED_ARROW
    display-name: "&aNext Page"
    type: NEXT_PAGE
    slots: [53]
    lore:
      - "&7Click to go to the next page."
    potion-type: LUCK
```

```yaml
# Collector sale logs viewer (read-only, paginated).
title: "&8Collector - Sale Logs"
size: 54

# Slots used to render log entries (paginated).
content-slots: [10, 11, 12, 13, 14, 15, 16, 19, 20, 21, 22, 23, 24, 25, 28, 29, 30, 31, 32, 33, 34, 37, 38, 39, 40, 41, 42, 43]

# Item rendered per log entry. Placeholders: {date} {items} {money}
log-entry:
  material: PAPER
  display-name: "&aSale &7- &f{date}"
  lore:
    - "&7Items sold: &f{items}"
    - "&7Earned: &2$&a{money}"

# Item rendered when there are no logs yet (placed in the middle content slot).
empty:
  material: BARRIER
  display-name: "&cNo sales recorded yet."
  lore: []

items:
  filler:
    material: GRAY_STAINED_GLASS_PANE
    display-name: "&7"
    type: DUMMY
    slots: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 17, 18, 26, 27, 35, 36, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53]
    lore: []

  back:
    material: ARROW
    display-name: "&cBack"
    type: BACK
    slots: [49]
    lore:
      - "&7Return to main menu."

  previous-page:
    material: TIPPED_ARROW
    display-name: "&cPrevious Page"
    type: PREVIOUS_PAGE
    slots: [45]
    lore:
      - "&7Click to go to the previous page."
    potion-type: HEALING

  next-page:
    material: TIPPED_ARROW
    display-name: "&aNext Page"
    type: NEXT_PAGE
    slots: [53]
    lore:
      - "&7Click to go to the next page."
    potion-type: LUCK
```

### gui/hopper_main.yml

The infinite hopper panel. The hopper has no sell button by design, so this menu shows what is
stored and lets the owner claim it: left click an entry to take one stack, shift left click to
take everything of that type.

`type-slots` decides where the locked in item types are drawn, so its length should match
`hopper.max-types` in `storages.yml`. Raise one without the other and the extra types are
stored but never shown.

```yaml
# Infinite Hopper main menu - opened on right-click on a placed Hopper.
# Placeholders: {items} {slots-used} {slots-max} {owner}
title: "&6Infinite Hopper"
size: 27

# Slots that render the lock-in type icons. The first N entries of the
# hopper's contents (LinkedHashMap insertion order) map 1:1 onto these
# slots. Default count = 5 - match storages.yml hopper.max-types.
type-slots: [10, 12, 13, 14, 16]

# Extra lore appended to each stored-item entry. Placeholders: {stored}.
entry-extra-lore:
  - ""
  - "&7Stored: &a{stored}"
  - ""
  - "&eLeft-click&7: claim 1 stack"
  - "&eShift-Left&7: claim all"

items:
  filler:
    material: GRAY_STAINED_GLASS_PANE
    display-name: "&7"
    type: DUMMY
    slots: [0, 1, 2, 3, 5, 6, 7, 8, 9, 11, 15, 17, 18, 19, 20, 21, 23, 24, 25, 26]
    lore: []

  info:
    material: HOPPER
    display-name: "&6&lInfinite Hopper"
    type: INFO
    slots: [4]
    lore:
      - "&7Owner: &e{owner}"
      - "&7Stored: &a{items}"
      - "&7Type slots: &a{slots-used}&7/&a{slots-max}"
      - ""
      - "&7Sell with a &esellwand &7(right-click)."

  remove:
    material: BARRIER
    display-name: "&cRemove Hopper"
    type: REMOVE
    slots: [22]
    lore:
      - "&7Drops contents on the ground"
      - "&7and returns the Hopper item."

  empty-slot:
    material: LIGHT_GRAY_STAINED_GLASS_PANE
    display-name: "&7Empty type slot"
    type: DUMMY
    slots: []
    lore:
      - "&7The first item type that lands"
      - "&7here will lock this slot in."
```

---

## lang/messages_en.yml

Every string the plugin sends to a player lives here. Nothing is hardcoded, so any message can
be reworded, recoloured, turned into several lines, or emptied out.

- `prefix` is prepended to single line messages. List values bypass it, which is how the
  multi line announcements stay clean.
- A key whose value is an empty string sends nothing, which is the way to silence a message you
  do not want.
- `{placeholders}` in braces are filled by the plugin. The comment above each key lists the
  ones it supports.
- `help-entries.<subcommand>.description` and `.usage` override what `/gens help` prints for
  that subcommand.
- The `sell.*` section is structured: a sale can show a title, an action bar, a chat line, play
  a sound, or any combination.

To translate, copy this file to `lang/messages_es.yml`, translate the values, and set
`lang: es` in `config.yml`. Keys added by later versions are merged into your translated file
from the English one, and the console tells you how many were added, so a translation never
ends up missing a message.

```yaml
# =============================================================================
#  SnGens - English messages (lang: en)
# =============================================================================
#  Selected by `lang: en` in config.yml. To translate, copy this file to
#  messages_<code>.yml and switch `lang` in config.yml. Missing keys fall back
#  to messages_en.yml so partial translations work.
#
#  Single-line strings get the prefix prepended. Multi-line lists do not -
#  format them however you like. Empty value ("" or []) disables a message.
#
#  Placeholders:
#   {player} {amount} {gen} {cost} {money} {upgradecost} {remaining}
#   {current} {max} {multiplier} {total} {uses} {event} {generator}
#   {successful_upgrade} {amount_short} {command}
#   {amount_formatted} {value_formatted} {previous}
# =============================================================================


# Schema version. Do not edit unless you know why.
# Prefix for every single-line message. List messages ignore it.
prefix: "&#43f072&lSnGens &8| &r"


# -----------------------------------------------------------------------------
# General
# -----------------------------------------------------------------------------
no-permission: "&cYou don't have permission."
player-only: "&cThis command can only be run by a player."
# Shown by %sngens_<currentplaced|max|multiplier>_hide% placeholders when the player has hide-stats on.
hide-stats-placeholder: "&7???"
hide-stats-on: "&aYour gens stats are now &fhidden&a."
hide-stats-off: "&aYour gens stats are now &fvisible&a."
usage-target: "&cUsage: &e/{command} <player>"
usage: "&cUsage: &e/{command}"
invalid-number: "&cInvalid number: &e{input}"
reload: "&aConfiguration reloaded."
generators-paused: "&eGenerator task paused. No drops will be generated until resumed."
generators-resumed: "&aGenerator task resumed."
invalid-user: "&cThere is no user with that name."
invalid-target: "&cThat player has never joined this server."


# -----------------------------------------------------------------------------
# Generator ownership / placement / pickup
# -----------------------------------------------------------------------------
not-owner: "&cYou don't own this generator."
max-gen: "&cYou have reached your generator placement limit."
invalid-world: "&cYou can't place generators in this world."
island-place-only: "&cYou can only place generators on an island you belong to."
wand-island-only: "&cYou can only use wands on an island you belong to."
pickup-broken: "&cYou can't pick up a corrupted generator."
pickup-gens: "&aStored &e{amount} &agenerator(s) in your vault. Run &e/gens recover&a to claim them."
pickup-busy: "&cYour generators are already being picked up - please wait a moment."
force-pickup: "&aStored &e{amount} &agenerator(s) in &e{player}&a's vault."
remove-all: "&aRemoved all of &e{player}&a's generators."

# /gens wipeuser - destructive purge.
# Placeholders: {target} {uuid} {command} {users} {refunds} {generators}
#               {collectors} {hoppers} {upgradeUses} {island}
wipeuser-usage: "&cUsage: &e/{command} <player|uuid> confirm [--include-island]"
wipeuser-warning:
  - "&c&l! WIPEUSER WARNING !"
  - "&7Target: &f{target} &8({uuid})"
  - "&7This will permanently DELETE the target's user row, refund vault,"
  - "&7generators, collectors, hoppers, and top-cache entries."
  - "&7No refunds. No drops. No money back. &cThis cannot be undone."
  - "&7Add &fconfirm &7to proceed. Add &f--include-island &7to also wipe"
  - "&7the island's upgrade-uses row (affects every island member)."
wipeuser-target-not-found: "&cNo player or UUID matches &e{target}&c."
wipeuser-running: "&7Wiping &f{target}&7… &8({uuid})"
wipeuser-success:
  - "&aWiped &f{target}&a."
  - "&7  user rows: &f{users}  &7refund entries: &f{refunds}"
  - "&7  generators: &f{generators}  &7collectors: &f{collectors}  &7hoppers: &f{hoppers}"
  - "&7  upgrade-uses rows: &f{upgradeUses}  &7island: &f{island}"
wipeuser-failure: "&cWipe of &e{target}&c failed - see server log. Some state may be partially purged."
# Fallback when a requirement entry omits its own .message.
requirement-fail-default: "&cYou don't meet the requirement to do this."


# -----------------------------------------------------------------------------
# Generator vault (/gens recover)
# -----------------------------------------------------------------------------
vault-empty: "&cYou have no stored generators."
vault-recover-all: "&aRecovered &e{amount} &agenerator(s) from your vault."
vault-recover-partial: "&aRecovered &e{recovered}&a. &e{remaining} &agenerator(s) still stored - make space and run &e/gens recover&a again."
vault-recover-all-confirm: "&e&lWARNING &7- &fThis will recover &eEVERYTHING&f from your vault. Items that don't fit in your inventory will be &cdropped on the ground&f (and may despawn). Run &e/gens recover all&f again within &e{seconds}s&f to confirm."
vault-recover-all-done: "&aRecovered &e{amount} &agenerator(s) from your vault. &7Overflow dropped on the ground."
vault-join-notice: "&eHey! You have &6{amount} &egenerator(s) stored ({reason}&e). Use &6/gens recover&e to claim them."
vault-island-notice: "&eYour &6{amount} &egenerator(s) were stored in your vault ({reason}&e). Use &6/gens recover&e."
# Storage blocks (Collectors and Infinite Hoppers) are refunded through the same
# vault as generators, so they get their own line next to the generator ones.
vault-recover-storages: "&aRecovered &e{hoppers} &ahopper(s) and &e{collectors} &acollector(s) from your vault."
vault-join-notice-storages: "&eYou also have &6{hoppers} &ehopper(s) and &6{collectors} &ecollector(s) stored ({reason}&e). Use &6/gens recover&e to claim them."
vault-island-notice-storages: "&eYour &6{hoppers} &ehopper(s) and &6{collectors} &ecollector(s) were stored in your vault ({reason}&e). Use &6/gens recover&e."
vault-reason-pickup: "&7you picked them up"
vault-reason-kick: "&7you were kicked from the island"
vault-reason-disband: "&7the island was disbanded"
vault-reason-leave: "&7you left the island"
vault-reason-ban: "&7you were banned from the island"
vault-reason-unknown: "&7for an unknown reason"


# -----------------------------------------------------------------------------
# Generator place / break / upgrade / corrupt-fix (chat notice)
# Sound and particles are hardcoded. Set list to [] to mute the chat line only.
# -----------------------------------------------------------------------------
generator-place:
  - "&aYou placed &f{gen} &7({current}/{max})"
generator-break:
  - "&cYou picked up &f{gen} &7({current}/{max})"
generator-upgrade:
  - "&aUpgraded generator from &f{previous} &ato &f{current}&a."
corrupt-fix:
  - "&aFixed &f{gen} &afor &2$&a{cost}&a."


# -----------------------------------------------------------------------------
# Upgrade
# -----------------------------------------------------------------------------
no-upgrade: "&cThis generator can't be upgraded any further."
upgrade-total-spent: "&7Spent &2$&a{amount_short} &7upgrading &a{successful_upgrade}&7 generators to &a{generator}&7."
upgrade-no-island: "&cYou need an island to use this menu."
upgrade-not-island-owner: "&cOnly the island owner can upgrade generators."
upgrade-no-uses-remaining: "&cNo upgrade uses remaining! Daily: &f{daily}/{daily_max} &c| Bonus: &f{bonus}"
upgrade-uses-reset: "&aYour daily upgrade uses have been reset! &7({amount} uses available)"
upgrade-addupgrades-sender: "&aAdded &e{amount} &aupgrade uses to &e{player}&a's island."
upgrade-addupgrades-target: "&aYou received &e{amount} &aupgrade uses!"
upgrade-addupgrades-sender-infinite: "&aGranted &eunlimited &aupgrade uses to &e{player}&a's island."
upgrade-addupgrades-target-infinite: "&aYou received &eunlimited &aupgrade uses!"
upgrade-feature-disabled: "&cThe upgrade menu is disabled."
upgrade-in-progress: "&cAn upgrade is already running. Wait for it to finish."


# -----------------------------------------------------------------------------
# Repair
# -----------------------------------------------------------------------------
fix-all: "&aAll of your broken generators have been repaired."
fix-empty: "&cYou don't have any corrupted generators."
player-repair: "&aRepaired all of &e{player}&a's broken generators."
gens-repaired: "&aAll of your broken generators have been repaired."


# -----------------------------------------------------------------------------
# Economy
# -----------------------------------------------------------------------------
not-enough-money: "&cNot enough money. &7(&e${money}&7/&e${upgradecost}&7, need &c${remaining}&7)"


# -----------------------------------------------------------------------------
# Admin - give / remove generators
# -----------------------------------------------------------------------------
invalid-gen: "&cNo generator exists with that id."
give-gen: "&aGave &e{amount}x {gen} &ato &e{player}&a."
receive-gen: "&aYou received &e{amount}x {gen}&a."


# -----------------------------------------------------------------------------
# Admin - slot bonus
# -----------------------------------------------------------------------------
add-max: "&aAdded &e{amount} &aslot(s) to &e{player}&a."
max-added: "&aYou can place &e{amount} &amore generators. &7({current}/{max})"
remove-max: "&aRemoved &e{amount} &aslot(s) from &e{player}&a."
max-removed: "&cYour bonus slots were reduced by &e{amount}&c. &7({current}/{max})"
reset-max: "&aReset &e{player}&a's bonus slots."
max-resetted: "&cYour bonus slot pool has been reset. &7({current}/{max})"


# -----------------------------------------------------------------------------
# Admin - sell multiplier
# -----------------------------------------------------------------------------
multiplier-increase: "&aIncreased &e{player}&a's multiplier by &e{multiplier}x &7(total: {total}x)"
increased-multiplier: "&aYour multiplier increased by &e{multiplier}x &7(total: {total}x)"
multiplier-decrease: "&aDecreased &e{player}&a's multiplier by &e{multiplier}x &7(total: {total}x)"
decreased-multiplier: "&cYour multiplier decreased by &e{multiplier}x &7(total: {total}x)"
set-multiplier: "&aSet &e{player}&a's multiplier to &e{multiplier}x&a."
multiplier-set: "&aYour multiplier is now &e{multiplier}x&a."


# -----------------------------------------------------------------------------
# Admin - silent sell handicap (/sngens handicap)
# Negative percent nerfs, positive buffs. Player never receives a message -
# their displayed multiplier stays real, only the actual payout is scaled.
# -----------------------------------------------------------------------------
handicap-set: "&aSet &e{player}&a's handicap to &e{percent}%&a."
handicap-cleared: "&aCleared &e{player}&a's handicap."
handicap-invalid-percent: "&cPercent must be between &e{min}&c and &e{max}&c."
handicap-player-not-found: "&cNo player matches &e{player}&c."
handicap-list-empty: "&7No players currently have a handicap set."
handicap-list-header: "&aActive handicaps &7({count}):"
handicap-list-entry: "&7- &f{player} &7→ &e{percent}%"


# -----------------------------------------------------------------------------
# Shop
# -----------------------------------------------------------------------------
gen-purchase: "&aYou purchased &e{gen}&a."
# Sent from the buy-quantity submenu. {cost} is the TOTAL charged for {amount}.
gen-purchase-bulk: "&aYou purchased &e{amount}x {gen}&a for &2${cost}&a."


# -----------------------------------------------------------------------------
# Sell - title / action-bar / chat each have their own enabled toggle.
# Sound is hardcoded.
# -----------------------------------------------------------------------------
sell:
  title:
    enabled: true
    title: "&cSold &f{amount_formatted} &cItems"
    sub-title: "&2$&a{value_formatted} &7(&e{multiplier}x&7)"
  action-bar:
    enabled: true
    message: "&aSold &f{amount_formatted} &aitems for &2$&a{value_formatted} &7(&e{multiplier}x&7)"
  chat:
    enabled: true
    message:
      - "&7You sold &e{amount_formatted} &7items for &2$&a{value_formatted} &7(&e{multiplier}x&7)"


# -----------------------------------------------------------------------------
# Sellwand
# -----------------------------------------------------------------------------
no-sell: "&cThere are no items to sell here."
sellwand-broke: "&cYour sellwand just broke!"
sellwand-give: "&aGave a sellwand &7({multiplier}x, {uses} uses)&a to &e{player}&a."
sellwand-receive: "&aYou received a sellwand with &e{multiplier}x &aand &e{uses} &auses."
sellwand-give-stacked: "&aAdded &e{added} &auses to &e{player}&a's &e{multiplier}x &asellwand &7(now {uses} uses)&a."
sellwand-receive-stacked: "&aYour &e{multiplier}x &asellwand gained &e{added} &auses &7(now {uses} uses)&a."
sellwand-same-chunk: "&cYour sellwand only works on containers in your current chunk."


# -----------------------------------------------------------------------------
# Events
# -----------------------------------------------------------------------------
invalid-event: "&cThere is no event with that name."
no-event: "&cThere is no active event right now."
event-is-running: "&cAn event is already running. Stop it first with &e/gens stopevent&c."
event-start: "&aStarted the &e{event} &aevent."
event-stop: "&aForced the current event to stop."
corrupt-gens: "&aExecuted the corruption event."


# -----------------------------------------------------------------------------
# Disable-crafting
# -----------------------------------------------------------------------------
disable-crafting-message: "&cYou can't craft with generator items."


# -----------------------------------------------------------------------------
# Corruption - broadcast + per-owner notify
# -----------------------------------------------------------------------------
corruption:
  broadcast:
    - "&7"
    - "  &c&lPower Outages"
    - "  &fDue to recent power outages, &c&n{amount}&f generators"
    - "  &fhave been broken - repair them immediately."
    - "  &7&o(( You will be notified if you were affected ))"
    - "&7"
  notify:
    - "&7"
    - "  &e&lGenerator Notification"
    - "  &fYou have &e&n{amount}&f corrupted generator(s)."
    - "  &fShift + right-click them to repair."
    - "&7"


# -----------------------------------------------------------------------------
# Help - built dynamically from registered subcommands.
# Placeholders: {label} {name} {usage} {description}
# -----------------------------------------------------------------------------
help-header: "&8&m-----------&r &#43f072&lSnGens Help &8&m-----------"
help-line: "&#43f072/{label} {name} {usage} &7- &f{description}"
help-footer: "&8&m----------------------------------------"

# Per-subcommand usage/description overrides for the help menu.
# Keys are the subcommand's internal name (the word after /gens).
# Leave a value empty ("") to fall back to the hardcoded English default.
# Only entries the player has permission for will be displayed.
help-entries:
  # --- Player-facing ---
  shop:
    usage: ""
    description: "Open the generator shop"
  upgrade:
    usage: ""
    description: "Open the generator upgrade menu"
  pickup:
    usage: "[player]"
    description: "Pick up your active generators into your vault"
  recover:
    usage: "[all]"
    description: "Claim generators stored in your vault"
  repair:
    usage: "[player]"
    description: "Repair corrupted generators"
  top:
    usage: ""
    description: "Open the gens-top leaderboard"
  hidestats:
    usage: ""
    description: "Toggle hiding your gens stats placeholders"
  # --- Admin ---
  give:
    usage: "<player> <id> [amount]"
    description: "Give generators to a player"
  sellwand:
    usage: "<player> <multiplier> <uses>"
    description: "Give a sellwand"
  upgradewand:
    usage: "<player> <free|radius> <uses> [radius]"
    description: "Give an upgrade wand"
  buildwand:
    usage: "<player> <distance> <uses>"
    description: "Give a build wand"
  offhand:
    usage: "give <player> <id>"
    description: "Give an offhand item"
  armor:
    usage: "give|giveset <player> <setId> [piece]"
    description: "Give an armor piece or full set"
  wand:
    usage: "<give|pick|fill|clear>"
    description: "Admin region wand for mass-placing generators"
  addmultiplier:
    usage: "<player> <amount>"
    description: "Add a sell multiplier"
  removemultiplier:
    usage: "<player> <amount>"
    description: "Remove a sell multiplier"
  setmultiplier:
    usage: "<player> <amount>"
    description: "Set a player's sell multiplier"
  handicap:
    usage: "<player|list|clear> [percent|player]"
    description: "Silently nerf/buff a player's sell payout"
  addmax:
    usage: "<player> <amount>"
    description: "Add bonus max generator slots"
  removemax:
    usage: "<player> <amount>"
    description: "Remove bonus max generator slots"
  resetmax:
    usage: "<player>"
    description: "Reset bonus max generator slots"
  removegenerators:
    usage: "<player>"
    description: "Remove all of a player's generators"
  addupgrades:
    usage: "<player> <amount>"
    description: "Add bonus upgrade-menu uses to a player's island"
  forcetop:
    usage: ""
    description: "Force a gens-top refresh"
  toplimit:
    usage: "<id|none>"
    description: "Set the gens-top counting cutoff and force a refresh"
  startevent:
    usage: "<event>"
    description: "Start an event (or 'random')"
  stopevent:
    usage: ""
    description: "Stop the active event"
  startcorruption:
    usage: ""
    description: "Force-corrupt all generators"
  reload:
    usage: ""
    description: "Reload the plugin configuration"
  stresstest:
    usage: "[confirm]"
    description: "Fill the current chunk with generators for performance testing"
  debugspawn:
    usage: "<iterations> [chunkRadius]"
    description: "Capture generator spawn telemetry across N task iterations into a JSON dump"
  collector:
    usage: "give|pickup <args>"
    description: "Manage Collectors (admin)"
  hopper:
    usage: "give|pickup <args>"
    description: "Manage Infinite Hoppers (admin)"
  wipeuser:
    usage: "<player|uuid> confirm [--include-island]"
    description: "Destructively purge a player's full state (no refunds, no drops)"

# -----------------------------------------------------------------------------
# Admin - stress test (chunk fill for performance profiling)
# Placeholders: {world} {chunk_x} {chunk_z} {layers} {amount} {gen} {seconds}
#               {x} {y} {z} {elapsed_ms}
# -----------------------------------------------------------------------------
stresstest-confirm:
  - "&c&l! STRESS TEST WARNING !"
  - "&7World: &f{world}  &7Chunk: &f{chunk_x},{chunk_z}"
  - "&7Generator: &f{gen}"
  - "&7Layers: &f{layers}  &7Total gens: &c&l{amount}"
  - "&7Type &f/sngens stresstest confirm &7within &f{seconds}s &7to proceed."
stresstest-no-pending: "&cNo pending stress test. Run &f/sngens stresstest &cfirst."
stresstest-running: "&cA stress test is already running for you."
stresstest-no-gen: "&cNo generator is registered. Configure one before running this command."
stresstest-not-empty: "&cChunk is not empty. First non-air block at &f{x}, {y}, {z}&c. Clear the chunk and retry."
stresstest-checking: "&7Verifying chunk is empty (&e{amount} &7gens queued)..."
stresstest-done: "&aStress test placed &e{amount} &agenerators across &e{layers} &alayers in &e{elapsed_ms}ms&a."



# -----------------------------------------------------------------------------
# Admin Region Wand (/gens wand)
# -----------------------------------------------------------------------------
wand-no-permission: "&cYou don't have permission to use the admin wand."
wand-pos1-set: "&aPosition 1 set: &e{x}, {y}, {z} &7({world})"
wand-pos2-set: "&aPosition 2 set: &e{x}, {y}, {z} &7({world})"
wand-no-selection: "&cSelect both corners with the wand first."
wand-different-worlds: "&cBoth corners must be in the same world."
wand-no-generator: "&cPick a generator with &e/gens wand pick &cfirst."
wand-fill-too-large: "&cSelection is &e{volume} &cblocks (max &e{max}&c)."
wand-fill-started: "&aFilling &e{volume} &ablocks with &f{gen}&a..."
wand-fill-completed: "&aFill complete. Placed &e{placed}&a, skipped &e{skipped} &awith &f{gen}&a."
wand-fill-in-progress: "&cA wand fill is already running. Wait for it to finish."
wand-cleared: "&aSelection cleared."
wand-given: "&aGave admin region wand to &e{player}&a."
wand-generator-picked: "&aGenerator set to &f{gen} &7({gen_id})&a."


# -----------------------------------------------------------------------------
# Gens Top (/gens top)
# Placeholders: {name} {player} {rank} {value} {value_short}
# -----------------------------------------------------------------------------
gens-top:
  empty: "&7No leaderboard data yet - wait for the first refresh."
  force-update: "&aForced gens-top refresh."
  disabled: "&cThe leaderboard is disabled."
  # Count-limit cutoff (/gens toplimit). Placeholders: {id} {material}
  limit-set: "&aGens-top cutoff set to &e{id} &7({material})&a. Generators after it no longer count."
  limit-cleared: "&aGens-top cutoff cleared. Every generator tier counts again."
  limit-invalid: "&cUnknown generator id: &e{id}&c."
  no-ssb2: "&cIsland mode requires a SuperiorSkyblock island plugin - feature disabled."
  # Fallback display when an SSB2 island has no explicit name set.
  # Placeholder: {owner}
  island-name-fallback: "{owner}'s Island"
  # Board label injected as {board} in broadcast-rank-change. Only shown when the
  # team/solo split is active - team changes use board-team,
  # solo changes use board-solo. The single combined board passes an empty label.
  # Include any leading space / formatting here.
  board-team: " &7(Team)"
  board-solo: " &7(Solo)"
  # Single broadcast emitted whenever a watched rank changes hands. When the
  # split is active it fires per board (team and solo separately) so a team
  # island is never announced as overtaking a solo island.
  # Multi-line list - format freely. List values bypass the prefix.
  # Placeholders: {name} {rank} {value} {value_short} {overtaken} {board}
  # {overtaken} is empty if no scope previously held that rank.
  broadcast-rank-change:
    - "&7"
    - "  &6&l▲ GENS TOP{board} ▲"
    - "  &e{name} &7took &6#{rank} &7from &c{overtaken}&7!"
    - "  &7Total value: &2$&a{value_short}"
    - "&7"

# -------- Upgrade wand --------
upgradewand-give: "&aGave &e{type} &aupgrade wand &7({uses} uses{radius_part})&a to &e{player}&a."
upgradewand-receive: "&aYou received a &e{type} &aupgrade wand with &e{uses} &auses{radius_part}."
upgradewand-give-stacked: "&aAdded &e{added} &auses to &e{player}&a's &e{type} &aupgrade wand &7(now {uses} uses{radius_part})&a."
upgradewand-receive-stacked: "&aYour &e{type} &aupgrade wand gained &e{added} &auses &7(now {uses} uses{radius_part})&a."
upgradewand-no-uses: "&cThis wand has no uses left."
upgradewand-no-generator: "&cNo generator at that block."
upgradewand-max-tier: "&cThis generator is already at max tier."
upgradewand-no-gen-radius: "&cNo upgradeable generators in the &e{radius}x{radius} &carea."
upgradewand-not-enough-money: "&cNot enough money. Cost: &e${cost}&c."
upgradewand-success-free: "&aUpgraded &e{previous} &a→ &e{current}&a."
upgradewand-success-radius: "&aUpgraded &e{amount} &agenerators for &2${cost}&a."
upgradewand-broke: "&cYour upgrade wand broke."
upgradewand-bad-radius: "&cRadius must be an odd integer ≥ 1."
upgradewand-bad-type: "&cUnknown wand type. Use &efree &cor &eradius&c."

# -------- Build wand --------
buildwand-give: "&aGave a build wand &7(distance {distance}, {uses} uses)&a to &e{player}&a."
buildwand-receive: "&aYou received a build wand &7(distance {distance}, {uses} uses)&a."
buildwand-give-stacked: "&aAdded &e{added} &auses to &e{player}&a's build wand &7(distance {distance}, now {uses} uses)&a."
buildwand-receive-stacked: "&aYour build wand &7(distance {distance})&a gained &e{added} &auses &7(now {uses} uses)&a."
buildwand-create-failed: "&cCould not build the wand item. Check wands.yml &7(buildwand.item)&c."
buildwand-no-uses: "&cThis wand has no uses left."
buildwand-no-generator: "&cNo generator at that block."
buildwand-need-generator: "&cClick a placed generator first to start a preview."
buildwand-bad-distance: "&cDistance must be a positive integer."
buildwand-obstructed: "&cBlocked at &e{x}, {y}, {z}&c. Clear the path before building."
buildwand-no-price: "&cNo shop price configured for &e{generator}&c. Cannot use the build wand on it."
buildwand-not-enough-money: "&cNot enough money. Cost: &e${cost}&c."
buildwand-slot-limit: "&cNot enough generator slots. &e{current}/{max} &cused, &e{needed} &cmore needed."
buildwand-preview-started: "&aPreview ready: &e{distance} &agens to the &e{direction}&a, cost &2${cost}&a. Click again to confirm."
buildwand-preview-cancelled: "&7Build wand preview cancelled."
buildwand-success: "&aPlaced &e{amount} &agenerators for &2${cost}&a."
buildwand-broke: "&cYour build wand broke."


# -----------------------------------------------------------------------------
# Equipment (offhands + armor sets)
# -----------------------------------------------------------------------------
equipment-not-found: "&cNo equipment with id &e{id}&c."
equipment-create-failed: "&cFailed to build equipment item &e{id}&c. Check console."
offhand-give: "&aGave offhand &e{id} &ato &e{player}&a."
offhand-receive: "&aYou received offhand &e{id}&a."
armor-give: "&aGave &e{id} {piece} &ato &e{player}&a."
armor-receive: "&aYou received &e{id} {piece}&a."
armor-giveset: "&aGave full &e{id} &aset to &e{player}&a."
armor-giveset-receive: "&aYou received the full &e{id} &aset."
armor-bad-piece: "&cInvalid piece &e{piece}&c. Use helmet, chestplate, leggings or boots."


# -----------------------------------------------------------------------------
# Debug spawn dump (/sngens debugspawn)
# -----------------------------------------------------------------------------
debug-spawn-started: "&aDebug spawn capture started: &e{iterations} &aiterations across &e{gens} &agenerators (&7scope: {scope}&a)."
debug-spawn-finished: "&aDebug spawn dump written after &e{ticks} &aticks: &7{path}"
debug-spawn-no-gens: "&cNo generators found in scope."
debug-spawn-already-running: "&cA debug spawn capture is already running. Wait until it finishes."
debug-spawn-not-on-island: "&eNot on a SuperiorSkyblock island; using chunk radius fallback."
debug-spawn-bad-iterations: "&cIterations must be between 1 and {max}."
debug-spawn-bad-radius: "&cChunk radius must be between 1 and {max}."


# -----------------------------------------------------------------------------
# Collector
# -----------------------------------------------------------------------------
collector-place-success: "&aCollector placed. Right-click it to open the menu."
collector-place-chunk-occupied: "&cThis chunk already has a Collector."
collector-place-cap-reached: "&cYou have reached your Collector cap ({current}/{max})."
collector-place-not-island-member: "&cYou can only place Collectors on an island you belong to."
collector-break-not-owner: "&cYou don't own this Collector."
collector-sell-not-owner: "&cYou don't own this Collector."
collector-break-success: "&aCollector removed."
collector-full: "&cThis Collector is full."
collector-given: "&aGave &e{amount} &aCollector(s) to &e{player}&a."
collector-pickup-success: "&aPicked up &e{amount} &aCollector(s) of &e{player}&a."

# Infinite Hopper
hopper-place-success: "&aHopper placed. Right-click to open. Drops in this column will be absorbed."
hopper-place-occupied: "&cThere is already a Hopper at this exact block."
hopper-place-cap-reached: "&cYou have reached your Hopper cap ({current}/{max})."
hopper-place-not-island-member: "&cYou can only place Hoppers on an island you belong to."
hopper-break-not-owner: "&cYou don't own this Hopper."
hopper-sell-not-owner: "&cYou don't own this Hopper."
displayshop-sell-not-owner: "&cYou don't own this Display Shop."
hopper-break-success: "&aHopper removed. Contents dropped on the ground."
hopper-claim-success: "&aClaimed &e{amount} &aitems."
hopper-claim-full: "&cYour inventory is full."
hopper-given: "&aGave &e{amount} &aHopper(s) to &e{player}&a."
hopper-pickup-success: "&aPicked up &e{amount} &aHopper(s) of &e{player}&a."
```
