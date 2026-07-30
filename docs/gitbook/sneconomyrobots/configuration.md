# Configuration

SnEconomyRobots ships with the following YAML files. New keys are auto-merged on boot; your edits
and comments are preserved.

Sections marked `# sn:extensible` hold entries you own. An entry you delete from one of those stays
deleted instead of being re-inserted on the next boot.

## config.yml

```yaml
# ============================================================
#  SnEconomyRobots - configuration
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

# Runtime debug output (also toggleable live via /robots debug).
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
  # Aliases of /robots. Re-read on /robots reload.
  aliases: [robot]

# ------------------------------------------------------------
#  Database. type=sqlite needs nothing else; type=mysql reads host/port/etc.
# ------------------------------------------------------------
database:
  # sqlite or mysql
  type: sqlite
  # MySQL connection (ignored when type is sqlite).
  host: localhost
  port: 3306
  database: sneconomyrobots
  username: root
  password: ""

# ============================================================
#  TIERS
#  The global tier ladder every robot instance is rolled against.
#  min/max bound the PRODUCTION MULTIPLIER rolled once at give-time and
#  stored on the instance. A robot's real accrual for an economy is
#  robots/<id>.yml -> production.<economy>.amount x instance multiplier.
#  Tier ids are the integers used in /robots admin give.
# ============================================================
# sn:extensible
tiers:
  1:
    # Tier label spliced into robot names and lore via {tier_display}.
    display-name: '&7[Tier I]'
    # Rarity word spliced via {tier_rarity}.
    rarity-display: '&7BASIC'
    # Lowest multiplier this tier can roll.
    min: 0.1
    # Highest multiplier this tier can roll.
    max: 0.35
  2:
    display-name: '&a[Tier II]'
    rarity-display: '&aCOMMON'
    min: 0.25
    max: 0.95
  3:
    display-name: '&b[Tier III]'
    rarity-display: '&bRARE'
    min: 0.75
    max: 1.6
  4:
    display-name: '&e[Tier IV]'
    rarity-display: '&eEPIC'
    min: 1.4
    max: 3.2
  5:
    display-name: '&6[Tier V]'
    rarity-display: '&6LEGENDARY'
    min: 2.8
    max: 5.0

# ============================================================
#  PRODUCTION
#  How fast equipped robots accrue. Robots accrue ONLY while their owner
#  is online, and a tick NEVER touches an economy API: it adds to the
#  player's claim bag below. Nothing is ever deposited automatically.
# ============================================================
production:
  # Ticks between payouts for a robot whose type sets no interval-ticks of its own.
  default-interval-ticks: 200
  # Players processed per server tick, so a full server is spread instead of spiking.
  players-per-tick: 20
  # Worlds where robots accrue. Empty = every world.
  active-worlds: []

# ------------------------------------------------------------
#  Claim bag. One bag per player, one balance per economy.
#  The player claims it from the menu or with /robots claim.
# ------------------------------------------------------------
bag:
  # Cap per economy. When reached, robots stop accruing THAT economy until
  # the player claims. 0 = unlimited (players may idle forever).
  max-per-economy: 0
  # Seconds between write-behind flushes of pending balances to the database.
  flush-interval-seconds: 60

# ------------------------------------------------------------
#  Robot boxes. A right-click opens every box of that tier the click
#  picked up, capped by mass-open-limit. Every box TIER rolls from its
#  own independent chance tables. Rewards go straight to storage; a full
#  storage refuses the open and consumes nothing.
# ------------------------------------------------------------
robot-boxes:
  # Master toggle of the box item and its right-click handling.
  enabled: true
  # Maximum boxes one right-click may open. This also bounds how many the
  # click removes from the inventory in the first place.
  mass-open-limit: 64
  # Whether a creative-mode player may open a box. Leave false: a box mints
  # a new robot that produces real currency forever, so a copied box in
  # creative is unlimited recurring income.
  allow-creative: false
  # Extra materials whose right-click must NOT open a box. Every vanilla
  # interactable block (chests, doors, anvils, furnaces...) is already
  # excluded; list only blocks added by other plugins.
  blocked-on: []
  # sn:extensible
  tiers:
    1:
      # Item material; any Bukkit material, or basehead-<base64>.
      material: CHEST
      # Head texture used when material is PLAYER_HEAD. Empty = none.
      head-texture: ""
      # Custom model data of the box item. 0 disables.
      custom-model-data: 0
      # Adds the enchantment glint.
      glow: true
      # Name of the box item.
      display-name: "&7&lRobot Box &8[Tier I]"
      # Lore of the box item.
      lore:
        - "&7Right-click to open your boxes"
        - "&7and receive random robots."
        - " "
        - "&8Opens up to mass-open-limit at once."
      # Which ROBOT TIER this box rolls. Weights; they need not add up to 100.
      tier-chances:
        1: 80.0
        2: 20.0
      # Which ROBOT TYPE this box rolls, by robots/<id>.yml id.
      # Empty = any defined robot with equal weight.
      robot-chances: {}
    2:
      material: CHEST
      head-texture: ""
      custom-model-data: 0
      glow: true
      display-name: "&a&lRobot Box &8[Tier II]"
      lore:
        - "&7Right-click to open your boxes"
        - "&7and receive random robots."
        - " "
        - "&8Opens up to mass-open-limit at once."
      tier-chances:
        1: 40.0
        2: 40.0
        3: 20.0
      robot-chances: {}
    3:
      material: ENDER_CHEST
      head-texture: ""
      custom-model-data: 0
      glow: true
      display-name: "&6&lRobot Box &8[Tier III]"
      lore:
        - "&7Right-click to open your boxes"
        - "&7and receive random robots."
        - " "
        - "&8Opens up to mass-open-limit at once."
      tier-chances:
        2: 25.0
        3: 45.0
        4: 25.0
        5: 5.0
      robot-chances: {}

# ------------------------------------------------------------
#  Merge. Three robots always produce exactly one robot - there is no
#  fail state. The output TIER is rolled from a weighted table built from
#  the inputs, and the merge menu shows that table live as it changes.
#  Merging RESETS every upgrade track and the produced counter to zero.
# ------------------------------------------------------------
merge:
  # Master toggle of the merge menu.
  enabled: true
  # How many robots one merge consumes.
  inputs: 3
  # Weight each input contributes to its own tier and to the tiers above it.
  tier-weights:
    # Weight added to the input's own tier.
    same: 50.0
    # Weight added to one tier above the input.
    plus-1: 32.0
    # Weight added to two tiers above the input.
    plus-2: 14.0
    # Weight added to three tiers above the input.
    plus-3: 4.0
  # How much an input's roll POSITION inside its tier band shifts weight upward.
  # 0 = position ignored (only the tier matters). 1 = a max-roll input moves its
  # whole same-tier weight one tier up. This is what makes a good roll worth keeping.
  position-influence: 0.5
  # Extra push when all inputs share one tier, so matched sets are the better play.
  matching-bonus:
    enabled: true
    # Multiplier applied to the plus-1 weight when every input is the same tier.
    multiplier: 1.75
  # The animation that plays before the output is committed. Inputs are consumed
  # and the result written only when it finishes; closing the menu mid-animation
  # never duplicates or destroys a robot.
  animation:
    enabled: true
    # Total length of the animation.
    duration-ticks: 60
    # Ticks between animation frames.
    frame-interval-ticks: 3
    # Materials cycled through the input slots while the animation runs.
    frame-materials:
      - IRON_INGOT
      - GOLD_INGOT
      - DIAMOND
      - EMERALD
      - NETHER_STAR
    # Sound played on every frame.
    sound-tick: BLOCK_NOTE_BLOCK_HAT
    # Sound played when the output is revealed.
    sound-finish: ENTITY_PLAYER_LEVELUP

# ------------------------------------------------------------
#  Upgrades. Levels belong to the ROBOT INSTANCE, not the player, and are
#  wiped when the robot is merged. Define as many tracks as you want.
#  effect values:
#    INTERVAL_MULTIPLIER - multiplies the robot's interval-ticks (lower = faster)
#    PRODUCTION_CAP      - the robot STOPS once its produced counter reaches value
#    NONE                - no effect of its own; used purely to gate economies
#  A cost is a map of economy id -> amount, so a level may cost several at once.
# ------------------------------------------------------------
upgrades:
  # What the PRODUCTION_CAP counter counts: CYCLES (production ticks) or
  # ECONOMY (the accrued total of limit-count-economy).
  limit-count-mode: CYCLES
  # Economy id counted when limit-count-mode is ECONOMY. Ignored otherwise.
  limit-count-economy: ""
  # Cap applied before any PRODUCTION_CAP level is bought. 0 = uncapped.
  # Keep this BELOW the level-1 value of every PRODUCTION_CAP track, or that first
  # level is a paid downgrade: 0 means uncapped, so buying a cap of 5000 would make
  # the robot stop where it previously ran forever. The cap is a LIFETIME total -
  # claiming does not reset it - and levels only ever rise, so the purchase cannot
  # be undone. The plugin warns on boot if a track breaks this.
  base-production-cap: 2500
  # sn:extensible
  tracks:
    speed:
      # Name shown in the upgrade menu.
      display-name: "&bSpeed"
      # Icon of the track in the upgrade menu.
      icon:
        material: SUGAR
        custom-model-data: 0
      # See the effect list above.
      effect: INTERVAL_MULTIPLIER
      levels:
        1:
          cost:
            vault: 50000.0
          value: 0.95
        2:
          cost:
            vault: 150000.0
          value: 0.90
        3:
          cost:
            vault: 400000.0
          value: 0.82
    limit:
      display-name: "&eCapacity"
      icon:
        material: HOPPER
        custom-model-data: 0
      effect: PRODUCTION_CAP
      levels:
        1:
          cost:
            vault: 75000.0
          value: 5000.0
        2:
          cost:
            vault: 250000.0
          value: 20000.0
        3:
          cost:
            vault: 600000.0
          value: 100000.0
    level:
      display-name: "&dCore Level"
      icon:
        material: NETHER_STAR
        custom-model-data: 0
      # NONE: this track exists only so robots/<id>.yml can gate an economy
      # behind unlocked-at: { track: level, level: N }.
      effect: NONE
      levels:
        5:
          cost:
            vault: 200000.0
        10:
          cost:
            vault: 750000.0

# ============================================================
#  SLOTS
#  Active robot slots. Extra slots are granted by an admin
#  (/robots admin giveslot) and stored per player, NOT by rank.
#  The number of active slot positions declared in guis/main.yml under
#  top-row.active-slots MUST equal max, or the plugin warns on load.
# ============================================================
slots:
  # Slots a player owns on first join.
  default: 1
  # Hard cap of slots any player may ever own.
  max: 6

# ------------------------------------------------------------
#  Storage. Paginated, and NOT unlimited: capacity comes from the player's
#  rank. Capacities do NOT stack - the HIGHEST matching entry wins, so a
#  player holding both vip and vip-plus below gets 150, never 250.
#  Capacity is resolved live from permissions, never stored, so a rank
#  change applies immediately with no data migration.
# ------------------------------------------------------------
storage:
  # Capacity of a player matching no rank entry below.
  default: 50
  # Node checked per rank key: permission-prefix + <key>.
  permission-prefix: "sneconomyrobots.storage."
  # Hard ceiling no rank may exceed. 0 = no ceiling.
  max: 0
  # sn:extensible
  ranks:
    vip: 100
    vip-plus: 150

# ------------------------------------------------------------
#  Cooldowns.
# ------------------------------------------------------------
cooldowns:
  # Seconds between equip/unequip actions on the active slots.
  equip-seconds: 3

# ============================================================
#  ECONOMY
#  An economy id is either an EdTools currency id or the reserved Vault id
#  below. Robots may accrue several at once; see robots/<id>.yml.
# ============================================================
economy:
  # Reserved id that routes to the Vault economy instead of EdTools.
  vault-economy-id: vault
  # Passed to EdTools addCurrency when a bag is CLAIMED. true lets the player's
  # active EdTools boosters multiply the claim. Note the timing: income accrues
  # on the tick but is multiplied at claim, so a player can bank all day and
  # claim under a booster. Set false to price the income at accrual value.
  affect-boosters: true

# ------------------------------------------------------------
#  Economy display names. AUTO-MANAGED: on boot and on /robots reload the
#  plugin writes an entry for every EdTools currency it finds (plus the
#  Vault id) and REMOVES entries whose currency no longer exists. Edit the
#  values freely - your edits are kept, only missing and stale keys move.
#  Every name shown to a player (bag lore, menus, merge info, messages,
#  placeholders) resolves through this map.
# ------------------------------------------------------------
# sn:extensible
boost-display-names: {}

# ============================================================
#  NOTIFICATIONS
# ============================================================
notifications:
  # Periodic reminder that the bag has income waiting.
  pending:
    enabled: true
    # ACTIONBAR, CHAT or NONE.
    mode: ACTIONBAR
    # Seconds between reminders. Ignored when mode is NONE.
    interval-seconds: 300
```

## robots/example-robot.yml

Each file in `robots/` defines one robot type, and the file name is its id, lowercased. Copy this
file to `robots/<id>.yml` to add a robot. It is seeded only when the folder holds no robot file at
all, so deleting it is permanent once you have your own.

```yaml
# ============================================================
#  SnEconomyRobots - robot type definition
#  Seeded once and never merged: SnEconomyRobots does not rewrite this file.
#  Copy it to robots/<id>.yml and edit freely.
#
#  THE FILE NAME IS THE ROBOT ID. This file defines the robot "example-robot",
#  which is what robot-boxes.tiers.<n>.robot-chances in config.yml and
#  /robots admin give reference. Renaming the file renames the robot.
#
#  Every field below is optional: a field you delete falls back to its default.
# ============================================================

# ------------------------------------------------------------
#  PLACEHOLDERS
#  Usable in display-name and in description. They resolve per ROBOT INSTANCE,
#  so two copies of this robot render different values.
#    {tier_display}  tiers.<n>.display-name of the instance's tier
#    {tier_rarity}   tiers.<n>.rarity-display of the instance's tier
#    {multiplier}    production multiplier rolled at give-time
#    {production}    what the instance accrues per payout, per economy
#    {interval}      seconds between payouts, after upgrade multipliers
#    {produced}      current value of the production counter
#    {limit}         production cap in force, or the status word for no cap
# ------------------------------------------------------------

# Name shown on the robot item and in messages.
display-name: "{tier_display} &f&lExample Robot"

# Lore lines shown on the robot item.
description:
  - "&7A demo robot. Copy this file to define your own."
  - " "
  - "&7Rarity: {tier_rarity}"
  - "&7Multiplier: &f{multiplier}x"
  - "&7Produces: &f{production}"
  - "&7Every: &f{interval}s"
  - "&7Produced: &f{produced}&7/&f{limit}"

# ------------------------------------------------------------
#  ITEM
# ------------------------------------------------------------

# Item material; any Bukkit material, or basehead-<base64>.
material: PLAYER_HEAD
# Head texture value used when material is PLAYER_HEAD. Empty = none.
head-texture: ""
# Custom model data of the robot item. 0 disables.
custom-model-data: 0
# Adds the enchantment glint.
glow: false

# ------------------------------------------------------------
#  PRODUCTION
# ------------------------------------------------------------

# Ticks between payouts of this robot. 0 = production.default-interval-ticks.
# An INTERVAL_MULTIPLIER upgrade track multiplies whichever value applies.
interval-ticks: 0

# Economies this robot accrues, one entry per economy. The key is an EdTools
# currency id, or the reserved economy.vault-economy-id from config.yml.
# A payout adds amount x the instance multiplier to the owner's claim bag; it
# never calls an economy API, so nothing is deposited until the player claims.
production:
  vault:
    # Base amount per payout, before the instance multiplier.
    amount: 100.0
    # Gates this economy behind an upgrade level of the instance.
    unlocked-at:
      # Track id from upgrades.tracks. Empty = always unlocked.
      track: ""
      # Level of that track the instance must own. Ignored when track is empty.
      level: 0
  # A second economy, gated until the instance buys Core Level 5. Uncomment it
  # after replacing "gems" with a currency id your EdTools install defines.
  # gems:
  #   amount: 2.0
  #   unlocked-at:
  #     track: level
  #     level: 5

# ------------------------------------------------------------
#  HOOKS
#  Commands run from the CONSOLE, one per line, without a leading slash.
#  {player} is the owner's name and {robot} this robot's display-name;
#  PlaceholderAPI tokens work too when PAPI is installed.
#  Example: ["broadcast &e{player} equipped {robot}&e!"]
# ------------------------------------------------------------

# Run when this robot is equipped into an active slot.
commands-on-equip: []
# Run when this robot leaves an active slot.
commands-on-unequip: []
```

## Menus

Three menu layouts ship under `guis/`, one file per menu. Each declares a `layout:` mask, its
templates and its clickable items, so the arrangement is yours to change. Remove a button by
deleting its character from the mask.

| File | Menu |
|------|------|
| `guis/main.yml` | Active slots, paged storage and the claim bag |
| `guis/merge.yml` | Merge inputs, the chance table and the run button |
| `guis/upgrades.yml` | Upgrade tracks of one robot |

## Language

Two locales ship: `lang/messages_en.yml` and `lang/messages_es.yml`. Pick one with the `lang` key at
the top of `config.yml`. Both files carry the same key set, so a value you restyle in one has a twin
in the other.

State words such as locked, stopped, full and empty live in the top-level `status:` section. They
are spliced into `{status}` placeholders across menus, item lore and the placeholders, so restyling
one there moves every surface that shows it.
