# Configuration

SnRotatinHeads ships with the following YAML files. New keys are auto-merged on boot; your edits
and comments are preserved. `heads.yml` is written by the plugin itself and is not shipped.

## config.yml

```yaml
# ============================================================
#  SnRotatinHeads - configuration
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

# Runtime debug output (also toggleable live via /rotatinheads debug).
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
  # Aliases of /rotatinheads. Re-read on /rotatinheads reload.
  aliases: [rheads, rh]

# ------------------------------------------------------------
#  Animation. One shared task drives every head.
# ------------------------------------------------------------
animation:
  # Ticks between animation frames (also the client interpolation duration).
  # Floored at 2: more frequent keyframes buy no visible smoothness.
  interval-ticks: 2

# ------------------------------------------------------------
#  Defaults applied to a NEWLY created head. Existing heads keep their own
#  values (edit them with /rotatinheads <size|rotationspeed|...> <id> <value>).
# ------------------------------------------------------------
defaults:
  # Uniform scale of the head model.
  size: 1.0
  # Radians added to the spin per animation frame; negative spins the other way, 0 disables.
  rotation-speed: 0.15
  # Multiplier on the shared bounce phase; 0 disables the bounce.
  bounce-speed: 0.12
  # Peak height of the bounce above the rest position, in blocks.
  bounce-height: 0.25
  # Distance in blocks from which the head is rendered (approximate; capped by the server view distance).
  view-range: 48
  # Base64 texture value used when /rotatinheads create omits one. Empty = plain player head.
  texture: ""
  # What a NEW head shows: <material>[:<custom-model-data>|:<item-model key>], for example
  # paper:1001 or paper:mypack:crown, or an engine model: meg:<id> (ModelEngine) or bm:<id>
  # (BetterModel). Empty = a player head showing the texture above.
  model: ""
  # Item display context of a NEW head, the pivot and base scale its model renders with:
  # ground, fixed, head, gui, none, thirdperson_righthand, ... Player heads read best as ground.
  # Ignored by engine models (meg:/bm:), which render with their own rig.
  transform: ground
  # Animation an engine model (meg:/bm:) plays on a NEW head; empty = none. Ignored by other models.
  animation: idle

# ------------------------------------------------------------
#  Text label above each head (requires the DecentHolograms plugin).
# ------------------------------------------------------------
hologram:
  # Master toggle of the labels. Heads work without labels when false or when DecentHolograms is absent.
  enabled: true
  # Blocks above the head position where the label of a NEWLY created head is anchored
  # (per head afterwards: /rotatinheads hologram offset <id> <blocks>).
  y-offset: 1.2
  # Blocks between two label lines of a NEWLY created head
  # (per head afterwards: /rotatinheads hologram spacing <id> <blocks>).
  line-height: 0.25
  # Default follow-bounce for a NEWLY created head: true makes the label bob with the head.
  follow-bounce: false
```

## lang/messages_en.yml

```yaml
# ============================================================
#  SnRotatinHeads - language file (English)
#  Managed by SnLib: new keys merge in on boot, your values are kept.
#  Restyle any line freely; your edits survive updates.
# ============================================================

# Prefix prepended to every single-line message sent via sn.lang().send(...)
prefix: "&#8354f2&lSnRotatinHeads &8| &7"

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

# Plugin messages. Do NOT write {prefix} in any value: SnLib auto-prepends the
# prefix above to every single-line message sent via sn.lang().send(...).
messages:
  # Generic
  player-only: "&cThis command must be run by a player."
  invalid-id: "&cInvalid id. Use only letters, numbers, '_' and '-' (1-32 characters)."
  head-not-found: "&cNo head found with id &f{id}&c."
  head-already-exists: "&cA head with id &f{id}&c already exists."
  head-world-unloaded: "&cThe world of head &f{id}&c (&f{world}&c) is not loaded."
  invalid-texture: "&cThat is not a head texture. Use a base64 textures value or a skin URL."
  invalid-model: "&cInvalid model &f{value}&c. Use &f<material>&c, &f<material>:<custom-model-data>&c, &f<material>:<namespace:key>&c, &fmeg:<model>&c, &fbm:<model>&c or &fhand&c."
  model-engine-missing: "&cThe model &f{model}&c needs &f{engine}&c, which is not installed or not enabled."
  model-engine-unknown: "&f{engine}&c has no model named &f{value}&c."
  invalid-transform: "&cInvalid display context: &f{value}"
  model-hand-empty: "&cHold the item whose model the head should show in your main hand."
  not-persisted: "&cWARNING: heads.yml could not be read at the last load, so edits are kept in memory only and NOT saved. Fix or remove the file, then run &f/{label} reload&c."
  hologram-provider-missing: "&eDecentHolograms is not installed or labels are disabled; the head works but its text label is not shown."

  # Head lifecycle
  head-created: "&aCreated rotating head &f{id}&a at your location."
  head-removed: "&aRemoved head &f{id}&a."
  head-moved: "&aMoved head &f{id}&a to your location."
  head-teleported: "&aTeleported to head &f{id}&a."
  texture-set: "&aUpdated the texture of head &f{id}&a."
  texture-hidden-by-model: "&eThis head shows the model &f{model}&e, so the texture is not visible until the model is a player head again."
  model-set: "&aHead &f{id}&a now shows &f{model}&a."
  animation-set: "&aHead &f{id}&a animation set to &f{value}&a."
  animation-waits-for-engine: "&eThis head shows &f{model}&e; the animation plays once it shows a ModelEngine or BetterModel model."
  transform-set: "&aSet display context of head &f{id}&a to &f{value}&a."
  size-set: "&aSet size of head &f{id}&a to &f{value}&a."
  rotation-set: "&aSet rotation speed of head &f{id}&a to &f{value}&a."
  bounce-speed-set: "&aSet bounce speed of head &f{id}&a to &f{value}&a."
  bounce-height-set: "&aSet bounce height of head &f{id}&a to &f{value}&a."
  view-range-set: "&aSet view range of head &f{id}&a to &f{value}&a blocks."

  # List and info. Continuation lines carry [noprefix] so only the header shows the prefix.
  list-empty: "&7There are no rotating heads yet. Create one with &f/{label} create <id>&7."
  list-header: "&fRotating heads &7({count})&f:"
  list-entry: "[noprefix]&8 - &f{id} &7@ {world} {x}, {y}, {z}"
  info-header: "&fHead &#8354f2{id}&f:"
  info-location: "[noprefix]&8 - &7Location: &f{world} {x}, {y}, {z}"
  info-model: "[noprefix]&8 - &7Model: &f{model} &7(&f{transform}&7, animation &f{animation}&7)"
  info-size: "[noprefix]&8 - &7Size: &f{size}"
  info-rotation: "[noprefix]&8 - &7Rotation speed: &f{rotation}"
  info-bounce: "[noprefix]&8 - &7Bounce: &fspeed {bounce-speed}&7, &fheight {bounce-height}"
  info-view-range: "[noprefix]&8 - &7View range: &f{view-range} blocks"
  info-hologram-lines: "[noprefix]&8 - &7Hologram lines: &f{count}"
  info-hologram-layout: "[noprefix]&8 - &7Hologram offset: &f{offset}&7, line spacing: &f{spacing}"
  info-actions: "[noprefix]&8 - &7Actions: &fleft {left}&7, &fright {right}"

  # Hologram lines (0-indexed)
  hologram-line-added: "&aAdded hologram line to head &f{id}&a."
  hologram-line-set: "&aSet hologram line &f{line}&a of head &f{id}&a."
  hologram-cleared: "&aCleared hologram lines of head &f{id}&a."
  hologram-follow-set: "&aHologram of head &f{id}&a follow-bounce set to &f{value}&a."
  hologram-offset-set: "&aSet hologram offset of head &f{id}&a to &f{value}&a blocks."
  hologram-spacing-set: "&aSet hologram line spacing of head &f{id}&a to &f{value}&a blocks."
  hologram-list-header: "&fHologram lines of head &#8354f2{id}&f &7(line numbers start at 0)&f:"
  hologram-list-entry: "[noprefix]&8 - &7[{index}] &r{line}"
  hologram-list-empty: "&7Head &f{id}&7 has no hologram lines yet. Add one with &f/{label} hologram add {id} <text>&7."
  hologram-no-lines: "&cHead &f{id}&c has no hologram lines to set. Add one first with &f/{label} hologram add {id} <text>&c."
  hologram-line-out-of-range: "&cLine &f{value}&c is out of range. This head has &f{count}&c line(s), so valid indexes are &f0-{max}&c."

  # Click actions (0-indexed, SnLib [tag] argument syntax)
  action-added: "&aAdded a &f{side}&a-click action to head &f{id}&a."
  action-removed: "&aRemoved &f{side}&a-click action &f#{index}&a from head &f{id}&a."
  action-cleared: "&aCleared &f{side}&a-click actions of head &f{id}&a."
  action-list-header: "&f{side}&f-click actions of head &#8354f2{id}&f:"
  action-list-entry: "[noprefix]&8 - &7[{index}] &f{action}"
  action-list-empty: "&7Head &f{id}&7 has no &f{side}&7-click actions."
  action-invalid-index: "&cInvalid action index: &f{value}"
  action-invalid-format: "&cAn action must start with a tag, for example &f[player] spawn&c, &f[console] give {player} diamond&c, &f[message] Hello&c or &f[sound] entity.experience_orb.pickup&c."

# Short state words. Each is substituted into a {value} or {side} slot of the
# messages above, so a restyle here applies everywhere at once.
status:
  on: "&aon"
  off: "&coff"
  side-left: "left"
  side-right: "right"
  side-any: "any"
  # Shown instead of a world name that no longer resolves. Color codes are stripped on read.
  unknown: "Unknown"
  # Shown as the model of a head that renders the plain textured player head. Color codes are stripped on read.
  player-head: "player head"
  # Shown as the animation of a head that plays none. Color codes are stripped on read.
  none: "none"
```

## heads.yml

Created the first time you save a head. One entry per head, keyed by id. You can edit it by hand and
run `/rh reload`; the plugin rewrites the whole file after every change. `model` is
`<material>[:<custom-model-data>|:<item-model key>]`, `meg:<model>` or `bm:<model>` (empty = the
textured player head) and
`transform` is the item display context token (`ground`, `fixed`, `head`, ...); `animation` is the
animation an engine model (`meg:`/`bm:`) plays, empty for none; `hologram-offset` and
`hologram-line-height` are the label height above the head and the distance between its lines. A
missing key means the configured default and an unreadable value falls back to it with a console
warning.

```yaml
heads:
  lobby:
    world: world
    x: 0.5
    y: 65.0
    z: 0.5
    yaw: 90.0
    pitch: 0.0
    texture: eyJ0ZXh0dXJlcyI6...
    model: ""
    transform: ground
    animation: idle
    size: 1.0
    rotation-speed: 0.15
    bounce-speed: 0.12
    bounce-height: 0.25
    view-range: 48.0
    follow-bounce: false
    hologram-offset: 1.2
    hologram-line-height: 0.25
    hologram-lines:
    - "&aWelcome"
    actions:
      left: []
      right:
      - "[player] spawn"
```

{% hint style="warning" %}
If `heads.yml` cannot be read (a YAML mistake, or `heads:` that is not a map), the plugin loads no
heads, refuses to save until you fix or remove the file and run `/rh reload`, and says so in the
console and after every command. Nothing is overwritten.
{% endhint %}
