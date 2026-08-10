# Configuration

SnChunkLoader ships with the following YAML files. New keys are auto-merged on boot; your edits and comments are preserved.

## config.yml

```yaml
# ============================================================
#  SnChunkLoader - configuration
#  Managed by SnLib: new keys are auto-merged on boot; your values and
#  comments are preserved. Do NOT add a config-version key (retired).
#  Set update-configs: false to freeze this file (SnLib only warns about
#  missing keys instead of inserting them).
# ============================================================

# Active language code; loads lang/messages_<code>.yml (falls back to en).
lang: en

# Master switch of the SnLib auto-updater for this plugin's managed files.
update-configs: true

# Runtime debug output (also toggleable live via /chunkloader debug).
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
  # Aliases of /chunkloader. Re-read on /chunkloader reload.
  aliases: [scl, snchunkloader]

# ------------------------------------------------------------
#  Database. type=sqlite needs nothing else; type=mysql reads host/port/etc.
# ------------------------------------------------------------
database:
  # sqlite or mysql
  type: sqlite
  # MySQL connection (ignored when type is sqlite).
  host: localhost
  port: 3306
  database: snchunkloader
  username: root
  password: ""

# ------------------------------------------------------------
#  Chunk loader - model. The item itself and the square it covers.
# ------------------------------------------------------------
chunk-loader:
  # Block placed as a chunk loader, and the material of its item. An unknown
  # value, or one that is not both a placeable block and a real item, falls back
  # to BEACON.
  #
  # DO NOT CHANGE THIS ONCE LOADERS EXIST. A placed loader is recognised by its
  # block material, so changing it makes every existing one unrecognisable.
  # PLACED loaders are recoverable: the reconciliation sweep unplaces them and
  # owes each item back to its owner, with its remaining time, re-minted in the
  # NEW material. The old blocks are left standing as ordinary blocks.
  # LOADER ITEMS ALREADY HANDED OUT ARE NOT. Every loader sitting in an inventory,
  # a chest or an ender chest carries the OLD material and placing one is refused,
  # so those stacks become unplaceable for good and nothing refunds them. Buy them
  # back or re-issue them by hand before you touch this key.
  # A TYPO here does all of that too, because the fallback to BEACON is itself a
  # change.
  material: BEACON
  # CustomModelData stamped on the loader item; 0 = none.
  custom-model-data: 0
  # Side of the square of chunks a loader keeps loaded, centered on its own
  # chunk. Always ODD: 1 keeps 1 chunk, 3 keeps 9, 5 keeps 25, 7 keeps 49.
  #
  # This range governs what /chunkloader give may MINT. It deliberately does NOT
  # confiscate: loaders already placed keep running, and loader items already in
  # players' hands can still be placed, even if their size now sits outside the
  # range. Narrowing it stops you SELLING a size; it never destroys one somebody
  # already paid for. A hard ceiling of 31 applies whatever you set here, because
  # the square is a direct multiplier on synchronous chunk loads.
  size:
    # Smallest size /chunkloader give may mint. Does NOT restrict placement.
    min: 1
    # Largest size /chunkloader give may mint. Does NOT restrict placement:
    # items already handed out stay placeable (see the note above). Hard cap 31.
    max: 7

  # ----------------------------------------------------------
  #  Features. Stored lifetime and the background tasks.
  # ----------------------------------------------------------
  time:
    # Ceiling on the lifetime one placed loader may accumulate by stacking, in
    # seconds. Stacking past it clamps to it. 0 = unlimited. 2592000 = 30 days.
    max-stack-seconds: 2592000
  tasks:
    # Period of the lifetime countdown, in ticks. Each run subtracts exactly
    # this many ticks of time, so the clock is tick-based, not wall-clock.
    countdown-interval-ticks: 20
    # Period of the flush that writes changed loaders to the database, in seconds.
    save-interval-seconds: 300
    # Delay between a player joining and the delivery of the loaders owed to
    # them, in ticks. Gives the client time to sync its inventory first.
    pending-delivery-delay-ticks: 20
    # Period of the reconciliation sweep, in seconds. 0 = disabled.
    # It does TWO jobs, and only the second one obeys return-on-membership-loss:
    #  1. Unplaces any loader whose BLOCK no longer exists and owes the item back
    #     to its owner. This is the ONLY thing that recovers a loader removed by
    #     something that fires no event at all - WorldEdit, a schematic paste, an
    #     island regen - and it runs whether or not returns are enabled.
    #  2. Re-checks that every owner is still a member of the island their loader
    #     sits on: the safety net for memberships changed through the
    #     SuperiorSkyblock API, which fires no event. Skipped entirely when
    #     SuperiorSkyblock is not installed, and skipped for any loader placed
    #     while it was not - (1) still runs for all of them.
    # Setting this to 0 therefore also switches off (1). Do that only if you are
    # certain no plugin on the server removes blocks without an event.
    reconciliation-interval-seconds: 120

  # ----------------------------------------------------------
  #  Limits. How many loaders may be placed, and where.
  # ----------------------------------------------------------
  limits:
    # Placed loaders per owner. 0 = unlimited. Applies on EVERY server, with or
    # without SuperiorSkyblock: it counts loaders, not islands.
    per-player: 3
    # Placed loaders per island, counted as NUMBER of loaders. 0 = unlimited.
    # IGNORED when SuperiorSkyblock is not installed - there are no islands to
    # count against, so per-player above is the only limit left.
    per-island: 5

  # ----------------------------------------------------------
  #  Island rules. SuperiorSkyblock is OPTIONAL: install it and this whole
  #  section applies, leave it out and the plugin is a plain chunk loader -
  #  loaders may be placed anywhere the server lets you place a block, anyone
  #  who can break the block gets the loader and its remaining time, and nothing
  #  below is evaluated. The keys stay here either way, ready for the day the
  #  island plugin is added.
  # ----------------------------------------------------------
  island:
    # Membership gate. This is NOT only a placement rule - read it before turning it off.
    # When true, a player must be a member of the island a loader stands on in order to
    # PLACE one there, and equally to BREAK one, FEED it more time, or TOGGLE it on and off.
    # When false, ALL FOUR gates open: any player can break someone else's placed loader and
    # walk away with the item and its remaining time. On a server that sells loaders that is
    # theft, so leave it true unless the whole server is deliberately free-for-all.
    require-own-island: true

    # Restrict placement to the island's PROTECTED range instead of its full
    # claimed area.
    protected-area-only: false
    # Return a loader to its owner when that owner stops being a member of the
    # island it stands on (kick, quit, ban, disband). false = leave it in place.
    return-on-membership-loss: true

# ------------------------------------------------------------
#  Feedback. The floating display above each placed loader.
# ------------------------------------------------------------
hologram:
  # Show a floating display above every placed loader.
  enabled: true
  # Display backend: snlib (built in, always available) or decentholograms.
  # decentholograms falls back to snlib when that plugin is not installed.
  provider: snlib
  # Height of the first line above the loader block, in blocks.
  offset-y: 2.5
  # Period of the display refresh, in ticks. 40 = every 2 seconds.
  update-interval-ticks: 40
```

## lang/messages_en.yml

```yaml
# ============================================================
#  SnChunkLoader - language file (English)
#  Managed by SnLib: new keys merge in on boot, your values are kept.
#  Restyle any line freely; your edits survive updates.
# ============================================================

# Prefix prepended to every single-line message sent via sn.lang().send(...)
prefix: "&#8354f2&lSnChunkLoader &8| &7"

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

# Your own plugin messages. Do NOT write {prefix} in any value: SnLib
# auto-prepends the prefix above to every single-line message sent via
# sn.lang().send(...).
messages:
  # --- command feedback ---
  players-only: "&cThis command can only be used by a player."
  invalid-number: "&cInvalid number: &f{value}"
  invalid-size: "&cSize must be an odd number between &f{min} &cand &f{max}&c."
  invalid-duration: "&cInvalid duration: &f{value} &c(use e.g. 1d, 12h, 30m, 1d12h, or -1 for infinite)."
  given: "&aGave &f{amount}x &achunk loader (&f{size}x{size}&a, &f{time}&a) to &f{player}&a."
  received: "&aYou received a &f{size}x{size} &achunk loader with &f{time}&a."
  # Sent to the RECEIVER of /chunkloader give when the item did not fit. It REPLACES `received`
  # above rather than following it, because the loader is owed and not held. Nothing is ever
  # dropped on the ground: it is handed over on their next join.
  inventory-full: "&cYour inventory is full. The chunk loader is waiting for you - make room and rejoin to receive it."
  # Told to the SENDER of /chunkloader give when the target could not hold it. It REPLACES `given`
  # above, so it carries the same facts: exactly one outcome line reaches each side.
  # Placeholders: {player}, {size}, {time}
  given-queued: "&e&f{player}&e could not hold the &f{size}x{size} &eloader (&f{time}&e), so it was queued for their next join."
  list-empty: "&7No chunk loaders found for &f{player}&7."
  chunks-info: "&aForce-loaded chunks: &f{chunks} &7| Loaders: &f{loaders} &7(&a{active} &7on)"

  # --- placing a loader ---
  no-permission: "&cYou don't have permission to do this."
  placed: "&aChunk loader placed. Keeping &f{chunks} &achunk(s) loaded."
  # Sent instead of the line above when the loader was placed with no time left: it stands there
  # switched off and holds no chunks until it is fed.
  placed-empty: "&eChunk loader placed, but it has no time left. Right-click it with another loader of the same size to feed it."
  place-failed: "&cThis chunk loader could not be placed here."
  # Shown while the plugin is still reading its loaders from storage on boot. During that window a
  # lookup cannot tell "no loader here" from "not read yet", so loader blocks are protected instead.
  not-ready: "&eChunk loaders are still loading. Try again in a moment."
  not-on-island: "&cYou can only place chunk loaders inside your own island."
  not-a-member: "&cYou are not a member of this island."
  limit-player: "&cYou have reached your limit of &f{limit} &cplaced chunk loaders."
  limit-island: "&cThis island has reached its limit of &f{limit} &cchunk loaders."

  # --- breaking one, and feeding it more time ---
  removed: "&aChunk loader removed. Returned to your inventory with &f{time}&a."
  # Sent instead of the line above when the inventory was full. The loader is NEVER dropped on the
  # ground - it is a paid item, so it is owed and handed over on your next join.
  removed-queued: "&eChunk loader removed with &f{time}&e, but your inventory is full. Make room and rejoin to receive it."
  cannot-break: "&cOnly island members can remove this chunk loader."
  not-your-loader: "&cThis chunk loader belongs to someone not on your island."
  time-added: "&aAdded time. This loader now has &f{time}&a."
  size-mismatch: "&cYou can only add time with another &f{size}x{size} &cchunk loader."
  already-infinite: "&cThis chunk loader already has infinite time."
  # The item you used had no time left on it, so it was not consumed.
  item-empty: "&cThat chunk loader has no time left to give."
  # The item's stored lifetime could not be read at all (a tampered or foreign stack). Nothing was
  # consumed and nothing was added.
  item-unreadable: "&cThat chunk loader item is damaged and cannot be used."
  max-stack-reached: "&cThis loader is already at the maximum stored time (&f{time}&c)."
  # The stack landed exactly on the ceiling and the excess time on the consumed item was lost.
  stack-clamped: "&eThis loader hit the &f{time} &eceiling; the time above it was lost."

  # --- loader menu ---
  toggled-on: "&aChunk loader &factivated&a."
  toggled-off: "&cChunk loader &fpaused&c. Time will not decrease while off."
  cannot-toggle-expired: "&cThis loader has no time left. Add time to activate it."

  # --- returns and owed deliveries ---
  returned-membership: "&eA chunk loader was returned to you (&f{time} &eleft) because you are no longer on that island."
  pending-delivered: "&aYou received &f{amount} &achunk loader(s) that were waiting for you."

# The /chunkloader list chat view. Both lines are rendered without the prefix.
lists:
  # Header of the list. Placeholders: {player}, {count}
  list-header: "&#8354f2&lChunk Loaders &8- &f{player} &7({count})"
  # One line per placed loader. Placeholders: {index}, {world}, {x}, {y}, {z},
  # {size}, {status}, {time}
  list-entry: "&8#{index} &7{world} &8[&f{x}, {y}, {z}&8] &7{size}x{size} &8| {status} &8| &7{time}"

# Name and lore of the loader item, rendered once when the item is created and
# identical for every viewer. Placeholders: {size}, {chunks}, {time}
#
# The lore is deliberately island-neutral, because SuperiorSkyblock is optional and the same item
# is handed out on servers that have no islands at all. On an island server, replace the third line
# with something like "&8Place inside your island to activate." - your value is preserved on every
# update.
item:
  name: "&#8354f2&lChunk Loader &7({size}x{size})"
  lore:
    - "&7Keeps &f{chunks} &7chunk(s) loaded and ticking."
    - "&7Remaining time: {time}"
    - ""
    - "&8Place it down to activate."
    - "&8Right-click a placed loader with another"
    - "&8loader to add its time."

# Lines of the floating display above a placed loader, refreshed on the interval
# set in config.yml. Placeholders: {owner}, {time}, {size}, {chunks}, {status}
hologram:
  lines:
    - "&#8354f2&lChunk Loader"
    - "&7Owner: &f{owner}"
    - "&7Time: {time}"
    - "&7Status: {status}"

# Short state words. Each is substituted into a {status} or {time} slot of the
# messages above, of the menus under guis/ and of the item lore, so a restyle
# here applies everywhere at once. Read by ONE labels class, never inline.
status:
  # A loader that is running, and one that is paused.
  loader-on: "&aON"
  loader-off: "&cOFF"
  # Shown in the {time} slot of a loader whose lifetime is spent.
  expired: "&cExpired"
  # Shown in the {time} slot of a loader that never expires (given with -1).
  infinite: "&bInfinite"
  # Shown in the {time} slot while less than one whole minute is left.
  less-than-minute: "&f<1&7m"
  # Shown instead of an owner name that no longer resolves. Color codes are
  # stripped on read: the value substitutes where a real name goes.
  unknown: "Unknown"

# Per-unit templates of the remaining-time formatter. Never seconds; zero
# components are omitted; non-zero components are joined by the separator.
# Placeholder: {value}
time:
  unit-day: "&f{value}&7d"
  unit-hour: "&f{value}&7h"
  unit-minute: "&f{value}&7m"
  separator: " "
```

## guis/loader.yml

```yaml
# ============================================================
#  SnChunkLoader - loader menu
#  Opened by right-clicking a placed loader while NOT holding a loader item.
#  (Holding a matching loader item feeds it time instead; anything else -
#  an empty hand, a sword, a stack of dirt - opens this menu.)
#  Managed by SnLib: new keys are auto-merged on boot; your values and
#  comments are preserved. Do NOT add a config-version key (retired).
#  Placeholders in the title, in every display-name and in every lore line:
#  {owner} {time} {size} {chunks} {status}
#  &#RRGGBB hex and MiniMessage work in all of them. PlaceholderAPI tokens
#  work too, but they resolve WITHOUT a viewer: server-wide tokens fill in,
#  per-player ones have no player to resolve against.
#  NOTE the values are a snapshot taken when the menu opens; they do not
#  tick while it stays open.
# ============================================================

# Window title.
title: "&#8354f2&lChunk Loader"

# Cell map, one character per slot, nine per row.
# f = border, i = info, t = the toggle button.
# To hide an element, delete BOTH its letter from the map below AND its own
# section further down. Removing only the letter leaves a defined element with
# nowhere to go, and SnLib warns about it on every boot.
layout:
  - "fffffffff"
  - "fffiftfff"
  - "fffffffff"

# Only the four basic mouse clicks act. Hotbar keys, drops, middle-clicks and
# off-hand swaps do nothing at all.
strict-clicks: true

items:
  # Background border.
  filler:
    key: f
    material: GRAY_STAINED_GLASS_PANE
    display-name: " "

templates:
  # Read-only summary of the loader. Carries no action, so clicking it does nothing.
  info:
    key: i
    material: BEACON
    glow: true
    display-name: "&#8354f2&lChunk Loader &7({size}x{size})"
    lore:
      - "&7Owner: &f{owner}"
      - "&7Chunks loaded: &f{chunks}"
      - "&7Remaining time: {time}"
      - "&7Status: {status}"

  # Face shown while the loader is ON. Clicking it pauses the loader.
  # Both faces share the cell 't': exactly one of them is placed at a time.
  toggle-on:
    key: t
    material: LIME_DYE
    glow: true
    display-name: "&aLoader is ON"
    lore:
      - "&7Time is counting down and chunks are loaded."
      - ""
      - "&eClick to pause &7(stops the timer)."
    click-actions:
      - "[chunkloader-toggle]"
      - "[sound] UI_BUTTON_CLICK"

  # Face shown while the loader is OFF. Clicking it starts the loader.
  toggle-off:
    key: t
    material: GRAY_DYE
    display-name: "&cLoader is OFF"
    lore:
      - "&7Timer is paused and chunks are not loaded."
      - ""
      - "&eClick to activate."
    click-actions:
      - "[chunkloader-toggle]"
      - "[sound] UI_BUTTON_CLICK"
```
