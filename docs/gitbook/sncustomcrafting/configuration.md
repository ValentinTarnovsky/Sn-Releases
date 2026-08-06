# Configuration

SnCustomCrafting ships with the following YAML files. New keys are auto-merged on boot; your edits and comments are preserved. Set `update-configs: false` in `config.yml` to freeze them, in which case SnLib only warns about missing keys instead of inserting them.

Sections marked `# sn:extensible` are yours: entries you delete there stay deleted.

`workstations.yml` is not listed below. It is runtime state written by `/customcrafting set` and `/customcrafting remove`, never merged and never seeded, so a workstation you remove stays removed.

## config.yml

```yaml
# ============================================================
#  SnCustomCrafting - configuration
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

# Runtime debug output (also toggleable live via /customcrafting debug).
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
  # Aliases of /customcrafting. Re-read on /customcrafting reload.
  aliases: [cc]

# ------------------------------------------------------------
#  Database. type=sqlite needs nothing else; type=mysql reads host/port/etc.
# ------------------------------------------------------------
database:
  # sqlite or mysql
  type: sqlite
  # MySQL connection (ignored when type is sqlite).
  host: localhost
  port: 3306
  database: sncustomcrafting
  username: root
  password: ""
  # How long the driver may block, in seconds. The shutdown flush has its own budget and
  # stops waiting when it expires; these two stop a SINGLE connect or read from outlasting
  # that budget inside the driver, where nothing here could interrupt it.
  # Opening a connection. 1 to 3600. Applies on BOTH backends. Raise it if a remote MySQL
  # under load logs "the crafts table could not be created" on boot - the table is created
  # once, and failing it turns persistence off for the whole session.
  connect-timeout-seconds: 10
  # A read on an already-open connection. MySQL only; SQLite is a local file and ignores it.
  # 0 to 3600, and 0 means unlimited - which is what hangs forever on a link that drops
  # silently instead of refusing.
  socket-timeout-seconds: 30

# ------------------------------------------------------------
#  Workstations and recipe sets.
# ------------------------------------------------------------
settings:
  # Recipe set used by a workstation that does not name one of its own.
  # It needs a matching section in recipes.yml and a menu at guis/<set>.yml.
  default-recipe-set: "default"
  # Protect registered workstation blocks from being broken, blown up, burnt or
  # pushed by a piston. /customcrafting remove is then the only way to unregister one.
  protect-workstation-blocks: true
  # Max distance of the /customcrafting set raycast, in blocks. 1 to 64;
  # anything outside is pulled to the nearest end and the console says so.
  target-block-range: 5
  # Seconds between two "workstation busy" messages for the same player.
  # 0 to 3600. 0 means no throttle at all: a player holding right-click on a
  # busy workstation is then messaged on every single click.
  busy-message-cooldown-seconds: 2
  # Seconds the server stop may spend saving running crafts. 1 to 45 - the ceiling stays
  # under the server watchdog, which would otherwise kill the JVM mid-save.
  # Keep it ABOVE database.connect-timeout-seconds. Crafts are saved in order so a
  # claimed one cannot come back, and this budget is what pays for that order; if the
  # first attempt against an unreachable database can eat all of it, every craft after
  # it is saved without that guarantee. Past the budget the crafts are still handed to
  # the database, just no longer waited for, and the console says so.
  shutdown-flush-seconds: 30

# ------------------------------------------------------------
#  PlaceholderAPI. Replace <id> with a workstation id.
#    %customcrafting_status_<id>%       -> recipe display name
#    %customcrafting_time_<id>_short%   -> remaining time, short format
#    %customcrafting_time_<id>_long%    -> remaining time, full format
#    %customcrafting_progress_<id>%     -> progress percent
#  Short-format aliases:
#    %customcrafting_time_<id>%  and  %customcrafting_<id>_time%
#
#  Only the progress TEMPLATE lives here. What a placeholder shows when there
#  is nothing to report is text a player reads, so it lives with the rest of
#  the text, under "status:" in lang/messages_en.yml.
# ------------------------------------------------------------
placeholders:
  # Template of the progress placeholder; {percent} is substituted.
  # Plain colour codes work: &a, &#RRGGBB.
  # Do NOT put {percent} inside anything that rewrites or colours the characters
  # one by one - [small], [rgb], a <gradient> or a <rainbow>. [center] is safe for
  # the token but pads the line for the literal "{percent}" (9 characters) rather
  # than for the 1-3 the number takes, so the result sits off-centre. The template is
  # rendered BEFORE the number is substituted, and those tags either change the
  # token's letters or push a colour code between them, so "{percent}" is no
  # longer there to replace and players see the token itself. Style the rest of
  # the line and leave the token bare:
  #   good: "&#8354f2Progress: &f{percent}%"
  #   bad:  "<gradient:#ff5555:#55ff55>{percent}%</gradient>"
  progress-format: "{percent}%"

# ------------------------------------------------------------
#  Duration rendering. Each unit is a template; {hours} {minutes} {seconds}
#  are substituted. hide-zero-units drops a unit whose value is 0, and zero is
#  what renders when nothing is left to show.
# ------------------------------------------------------------
time-format:
  # Full format: used in messages and by the _long placeholder.
  full:
    hours: "{hours}h"
    minutes: "{minutes}m"
    seconds: "{seconds}s"
    separator: " "
    hide-zero-units: true
    zero: "0s"

  # Short format: used by the short placeholders. It has no seconds unit by design.
  short:
    hours: "{hours}h"
    minutes: "{minutes}m"
    separator: " "
    hide-zero-units: true
    zero: "0m"
```

## items.yml

Every item the plugin can create, ingredients and results alike. Each top-level key is an item id that `recipes.yml` refers to.

```yaml
# ============================================================
#  SnCustomCrafting - items
#  Every item this plugin can create. Each top-level key is an item id.
#  Recipes reference these ids, and an item only counts as an ingredient
#  when this plugin created it - a renamed vanilla lookalike never does.
#  Hand one out with /customcrafting give <player> <item> [amount].
#  Managed by SnLib: new keys merge in on boot, your values are kept.
# sn:extensible-root
# ============================================================
#
#  Every item below is an example: rename it, restyle it or delete it.
#  A deleted entry stays deleted, so keep the ids used by recipes.yml in
#  sync with this file - a recipe requiring an id that is not defined here
#  is skipped with a warning on boot.
#
#  Keys used by these items:
#    material          vanilla material, or a head texture / base64 / URL
#    display-name      colour codes &a-&f, hex &#RRGGBB, tags [rgb] [small]
#    lore              one list entry per line; "" is a blank line
#    custom-model-data resource-pack model id; omit it when you have no pack
#  The full key list (glow, enchantments, flags, colour, durability, item
#  properties, interact actions) is in SnLib's docs/item-example.yml.

# ------------------------------------------------------------
#  Ingredients - what a player spends to start a craft.
# ------------------------------------------------------------

# Required by the magic_seed recipe.
magic_stick:
  material: STICK
  display-name: "&bMagic Stick"

# Required by the magic_seed recipe.
common_seeds:
  material: WHEAT_SEEDS
  display-name: "&fCommon Seeds"
  lore:
    - "&7Crafting ingredient."

# Required by the fertilizer recipe.
bone_dust:
  material: BONE_MEAL
  display-name: "&fBone Dust"
  lore:
    - "&7Crafting ingredient."

# ------------------------------------------------------------
#  Results - what a recipe hands back, through its reward lines.
# ------------------------------------------------------------

# Handed out by the magic_seed recipe.
# The lore describes the item itself. The menu icon is a separate thing, declared
# under the recipe's display: block in recipes.yml - that is where the crafting
# time, the requirement list and "Click to craft" belong.
magic_seed:
  material: WHEAT_SEEDS
  display-name: "&aMagic Seed"
  custom-model-data: 1001
  lore:
    - "&7A seed charged with magic."

# Handed out by the fertilizer recipe.
fertilizer:
  material: BONE_MEAL
  display-name: "&eFertilizer Plus"
  lore:
    - "&7Makes your crops grow faster."
```

## recipes.yml

Recipe sets, and the recipes in each. A recipe declares its menu icon and slot, a `min`/`max` duration in whole minutes, the items it consumes and the console commands it runs when the craft is claimed.

```yaml
# ============================================================
#  SnCustomCrafting - recipes
#  Managed by SnLib: new keys are auto-merged on boot; your values and
#  comments are preserved.
#
#  A recipe set groups the recipes one workstation offers. Register a
#  workstation with /customcrafting set <id> [recipe-set]; a workstation that
#  names no set uses settings.default-recipe-set from config.yml.
#
#  Every set needs a menu file at guis/<set>.yml - the file name IS the set id.
#  A set with no menu file, no recipes or an unknown name falls back to the
#  default set instead of opening an empty menu.
#
#  Requirements name item ids declared in items.yml, never materials. Only this
#  plugin's own items are consumed, so a renamed vanilla stack never counts as
#  an ingredient. A requirement naming an id that items.yml does not declare
#  disables the whole recipe, with one line in the console saying which.
# ============================================================

# Recipe sets. Each key is one set id.
# sn:extensible
recipe-sets:

  default:
    recipes:

      magic_seed:
        # Menu slot of the icon, 0-based. A slot outside the menu is skipped.
        slot: 11
        # Icon shown in the menu, in the same format as items.yml. A recipe with
        # no display section is skipped.
        display:
          material: WHEAT_SEEDS
          display-name: "&aMagic Seed"
          lore:
            - "&7Creates a powerful seed."
            - "&7Duration: &f90-120m"
            - ""
            - "&7Requirements:"
            - "&8- &fx10 Common Seeds"
            - "&8- &fx1 Magic Stick"
            - ""
            - "&eClick to craft"
          glow: false
          custom-model-data: 1001
        # Craft time in whole minutes. A value between min and max is rolled when
        # the craft starts, and only counts down while the player is online.
        duration:
          min: 90
          max: 120
        # Items consumed when the craft starts; item is an id from items.yml.
        requirements:
          - item: common_seeds
            amount: 10
          - item: magic_stick
            amount: 1
        # Console commands run when the player claims the finished craft.
        # {player} becomes their name, and customcrafting give is how a plugin
        # item is handed out - a vanilla /give cannot produce one.
        #
        # Use the full command name here, not the cc alias: aliases are owner
        # editable in config.yml and another plugin may already own /cc, and a
        # reward line that stops resolving pays out nothing while the craft is
        # still consumed.
        rewards:
          - "customcrafting give {player} magic_seed 1"
          - "msg {player} &aYou received your Magic Seed!"

      fertilizer:
        slot: 15
        display:
          material: BONE_MEAL
          display-name: "&eFertilizer Plus"
          lore:
            - "&7Speeds up your crops."
            - "&7Duration: &f45-60m"
          glow: false
        duration:
          min: 45
          max: 60
        requirements:
          - item: bone_dust
            amount: 64
        rewards:
          - "customcrafting give {player} fertilizer 10"
```

## guis/default.yml

The menu of the recipe set named `default`. The file name **is** the set id: copy this file to `guis/<set>.yml` to give another set its own menu. A set with no file of its own opens this one.

Recipes are not listed here. Each is placed by its own `slot` in `recipes.yml` and looks exactly like its `display` block there.

```yaml
# ============================================================
#  SnCustomCrafting - menu of the "default" recipe set
#
#  The file name IS the recipe set id: this file is the menu of the set named
#  "default" in recipes.yml. To give another set its own menu, copy this file
#  to guis/<set>.yml and restyle it - a set with no file of its own opens this
#  one instead.
#
#  Recipes are NOT listed here. Every recipe of the set is placed by its own
#  "slot" in recipes.yml and looks exactly like its "display" block there, so
#  adding a recipe never means editing this file.
#
#  Managed by SnLib: new keys merge in on boot, your values are kept.
# ============================================================

# Menu title. Read once per boot; PlaceholderAPI tokens resolve per viewer.
title: "&8Custom Crafting"

# Only a plain left/right click (and its shift form) may act on a cell. Leave
# this on. Clicking a recipe SPENDS the ingredients and occupies the workstation
# for the whole craft, and there is no undo; without this, a number key, Q, F or
# the middle button over a recipe pays for a craft the player never asked for.
strict-clicks: true

# Cell mask, one line per row, 9 characters per line. 'f' is a border cell and
# a space is a free cell a recipe can sit on. The number of lines is the number
# of rows.
layout:
  - "fffffffff"
  - "         "
  - "fffffffff"

# sn:extensible
items:

  # Border. Renders in every 'f' cell above; take the character out of the
  # layout to drop it, or give it more cells to widen the frame.
  border:
    material: GRAY_STAINED_GLASS_PANE
    display-name: "&f"
    key: f

templates:

  # Behaviour of a recipe cell. The plugin binds one of these per recipe of the
  # set and hands over the icon built from that recipe's "display" block, so
  # the look lives in recipes.yml and only the click lives here. {id} is the
  # clicked recipe's id.
  #
  # Declare NO display-name and NO lore here: a display-name REPLACES the
  # recipe's own name and lore lines are APPENDED after its own, so the icon
  # would stop matching what recipes.yml says.
  recipe:
    click-actions:
      - "[craft] {id}"
```

## lang/messages_en.yml

All player-facing text, including the placeholder stand-ins under `status:`.

```yaml
# ============================================================
#  SnCustomCrafting - language file
#  Managed by SnLib: new keys merge in on boot, your values are kept.
#  Restyle or translate any line freely; your edits survive updates.
#
#  This path is FIXED. SnLib only ever copies lang/messages_en.yml out of the
#  jar, so this file is the one that reaches disk no matter what config.yml's
#  "lang" key says. To run another language, translate the values here, or add
#  your own lang/messages_<code>.yml BY HAND next to this file and point
#  "lang" at it - a file the jar does not ship is never written for you.
# ============================================================

# Prefix prepended to every single-line message sent via sn.lang().send(...)
prefix: "&#8354f2&lSnCustomCrafting &8| &7"

# snlib.* is SnLib's shared command contract (12 keys). Every Sn plugin ships
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
    entry: "&#8354f2{usage} &7{description}"
    footer: "&7Page &f{page}&7/&f{total} &8- &#8354f2/{command} help <page>"

# Your own plugin messages. Do NOT write {prefix} in any value: SnLib
# auto-prepends the prefix above to every single-line message sent via
# sn.lang().send(...). A literal {prefix} renders verbatim, and SnLib
# logs a one-time WARN when the active lang file contains one.
success:
  # {time} is the rolled duration of the craft that just started.
  craft-started: "Craft started! It will take &f{time}&7."
  craft-claimed: "Job finished! Rewards received."
  # {count} crafts were finished instantly by /customcrafting bypass.
  craft-bypassed: "&aCompleted &f{count}&a active crafts."
  workstation-set: "Workstation &e{id} &7registered on this block."
  workstation-removed: "Workstation &c{id} &7removed."
  # {amount}x {item} handed to {player} by /customcrafting give.
  item-given: "&aYou gave &f{amount}x {item} &ato &f{player}&a."

errors:
  players-only: "&cThis command is for players only."
  data-not-loaded: "&cYour data is still loading. Try again in a moment."
  recipe-set-not-found: "&cNo recipe set called &f{set} &cexists."
  workstation-not-found: "&cNo workstation called &f{id} &cexists."
  # Rejected before anything is written: the id becomes a key in workstations.yml, and a
  # separator like a dot is read back as a nested path instead of as an id.
  invalid-workstation-id: "&cThe id &f{id} &cis not valid. Use lowercase letters, numbers, &f_ &cand &f-&c only."
  item-not-found: "&cNo item called &f{item} &cexists."
  look-at-block: "&cLook at a valid block."
  missing-materials: "&cYou do not have enough materials."
  # One line per missing ingredient, listed under missing-materials. [noprefix]
  # keeps it flush with the list instead of repeating the plugin prefix.
  missing-item: "[noprefix]&8- &c{amount}x {item}"
  # {time} is what is left on the craft occupying the workstation.
  craft-occupied: "&cWorkstation busy! It finishes in &e{time}"
  no-active-crafts: "&cYou have no active crafts."
  # The craft finished, but its recipe is no longer in recipes.yml, so there is nothing to
  # hand over yet. The craft is NOT lost: it stays claimable and pays out as soon as the
  # recipe (and its set) are back. Without this line the block would simply go dead.
  craft-unavailable: "&cThat craft finished, but its recipe no longer exists. Tell an admin: it is kept until the recipe is back."

# ------------------------------------------------------------
#  Stand-ins the PlaceholderAPI placeholders show when there is nothing to
#  report. They live here rather than in config.yml because they are text a
#  player reads, so translating this file translates them too.
#
#  These are NOT colour-rendered the way a message is: they are written into
#  whatever scoreboard, tab or hologram asked for the placeholder. Plain codes
#  (&7, &#RRGGBB) work; MiniMessage tags do not.
# ------------------------------------------------------------
status:
  # Shown when no craft is running, or the player is offline or still loading.
  # It is what the status, time AND progress placeholders answer for a player
  # whose data has not been read yet, so it is seen for a second after a join.
  no-craft: "&7None"
  # Shown by the time placeholders when there is no craft to report. Delete the
  # key to reuse no-craft instead.
  time-fallback: "0m"
  # Shown by the progress placeholder when there is no craft to report.
  progress-fallback: "0%"
```
