# Configuration

SnPickaxes ships with three YAML files. New keys are auto-merged on boot; your edits and comments are preserved.

- `config.yml` decides what the pickaxes **do**: cube size, extra drops, region rules, the omnitool switch.
- `items.yml` decides what they **look like**: material, name, lore, model data, enchantments.
- `lang/messages_en.yml` holds all player-facing text.

## Region restrictions

Each pickaxe has its own `regions` block, so you can limit where its special ability works. This needs WorldGuard. Without it the block is ignored and both pickaxes work everywhere.

| `mode` | Behaviour |
|--------|-----------|
| `none` | No restriction. The ability works everywhere. This is the default. |
| `blacklist` | The ability is disabled inside the listed regions, and works everywhere else. |
| `whitelist` | The ability only works inside the listed regions. |

The two lists are independent, so each pickaxe can have a different mode and a different set of regions. Region names are case-insensitive and match WorldGuard region ids. When the ability is blocked, the item still mines as a normal vanilla pickaxe: no area break, no extra drops, and the swing itself is never cancelled.

{% hint style="warning" %}
The shipped list contains one placeholder entry, `example_region`. Replace it with your own. A leftover example that happens to match a real region on your server applies the rule there, with no warning.
{% endhint %}

## config.yml

```yaml
# ============================================================
#  SnPickaxes - configuration
#  Managed by SnLib: new keys are auto-merged on boot; your values and
#  comments are preserved. Do NOT add a config-version key (retired).
#  Set update-configs: false to freeze this file (SnLib only warns about
#  missing keys instead of inserting them).
#  Item appearance lives in items.yml; all text lives in lang/messages_en.yml.
# ============================================================

# Active language code; loads lang/messages_<code>.yml (falls back to en).
lang: en

# Master switch of the SnLib auto-updater for this plugin's managed files.
update-configs: true

# Runtime debug output (also toggleable live via /snpickaxes debug).
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
  # Aliases of /snpickaxes. Re-read on /snpickaxes reload.
  aliases: []

# ------------------------------------------------------------
#  Pickaxes - what they are and what they do.
# ------------------------------------------------------------

# Global omni-tool switch. When true, the moment a player starts mining, the pickaxe MORPHS in
# hand into the tool that matches the block, at the SAME tier as its material: a Diamond Pickaxe
# becomes a Diamond Axe on wood, a Diamond Shovel on sand, a Diamond Hoe on leaves, and a
# Pickaxe again on stone and ores. It mines and drops every block type as its proper tool, and
# this also lets BOTH pickaxes' abilities work on all those block types, overriding each
# pickaxe's 'only-pickaxe-mineable'.
# The morphed tool is rebuilt from items.yml, so it keeps the name, lore and enchantments you
# CONFIGURED there - but anything added to the item afterwards does not survive a morph: an anvil
# rename, an enchantment from a book, and any durability already spent.
omnitool: false

pickaxes:
  # ----- Area pickaxe: breaks an N x N x N cube around the mined block. -----
  area:
    # Master switch for this pickaxe's area-breaking behaviour.
    enabled: true
    # Cube edge length, always centered on the block you mine.
    # MUST be an odd number >= 1: 1 = single block (vanilla), 3 = 3x3x3, 5 = 5x5x5 ...
    # An even value is rounded up to the next odd number on load.
    size: 3
    # Safety cap on 'size'. A swing breaks size^3 blocks on the main thread, so a typo like
    # 51 (132651 blocks) would freeze the server. Values above this are clamped with a warning.
    # Raise it if you intentionally want bigger cubes.
    max-size: 15
    # Only break blocks tagged MINEABLE_PICKAXE (stone, ores, deepslate, ...).
    # If false, every non-air breakable block in range is attempted.
    only-pickaxe-mineable: true
    # ----- Region restrictions (optional; needs WorldGuard) -----
    # Limit WHERE this pickaxe's area-breaking works. Each pickaxe has its OWN list. Without
    # WorldGuard this whole block is ignored and the pickaxe works everywhere. When blocked
    # here the item still mines as a normal vanilla pickaxe, it just does not expand.
    # Region names are case-insensitive.
    regions:
      # none      = no restriction; works everywhere (default).
      # blacklist = area-breaking DISABLED inside the listed regions, works everywhere else.
      # whitelist = area-breaking ONLY works inside the listed regions.
      mode: none
      # WorldGuard region names this rule applies to (used only when mode is not 'none').
      # The entry below is a placeholder - replace it with your own. A leftover example that
      # happens to match a real region on your server applies this rule there with no warning,
      # because a well-formed list is not something the plugin can second-guess.
      list:
        - example_region

  # ----- Duplicator pickaxe: multiplies the drops of the block it breaks. -----
  duplicator:
    # Master switch for this pickaxe's drop-duplication behaviour.
    enabled: true
    # Number of EXTRA drop copies per block, on top of the normal drop.
    # 0 = vanilla drops only, 1 = double, 2 = triple ...
    # There is NO upper cap here, unlike area size's max-size. The copies are produced on the
    # main thread inside the break, and with give-to-inventory: false each one is a separate
    # item entity, so a mistyped digit is a server freeze. Large values are warned about on
    # boot but honoured. Blocks that carry contents, such as a filled shulker box or a bee-
    # filled hive, are never duplicated.
    extra-drops: 1
    # Where the EXTRA copies go:
    #   false = drop them on the ground at the broken block (vanilla-like).
    #   true  = put them straight into the miner's inventory; the overflow drops on the ground.
    # Set this to true if your server core auto-collects ground items and swallows the copies.
    give-to-inventory: true
    # Compatibility with auto-pickup cores that suppress the break's drops via
    # event.setDropItems(false), collecting them themselves WITHOUT cancelling the event.
    # Normally that suppression also skips duplication; set this true to still produce the
    # copies. Only takes effect together with give-to-inventory: true, because ground copies
    # would just be re-collected by that same core. Region protection is unaffected: a denied
    # break is a CANCELLED event, never a drop-suppressed one.
    # If your core instead CANCELS the break entirely, leave this on and also lower
    # advanced.block-break-event-priority at the bottom of this file.
    duplicate-when-drops-suppressed: true
    # Only duplicate drops from blocks tagged MINEABLE_PICKAXE.
    only-pickaxe-mineable: true
    # ----- Region restrictions (optional; needs WorldGuard) -----
    # Limit WHERE this pickaxe's duplication works. Independent of the area pickaxe's list.
    # When blocked here the item still mines as a normal vanilla pickaxe, it just does not
    # duplicate. Region names are case-insensitive.
    regions:
      # none      = no restriction; works everywhere (default).
      # blacklist = duplication DISABLED inside the listed regions, works everywhere else.
      # whitelist = duplication ONLY works inside the listed regions.
      mode: none
      # WorldGuard region names this rule applies to (used only when mode is not 'none').
      # The entry below is a placeholder - replace it with your own. A leftover example that
      # happens to match a real region on your server applies this rule there with no warning,
      # because a well-formed list is not something the plugin can second-guess.
      list:
        - example_region

# ------------------------------------------------------------
#  Advanced - event handling. Leave the default unless an auto-pickup core
#  fights the duplicator.
# ------------------------------------------------------------
advanced:
  # Bukkit EventPriority of our BlockBreakEvent handler.
  # Valid: LOWEST, LOW, NORMAL, HIGH, HIGHEST, MONITOR (case-insensitive).
  # Default MONITOR = run last and observe the final state: safest for region protection, and
  # enough to beat a core that only SUPPRESSES drops (see duplicate-when-drops-suppressed).
  # Lower this (e.g. LOW) ONLY if a core CANCELS the break to auto-collect. Applied on server
  # start/restart, NOT on /snpickaxes reload: the listener is registered once, at enable.
  # WARNING: at ANY value other than MONITOR we stop ignoring cancelled breaks and can run
  # before WorldGuard, Lands and GriefPrevention deny handlers, so the duplicator hands out
  # copies for breaks those plugins then refuse - a protected block becomes an item generator.
  # The exact exposure depends on the priority each protection plugin uses. Use a non-MONITOR
  # value only on a server without block protection; the plugin warns on every boot while one
  # is set.
  block-break-event-priority: MONITOR
```

## items.yml

The two ids, `area` and `duplicator`, match the `config.yml` sections and the `/snpickaxes give` argument. Adding a third id does nothing.

`display-name` and `lore` support `&` colour codes and `&#RRGGBB` hex. The placeholders `%size%`, `%extra%` and `%multiplier%` are available in both fields, and the `{size}` brace form works too. Use each on the pickaxe it means something for. Placeholders resolve once, when the item is created.

{% hint style="warning" %}
`enchantments` is a flat list of id and level pairs, such as `[EFFICIENCY, 255, UNBREAKING, 3]`. A map does not parse.
{% endhint %}

```yaml
# ============================================================
#  SnPickaxes - item definitions
#  Managed by SnLib: new keys are auto-merged on boot; your values and
#  comments are preserved.
#
#  Exactly two ids exist, 'area' and 'duplicator', and they match the section
#  names in config.yml and the argument of /snpickaxes give. Adding a third id
#  here does nothing: the plugin only ever looks these two up.
#
#  What each pickaxe DOES (cube size, extra drops, region limits) is configured
#  in config.yml. This file is only what it looks like and how it behaves as an
#  item. Every field used below is explained in README.md.
# ============================================================

# The Area pickaxe. %size% renders the cube edge length from config.yml.
area:
  display-name: "&b&lArea Pickaxe"
  # Use a real pickaxe material: it decides the tier every omnitool morph keeps.
  material: DIAMOND_PICKAXE
  lore:
    - "&7Breaks a &b%size%x%size%x%size% &7area."
    - "&7Respects region protection."
  custom-model-data: 1001
  glow: true
  # A FLAT list of id, level pairs: [EFFICIENCY, 255, UNBREAKING, 3]. A map does not parse.
  enchantments: [EFFICIENCY, 255]
  # false makes the pickaxe take durability damage. Mind that a cube swing damages it once per
  # block broken, so it can snap partway through one; the cube then stops where it snapped.
  unbreakable: true
  # Kept in the inventory on death, no corpse drop.
  keep-on-death: true
  # The player cannot drop it (Q key, inventory drag-out).
  no-drop: true

# The Duplicator pickaxe. %extra% renders the extra copies and %multiplier% the
# resulting total (extra + 1), both from config.yml.
duplicator:
  # The field comments in the 'area' section above apply to every key here as well.
  display-name: "&d&lDuplicator Pickaxe"
  material: NETHERITE_PICKAXE
  lore:
    - "&7Drops &dx%multiplier% &7the blocks you mine."
    - "&7Respects region protection."
  custom-model-data: 1002
  glow: true
  enchantments: [EFFICIENCY, 255]
  unbreakable: true
  keep-on-death: true
  no-drop: true
```

## lang/messages_en.yml

```yaml
# ============================================================
#  SnPickaxes - language file (English)
#  Managed by SnLib: new keys merge in on boot, your values are kept.
#  Restyle any line freely; your edits survive updates.
# ============================================================

# Prefix prepended to every single-line message sent via sn.lang().send(...)
prefix: "&#8354f2&lSnPickaxes &8| &7"

# snlib.* is SnLib's shared command contract. Every Sn plugin ships
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

# Your own plugin messages. Do NOT write {prefix} in any value: SnLib
# auto-prepends the prefix above to every single-line message sent via
# sn.lang().send(...). A literal {prefix} renders verbatim, and SnLib
# logs a one-time WARN when the active lang file contains one.
messages:
  # Shown to the sender of /snpickaxes give when they gave the pickaxe to themselves.
  given-self: "&aYou received &e%amount% &ax %pickaxe%&a."
  # Shown to the sender of /snpickaxes give when the target is someone else.
  given-other: "&aGave &e%amount% &ax %pickaxe% &ato &e%player%&a."
  # Shown when the pickaxe could not be built into a real item: its section is
  # missing from items.yml, or that section's material is an air type. Nothing
  # was handed out when this appears.
  give-failed: "&cCould not create the &e%pickaxe% &cpickaxe. Check its section in items.yml."
```

## Debug categories

`debug.categories` narrows the runtime output to the areas you care about. The categories are `break`, `omnitool`, `regions`, `items`, `config` and `command`. An empty list lets every one through. A list naming none of them silences the output entirely.
