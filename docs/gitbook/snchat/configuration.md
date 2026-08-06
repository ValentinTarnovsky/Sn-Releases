# Configuration

SnChat ships with the following YAML files. New keys are auto-merged on boot; your edits and comments are preserved. Set `update-configs: false` in `config.yml` to freeze that behaviour, and SnLib will only warn about missing keys instead of inserting them.

Sections marked `# sn:extensible` are yours. Entries you delete there stay deleted.

## config.yml

```yaml
# ============================================================
#  SnChat - configuration
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

# Runtime debug output (also toggleable live via /snchat debug).
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
  # Aliases of /snchat. Re-read on /snchat reload.
  aliases: []

# ============================================================
#  BAND 1 - MODEL: the chat line itself
# ============================================================

# Default format for every player whose LuckPerms primary group has no
# group-formats entry below.
# Placeholders: {prefix}, {suffix}, {displayname}, {name}, {message}, %papi%
# A format WITHOUT {message} renders the format alone and drops the message.
# The colour in front of {message} carries onto the message body; codes the
# sender typed themselves override it.
#
# What a player may put in their own message is split across two permissions:
#   snchat.color       - &-codes and &#RRGGBB hex. Cosmetic MiniMessage tags
#                        (<red>, <gradient>) render too.
#   snchat.prefixtags  - the line tags [center], [rgb], [small] and [noprefix],
#                        which only work at the very start of a message and
#                        affect the whole line.
#
# BOTH need a & somewhere in the message to take effect at all. A message with
# no & is published exactly as typed, so "<red>hi" and "[center]hi" reach chat
# as those literal characters. That guard is deliberate: it is what stops this
# plugin overwriting a line another plugin already coloured. Type "&r<red>hi"
# to get the tag rendered.
#
# Without snchat.prefixtags the leading tags never take effect: they are removed
# before the line is rendered, or left as visible text when nothing renders it.
# Either way the caps and flood filters measure exactly what readers receive.
# Prefix tags are NOT supported inside chat-format or group-formats: each half
# of the format is rendered separately, so a leading tag would apply to the
# left half alone.
chat-format: "{prefix}{displayname} &8&l>> &7{message}"

# ------------------------------------------------------------
#  Per-group overrides, keyed by LuckPerms primary group.
#  Case is ignored, so "Admin" and "admin" are the same group - the same rule
#  blockcommands.yml uses. Unlisted groups fall back to chat-format above.
# ------------------------------------------------------------
# sn:extensible
group-formats: {}
#  founder: "{prefix}{displayname}&8&l> &#FF9B00{message}"
#  admin: "{prefix}{displayname} &8&l> &f{message}"

# ============================================================
#  BAND 2 - FEATURES
# ============================================================

# ------------------------------------------------------------
#  Tooltip and click attached to the whole chat line.
# ------------------------------------------------------------
message-hover:
  # Whether the chat line carries a hover tooltip and a click at all.
  enabled: true
  # Tooltip lines. Placeholders: {player}, {name}, {displayname}, {prefix},
  # {suffix}, %papi%. {player} and {name} are both the account name; {displayname}
  # is the nickname other plugins may have set.
  lines:
    - "&7Player: &f{displayname}"
    - "&7World: &f%player_world%"
    - "&7Health: &c%player_health_rounded%&7/&c%player_max_health_rounded%"
    - ""
    - "&eClick to message"
  # Command suggested (never run) when the line is clicked. The trailing space
  # matters. {player} is the only token expanded here (plus %papi%); the other
  # format placeholders are not, because a command box is not styled text.
  click-action: "/msg {player} "

# ------------------------------------------------------------
#  [item] token: shows the sender's held item inline, with a tooltip.
# ------------------------------------------------------------
item-display:
  # Master switch. When false the tokens below stay literal text.
  enabled: true
  # Tokens replaced by the held item. Matching is case-insensitive.
  aliases:
    - "[item]"
    - "[i]"
    - "[itm]"
  # Materials that cannot be shared. Sharing one CANCELS the whole message.
  # Exact uppercase Material names; a typo is warned about on boot.
  blacklist:
    - BARRIER
    - BEDROCK
    - COMMAND_BLOCK
    - CHAIN_COMMAND_BLOCK
    - REPEATING_COMMAND_BLOCK
    - STRUCTURE_BLOCK
    - STRUCTURE_VOID
    - JIGSAW
    - DEBUG_STICK

# ------------------------------------------------------------
#  [inv] token: a clickable tag opening a frozen, view-only replica of the
#  sender's inventory as it was when they sent the message.
# ------------------------------------------------------------
inventory-display:
  # Master switch. When false the tokens below stay literal text.
  enabled: true
  # Tokens replaced by the clickable tag. Matching is case-insensitive.
  # An alias listed under two families answers for only one of them; the log
  # says which on boot.
  aliases:
    - "[inv]"
    - "[inventory]"
  # Seconds the snapshot stays clickable. Clamped to 1-86400; never infinite.
  # The click dies at exactly the same moment the snapshot does.
  snapshot-ttl-seconds: 600
  # Materials rendered as an empty slot in the replica. Silent, no message.
  # Exact uppercase Material names; a typo is warned about on boot.
  blacklist: []

# ------------------------------------------------------------
#  [ec] token: same as [inv], for the ender chest.
# ------------------------------------------------------------
enderchest-display:
  # Master switch. When false the tokens below stay literal text.
  enabled: true
  # Tokens replaced by the clickable tag. Matching is case-insensitive.
  aliases:
    - "[ec]"
    - "[enderchest]"
  # Seconds the snapshot stays clickable. Clamped to 1-86400.
  snapshot-ttl-seconds: 600
  # Materials rendered as an empty slot in the replica. Silent, no message.
  # Exact uppercase Material names; a typo is warned about on boot.
  blacklist: []

# ------------------------------------------------------------
#  Global chat mute, toggled by /snchat mutechat or /mutechat.
#  Only players with snchat.bypass.mutechat can talk while it is on.
# ------------------------------------------------------------
mutechat:
  # Whether the mute COMMAND works. It does not gate the mute check itself.
  enabled: true

# ------------------------------------------------------------
#  Chat clearing, run by /snchat clearchat or /clearchat.
# ------------------------------------------------------------
clearchat:
  # Whether the clear COMMAND works.
  enabled: true
  # Blank lines pushed to every online player to scroll the chat away.
  # Clamped to 1-500.
  lines: 100

# ============================================================
#  BAND 3 - LIMITS
# ============================================================

# ------------------------------------------------------------
#  Live [inv] and [ec] snapshots.
#  Each one is a full deep copy of a container, held in memory until its TTL
#  expires. The TTL bounds how long one lives; this bounds how many exist, so
#  a player cannot fill the heap by typing [inv] over and over.
# ------------------------------------------------------------
snapshots:
  # Most live snapshots one player may hold at once, counting [inv] and [ec]
  # together. Past it their OLDEST is dropped and its tag answers "expired".
  # Clamped to 2-64. The floor is 2, not 1: one message may carry both an [inv]
  # and an [ec] token, and at a cap of 1 the second would evict the first.
  max-per-player: 8

# ------------------------------------------------------------
#  Chat moderation. Checks run in order: mute, cooldown, flood, caps.
#  A block short-circuits; corrections accumulate.
# ------------------------------------------------------------
chat-control:
  # Master switch of every module below. It also gates the global mute CHECK, so
  # turning this off stops an active mute from being enforced and /mutechat
  # refuses to toggle one.
  enabled: true

  # Bypass: snchat.bypass.cooldown. All delays are milliseconds; 0 disables one.
  cooldown:
    # Whether the three cooldown clocks below run at all.
    enabled: true
    # Minimum gap between any two messages from the same player.
    message-delay: 0
    # Minimum gap before the same player may repeat the same message.
    repeat-delay: 2500
    # Minimum gap before anyone may repeat a message another player just sent.
    global-repeat-delay: 1000
    # Minimum gap between any two commands from the same player.
    command-delay: 0
    # Whether a cooldown violation also alerts snchat.notify holders.
    notification: true

  # Bypass: snchat.bypass.caps.
  caps:
    # Whether excessive capitalization is checked at all.
    enabled: false
    # true corrects the message to lowercase; false blocks it outright.
    replace: true
    # Maximum consecutive uppercase LETTERS. Non-letters neither count nor
    # reset the run, so "ABCD EFGH" reads as 8 consecutive.
    max-caps: 8
    # Maximum share of uppercase among all letters, 0-100. Catches full-caps
    # sentences whose individual words stay under max-caps. 0 disables this half.
    percentage: 70
    # Messages shorter than this skip the check (avoids "OK", "GG").
    min-length: 4
    # Whether a caps violation also alerts snchat.notify holders.
    notification: true

  # Bypass: snchat.bypass.flood.
  flood:
    # Whether repeated characters are checked at all.
    enabled: false
    # true collapses the repeated run; false blocks the message outright.
    replace: true
    # Maximum consecutive identical characters. 4 allows four and catches five.
    # Floored at 1: a 0 would strip every character of a corrected message and
    # deliver a blank line.
    max-repeat: 4
    # Messages shorter than this skip the check.
    min-length: 4
    # Whether a flood violation also alerts snchat.notify holders.
    notification: true

  # Blocks plugin:command syntax such as /essentials:home.
  # Bypass: snchat.bypass.syntax.
  # /paper:callback is always allowed: it is not a real command, it is how Paper
  # delivers a click on an [inv] or [ec] tag back to the server.
  syntax:
    # Whether the plugin:command form is blocked at all.
    enabled: true
    # Case-insensitive PREFIX matches that stay allowed, e.g. "/strikepractice:".
    # Blank entries are ignored: one would otherwise match every command.
    whitelist: []
    # Whether a syntax violation also alerts snchat.notify holders.
    notification: true

# ------------------------------------------------------------
#  Per-group command whitelist. The group lists themselves live in
#  blockcommands.yml.
# ------------------------------------------------------------
command-blocker:
  # Master switch. When false nothing is blocked and no placeholder command
  # is registered.
  enabled: true
```

## announcements.yml

Created after the plugin enables, so it appears only once the license check has passed.

```yaml
# ============================================================
#  SnChat - automatic announcements
#  Managed by SnLib: new keys are auto-merged on boot; your values and
#  comments are preserved.
#  Supports [center], &#RRGGBB, & codes and %papi%.
# ============================================================

# Whether the rotation timer runs at all.
enabled: true

# Seconds between announcements. This is BOTH the initial delay and the period,
# so nothing is broadcast at server start.
interval: 300

# Sound played to each recipient after the lines, as "SOUND_ID [volume] [pitch]".
# The id resolves as an open set, so both ENTITY_EXPERIENCE_ORB_PICKUP and
# minecraft:entity.experience_orb.pickup work. Use "none" for silence.
sound: "ENTITY_EXPERIENCE_ORB_PICKUP 10.0 1.0"

# The keys below, cycled in order. A key with no entry under announcements:
# is skipped for that turn. An empty list disables the rotation.
rotation:
  - vote
  - discord

# ------------------------------------------------------------
#  Announcement definitions. Each entry needs lines, hover, click-action and
#  click-type. click-type is OPEN_URL, RUN_COMMAND or SUGGEST_COMMAND; an
#  unrecognised value falls back to SUGGEST_COMMAND with a warning.
#  Every line of one announcement carries the same hover and the same click.
# ------------------------------------------------------------
# sn:extensible
announcements:
  vote:
    hover:
      - "&aClick to vote for the server!"
    click-action: "https://example.com/vote"
    click-type: OPEN_URL
    lines:
      - "&8&m                                                       "
      - "[center]&#FF9B00&lVOTE FOR US"
      - "[center]&7Support the server by voting daily!"
      - "[center]&eClick here to vote"
      - "&8&m                                                       "
  discord:
    hover:
      - "&bJoin our Discord!"
    click-action: "https://discord.gg/example"
    click-type: OPEN_URL
    lines:
      - "&8&m                                                       "
      - "[center]&#7289DA&lDISCORD"
      - "[center]&7Join our community!"
      - "[center]&eClick to join"
      - "&8&m                                                       "
```

An announcement key does not have to be in `rotation`. A key configured but left out of the rotation is still sendable with `/snchat announce <name>`, which is how you keep a manual-only announcement.

## blockcommands.yml

Created after the plugin enables. Read the header carefully before your first restart.

```yaml
# ============================================================
#  SnChat - command whitelist per LuckPerms group
#  Managed by SnLib: new keys are auto-merged on boot; your values and
#  comments are preserved.
#
#  Each group lists the commands its members may run AND see in tab
#  completion. Anything not listed is blocked and hidden.
#  The master switch lives in config.yml under command-blocker.enabled.
# ============================================================
#
#  READ THIS BEFORE YOUR FIRST RESTART
#  This module is ACTIVE out of the box: command-blocker.enabled defaults to
#  true and the "default" group below is already populated. So from the first
#  boot, every player in "default" - and every group NOT listed here, which
#  falls back to "default" - can only run the commands listed under it.
#  Operators hold snchat.bypass.blockcommands and notice nothing, so this is
#  easy to miss until a player reports it.
#  To turn the module off entirely, set command-blocker.enabled: false in
#  config.yml. To keep it on, add your own groups below.
#
#  HOW A GROUP IS RESOLVED
#    bypass: true       - the group sees and runs everything
#    inherit: <group>   - adds every command of that group (single group, not
#                         a list; the chain is followed and cycles are safe)
#    commands:          - the group's own commands, written without a leading /
#
#  Group names are matched case-insensitively against the player's LuckPerms
#  primary group.
#
#  TWO SAFETY NETS, in this order:
#    1. A group with no entry here falls back to the "default" group's list.
#    2. If "default" is missing too, the resolved list is empty and EVERY
#       command is allowed. A server that never configured this module is
#       never locked out of it.
#
#  A consequence worth knowing: an EMPTY list is treated the same as no entry.
#  Writing "commands: []" for a group does NOT mean "this group runs nothing" -
#  it falls back to "default", and to allow-all if "default" is empty too. To
#  restrict a group to nothing, give it a list with one harmless command in it.
#
#  SnChat's own commands - and any alias you set in config.yml - are always
#  runnable and always visible, whatever the whitelist says, so a typo here can
#  never lock you out of fixing it.
#
#  A whitelisted command no installed plugin provides is registered as a silent
#  placeholder, so it tab-completes instead of printing "Unknown command".
#  Those placeholders carry no permission, so on a server missing the plugin
#  that owns them they are visible in tab to anyone who is not filtered - each
#  one doing nothing when run. That is the cost of the line above.
#
#  ONE COMMAND IS ALWAYS ALLOWED: /paper:callback. It is not a real command and
#  it is never typed - it is how Paper delivers a click on a [inv] or [ec] tag
#  back to the server. Blocking it would make those tags do nothing for every
#  player without a bypass. The same exception is made by the plugin:command
#  syntax check in config.yml.
# ============================================================

# sn:extensible
groups:
  default:
    commands:
      - help
      - spawn
      - home
      - sethome
      - delhome
      - tpa
      - tpaccept
      - tpdeny
      - msg
      - r
      - reply
      - snchat

  mod:
    inherit: default
    commands:
      - ban
      - tempban
      - kick
      - mute
      - warn
      - tp
      - vanish

  admin:
    bypass: true
```

## lang/messages_en.yml

Every player-facing string. Set `lang:` in `config.yml` to load a different `messages_<code>.yml`.

```yaml
# ============================================================
#  SnChat - language file (English)
#  Managed by SnLib: new keys merge in on boot, your values are kept.
# ============================================================

# Prefix prepended to every single-line message sent via sn.lang().send(...)
prefix: "&#8354f2&lSnChat &8| &7"

# snlib.* are the shared command messages: permission errors, usage lines, the
# generated help. Restyle them like any other value here.
# Deleting one does not stick: this block is schema rather than owner content,
# so the styled value shipped in the jar is merged back in on the next boot.
# Placeholders {plugin} {usage} {description} {page} {total} {min} {max}
# {value} {command} are filled in for you. NEVER put {prefix} here (it is
# auto-prepended).
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
    header: "&8&m----------&r &#8354f2&lSnChat &8&m----------"
    entry: "&#8354f2{usage} &7{description}"
    footer: "&7Page &f{page}&7/&f{total} &8- &#8354f2/{command} help <page>"

# Your own plugin messages. Do NOT write {prefix} in any value: SnLib
# auto-prepends the prefix above to every single-line message sent via
# sn.lang().send(...). A literal {prefix} renders verbatim, and SnLib
# logs a one-time WARN when the active lang file contains one.
messages:

  # ---------- General ----------
  # Sent when a command that needs a player is run from the console.
  player-only: "&cThis command can only be used by players."
  # The only place the three chat tokens are ever advertised to a player.
  # Generated command help cannot cover them: they are not commands.
  # {item} {inventory} {enderchest} are the FIRST alias configured for each
  # family in config.yml, so renaming a token there renames it here too. A
  # family you disabled contributes an empty value.
  showcase-hint: "&#8354f2{item} {inventory} {enderchest} &8- &7Share your held item, inventory or ender chest"

  # ---------- Showcase tokens ----------
  showcase-no-permission: "&cYou do not have permission to share items."
  item-blacklisted: "&cYou cannot share this item in chat."
  snapshot-expired: "&cThat snapshot has expired."
  snapshot-no-permission: "&cYou do not have permission to open this."
  # Shown when the viewer cannot be opened at all, which means one of the
  # guis/*.yml files is missing or unreadable. Check the server log.
  snapshot-unavailable: "&cThe snapshot viewer is unavailable right now."

  # ---------- Announcements ----------
  announcement-not-found: "&cAnnouncement not found: &f{name}"
  # The announcement exists but could not be rendered for that player. The
  # console line says which placeholder or click value broke.
  announcement-failed: "&cAnnouncement &f{name} &ccould not be sent. Check the console."
  announcement-sent: "&aAnnouncement &f{name} &abroadcast."
  announcement-sent-to-player: "&aAnnouncement &f{name} &asent to &f{player}&a."

  # ---------- Chat control ----------
  cooldown-message: "&cPlease wait before sending another message."
  cooldown-repeat: "&cPlease do not repeat the same message."
  cooldown-global-repeat: "&cThat message was sent too recently."
  cooldown-command: "&cPlease wait before using another command."
  caps-blocked: "&cExcessive capitalization is not allowed."
  flood-blocked: "&cExcessive character repetition is not allowed."
  syntax-blocked: "&cUsing plugin:command syntax is not allowed."
  mutechat-denied: "&cChat is currently muted."
  mutechat-enabled: "&cChat has been muted by staff."
  mutechat-disabled: "&aChat has been unmuted."
  clearchat-cleared: "&aChat has been cleared by &f{player}&a."
  # The alerts toggle lasts for the session: alerts come back on at next login.
  alerts-enabled: "&aViolation alerts enabled."
  alerts-disabled: "&cViolation alerts disabled &7(until you relog)&c."
  feature-disabled: "&cThis feature is disabled."
  # Staff violation alert. This one is spliced into chat rather than sent as a
  # message, so it never receives the SnChat prefix and carries its own [CC]
  # badge instead - no [noprefix] tag needed.
  # {detail} is what the player typed, neutralized so it cannot restyle the
  # alert or run anything. Be aware of exactly what that means for a reader:
  # a MiniMessage tag is shown as plain text ("<red>" arrives as "<red>"), but
  # colour codes and percent signs are DELETED, not shown - "&cSHOUT" arrives
  # as "SHOUT" and "50% off" as "50 off".
  staff-notification: "&8[&cCC&8] &f{player} &7triggered &c{module}&7: &f{detail}"

  # ---------- Command blocker ----------
  command-blocked: "&cYou do not have permission to use this command."
  blocker-status: "&7Command Blocker is {status} &7(&f{groups} &7groups configured)."

# ------------------------------------------------------------
#  Text spliced INTO a chat line rather than sent as a message. These never
#  receive the prefix, because they are fragments, not messages.
# ------------------------------------------------------------
chat:
  # The [item] tag itself. {item_name} is replaced by the item COMPONENT, so the
  # item keeps its own colours and its tooltip. A value without {item_name}
  # renders alone and drops the name.
  item-format: "&8[&f{item_name}&8]"
  # The [inv] and [ec] tags, and the tooltip shown when hovering them.
  inventory-format: "&8[&aInventory&8]"
  inventory-hover:
    - "&aInventory of &f{player}"
    - "&7Click to open"
  enderchest-format: "&8[&dEnder Chest&8]"
  enderchest-hover:
    - "&dEnder Chest of &f{player}"
    - "&7Click to open"

# ------------------------------------------------------------
#  Short state words. Each is substituted into a {status}, {module} or {detail}
#  slot of the messages above, so a restyle here applies everywhere at once.
# ------------------------------------------------------------
status:
  enabled: "&aenabled"
  disabled: "&cdisabled"
  # Shown in place of the item tag when the sender's hand is empty.
  none: "&8[&7Nothing&8]"
  # Shown instead of a player name when the sender is the console.
  console: "Console"

# ------------------------------------------------------------
#  Names of the chat control modules, substituted into {module} of the staff
#  alert above. Restyle the values; the keys are read by name, so renaming one
#  makes its module fall back to the English word shipped here.
# ------------------------------------------------------------
# sn:extensible
chat-control-modules:
  cooldown: "cooldown"
  caps: "caps"
  flood: "flood"
  syntax: "syntax"

# ------------------------------------------------------------
#  The pieces a {detail} value is built from. Same key rule as above: restyle
#  the values, do not rename the keys.
#  The first three name a cooldown reason. The command-delay clock has no label
#  of its own - its detail is the command that was typed.
#  The last two belong to the detail formatter and are used by every module.
# ------------------------------------------------------------
# sn:extensible
chat-control-details:
  too-fast: "too fast"
  repeat: "repeat: "
  global-repeat: "global repeat: "
  # Appended to any detail longer than 40 characters.
  truncated: "..."
  # Shown in place of a command's arguments. They are never put in an alert: the
  # arguments of a login or password command are the most sensitive text this
  # plugin handles, and naming the command is enough to judge command spam.
  redacted: "&8[args hidden]"
```

## guis/inventory.yml

The window opened by clicking an `[inv]` tag. `{player}` in the title is the snapshot owner, never the person looking at it.

```yaml
# ============================================================
#  SnChat - [inv] snapshot viewer
#  Managed by SnLib: new keys are auto-merged on boot; your values and
#  comments are preserved.
#
#  Opened by clicking an [inv] tag in chat. It shows a frozen copy of the
#  sender's inventory as it was when the message was sent. Every click and
#  drag inside it is cancelled and no item can leave it.
# ============================================================

# Title of the viewer window. {player} is the name of the player whose inventory
# this is - the sender of the message, never whoever is looking at it.
# Any OTHER %placeholder% written here resolves against the VIEWER, not the
# owner, which is usually the opposite of what a title like this wants.
# (%player% is the one exception: it is intercepted and gives the owner's name,
# same as {player}. Write {player} anyway - it is the form that always means
# the owner.)
title: "&#8354f2&l{player}'s Inventory"

# Slot map: one line per row, one character per slot, read left to right and
# top to bottom. A space is an empty slot. Move a letter to move what it holds;
# delete it to leave that content out.
#   h helmet      c chestplate   l leggings   b boots
#   m main hand   o off hand     p the sender's head
#   s the sender's main inventory, 27 cells in order
#   t the sender's hotbar, 9 cells in order
#   # filler
layout:
  - "hclb#m#op"
  - "#########"
  - "sssssssss"
  - "sssssssss"
  - "sssssssss"
  - "ttttttttt"

# Ordered cell groups the plugin fills one item per cell. Reordering their
# cells reorders the picture only; each item keeps its own identity.
regions:
  storage: s
  hotbar: t

items:
  # Placed in every cell marked '#'.
  filler:
    material: GRAY_STAINED_GLASS_PANE
    display-name: " "
    key: "#"

# The cells the sender's real items are painted into. The item supplies
# everything visible - material, name, lore, enchantments - so of what is
# declared here only a display-name or a lore would reach the cell, and neither
# is declared: the replica shows each item exactly as its owner had it.
#
# A cell the sender had empty stays empty: nothing is painted into it at all,
# and nothing you set here changes that. These entries only describe cells that
# DO hold an item.
templates:
  helmet:
    key: h
  chestplate:
    key: c
  leggings:
    key: l
  boots:
    key: b
  main-hand:
    key: m
  off-hand:
    key: o
  # Painted over every cell of the storage and hotbar regions. Both fields are
  # left empty on purpose: that is "declare nothing", so the item crosses to the
  # cell untouched, keeping its own name and lore.
  slot:
    display-name: ""
    lore: []

  # Head of the player the snapshot belongs to. This cell is rendered from the
  # definition below rather than from an item. {player} is their name.
  owner:
    key: p
    material: PLAYER_HEAD
    skull-owner: "{uuid}"
    display-name: "&#8354f2&l{player}"
    lore:
      - "&7Inventory as it was when"
      - "&7the message was sent."
```

## guis/enderchest.yml

The window opened by clicking an `[ec]` tag.

```yaml
# ============================================================
#  SnChat - [ec] snapshot viewer
#  Managed by SnLib: new keys are auto-merged on boot; your values and
#  comments are preserved.
#
#  Opened by clicking an [ec] tag in chat. It shows a frozen copy of the
#  sender's ender chest as it was when the message was sent. Every click and
#  drag inside it is cancelled and no item can leave it.
# ============================================================

# Title of the viewer window. {player} is the name of the player whose ender
# chest this is - the sender of the message, never whoever is looking at it.
# Any OTHER %placeholder% written here resolves against the VIEWER, not the
# owner, which is usually the opposite of what a title like this wants.
# (%player% is the one exception: it is intercepted and gives the owner's name,
# same as {player}. Write {player} anyway - it is the form that always means
# the owner.)
title: "&#8354f2&l{player}'s Ender Chest"

# Slot map: one line per row, one character per slot, read left to right and
# top to bottom. A space is an empty slot.
#   e the sender's ender chest, 27 cells in order
layout:
  - "eeeeeeeee"
  - "eeeeeeeee"
  - "eeeeeeeee"

# Ordered cell group the plugin fills one item per cell. Reordering its cells
# reorders the picture only; each item keeps its own identity.
regions:
  contents: e

# The cells the sender's real items are painted into. The item supplies
# everything visible - material, name, lore, enchantments - so of what is
# declared here only a display-name or a lore would reach the cell, and neither
# is declared: the replica shows each item exactly as its owner had it.
#
# A cell the sender had empty stays empty: nothing is painted into it at all,
# and nothing you set here changes that. The entry below only describes cells
# that DO hold an item.
templates:
  # Both fields are left empty on purpose: that is "declare nothing", so the
  # item crosses to the cell untouched, keeping its own name and lore.
  slot:
    display-name: ""
    lore: []
```
