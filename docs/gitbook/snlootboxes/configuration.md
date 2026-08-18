# Configuration

SnLootBoxes ships with the following YAML files. New keys are auto-merged on boot; your edits and comments are preserved.

The `lootboxes/` folder is the exception: it is yours. The plugin never merges or rewrites your lootbox files.

## config.yml

```yaml
# ============================================================
#  SnLootBoxes - configuration
#  Managed by SnLib: new keys are auto-merged on boot; your values and
#  comments are preserved. Do NOT add a config-version key (retired).
#  Set update-configs: false to freeze this file (SnLib only warns about
#  missing keys instead of inserting them).
# ============================================================

# Active language code; loads lang/messages_<code>.yml (falls back to en).
lang: en

# Master switch of the SnLib auto-updater for this plugin's managed files.
update-configs: true

# Runtime debug output (also toggleable live via /lootbox debug).
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
  # Aliases of /lootbox. Re-read on /lootbox reload.
  aliases: []

# ------------------------------------------------------------
#  Key items. How generated keys behave in inventories.
# ------------------------------------------------------------
key-items:
  # true  -> keys of the same lootbox stack up to the vanilla max.
  # false -> keys from DIFFERENT grants never stack: each give/giveall/editor
  #          grant carries its own unique tag. The keys of one grant still
  #          stack with each other.
  stackable: true

# ------------------------------------------------------------
#  Display-name template. Renders the key-item name for every lootbox.
#  Each lootbox's raw display-name is injected into {name} as a 3-stop
#  hex gradient (hex-1 -> hex-2 -> hex-3).
#  Placeholders: {hex-1} {hex-2} {hex-3} {bullet} {name}
# ------------------------------------------------------------
display-name:
  format-template: "&{hex-1}&l{bullet}&{hex-2}&l{bullet}&f&l{bullet} {name} &f&l&{hex-2}&l{bullet}&{hex-1}&l{bullet}"

# ------------------------------------------------------------
#  Auto-lore. When enabled, the key-item lore in each lootboxes/*.yml is
#  replaced by the template below.
#    key-item-lore placeholders : {display_name} {normal_rewards}
#                                 {super_rewards} {hex-1/2/3} {bullet}
#    *-line-format placeholders : {amount} {displayname} {bullet} {hex-1/2/3}
#  NOTE: a line containing {normal_rewards}/{super_rewards} is REPLACED by one
#  formatted line per reward, so put indentation inside the *-line-format.
# ------------------------------------------------------------
auto-lore:
  enabled: true
  normal-line-format: "&8 {bullet} &fx{amount} &7{displayname}"
  super-line-format: "&{hex-2} {bullet} &fx{amount} &7{displayname}"
  key-item-lore:
    - "&7Open &r{display_name}&7 for a chance at:"
    - ""
    - "&#8354f2&lNormal Rewards"
    - "{normal_rewards}"
    - ""
    - "&#FFD700&lSuper Rewards"
    - "{super_rewards}"
    - ""
    - "&8> Right-click to open"

# ------------------------------------------------------------
#  Fast open. Shift + right-click a key to skip the animation and
#  instantly roll and deliver the full reward set (one reward per
#  normal-chest grid slot in opening.yml, plus the super reward).
# ------------------------------------------------------------
fast-open:
  enabled: true

# ------------------------------------------------------------
#  Session. GUI timeout behaviour.
# ------------------------------------------------------------
session:
  # Seconds before an open GUI auto-closes (applies to every lootbox).
  timeout-seconds: 300
  # true  -> a timed-out session rolls every missing reward and delivers them.
  # false -> only already-rolled rewards are delivered, then the GUI closes.
  auto-complete-on-timeout: true

# ------------------------------------------------------------
#  Delivery. Caps how many accounts sharing one IP receive keys from
#  /lootbox giveall:
#    0 -> no limit; every online player receives a key.
#    N -> at most N accounts per IP. When an IP has more than N accounts
#         online, the N with the highest playtime win the keys (playtime is
#         read live from the vanilla stat; this plugin never stores it).
# ------------------------------------------------------------
delivery:
  max-accounts-per-ip: 0

# ------------------------------------------------------------
#  Roll animation (ticks; 20 ticks = 1 second).
# ------------------------------------------------------------
roll-settings:
  normal-duration-ticks: 60
  normal-interval-ticks: 3
  super-duration-ticks: 80
  super-interval-ticks: 3
  final-visual-delay-ticks: 20

# ------------------------------------------------------------
#  Access. Where lootboxes may be opened.
#  Empty lists = no restriction. allowed-regions requires WorldGuard.
# ------------------------------------------------------------
access:
  allowed-worlds: []
  allowed-regions: []

# ------------------------------------------------------------
#  Effects. Sounds and particles of the opening animation.
# ------------------------------------------------------------
effects:
  normal-roll-sound: "BLOCK_NOTE_BLOCK_HAT"
  normal-complete-sound: "ENTITY_EXPERIENCE_ORB_PICKUP"
  super-roll-sound: "BLOCK_NOTE_BLOCK_BELL"
  super-complete-sound: "UI_TOAST_CHALLENGE_COMPLETE"
  complete-particle: "HAPPY_VILLAGER"
  complete-particle-count: 40

# ------------------------------------------------------------
#  Announce. Message sent when a player finishes opening a lootbox.
#    message placeholders      : {player} {display_name} {normal_rewards}
#                                {super_reward} {hex-1/2/3} {bullet}
#    *-line-format placeholders: {amount} {displayname} {bullet} {hex-1/2/3}
#  mode: "broadcast" (everyone) | "player" (opener only) | "none" (disable)
#  A lootbox with announce-personal: true always announces to the opener only.
#  NOTE: [center] works in both modes. The difference: "player" renders the
#  full text pipeline (PAPI placeholders resolve against the opener), while
#  broadcast renders legacy & / hex colors only - PAPI and MiniMessage tags
#  stay as-is (there is no single viewer to resolve them against).
# ------------------------------------------------------------
announce:
  enabled: true
  mode: "broadcast"
  normal-line-format: "&8   {bullet} &fx{amount} &7{displayname}"
  super-line-format: "&{hex-2}   {bullet} &fx{amount} &7{displayname}"
  message:
    - "&8&m                                                  "
    - "&f{player} &7opened &r{display_name}&7!"
    - ""
    - "&#8354f2&lNormal Rewards"
    - "{normal_rewards}"
    - ""
    - "&#FFD700&lSuper Reward"
    - "{super_reward}"
    - "&8&m                                                  "
```

## opening.yml

```yaml
# ============================================================
#  SnLootBoxes - opening GUI layout
#  Shown when a player right-clicks a lootbox key.
#  Managed by SnLib: new keys are auto-merged on boot; your values and
#  comments are preserved.
#
#  title: {lootbox} is the rendered lootbox name; PAPI placeholders
#  resolve against the opener.
#  size: must be a multiple of 9 between 9 and 54 (invalid -> 54).
#  normal-chest.slots: the openable chest grid. The LIST ORDER is the
#  grid order and its LENGTH is how many normal rewards one opening
#  rolls. Slots outside the GUI size are ignored.
#  Item entries follow the SnLib item spec: material, display-name,
#  lore, glow, custom-model-data (plus heads, colors, enchantments...).
# ============================================================

title: "{lootbox}"
size: 54

# Decorative background; fills every slot no chest occupies.
filler:
  material: BLACK_STAINED_GLASS_PANE
  display-name: " "
  lore: []

# The openable chest grid (default 3x3).
normal-chest:
  slots: [12, 13, 14, 21, 22, 23, 30, 31, 32]
  material: ENDER_CHEST
  display-name: "&#8354f2&lClick to Open"
  lore:
    - "&7Reveal a random reward."
    - ""
    - "&8> Click to roll"

# The super chest: locked until every normal chest is opened.
super-chest:
  slot: 49
  locked:
    material: RED_STAINED_GLASS_PANE
    display-name: "&c&lSuper Chest &8(Locked)"
    lore:
      - "&7Open all chests to unlock."
  unlocked:
    material: ENDER_CHEST
    display-name: "&#FFD700&lSuper Chest"
    lore:
      - "&eThe big one - click to roll!"
      - ""
      - "&8> Click to roll"
    glow: true
```

## lootboxes/example.yml

```yaml
# ============================================================
#  SnLootBoxes - example lootbox
# ------------------------------------------------------------
#  This folder is yours: the plugin never merges or rewrites
#  your files on update. Copy this file to create a lootbox by
#  hand (the file name minus .yml is the lootbox id), or manage
#  everything in-game with /lootbox editor.
#
#  NOTE: when the in-game editor saves a lootbox it rewrites the
#  whole file in its own serialized reward form (see the REWARDS
#  banner below), replacing hand-written display-item blocks and
#  all comments.
#
#  Rarity colors used below:
#    &f Common   &a Uncommon   &b Rare   &5 Epic   &6 Legendary
# ============================================================

# Name shown to players. Rendered as a 3-stop hex gradient
# (hex-1 -> hex-2 -> hex-3 below) through config.yml's
# display-name template.
display-name: "Example Crate"

# false = one opening never rolls the same normal reward twice;
#         needs at least 9 enabled normal rewards to open.
# true  = every roll is independent; needs at least 1.
allow-duplicates: false

# true  = the opening announcement is shown only to the opener.
# false = it follows the global announce settings in config.yml.
announce-personal: false

# The physical key players right-click to open this lootbox.
key-item:
  # Item material of the key.
  material: TRIPWIRE_HOOK
  # Raw key name; the rendered name normally comes from
  # display-name above plus the gradient template.
  display-name: "Example Crate"
  # Key lore; ignored while auto-lore is enabled in config.yml.
  # {hex-1} {hex-2} {hex-3} {bullet} are replaced when shown.
  lore:
    - "&7Right-click to open this crate."
    - ""
    - "&7Rarity: &6Legendary"
  # Stack amount per generated key.
  amount: 1
  # Custom model data for resource packs; 0 = none.
  custom-model-data: 0
  # true adds an enchantment glint to the key.
  glow: false
  # Bullet character used by the gradient template and reward lines.
  bullet: "•"
  # The three gradient stops of the rendered name (#RRGGBB).
  hex-1: "#FFD700"
  hex-2: "#FFFFFF"
  hex-3: "#FFD700"

# ============================================================
#  REWARDS - two pools: "normal" (the 3x3 chest grid) and
#  "super" (the big chest unlocked after all 9 are opened).
#
#  Hand-written reward form (used in this file):
#    weight        relative win chance, any number above 0
#    display-item  the item shown in the GUI and delivered:
#                  material (required), display-name, lore,
#                  amount (stack shown AND delivered),
#                  custom-model-data
#    amount        optional, next to weight: overrides
#                  display-item.amount as the delivered amount
#    actions       optional, one entry decides delivery:
#                    "[console] cmd" or "[command] cmd" runs the
#                    command as console on win ({player} = winner
#                    name, {amount} = reward amount); with no such
#                    entry the display item itself is delivered.
#    enabled       optional, default true. false = the reward
#                  still shows in previews and key lore but can
#                  never be won.
#
#  Editor-saved reward form (equivalent, written on every editor
#  save): weight, nbt (base64 item bytes), amount (delivered
#  amount), method (NBT | COMMAND), command-template, enabled.
#  An explicit method always wins over actions inference.
# ============================================================
rewards:
  normal:
    coal:
      # Relative win chance within this pool.
      weight: 60.0
      display-item:
        material: COAL
        display-name: "&fCoal"
        lore:
          - "&7Rarity: &fCommon"
        amount: 32
    iron:
      weight: 50.0
      display-item:
        material: IRON_INGOT
        display-name: "&fIron Ingot"
        lore:
          - "&7Rarity: &fCommon"
        amount: 16
    money:
      weight: 35.0
      display-item:
        material: GOLD_NUGGET
        display-name: "&6$1,000"
        lore:
          - "&7Rarity: &aUncommon"
      # Delivered as a console command instead of the item above.
      actions:
        - "[console] eco give {player} 1000"
    gold:
      weight: 30.0
      display-item:
        material: GOLD_INGOT
        display-name: "&eGold Ingot"
        lore:
          - "&7Rarity: &aUncommon"
        amount: 8
    redstone:
      weight: 25.0
      display-item:
        material: REDSTONE
        display-name: "&cRedstone Dust"
        lore:
          - "&7Rarity: &aUncommon"
        amount: 24
    emerald:
      weight: 20.0
      display-item:
        material: EMERALD
        display-name: "&aEmerald"
        lore:
          - "&7Rarity: &aUncommon"
        amount: 5
    diamond:
      weight: 15.0
      display-item:
        material: DIAMOND
        display-name: "&bDiamond"
        lore:
          - "&7Rarity: &bRare"
        amount: 3
    ender-pearl:
      weight: 10.0
      display-item:
        material: ENDER_PEARL
        display-name: "&3Ender Pearl"
        lore:
          - "&7Rarity: &bRare"
        amount: 4
    netherite:
      weight: 5.0
      display-item:
        material: NETHERITE_SCRAP
        display-name: "&5Netherite Scrap"
        lore:
          - "&7Rarity: &5Epic"
        amount: 2

  super:
    diamond-block:
      weight: 40.0
      display-item:
        material: DIAMOND_BLOCK
        display-name: "&bDiamond Block"
        lore:
          - "&7Rarity: &bRare"
          - "&#FFD700Super Reward"
        amount: 5
    big-money:
      weight: 30.0
      display-item:
        material: GOLD_BLOCK
        display-name: "&6$10,000"
        lore:
          - "&7Rarity: &6Legendary"
          - "&#FFD700Super Reward"
      actions:
        - "[console] eco give {player} 10000"
    netherite-sword:
      weight: 20.0
      display-item:
        material: NETHERITE_SWORD
        display-name: "&5Netherite Sword"
        lore:
          - "&7Rarity: &5Epic"
          - "&#FFD700Super Reward"
    elytra:
      weight: 10.0
      display-item:
        material: ELYTRA
        display-name: "&dElytra"
        lore:
          - "&7Rarity: &5Epic"
          - "&#FFD700Super Reward"
```

## Other files

- `lang/messages_en.yml` holds every message the plugin sends; it auto-merges on boot.
- `guis/editor-main.yml`, `guis/editor-lootbox.yml`, `guis/editor-items.yml` and `guis/editor-reward.yml` define the editor menus; they auto-merge on boot.
