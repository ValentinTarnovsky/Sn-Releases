# Configuration

SnElementalGems ships with the following YAML files. New keys are auto-merged on boot; your edits and comments are preserved.

## config.yml

```yaml
# ============================================================
#  SnElementalGems - configuration
#  Managed by SnLib: new keys are auto-merged on boot; your values and
#  comments are preserved. Do NOT add a config-version key (retired).
#  Set update-configs: false to freeze this file.
# ============================================================

# Active language code; loads lang/messages_<code>.yml (falls back to en).
lang: en

# Master switch of the SnLib auto-updater for this plugin's managed files.
update-configs: true

# Runtime debug output (also toggleable live via /gems debug).
debug:
  enabled: false

# ------------------------------------------------------------
#  Main command.
# ------------------------------------------------------------
command:
  # Aliases of /gems. Re-read on /gems reload.
  aliases: [gem, eg]

# ------------------------------------------------------------
#  Database. type=sqlite needs nothing else; type=mysql reads host/port/etc.
# ------------------------------------------------------------
database:
  # sqlite or mysql
  type: sqlite
  # MySQL connection (ignored when type is sqlite).
  host: localhost
  port: 3306
  database: snelementalgems
  username: root
  password: ""

# ------------------------------------------------------------
#  Presentation. Controls what the bare /gems command does.
# ------------------------------------------------------------
presentation:
  # Bare /gems: gui opens the shop, chat prints the generated help.
  main: gui

# ------------------------------------------------------------
#  Shop
# ------------------------------------------------------------
shop:
  # Enable /gems shop and the bare-command shop GUI. false blocks it with
  # messages.shop-disabled.
  enabled: true
  # Label shown in place of an upgrade's price in the upgrades menu when the
  # upgrade is already at its maximum level.
  max-level-label: "&aMAX"

# ------------------------------------------------------------
#  Economy
# ------------------------------------------------------------
economy:
  # Gems a brand-new player starts with.
  starting-balance: 0
  # Show 2 decimals in formatted balances (false = whole numbers).
  decimals: false
  # Enable /gems withdraw. false blocks it with messages.withdraw-disabled.
  withdraw-enabled: true
  # Max gems withdrawable in a single /gems withdraw (0 = no limit).
  withdraw-limit: 2304
  # Cap on a single /gems give amount to avoid item floods (0 = no limit).
  max-give: 10000
  # Anti-abuse: hard cap a single drop can ever award (0 = no cap). Prevents the
  # legacy name-tag inflation exploit even with a misconfigured stacker.
  max-drop-amount: 100000

# ------------------------------------------------------------
#  Gem item (the physical, redeemable gem). Definition lives in items.yml;
#  these are the behavioural toggles.
# ------------------------------------------------------------
gem-item:
  # Allow right-click / place to redeem a gem item into balance.
  can-redeem: true
  # Block materials that must never redeem a gem on right-click (1.21 names).
  redeem-blocked-on: [CHEST, OAK_SIGN, ENDER_CHEST, BARREL]
  # Allow gem items inside villager trade menus.
  allow-villager-trades: false
  # Allow crafting with gem items.
  allow-crafting: false

# ------------------------------------------------------------
#  Drop notifications
# ------------------------------------------------------------
notifications:
  # Send a message when a player receives gems from a drop.
  gem-received: true
  # Route the gem-received message to the action bar instead of chat.
  action-bar: true

# ------------------------------------------------------------
#  Sounds. Valid 1.21 Sound enum names. Empty string = silent.
# ------------------------------------------------------------
sounds:
  redeem:         { sound: ENTITY_EXPERIENCE_ORB_PICKUP, volume: 1.0, pitch: 1.0 }
  withdraw:       { sound: ENTITY_EXPERIENCE_ORB_PICKUP, volume: 1.0, pitch: 1.0 }
  gem-dropped:    { sound: ENTITY_EXPERIENCE_ORB_PICKUP, volume: 1.0, pitch: 1.2 }
  open-shop:      { sound: BLOCK_CHEST_OPEN,             volume: 1.0, pitch: 1.0 }
  open-upgrades:  { sound: BLOCK_CHEST_OPEN,             volume: 1.0, pitch: 1.0 }
  purchase:       { sound: ENTITY_PLAYER_LEVELUP,        volume: 1.0, pitch: 1.0 }
  upgrade:        { sound: ENTITY_PLAYER_LEVELUP,        volume: 1.0, pitch: 1.2 }

# ------------------------------------------------------------
#  Vault economy provider
# ------------------------------------------------------------
vault:
  # Register gems as the server's Vault economy.
  hook: false
  # Register even if another Vault economy already exists, displacing it.
  override-existing: false

# ------------------------------------------------------------
#  Leaderboard
# ------------------------------------------------------------
leaderboard:
  # How many entries /gems top and the top-N placeholders expose.
  size: 10
  # How often (seconds) the cached leaderboard refreshes.
  refresh-seconds: 300
```

## droprates.yml

```yaml
# ============================================================
#  SnElementalGems - drop rates
#  Managed by SnLib: new keys are auto-merged on boot; your values and comments
#  are preserved.
#
#  Gems drop from four sources: killing entities, breaking blocks, fishing, and
#  automated (non-player-kill) deaths such as mob grinders. Every TYPE below uses
#  1.21 Material / EntityType names. Legacy 1.8 names (SUGAR_CANE_BLOCK, CROPS,
#  LOG-1, RAW_FISH) never match on 1.21 and silently drop nothing.
#
#  Per TYPE:
#    chance         0-100 roll to drop at all.
#    amount         gems awarded (the ceiling when random-amount is true).
#    random-amount  true  -> a random 1..amount; false -> exactly amount.
#    message        sent to the receiver on a drop; {amount} is filled and the
#                   plugin prefix is applied automatically. Empty = no message.
#  Automated TYPEs additionally read:
#    death-reasons    the EntityDamageEvent causes that count as an automated kill.
#    support-stacking honor a real stacker plugin's stack size (NEVER a name tag).
#
#  Use '*' as a TYPE inside a section to define a fallback for every entry in
#  that section; a specific TYPE always overrides the wildcard.
# ============================================================

# Add dropped gems straight to the balance instead of spawning a physical item.
auto-redeem: true

# Worlds where NO gem drops occur, from any source.
disabled-worlds:
  - world_nether

# Only enable if entity-kill drops fail on your server build; adds a second,
# cancellation-checked death path. Default false (the single kill path is enough).
mob-death-patch: false

# Whether blocks broken with a Silk Touch tool still drop gems. This toggle is
# owned HERE (the drop engine reads it live from this file, re-read on reload).
silk-touch-drops: false

# How long (seconds) a player-placed block is remembered so breaking it yields no
# gems (anti place-break farm). Entries expire individually.
place-break-tracking-seconds: 600

# Permission a player must hold to receive drops from each source.
permissions:
  entity: snelementalgems.gemdrop.entity
  block: snelementalgems.gemdrop.block
  fish: snelementalgems.gemdrop.fish

# ------------------------------------------------------------
#  Entity kills: a player kills a mob. Keys are EntityType names (1.21).
# ------------------------------------------------------------
entity-drops:
  ZOMBIE:
    chance: 5
    amount: 1
    random-amount: true
    message: "&a{amount}x gem(s) dropped from killing a zombie."
  SKELETON:
    chance: 5
    amount: 1
    random-amount: true
    message: "&a{amount}x gem(s) dropped from killing a skeleton."
  BLAZE:
    chance: 10
    amount: 1
    random-amount: true
    message: "&a{amount}x gem(s) dropped from killing a blaze."

# ------------------------------------------------------------
#  Block breaks. Keys are Material names (1.21).
# ------------------------------------------------------------
block-drops:
  EMERALD_ORE:
    chance: 30
    amount: 1
    random-amount: true
    message: "&a{amount}x gem(s) dropped from mining emerald ore."
  DIAMOND_ORE:
    chance: 20
    amount: 1
    random-amount: true
    message: "&a{amount}x gem(s) dropped from mining diamond ore."
  SUGAR_CANE:
    chance: 10
    amount: 1
    random-amount: true
    message: "&a{amount}x gem(s) dropped from farming sugar cane."
  WHEAT:
    chance: 10
    amount: 1
    random-amount: true
    message: "&a{amount}x gem(s) dropped from farming wheat."
  SPRUCE_LOG:
    chance: 5
    amount: 1
    random-amount: true
    message: "&a{amount}x gem(s) dropped from chopping spruce log."

# ------------------------------------------------------------
#  Fishing catches. Keys are the Material of the caught item (1.21).
# ------------------------------------------------------------
fish-drops:
  COD:
    chance: 50
    amount: 1
    random-amount: true
    message: "&a{amount}x gem(s) dropped from fishing."
  SALMON:
    chance: 50
    amount: 1
    random-amount: true
    message: "&a{amount}x gem(s) dropped from fishing."
  PUFFERFISH:
    chance: 50
    amount: 1
    random-amount: true
    message: "&a{amount}x gem(s) dropped from fishing."
  TROPICAL_FISH:
    chance: 50
    amount: 1
    random-amount: true
    message: "&a{amount}x gem(s) dropped from fishing."

# ------------------------------------------------------------
#  Automated deaths (mob grinders): a mob dies WITHOUT a player kill, by one of
#  the listed death-reasons. Keys are EntityType names (1.21).
# ------------------------------------------------------------
automated-drops:
  VILLAGER:
    chance: 50
    amount: 1
    random-amount: true
    support-stacking: true
    message: "&a{amount}x gem(s) dropped from an automated villager death."
    death-reasons:
      - FALL
      - LAVA
  IRON_GOLEM:
    chance: 50
    amount: 1
    random-amount: true
    support-stacking: true
    message: "&a{amount}x gem(s) dropped from an automated iron golem death."
    death-reasons:
      - FALL
      - LAVA
```

## upgrades.yml

```yaml
# ============================================================
#  SnElementalGems - upgrades
#  Managed by SnLib: new keys are auto-merged on boot; your values and comments
#  are preserved.
#
#  Five account-wide upgrades bought with gems. A player's current level per
#  upgrade is stored in the database. Each upgrade has:
#    enabled     turn the whole upgrade on/off.
#    max-level   highest purchasable level.
#    levels.<n>  the price to reach level <n> plus its effect value:
#      Fortune    multiplier  -> gems per drop are multiplied.
#      Luck       multiplier  -> drop CHANCE is multiplied.
#      Discount   percent     -> shop prices reduced by this percent.
#      Charity    amount+chance -> chance to also give 'amount' gems to everyone.
#      Execution  chance+commands -> chance to run reward commands on a drop.
# ============================================================

Upgrades:

  # More gems per drop (multiplies the drop amount).
  Fortune:
    enabled: true
    max-level: 3
    levels:
      '1':
        price: 5000
        multiplier: 2.0
      '2':
        price: 25000
        multiplier: 3.0
      '3':
        price: 100000
        multiplier: 4.0

  # Higher chance to receive a gem (multiplies the drop chance).
  Luck:
    enabled: true
    max-level: 3
    levels:
      '1':
        price: 5000
        multiplier: 2.0
      '2':
        price: 25000
        multiplier: 3.0
      '3':
        price: 100000
        multiplier: 4.0

  # Percentage discount applied to every Gem Shop price.
  Discount:
    enabled: true
    max-level: 3
    levels:
      '1':
        price: 5000
        percent: 10
      '2':
        price: 25000
        percent: 20
      '3':
        price: 100000
        percent: 30

  # Chance to also award gems to every online player when you get a drop.
  Charity:
    enabled: true
    max-level: 3
    levels:
      '1':
        price: 5000
        amount: 1
        chance: 1.0
      '2':
        price: 25000
        amount: 2
        chance: 2.0
      '3':
        price: 100000
        amount: 3
        chance: 3.0

  # Chance to run reward commands when you get a drop.
  #   chance                   0-100 roll to trigger.
  #   random-executed.enabled  true -> run a random subset; false -> run all commands.
  #   random-executed.amount   how many random commands to run when enabled.
  #   commands                 console commands; {player} is replaced with the receiver.
  Execution:
    enabled: true
    max-level: 3
    levels:
      '1':
        price: 25000
        chance: 2.5
        random-executed:
          enabled: true
          amount: 1
        commands:
          - "msg {player} &aCongrats! Extra rewards dropped with your gems!"
          - "give {player} diamond 8"
      '2':
        price: 100000
        chance: 5.0
        random-executed:
          enabled: true
          amount: 2
        commands:
          - "msg {player} &aCongrats! Extra rewards dropped with your gems!"
          - "give {player} diamond 16"
      '3':
        price: 500000
        chance: 10.0
        random-executed:
          enabled: false
          amount: 3
        commands:
          - "msg {player} &aCongrats! Extra rewards dropped with your gems!"
          - "give {player} diamond 32"
```

## shops.yml

```yaml
# ============================================================
#  SnElementalGems - shop data
#  Managed by SnLib: new keys are auto-merged on boot; your values and comments
#  are preserved.
#
#  This is the shop DATA (categories and priced rewards). The menu LAYOUT lives
#  in guis/shop.yml (the category hub) and guis/shop-category.yml (the paged
#  sub-shop). This file defines what each purchase costs and grants. The category
#  ids below match the [custom] open-category <id> actions in the hub, and every
#  reward id is the stable token the [custom] buy-reward <category> <reward>
#  action carries.
#
#  Per reward:
#    price                gems the purchase costs (before the Discount upgrade).
#    give-item            true  -> hand the player the item below;
#                         false -> the item is display-only and 'commands' grant it.
#    material             1.21 Material of the reward / display item.
#    amount               stack size handed out when give-item is true.
#    display-name         item name shown in the menu and the purchase message.
#    description          one-line flavour text shown in the reward's menu lore.
#    commands             commands run on purchase. A line with no [tag] runs as
#                         console; [player]/[console] tags are honoured. {player},
#                         {price} and {item} are substituted.
#    required-permission  permission the buyer must hold (empty = none).
#    cancel-if-has-perm   deny if the buyer already holds this permission (empty = off).
# ============================================================

# Seconds a player must wait between purchases (anti-spam; 0 disables the wait).
# The atomic balance withdrawal is the real double-buy guard, this is only UX.
purchase-cooldown-seconds: 2

categories:

  # ----- Crate keys -----
  crates:
    display: "&#8354f2&lCrate Keys"
    rewards:
      common-key:
        price: 1000
        give-item: false
        material: TRIPWIRE_HOOK
        amount: 1
        display-name: "&aCommon Key"
        description: "A common crate key."
        commands:
          - "crates give {player} common 1"
        required-permission: ""
        cancel-if-has-perm: ""
      legendary-key:
        price: 5000
        give-item: false
        material: TRIPWIRE_HOOK
        amount: 1
        display-name: "&6Legendary Key"
        description: "A legendary crate key."
        commands:
          - "crates give {player} legendary 1"
        required-permission: ""
        cancel-if-has-perm: ""

  # ----- Ranks -----
  ranks:
    display: "&#8354f2&lRanks"
    rewards:
      vip-rank:
        price: 50000
        give-item: false
        material: NAME_TAG
        amount: 1
        display-name: "&bVIP Rank"
        description: "Unlock the VIP rank."
        commands:
          - "lp user {player} parent add vip"
        required-permission: ""
        cancel-if-has-perm: "group.vip"
      diamond-bundle:
        price: 25000
        give-item: true
        material: DIAMOND
        amount: 16
        display-name: "&bDiamond Bundle"
        description: "A stack of 16 diamonds."
        commands: []
        required-permission: ""
        cancel-if-has-perm: ""
```

## items.yml

```yaml
# ============================================================
#  SnElementalGems - physical items
#  Managed by SnLib: new keys merge in on boot, your values are kept.
# ============================================================
# The redeemable gem item. Its right-click redeem behaviour is handled by the
# plugin listener (PDC-tagged), so the appearance lives here and the behaviour
# lives in code. Supports SKULL:<owner> / HDB:<id> via the material field when
# HeadDatabase is installed.

gem:
  display-name: "&a&lGem &7(&fRight-Click&7)"
  material: EMERALD
  glow: true
  lore:
    - "&7Right-click to redeem this gem"
    - "&7into your balance."
  # Max stack keeps redeems sane; the plugin reads the stack size on redeem.
  max-stack-size: 64
```
