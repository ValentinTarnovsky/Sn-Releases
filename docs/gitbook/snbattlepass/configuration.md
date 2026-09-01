# Configuration

SnBattlePass ships with the following YAML files. New keys are auto-merged on boot; your edits and comments are preserved.

Two files are yours to curate: the `pool` in `challenges.yml` and the `tiers` in `rewards.yml`. Entries you delete from them stay deleted, so the merge never revives a challenge or a tier you removed.

## config.yml

```yaml
# ============================================================
#  SnBattlePass - configuration
#  Managed by SnLib: new keys are auto-merged on boot; your values and
#  comments are preserved.
#  Set update-configs: false to freeze this file (SnLib only warns about
#  missing keys instead of inserting them).
#  Sections marked "# sn:extensible" are yours: entries you delete there
#  stay deleted.
# ============================================================

# Active language code; loads lang/messages_<code>.yml (falls back to en).
lang: en

# Master switch of the SnLib auto-updater for this plugin's managed files.
update-configs: true

# Runtime debug output (also toggleable live via /battlepass debug).
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
  # Aliases of /battlepass. Re-read on /battlepass reload.
  aliases: [bp, pass]

# ------------------------------------------------------------
#  Database. type=sqlite needs nothing else; type=mysql reads host/port/etc.
#  Every key here needs a RESTART, not a reload: the connection pool is
#  built once when the plugin loads.
# ------------------------------------------------------------
database:
  # sqlite or mysql
  type: sqlite
  # MySQL connection (ignored when type is sqlite).
  host: localhost
  port: 3306
  database: snbattlepass
  username: root
  password: ""

# ------------------------------------------------------------
#  Presentation. gui opens the menu; chat prints the text view.
# ------------------------------------------------------------
presentation:
  # Bare /battlepass: gui opens the main pass menu, chat prints the generated help.
  main: gui

# ------------------------------------------------------------
#  Passes. Everyone starts on Free; Gold and Diamond are the paid
#  upgrades, and Diamond covers everything Gold covers.
#  A pass can also be granted by permission (snbattlepass.pass.gold
#  and .diamond), which lasts only as long as the permission does.
# ------------------------------------------------------------
passes:
  # false stops the plugin selling the upgrade at all: the menu says so instead
  # of opening the confirmation, and a purchase is refused however it was reached.
  # Use it when the pass is sold from a web store.
  allow-purchase: true
  free:
    # Hours a challenge slot waits before it can be rolled again, for a Free player.
    challenge-cooldown-hours: 24
  gold:
    # Charged through economy.charge-command when a player buys Gold.
    price: 1000.0
    # Hours a challenge slot waits before it can be rolled again, for a Gold player.
    challenge-cooldown-hours: 20
  diamond:
    # Charged through economy.charge-command when a player buys Diamond.
    price: 2500.0
    # Hours a challenge slot waits before it can be rolled again, for a Diamond player.
    challenge-cooldown-hours: 16

# ------------------------------------------------------------
#  Reward cadence. Both lists loop over the tiers, so the entry at
#  position N also applies to tier N + length, N + 2*length, and so on.
#  What each tier actually gives is in rewards.yml.
# ------------------------------------------------------------
reward-patterns:
  # Which tiers carry a free reward. A tier switched off here shows no free
  # reward and cannot be claimed, whatever rewards.yml lists for it.
  free: [true, false]
  # Pass each tier's premium reward asks for. GOLD or DIAMOND; a tier that pins
  # its own premium pass in rewards.yml ignores this.
  premium: [GOLD, GOLD, DIAMOND]

# ------------------------------------------------------------
#  Progression. The curve fixes what every level costs and how
#  far the reward track goes; the passive block is what farming
#  a single block is worth.
# ------------------------------------------------------------
xp:
  curve:
    # XP required to leave level 0.
    base: 5000.0
    # XP added to the requirement of each further level: base + increment * level.
    increment: 7500.0
  # Levels a player can climb, and the number of reward tiers. Hard cap 54.
  max-tier: 54
  passive:
    # XP granted per block farmed with a tool that has no override below.
    xp-per-block: 0.5
    # XP per farmed block for specific EdTools tool ids; anything not listed
    # earns xp-per-block.
    # sn:extensible
    tool-overrides:
      example_drill: 1.5
      example_harvester: 2.0
    # Queued farm events applied per tick; the remainder waits for the next tick.
    # Floored at 100: the farm queue is unbounded, so a cap below what your
    # farmers produce would grow it without limit.
    drain-max-per-tick: 2000

# ------------------------------------------------------------
#  Challenges. What a player can be asked to do is in challenges.yml;
#  these two decide how many they juggle at once and how promptly a
#  finished deadline is noticed. The cooldown after a challenge closes
#  is per pass, under "passes" above.
# ------------------------------------------------------------
challenges:
  # Challenge slots a player can roll, 1 to 5. Storage always holds 5, so
  # lowering this leaves the higher slots empty in the menu, in the commands and
  # in the placeholders alike. A challenge already running in a removed slot
  # freezes (no progress, no payout); its deadline keeps counting in real time,
  # so raising the count back resumes it or expires it if the time ran out.
  # Applied on /battlepass reload.
  slots: 5
  # Ticks between the passes that close expired challenges. A challenge stops
  # counting the moment it expires whatever this says, so it only decides how
  # quickly the menu and the cooldown catch up. Applied on /battlepass reload.
  expiry-check-interval-ticks: 20

# ------------------------------------------------------------
#  Playtime. One minute is credited per minute of activity; moving
#  across blocks, interacting or chatting all count as active.
# ------------------------------------------------------------
playtime:
  # Seconds without any activity before a player counts as AFK and stops
  # accruing playtime. Applied on /battlepass reload.
  afk-window-seconds: 300

# ------------------------------------------------------------
#  Persistence.
# ------------------------------------------------------------
data:
  # Seconds between writes of the players whose progress changed. Progress is also
  # written when a player disconnects and when the server stops, so this only caps
  # how much a crash can cost. Applied on /battlepass reload.
  flush-interval-seconds: 30

# ------------------------------------------------------------
#  Economy. The plugin never talks to an economy plugin directly:
#  it READS a balance through a placeholder and CHARGES by running a
#  console command, so any currency offering those two works.
# ------------------------------------------------------------
economy:
  # Placeholder expanded per buyer to read their balance. Needs PlaceholderAPI.
  # Use one that returns EXACT digits. An abbreviated placeholder ("1.5M") is
  # read as the rounded-up number and can pass the balance check for a player
  # who cannot actually pay.
  balance-placeholder: "%vault_eco_balance%"
  # Console command charging a purchase. {player} is the buyer, {price} the amount.
  charge-command: "eco take {player} {price}"
  # true refuses the purchase when the balance cannot be read at all - a missing
  # PlaceholderAPI, a mistyped placeholder. false charges anyway and trusts the
  # charge command to reject a buyer who cannot pay, which is the setting for a
  # currency that has no readable placeholder.
  deny-on-unreadable-balance: true
  # Literal pieces removed from the balance before it is read as a number, so a
  # placeholder returning "$1,234.50" still parses.
  strip-characters: [",", " ", "$"]

# ------------------------------------------------------------
#  Integrations. Optional plugins the battle pass listens to; each is
#  detected at boot and simply skipped when absent.
# ------------------------------------------------------------
integrations:
  rival-pets:
    # Reads the equipped pet's buff and boosts PASSIVE farming XP by it (a +20%
    # buff pays 1.2x per block). Challenge rewards are never boosted. Needs the
    # RivalPets plugin. Both keys here need a RESTART, not a reload: the boost
    # provider is built once at boot.
    enabled: true
    # RivalPets buff name read as the farming boost.
    buff-name: battlepass-farm
  sn-pets:
    # Reads the equipped SnPets pets whose id starts with the prefix below and
    # boosts PASSIVE farming XP by their summed boost. Challenge rewards are never
    # boosted. Needs the SnPets plugin.
    #
    # Per pet:  min(level x percent-per-level,
    #               max-percent-base + (rarity - 1) x max-percent-per-tier)
    # where the rarity is the number the pet id ends with (Pase_3 is rarity 3, so
    # its ceiling is 20 + 2 x 2 = 24%). Each pet is capped on its own and the
    # results are added, so a level-100 Pase_3 pays 24% -> x1.24.
    #
    # If rival-pets and sn-pets are BOTH active, the two multipliers multiply.
    # ALL the keys here need a RESTART, not a reload: the boost provider is built
    # once at boot.
    enabled: true
    # Pet ids starting with this prefix count as battle-pass pets. Case-sensitive.
    pet-id-prefix: "Pase_"
    # Percentage points each pet level is worth.
    percent-per-level: 0.4
    # Ceiling of a rarity-1 pet, in percentage points.
    max-percent-base: 20.0
    # Percentage points the ceiling widens by for each rarity above 1.
    max-percent-per-tier: 2.0

# ------------------------------------------------------------
#  Feedback. Timing of the tier-up and challenge-complete title;
#  the text itself is in lang/messages_en.yml under notifications.
# ------------------------------------------------------------
notifications:
  title:
    # Ticks the title takes to fade in.
    fade-in-ticks: 10
    # Ticks the title stays on screen.
    stay-ticks: 40
    # Ticks the title takes to fade out.
    fade-out-ticks: 10
```

## challenges.yml

```yaml
# ============================================================
#  SnBattlePass - challenge pool
#  Managed by SnLib: new keys are auto-merged on boot; your values and
#  comments are preserved.
#  The pool below is marked "# sn:extensible": entries you delete stay
#  deleted and entries you add are never touched.
#  How many slots a player has, and how often expiries are swept, live in
#  config.yml under "challenges".
# ------------------------------------------------------------
#  Each entry under "pool" is one challenge the roller can hand out:
#    enabled         false keeps the entry in the file but out of the game.
#    type            CRATE_OPEN, GEN_UPGRADE, GEN_REPAIR, EDTOOLS_FARM,
#                    PLAYTIME, PET_OPEN, OFFHAND_OPEN or ENVOY_OPEN. The short
#                    form works too: keys, gen-upgrade, gen-repair, farm,
#                    playtime, pets, offhands, envoys.
#    weight          relative roll weight; equal weights give a uniform roll.
#                    0 keeps the entry out of random rolls while /battlepass
#                    givechallenge can still hand it out.
#    filter          "all" counts every signal of that type, or name one id: a
#                    crate, an EdTools tool, a SnPets box, an offhand box, a
#                    generator or a SnEnvoys reward. Ids are matched ignoring
#                    case. A typo shows up as a challenge that never advances,
#                    so turn debug on to print the id a source really reports.
#    target-amounts  the goals to pick between, one per roll. An entry is a
#                    fixed number (50) or an inclusive range (1-100) that rolls
#                    a value inside it, so [1-100, 125, 150] gives 1..100, 125
#                    or 150 with equal odds. Every goal must be at least 1; an
#                    entry allowing 0 is skipped with a warning. PLAYTIME counts
#                    MINUTES; every other type counts events.
#    duration        how long the player has, e.g. 24h, 12h, 1d.
#    reward-percent  XP paid on completion, as a percentage of the requirement
#                    of the level the player is on at that moment.
#    icon            Material shown for this challenge in the menu.
#    display-name    coloured name shown to the player.
#
#  ENVOY_OPEN counts SnEnvoys claims and only advances while SnEnvoys is
#  installed; disable that entry on a server without it.
# ============================================================

# The challenges a player can roll. One per type at equal weight, so the shipped
# pool rolls uniformly across all eight.
# sn:extensible
pool:
  crate-open:
    enabled: true
    type: CRATE_OPEN
    weight: 10
    filter: all
    target-amounts: [25, 50, 100]
    duration: 24h
    reward-percent: 50.0
    icon: TRIPWIRE_HOOK
    display-name: "&#FFBE5DOpen Crates"

  gen-upgrade:
    enabled: true
    type: GEN_UPGRADE
    weight: 10
    filter: all
    target-amounts: [10, 25, 50]
    duration: 24h
    reward-percent: 40.0
    icon: PISTON
    display-name: "&#FFBE5DUpgrade Generators"

  gen-repair:
    enabled: true
    type: GEN_REPAIR
    weight: 10
    filter: all
    target-amounts: [10, 20, 40]
    duration: 24h
    reward-percent: 40.0
    icon: ANVIL
    display-name: "&#FFBE5DRepair Generators"

  edtools-farm:
    enabled: true
    type: EDTOOLS_FARM
    weight: 10
    filter: all
    target-amounts: [5000, 10000, 25000]
    duration: 24h
    reward-percent: 75.0
    icon: DIAMOND_HOE
    display-name: "&#37AAFFFarm Blocks"

  playtime:
    enabled: true
    type: PLAYTIME
    weight: 10
    filter: all
    target-amounts: [30, 60, 120]
    duration: 24h
    reward-percent: 60.0
    icon: CLOCK
    display-name: "&#37AAFFPlay Time"

  pet-open:
    enabled: true
    type: PET_OPEN
    weight: 10
    filter: all
    target-amounts: [10, 25, 50]
    duration: 24h
    reward-percent: 50.0
    icon: BONE
    display-name: "&#8354f2Open Pet Boxes"

  offhand-open:
    enabled: true
    type: OFFHAND_OPEN
    weight: 10
    filter: all
    target-amounts: [10, 25, 50]
    duration: 24h
    reward-percent: 50.0
    icon: SHIELD
    display-name: "&#8354f2Open Offhand Boxes"

  envoy-open:
    enabled: true
    type: ENVOY_OPEN
    weight: 10
    filter: all
    target-amounts: [5, 10, 20]
    duration: 24h
    reward-percent: 50.0
    icon: CHEST
    display-name: "&#55FFAAOpen Envoys"
```

## rewards.yml

```yaml
# ============================================================
#  SnBattlePass - reward track
#  Managed by SnLib: new keys are auto-merged on boot; your values and
#  comments are preserved.
#
#  Every tier has two lanes:
#    free    - claimable once the tier is reached, but ONLY on tiers the
#              config.yml 'reward-patterns.free' cadence switches on. On a
#              tier the cadence switches off the free lane is empty and
#              unclaimable, whatever you list here.
#    premium - additionally needs a pass. Which pass comes from the looping
#              'reward-patterns.premium' in config.yml, unless the tier pins
#              one with its own 'pass' key (see tier 3 below).
#                pass: GOLD    -> Gold or Diamond may claim it
#                pass: DIAMOND -> Diamond only
#
#  Tiers are written 1-based, the way players read them in the menu. Any
#  tier you leave out uses the 'default' block, so only write the ones that
#  differ. Commands run from the console; {player} becomes the receiver.
#
#  free-display / premium-display are optional friendly lore for the menu.
#  A lane with no display list shows its raw commands instead.
# ============================================================

# ------------------------------------------------------------
#  Applied to every tier with no entry of its own below.
# ------------------------------------------------------------
default:
  # Console commands run when the free lane is claimed.
  free:
    - "eco give {player} 500"
  # Menu lore of the free lane; set it to [] to show the commands above instead.
  # (Set, not delete: this section is managed, so a deleted key is re-inserted
  # on the next boot while an emptied list is yours and stays.)
  free-display:
    - " &8- &#FFD700$500 Coins"
  premium:
    # No 'pass' key here, so every tier using this block follows the cadence
    # in config.yml reward-patterns.premium.
    # Console commands run when the premium lane is claimed.
    commands:
      - "eco give {player} 1000"
  # Menu lore of the premium lane; set it to [] to show the commands above
  # instead (same managed-file rule as free-display).
  premium-display:
    - " &8- &#FFD700$1,000 Coins"

# ------------------------------------------------------------
#  Per-tier rewards, keyed by the tier number the player sees.
#  The three below are examples: rename, replace or delete them.
# ------------------------------------------------------------
# sn:extensible
tiers:
  "1":
    free:
      - "eco give {player} 1000"
    free-display:
      - " &8- &#FFD700$1,000 Coins"
    premium:
      commands:
        - "eco give {player} 2000"
    premium-display:
      - " &8- &#FFD700$2,000 Coins"
  "2":
    free:
      - "eco give {player} 1500"
    free-display:
      - " &8- &#FFD700$1,500 Coins"
    premium:
      commands:
        - "eco give {player} 2500"
        - "give {player} diamond 4"
    premium-display:
      - " &8- &#FFD700$2,500 Coins"
      - " &8- &f4x Diamond"
  "3":
    free:
      - "eco give {player} 2000"
      - "give {player} emerald 8"
    free-display:
      - " &8- &#FFD700$2,000 Coins"
      - " &8- &f8x Emerald"
    premium:
      # Pins this tier to Diamond, ignoring the cadence.
      pass: DIAMOND
      commands:
        - "eco give {player} 5000"
        - "give {player} netherite_ingot 1"
    premium-display:
      - " &8- &#FFD700$5,000 Coins"
      - " &8- &f1x Netherite Ingot"
```

## lang/messages_en.yml

Every player-visible string lives here: messages, notification titles, and the `status:` labels the menus use for states such as locked, claimed or on cooldown. One language file ships; add more by copying it and pointing `lang.code` at the new file.

## guis/

Three files, one per menu: `main.yml`, `challenges.yml` and `confirm-purchase.yml`.

Each menu is a title, a row count, a `layout:` character grid and an `items:` map keyed by the letters in that grid. To move a button, move its letter. To remove one, delete its letter from the layout: a key the layout does not use is hidden, and because the layout is a list value rather than a key, the auto-merge never puts it back.

> Removed in 2.1.0: menus used to ship a second `-alt.yml` layout each, picked per viewer by `presentation.alternate-gui-placeholder`. Both the key and the alternate files are gone, and every player now sees the one layout. Upgrading from 2.0.0 leaves the old key and the three `*-alt.yml` files behind in your data folder: nothing reads them, and you can delete them.
