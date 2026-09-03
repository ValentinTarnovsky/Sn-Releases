# Configuration

SnCompanions ships with the YAML files below. `config.yml`, the language files and the menu layouts
are managed by SnLib: new keys are auto-merged on boot, and your values and comments are
preserved. Set `update-configs: false` to freeze them, in which case SnLib only warns about
missing keys instead of inserting them.

`eggs.yml` and the `companions/` folder are seeded once, on a
fresh install, and never written to again. A companion file you delete stays deleted, and a file you
add is picked up on the next reload; one file is one companion, and the file name is its id. Eggs
work the same way but live as keys inside `eggs.yml` rather than as separate files - see
[eggs.yml](#eggsyml).

## config.yml
```yaml
# ============================================================
#  SnCompanions - configuration
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

# Runtime debug output (also toggleable live via /companions debug).
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
  # Aliases of /companions. Re-read on /companions reload.
  aliases: [companion]

  # Companions listed per page by /companions admin list. Raising it makes one command
  # print more lines at once and pushes more of the sender's chat history
  # off screen; 1 to 50, values outside that are clamped.
  list-page-size: 8

# ------------------------------------------------------------
#  Database. type=sqlite needs nothing else; type=mysql reads host/port/etc.
#  ONE SERVER PER DATABASE: SnCompanions loads a player on join, caches them in
#  memory and saves on quit, so two servers pointing at the same MySQL will
#  overwrite each other and silently destroy companions. Cross-server sharing is
#  not supported.
# ------------------------------------------------------------
database:
  # sqlite or mysql
  type: sqlite
  # MySQL connection (ignored when type is sqlite).
  host: localhost
  port: 3306
  database: sncompanions
  username: root
  password: ""

# ------------------------------------------------------------
#  Public developer API.
# ------------------------------------------------------------
# Public developer API events. When false, no API event is dispatched (zero
# cost) and cancellable hooks report "not cancelled" so gameplay proceeds.
# The query facade stays available either way.
api-events:
  enabled: true

# ------------------------------------------------------------
#  Groups. A group is a free-text label a companion file points at with its
#  "group" key. It is not a rarity system: the plugin uses it for the
#  storage sort order, the bulk delete buttons, the companion lore and the colour
#  the menus draw that companion's lines in, nothing else. A companion whose group is
#  not listed here sorts last and gets no bulk delete button.
# ------------------------------------------------------------
# sn:extensible
groups:
  common:
    # Name shown in the companion lore and on the bulk delete button.
    display: "&7Common"
    # Colour prefix exposed to the menus as {group-color}: a legacy code (&7), a hex
    # (&#55ff55) or the SnLib [rgb] gradient tag. [rgb] only works when {group-color}
    # is the first thing on the line. Leave empty to reuse the colour display starts
    # with, which is what every install made before this key existed does.
    color: "&7"
    # Sort weight in the storage grid; lower sorts first.
    order: 1
  rare:
    display: "&9Rare"
    color: "&9"
    order: 2
  epic:
    display: "&5Epic"
    color: "&5"
    order: 3

# ------------------------------------------------------------
#  Companion storage.
# ------------------------------------------------------------
storage:
  # Companions a player may keep before permissions or purchases raise it. The
  # effective capacity is max(base-capacity, sncompanions.storage.<n>) plus whatever
  # was bought with "/companions admin storage give|set": the permission names the
  # absolute capacity a rank grants, and purchases add on top of it.
  base-capacity: 54

  # Ceiling of the TOTAL storage that "/companions admin storage give|set" may leave
  # a player with. 0 disables it, which is the shipped value: storage is
  # paginated, so unlike the equip slots it has no visual limit to agree with.
  # A command that would go past the ceiling is NOT cancelled - the purchase is
  # trimmed so the total lands on the ceiling, and says so. This does not limit
  # the "sncompanions.storage.<n>" permission grants, which are the floor purchases
  # stack onto.
  max-capacity: 0

# ------------------------------------------------------------
#  Equip slots.
# ------------------------------------------------------------
slots:
  # Companions a player may equip at once before permissions or purchases raise it.
  # The effective count is max(base-count, sncompanions.slots.<n>) plus whatever was
  # bought with "/companions admin slots give|set": the permission names the absolute
  # count a rank grants, and purchases add on top of it, so with base-count 1
  # a player who buys one slot equips two companions.
  base-count: 1

  # Ceiling of the TOTAL slots that "/companions admin slots give|set" may leave a
  # player with. 0 disables it. Keep it equal to the number of 's' cells in the
  # layout of guis/main.yml (7 in the shipped menu): the menu can only draw that
  # many, so slots past it are bought and never usable. The two files are not
  # read from each other on purpose - this comment is the link.
  # A command that would go past the ceiling is NOT cancelled - the purchase is
  # trimmed so the total lands on the ceiling, and says so. This does not limit
  # the "sncompanions.slots.<n>" permission grants, which are the floor purchases
  # stack onto.
  max-count: 7

  # Forbid equipping two companions that belong to the same group. Off by default: a
  # player may equip as many companions of one group as they have slots for. Companions that
  # declare no group are never blocked by this rule.
  unique-group: false

# ------------------------------------------------------------
#  Formation. The equipped companions stand behind their owner and turn with the
#  owner's camera. By default they form a straight LINE, side by side, at the
#  owner's feet, ordered by equip slot from the owner's right.
#  Set "shape" to CIRCLE or OVAL instead and they stand on an arc around the owner.
#  The arc is always used whole, end to end: the companions spread evenly across it
#  and slide over whenever one is equipped or unequipped.
#  "Evenly" means an even share of the ANGLE, which is an even spacing on the
#  ground only on a CIRCLE. On an OVAL the companions near the ends sit closer together
#  than the ones near the middle - about 23% closer at 4 companions and 28% at 6, at the
#  values below. 1, 2 and 3 companions are unaffected.
#  The radius / back-radius / arc-degrees / arc-center-offset keys apply to
#  CIRCLE and OVAL only; LINE reads the "line" block instead.
# ------------------------------------------------------------
formation:
  # CIRCLE and OVAL only. Blocks between the owner and the arc at their SIDES.
  # Under shape CIRCLE this is the distance in every direction; under shape OVAL
  # the back of the arc uses back-radius instead.
  radius: 2.0
  # LINE puts every companion on one straight line behind the owner, see "line" below.
  # CIRCLE keeps every companion at "radius" blocks. OVAL keeps "radius" at the owner's
  # sides and "back-radius" straight behind (and in front), so the arc hugs the
  # owner's back and opens out at the flanks.
  # Delete this key and the plugin uses CIRCLE, not the LINE shipped here. A value
  # that names none of the three logs one warning and falls back to CIRCLE too.
  shape: LINE
  # LINE only.
  line:
    # Blocks between two neighbouring companions on the line.
    spacing: 0.8
    # Blocks behind the owner the line runs. Negative puts it in front (screenshots).
    distance: 1.0
  # CIRCLE and OVAL only. Blocks between the owner and the arc straight behind them.
  # Read on every load but only USED when shape is OVAL; a CIRCLE ignores it in
  # favour of radius.
  back-radius: 1.4
  # Blocks above the owner's feet the companion ORIGIN sits at. A player head drawn
  # as a ground item hangs 1/16 of its scale below its origin, so
  # 0.0625 x animation.head-size (0.08 at the 1.3 below) rests it on the floor.
  # Raise it to make the companions float instead: 1.9 puts them at head height.
  height-offset: 0.08
  # CIRCLE and OVAL only. Total width of the arc in degrees, capped at 360. Read together with
  # arc-center-offset below: at the default centre of 180, an arc of 180 puts the
  # first companion exactly at the owner's right and the second exactly at their left,
  # and going past 180 swings both ends a little in FRONT of the shoulders (at the
  # 190 shipped here, 0.12 blocks). A LONE companion takes the right END of the arc, not
  # the middle, so a single companion moves forward with them. At a different
  # arc-center-offset "past 180" swings the ends somewhere else entirely.
  arc-degrees: 190.0
  # CIRCLE and OVAL only. Degrees from the owner's facing to the centre of the arc.
  # 180 puts the arc behind the owner, 0 in front of them, 90 at their right.
  arc-center-offset: 180.0
  # Where a companion looks: OWNER_YAW (the same way as the owner), OUTWARD (away from
  # the owner) or CENTER (at the owner).
  facing: OWNER_YAW
  # Degrees added to every companion's facing, to correct a head texture that is not
  # drawn looking forward. Under the default facing of OWNER_YAW, 180 points every
  # head OPPOSITE the owner's yaw - which is what faces them toward the camera in
  # third person, and toward anyone standing behind the owner. (Turning around
  # never shows you your own companions: the formation is camera-relative and comes with you.)
  # Note that 180 also swaps OUTWARD and CENTER.
  facing-offset: 180.0

# ------------------------------------------------------------
#  Animation. ONE shared task moves every companion of every player; there is no task
#  per companion and no task per player.
# ------------------------------------------------------------
animation:
  # Ticks between formation updates, minimum 2. The client interpolates between
  # updates, so 2 already looks fluid and a lower value would only cost tick.
  interval-ticks: 2
  # Uniform scale of a floating companion head.
  head-size: 1.3
  # Blocks a client keeps rendering a companion for, and the radius the plugin scans
  # for viewers.
  view-range: 48
  # Hands the movement packet write to a dedicated thread instead of the server
  # thread. Turn it off to send inline.
  async-packet-send: true
  # Vertical bob of a companion.
  bounce:
    # Blocks the companion rises and falls; 0 keeps the companions still on the ground.
    # 0.06 was the floating bob the companions had while they hovered.
    height: 0.0
    # Radians of bob phase advanced per animation tick; higher bobs faster.
    speed: 0.1

# ------------------------------------------------------------
#  BetterModel models. A companion whose file declares a "model" block is drawn as an
#  animated model instead of a floating head, with one animation while it holds
#  still and another while it moves. BetterModel is optional: without it, with
#  the switch below off, or when the engine does not know the model id, the companion
#  falls back to its head and nothing breaks. Companions that fell back pick their
#  model up on their own when BetterModel enables or reloads its models.
#  The vertical bob under "animation" is never applied to a model: a model
#  animates itself, so use height-offset below to place it instead.
# ------------------------------------------------------------
models:
  # Draw companions that declare a model as models. Off renders every companion as its head
  # even with BetterModel installed.
  enabled: true
  # Blocks per second of companion movement above which the moving animation plays.
  # The companions also travel when the owner only turns the camera, because the whole
  # formation swings with it, so looking around counts as moving.
  move-threshold: 0.8
  # Ticks the companion must stay below the threshold before the idle animation comes
  # back. Keeps a single step from flickering between the two animations.
  idle-delay-ticks: 8
  # Blocks added to a model companion's height, on top of formation.height-offset.
  # The two are SUMMED, and this one moves ONLY the companions drawn as models.
  # A model's own origin is at its feet, so at the shipped values 0.0 leaves it
  # standing 0.08 blocks above the floor; -0.08 (minus formation.height-offset)
  # puts its feet exactly on the ground.
  height-offset: 0.0
  # There is nothing to smooth here, which is why no setting for it exists: a
  # model companion's bones ride an invisible carrier that moves exactly like the heads
  # beside them, and the client interpolates that carrier on its own.

# ------------------------------------------------------------
#  EdTools boosters. An equipped companion whose file declares an "edtools-boosts"
#  block grants EdTools boosters: one per currency it names, plus the GLOBAL
#  enchant multiplier. EdTools is optional - without it, or with the switch
#  below off, nothing is granted and nothing breaks.
#
#  How the numbers work, because it surprises people:
#   - A companion declares its boost in PERCENT, exactly like its buff: 10.0 is +10%.
#     Every equipped companion's percentages for the same currency are SUMMED, and
#     nothing widens them: the value a companion file writes is the value that is
#     granted, exactly like its buff.
#   - EdTools is then handed a fraction (0.5 is "+50%", 1.0 is "double"),
#     ROUNDED TO ONE DECIMAL. That is a hard rule of this integration and it
#     means the granted boost moves in steps of 10%: a total of 4% rounds down
#     to nothing and grants no booster at all, 5% rounds up to +10%, and 26%
#     lands on +30%. Write your companions in tens if you want what you wrote.
#   - The boosters never expire and are never saved by EdTools. They exist for
#     exactly as long as the companion is equipped, and are re-applied on every join.
#   - Each entry of a companion's "edtools-boosts" block accepts an optional "max":
#     a ceiling in percentage points for THAT companion, applied before the summing
#     and before the rounding. It caps one companion, never the player's total.
#     Documented in companions/ember_fox.yml and in the GitBook.
#
#  Only the GLOBAL enchant multiplier can be boosted, never one named enchant:
#  that is the whole of what EdTools exposes to other plugins.
# ------------------------------------------------------------
edtools:
  # Let equipped companions grant EdTools boosters. Turning this off and reloading
  # removes the ones already granted; it does not wait for a restart.
  enabled: true
  # Name EdTools shows for this plugin's boosters in its own booster list.
  booster-display-name: "&dCompanions"
  # Ticks to wait after a player joins before writing their boosters, so EdTools
  # has finished restoring its own state first. 20 ticks = 1 second.
  join-delay-ticks: 20

# ------------------------------------------------------------
#  Holograms. Text drawn above every companion, riding the companion itself: the client
#  moves it with the companion, so it never lags behind. One line of text is one
#  packet entity; nothing is spawned for a companion whose lines are empty.
#  A companion file may declare its own "hologram:" block (enabled, lines,
#  height-offset); a companion that declares none uses default-lines below.
#  Placeholders: every companion placeholder of the menus ({companion} {level} {level-cap}
#  {exp} {exp-next} {percent} {group} {group-color} {buff} {buff-value}
#  {owner}). PlaceholderAPI tokens resolve against the OWNER.
#  The text is only rewritten when something on the companion changes - a level up,
#  an admin edit - never on the animation tick.
# ------------------------------------------------------------
holograms:
  # Master switch. Off spawns nothing at all.
  enabled: true
  # Blocks above the companion's origin the FIRST (top) line sits at. Careful: the
  # origin is not the same for both kinds of companion. A head companion's is the head
  # itself (formation.height-offset above the owner's feet); a model companion's is
  # its carrier, which also carries models.height-offset - so a model pushed
  # down to stand on the ground needs a LARGER offset here, or the text lands
  # inside it. A companion file can override this per companion, which is what the golem
  # example does.
  height-offset: 0.9
  # Blocks between two lines.
  line-spacing: 0.25
  # Uniform scale of the text.
  scale: 1.0
  # Background colour as ARGB hex; empty for no background. 40000000 is the
  # vanilla translucent black.
  background: ""
  # Draw the text with a shadow.
  shadow: false
  # Let the text show through blocks.
  see-through: false
  # Pixels before a line wraps.
  line-width: 200
  # How the text turns towards the player.
  #   vertical   - stays upright and turns only around the vertical axis, so the
  #                lines never lean however far above or below you stand
  #   center     - turns on both axes and tilts towards the camera
  #   horizontal - turns around the horizontal axis only
  #   fixed      - never turns, keeping the facing it was spawned at
  billboard: vertical
  # Lines of a companion whose file declares no hologram block. Empty draws nothing.
  default-lines:
    - "{companion}"
    - "&7Lv. &f{level}&7/&f{level-cap}"

# ------------------------------------------------------------
#  Visibility. Two independent switches per player, stored in the database and
#  kept across relogs, so all four combinations are valid:
#    /companions toggle -> hides the player's OWN companions, from themselves AND from
#                    everyone else (permission sncompanions.toggle)
#    /companions hide   -> hides EVERY other player's companions, for that player only
#                    (permission sncompanions.hide)
# ------------------------------------------------------------
visibility:
  # Seconds a player must wait between two uses of /companions toggle. 0 disables the
  # cooldown. It survives a relog, so reconnecting does not reset it.
  own-cooldown-seconds: 3
  # Seconds a player must wait between two uses of /companions hide. Counted
  # separately from the one above, so one toggle never blocks the other.
  others-cooldown-seconds: 3

# ------------------------------------------------------------
#  Experience. Every companion declares its own source and its own ratio in its
#  companions/<id>.yml file; this section only decides which sources are live at all,
#  how often the playtime one pays and what a level up feels like. All equipped
#  companions gain from the same event, each one from its own source. Stored companions
#  never gain anything, and a companion at its level cap freezes: it keeps what it had
#  and resumes exactly there if the cap ever rises.
# ------------------------------------------------------------
experience:
  # Master switch of every experience source. Off freezes every companion where it is.
  enabled: true

  # Per-source switches. Turning one off only stops the gain: no level is ever
  # touched and no companion loses anything.
  sources:
    # Companion experience per point of vanilla XP the owner picks up. The owner's XP
    # is observed, never consumed.
    vanilla-xp: true
    # Companion experience per block broken. A companion may declare a material whitelist in
    # its own file; without one every block counts, which on a mine or generator
    # world is thousands of events a second.
    block-break: true
    # Companion experience per block broken through an EdTools omnitool. This is NOT
    # the same event as block-break above: EdTools consumes the blocks its tools
    # break without ever firing a vanilla BlockBreakEvent, so a companion has to pick
    # the source its server actually produces. Needs EdTools installed; without
    # it nothing is registered and no class of it is ever loaded. A companion may also
    # restrict itself to certain tool ids with `experience.tools` in its own file.
    edtools-block-break: true
    # Companion experience per mob killed. A dead player is not a mob kill: PvP feeds
    # the damage-dealt source instead.
    mob-kill: true
    # Companion experience per point of damage the owner deals, melee and projectile,
    # counted after armor and after the companions' own damage buff.
    damage-dealt: true
    # Companion experience per minute the owner stays connected.
    playtime: true

  playtime:
    # Seconds between playtime grants. Every companion's playtime ratio is read per
    # MINUTE whatever this says, so changing it only changes how often the grant
    # is paid, never what a companion earns per minute of play.
    interval-seconds: 60

  level-up:
    # Sends messages.companion-level-up to the owner when one of their equipped companions
    # gains a level.
    message: true
    # Sound played to the owner on level up. "none" plays nothing.
    sound: "ENTITY_PLAYER_LEVELUP 1.0 1.6"

# ------------------------------------------------------------
#  Buffs. A companion declares which of the three it grants, what it gives at level 1
#  and what each level adds. The engine then computes, per buff:
#
#    base       = initial + (level - 1) x per-level
#    companion buff   = base, the value the companion's own file declares
#    player     = the sum of every equipped companion
#    effect     = that sum capped by the "cap" below
#
#  The two modifiers are added to each other and multiply the base once; the cap
#  applies LAST, to the summed total, never per companion. A total that comes out
#  negative is treated as zero.
# ------------------------------------------------------------
buffs:
  # Master switch of all three buffs.
  enabled: true

  damage:
    # Whether companions grant the damage buff at all.
    enabled: true
    # Highest total the buff may reach, in percent. 100 means at most double
    # damage.
    cap: 100.0
    # Apply the buff to melee hits.
    melee: true
    # Apply the buff to projectiles the owner fired.
    projectile: true
    # Apply the buff when the victim is another player. Off leaves PvE buffed
    # and PvP vanilla.
    pvp: true

  resistance:
    # Whether companions grant the resistance buff at all.
    enabled: true
    # Highest total the buff may reach, in percent. Capped at 95 whatever you
    # write here: invulnerability is not a state this plugin can produce.
    cap: 50.0
    # Damage causes the reduction applies to. VOID, KILL, FALL, DROWNING,
    # POISON, SUFFOCATION, SUICIDE and CUSTOM are refused with a warning: those
    # would either be an exploit or would shield a player from /kill and the
    # rest of the staff tools.
    causes:
      - ENTITY_ATTACK
      - ENTITY_SWEEP_ATTACK
      - PROJECTILE
      - ENTITY_EXPLOSION
      - BLOCK_EXPLOSION

  speed:
    # Whether companions grant the speed buff at all.
    enabled: true
    # Highest total the buff may reach, in percent. Applied as an attribute
    # modifier on movement speed, never through setWalkSpeed, so EssentialsX
    # /speed and impulse plates keep working. Values much above 40 are what
    # anticheats start flagging.
    cap: 40.0
    # Also apply the buff to flight speed. Off by default; turning it back off
    # removes the modifier from every player on the next reload.
    affect-flying: false

# ------------------------------------------------------------
#  Fusion. Two companions of the SAME id become the companion that their own
#  companions/<id>.yml names as its fusion target: it is a companion-to-companion map, never a
#  rarity ladder, and a companion whose file names no target cannot be fused at all.
#  The target, the success chance and the price of one attempt are declared per
#  companion in that file; this section holds only the rules every fusion obeys.
#  On a FAILED roll both parents are lost and the price is still paid. That is
#  a normal outcome of a legal attempt, not an error.
# ------------------------------------------------------------
fusion:
  # Master switch of the whole feature. Off refuses every fusion and every
  # Fuse All; nothing is deleted and no companion loses its fusion target.
  enabled: true

  # Which parent the result takes each preserved field from.
  #   BEST   - per FIELD, whichever parent holds the better value: the higher
  #            level and the higher experience. The result may take its level from
  #            one parent and its experience from the other.
  #   FIRST  - everything comes from the companion in the left input slot.
  #   SECOND - everything comes from the companion in the right input slot.
  keep-from: BEST

  # What survives a fusion at all. A field switched off here is not taken from
  # either parent: the result carries what a brand new companion carries, which is
  # level 1 and no experience.
  keep:
    # Carry over a level. It is capped by the TARGET companion's own max-level, since
    # every companion declares its own ceiling.
    level: true
    # Carry over the experience banked toward the next level.
    exp: true

  cost:
    # Charge the `fusion.cost` each companion file declares. The price is money, taken
    # through Vault or through whatever economy backend SnLib is configured
    # with. A server with no economy plugin at all fuses for FREE and says so
    # once in the console: refusing instead would make every companion that names a
    # price unfusable. Off makes every attempt free without editing companion files.
    enabled: true

  fuse-all:
    # Let the Fuse All button fuse every possible pair at once. Off leaves
    # single fusions working.
    enabled: true
    # Seconds the confirmation stays armed. Fuse All rolls the chance PER PAIR
    # and can destroy most of a collection, so the first click only shows how
    # many pairs would enter and a second click inside this window commits.
    # Clamped to 1-60.
    confirm-seconds: 5

  # Default for a companion whose companions/<id>.yml fusion block declares no broadcast key
  # of its own; a companion file overrides it either way, so you can announce the rare
  # fusions and stay quiet about the common ones. Fuse All never broadcasts,
  # whatever any companion says: it rolls per pair, so a good run would post one line
  # per winning pair.
  # HEADS UP: two shipped companion files already override this - stone_golem.yml sets
  # broadcast: true and ember_fox.yml sets false - and companions/ is seed-only, so
  # those stay whatever they are no matter what you put here. Turning this on will
  # NOT make Ember Fox announce; edit or delete the key in its file to do that.
  broadcast: false

  # Sound played to the owner when a fusion produces a companion. "none" plays nothing.
  success-sound: "ENTITY_PLAYER_LEVELUP 1.0 1.2"
  # Sound played to the owner when a fusion fails and both parents are lost.
  # "none" plays nothing.
  failure-sound: "ENTITY_ITEM_BREAK 1.0 0.8"

# ------------------------------------------------------------
#  Companion eggs. An egg is a PURCHASE, not an item: nothing is minted, traded
#  or right-clicked. A player buys one from the egg menu, it rolls its weighted
#  table straight away and the companions land in their storage.
#  What a single egg contains, costs, looks like on hatching and says lives in
#  eggs.yml, one key per egg; this section holds only the rules every egg obeys.
#  An egg that opens ALWAYS produces a companion: the numbers in its drops table
#  are weights, not percentages, and there is no failure roll.
#  Storage is checked ONCE, up front, for every companion the roll produced: if
#  the whole bundle does not fit, nothing is charged and nothing is granted.
#  WHAT AN EGG COSTS is eggs.yml's price block, not this section:
#    price.currency: vault          the server economy, through Vault. With no
#                                   economy plugin installed the egg is REFUSED,
#                                   never given away: opening one produces
#                                   companions, so a free egg would mint them.
#    price.currency: edtools:<id>   one of your EdTools currencies, by its own
#                                   id. This is INDEPENDENT of the edtools
#                                   section below, whose enabled switch governs
#                                   the companion BOOSTERS and nothing else. An
#                                   id EdTools does not serve is named in the
#                                   console when eggs.yml loads, and the egg
#                                   refuses every click until the id is real.
#  The opens: buttons price a BUNDLE each; an amount with no button of its own
#  costs the first button's price in proportion. If the companions cannot be
#  saved after the money was taken, the whole price is refunded automatically.
# ------------------------------------------------------------
eggs:
  # Master switch of egg opening: the menu, the command and the admin open. Off
  # refuses every one of them.
  enabled: true

  # The egg the main menu button opens. Must be an id declared in eggs.yml; an
  # id that names nothing leaves that button doing nothing.
  default: basic_egg

  # Let a creative-mode player BUY eggs. Off by default: a creative player has
  # unlimited resources on most servers, so paying for an egg means nothing.
  # This gate applies to PURCHASES only, like the egg's own cooldown-seconds:
  # /companions admin openegg ignores both, because nobody is charged there.
  allow-creative: false

# ------------------------------------------------------------
#  Companion items. A stored companion can be taken out of the storage as a physical head
#  (shift + right click on it in the main menu) and redeemed back by anyone
#  with a right click, which is how companions change hands. The item carries the
#  companion's whole state - level and experience - so
#  a companion that comes back is the companion that left. A redeem into a full storage
#  hands the item straight back, and so does every other refusal: the item is
#  the companion, and it is never destroyed by a refusal.
#  An EQUIPPED companion cannot be taken out (unequip it first) and neither can one
#  whose companions/<id>.yml is gone, because it has no face left to travel with.
#  The blocked-blocks list below makes the redeem click step aside for a chest,
#  a door or a workstation, so the block wins the click instead of the item.
# ------------------------------------------------------------
companion-items:
  # Master switch of extracting and redeeming. Off refuses both and hands a
  # clicked item back. Items already in circulation stay in inventories.
  enabled: true

  # Let a creative-mode player redeem companion items. Off by default: creative
  # middle-click copies any stack with its NBT intact, so one companion item would
  # mint companions without limit.
  allow-creative: false

  # Blocks the redeem right click steps aside for, so the block wins the click.
  # Without them, clicking a chest with a companion item in hand would redeem the
  # companion instead of opening the chest. An empty list makes every block
  # redeemable-on.
  #
  # An entry starting with # is a vanilla BLOCK TAG and expands to everything
  # in it on load, following whatever the running server version ships. That is
  # what keeps this list short: #doors alone covers every wood and copper door,
  # and #shulker_boxes covers the undyed box AND all sixteen dyed ones, which a
  # bare SHULKER_BOX line does not. Anything else is one material name, and an
  # unknown tag or material is skipped with one console warning.
  #
  # A server on 1.21 can also add CRAFTER, VAULT and TRIAL_SPAWNER. They are
  # left out here because they do not exist on 1.20, which this plugin also
  # supports, and an unknown name costs a console warning on every boot.
  blocked-blocks:
    # Tags: whole families in one line.
    - "#shulker_boxes"
    - "#doors"
    - "#trapdoors"
    - "#fence_gates"
    - "#beds"
    - "#buttons"
    - "#anvil"
    - "#cauldrons"
    - "#all_signs"
    - "#candles"
    - "#flower_pots"
    # Containers.
    - CHEST
    - TRAPPED_CHEST
    - ENDER_CHEST
    - BARREL
    - HOPPER
    - DISPENSER
    - DROPPER
    # Workstations and other blocks that open a screen.
    - FURNACE
    - BLAST_FURNACE
    - SMOKER
    - BREWING_STAND
    - CRAFTING_TABLE
    - ENCHANTING_TABLE
    - BEACON
    - LECTERN
    - GRINDSTONE
    - SMITHING_TABLE
    - STONECUTTER
    - LOOM
    - CARTOGRAPHY_TABLE
    - CHISELED_BOOKSHELF
    # Blocks that do something visible when you click them.
    - LEVER
    - NOTE_BLOCK
    - BELL
    - COMPARATOR
    - REPEATER
    - DAYLIGHT_DETECTOR
    - JUKEBOX
    - RESPAWN_ANCHOR
    - CAKE
    - COMPOSTER
    - DECORATED_POT
    - CAMPFIRE
    - SOUL_CAMPFIRE
    - BEEHIVE
    - BEE_NEST
    - LODESTONE

  # Look of the extracted item. Its MATERIAL is always the companion's own head and
  # cannot be set here; the name and lore below are yours.
  # Every companion placeholder the menus use works here:
  #   {companion} {companion-lore} {group} {group-color} {level} {level-cap} {exp}
  #   {exp-next} {percent} {bar} {buff} {buff-value} {owner}
  # {companion-lore} is multi-line and expands to one lore line each.
  # {owner} is the player the companion was taken out by. A companion item also REMEMBERS
  # them: only that player can redeem it, and anyone else is refused and handed
  # the item straight back. An item that records nobody is not locked and stays
  # redeemable by whoever holds it.
  item:
    display-name: "{companion}"
    lore:
      - "{companion-lore}"
      - ""
      - "&8Group: &r{group-color}{group}"
      - "&7Level &f{level}&7/&f{level-cap}"
      - "&7Exp &f{exp}&7/&f{exp-next} &8({percent}%)"
      - "&7Owner&8: &f{owner}"
      - ""
      - "&a&lRIGHT CLICK"
      - "&2Redeem this companion into your storage"
```

### A companion item remembers who took it out

An extracted companion carries two more tags in its item data: the UUID of the
player who took it out, and their name. A player who right clicks a head that belongs to somebody
else is refused with `messages.companion-item-not-yours`, which names the owner, and the head goes
straight back into their inventory untouched.

- **The UUID is the truth; the name is only printed.** A player who changes their nick keeps their
  companions, and a player who takes the owner's old nick does not inherit them.
- **A head that remembers nobody is not locked and stays redeemable by
  whoever holds it.** That is deliberate: nothing on the server records who extracted such an item,
  so there is no owner to restore.
- There is **no config switch**. An item with no owner tag is free and an item with one is locked,
  which is the whole rule.
- Redeeming your OWN head always works, and the companion it creates belongs to whoever
  redeemed it - so taking it out again stamps the new holder.

{% hint style="warning" %}
`config.yml` is **managed**: a merge adds keys you are missing but never rewrites a value you
already have. The `&7Owner&8: &f{owner}` line above is one entry of a `lore` list, so on a server
whose list already exists it reaches **fresh installs only**. To show the owner there, add
that one line to `companion-items.item.lore` by hand, or delete the whole `lore` list and let the next
boot write the shipped one back. The lock itself does not depend on the lore line - it works
whether or not the item advertises an owner.
{% endhint %}

`{owner}` is **not hologram-only**. It is an ordinary companion placeholder and works in
`companion-items.item.lore` and in the companion cells of `guis/main.yml` too.

```yaml
# ------------------------------------------------------------
#  Worlds. TWO separate lists, on purpose: where companions are DRAWN and where their
#  buffs APPLY are different questions, and mixing them would turn a cosmetic
#  decision into a combat-balance one.
# ------------------------------------------------------------
worlds:
  # Whitelist of the worlds companions are drawn in. An EMPTY list means every world.
  # Outside this list the companions stay equipped and their buffs keep applying;
  # only the rendering stops. Use worlds.buff-disabled below to switch a buff
  # off - this list never does.
  render: []
  # Narrows the list above for OTHER players' companions only. An EMPTY list means
  # "wherever render allows". Name a world here to let players see their OWN
  # companions there while everyone else's stay hidden, as in a lobby where a crowd of
  # formations is noise. A world absent from render draws nothing either way.
  render-others: []
  # Blacklist per buff: the worlds that buff does NOT apply in. An empty list
  # means the buff applies everywhere, and so does a buff removed from here.
  # Entering or leaving one of these worlds removes or re-applies the buff.
  # Example: leave speed on in your pvp world while damage is off there.
  buff-disabled:
    speed: []
    resistance: []
    damage: []

# ------------------------------------------------------------
#  Menus. Everything the four menus look like lives in guis/<id>.yml: the
#  titles, the masks, every material, every name, every lore line and every
#  click action. These two keys are the exceptions, because neither can be
#  written as an item field.
#  There is deliberately no page-size key: the number of cells a menu gives its
#  paged region IS its page size, so widening the letter run in that menu's
#  layout widens the page and the arrows follow. A second knob could only ever
#  disagree with the picture.
# ------------------------------------------------------------
menus:

  # The experience bar spliced into a companion's {bar} lore line.
  progress-bar:
    # Cells the bar is drawn with, 1 to 64.
    length: 20
    # Glyph of a cell the companion already earned.
    filled: "&a|"
    # Glyph of a cell it has not.
    empty: "&8|"

  bulk-delete:
    # Icon of a group button whose group names no material of its own below.
    default-icon: RED_DYE
    # Icon per group id, matching the keys of the `groups:` section above. A
    # value may also be a head texture (a raw base64 payload or a basehead-
    # prefixed one), exactly like any other material field.
    # sn:extensible
    icons:
      common: WHITE_DYE
      rare: BLUE_DYE
      epic: PURPLE_DYE

# ------------------------------------------------------------
#  PlaceholderAPI. SnCompanions exposes %sncompanions_...% when PlaceholderAPI is
#  installed; without it nothing is registered and no key here does
#  anything. The tokens themselves are not configurable - another plugin's
#  scoreboard refers to them by name - so this section only decides how the
#  NUMBERS are written.
#
#  The full list:
#    %sncompanions_companions_total%      %sncompanions_companions_stored%     %sncompanions_companions_equipped%
#    %sncompanions_storage_used%    %sncompanions_storage_total%   %sncompanions_storage_free%
#    %sncompanions_slots_used%      %sncompanions_slots_total%     %sncompanions_slots_free%
#    %sncompanions_buff_damage%     %sncompanions_buff_resistance% %sncompanions_buff_speed%
#    %sncompanions_equipped_<n>%    the companion in equip slot <n>, or the status.none word
#    %sncompanions_equipped_<n>_level%   _exp%   _exp_next%   _cap%
#
#  Every one of them is answered from memory: a player who is not loaded
#  reads as zero rather than making the server wait for a database read,
#  which is what lets a scoreboard use them every tick.
# ------------------------------------------------------------
placeholders:
  # Decimal places of the three buff percentages (%sncompanions_buff_*%). 0 prints
  # whole numbers, 2 prints 12.50. Written with a dot on every server no
  # matter its language, so a scoreboard condition comparing the value to a
  # number keeps working. 0 to 6.
  #
  # The default is 1 because that is what the MENUS print: raise it and the
  # same buff reads 12.50 on your scoreboard and 12.5 in the companion lore at the
  # same time. Raise it anyway if another plugin compares the token and needs
  # the precision - the menus are a house style, this is a value.
  decimals: 1

  # Whether the counters and the experience numbers are abbreviated: on,
  # 1500 prints as 1.5K. Leave it off if another plugin compares these
  # placeholders against a number - an abbreviated value is text, not a
  # number, and a requirement check on it will fail. The percentages above
  # are never abbreviated.
  compact-numbers: false
```

### How capacity is resolved

The effective slot count and storage capacity are
**`max(config base, permission) + purchased`**: the config base (`slots.base-count` /
`storage.base-capacity`) and the rank permission (`sncompanions.slots.<n>` / `sncompanions.storage.<n>`)
compete - the permission names the absolute count a rank grants, so the higher of the two is the
player's floor - and everything sold through `/companions admin slots|storage give` adds on top of that
floor. With `base-count: 1`, a player who buys one slot equips two companions.

Nothing about that is stored: the database holds only the purchased half, and the floor is
resolved from the config and the permissions on every join.

### Capacity ceilings

`slots.max-count` and `storage.max-capacity`
cap the total the two admin capacity commands may leave a player with.

| Key | Default | Caps |
|---|---|---|
| `slots.max-count` | `7` | the total equip slots `/companions admin slots give\|set` may leave |
| `storage.max-capacity` | `0` (off) | the total storage `/companions admin storage give\|set` may leave |

A command that would go past its ceiling is **not cancelled**. The purchase is trimmed so the
total lands exactly on the ceiling and the admin is told, so on a stock install (base `1`,
ceiling `7`) `/companions admin slots give Snopeyy 100` leaves 6 purchased slots - a total of 7 - and
prints `messages.admin-slots-clamped` before the usual confirmation. That notice is shown even
to an admin who typed `-sf`: that flag mutes success confirmations, and a number other than the one
typed is corrective information. Set a key to `0` to remove its ceiling.

{% hint style="info" %}
Keep `slots.max-count` equal to the number of `s` cells in the layout of `guis/main.yml` - 7 in the
shipped menu. The menu can only draw that many, so slots past it are bought and never usable.
SnCompanions does not read `guis/main.yml` to derive the ceiling, on purpose: two files you edit
independently should not silently depend on each other.
{% endhint %}

{% hint style="warning" %}
The ceiling bounds **purchases only**. It does not limit the `sncompanions.slots.<n>` /
`sncompanions.storage.<n>` permission grants, so a rank may still grant more than an admin command can -
purchases stack on top of whatever floor the rank sets.

It also never lowers a row on its own. If you reduce `slots.max-count` on a live server, players
already above it keep what they have until the next `slots give`/`set` on them, which then clamps
them down to the new ceiling.
{% endhint %}

### How the label turns

`holograms.billboard` decides whether the text above a companion leans towards the
player or stays upright.

| Value | What the text does |
|---|---|
| `vertical` (default) | turns only around the vertical axis, so the lines stay upright at every angle |
| `center` | turns on both axes and tilts towards the camera |
| `horizontal` | turns around the horizontal axis only |
| `fixed` | never turns, keeping the facing it was spawned at |

`vertical` is the DecentHolograms look and the shipped default: a label read from a
rooftop or from the bottom of a ravine is as straight as one read at eye level. `center` tilts the
whole stack towards the camera - write it if that is what you want, and nothing else
changes. `horizontal` and `fixed` complete the set of what a text display can do; neither reads
well on a name plate.

An unknown value falls back to `vertical` and logs one line in the console naming what it read. The
setting is global: there is no per-companion override, so a server has one label aesthetic rather than a
mix. It applies on the next formation rebuild, which `/companions reload` performs.

{% hint style="info" %}
`config.yml` is managed, so a server whose file lacks the key receives `billboard: vertical`
on the next boot and its labels straighten up with no file editing.
{% endhint %}

## companions/

One file per companion type, named after its id. Three examples are seeded on a fresh install:
`ember_fox.yml`, `gale_sprite.yml` and `stone_golem.yml`. A companion file declares the display
name, the group it belongs to, the head or BetterModel it renders as, its level cap and
experience curve, the buffs it grants per level, the hologram drawn above it,
and the optional `edtools-boosts` and `buff-display` blocks. Copy one of the examples to add your
own.

The `hologram:` block is optional and, because this folder is seeded once and never merged
again, it is never added to the companion files you already have. That is what
`holograms.default-lines` in `config.yml` is for: a companion that declares no block uses those
lines, so you only write a block for the companions that should differ. Only an explicit
`enabled: false` silences a companion - `gale_sprite.yml` ships that way as the mute example, and
`stone_golem.yml` ships its own `height-offset` because a model companion's label is measured from
the carrier its bones ride, which also carries `models.height-offset`.

The `edtools-boosts:` block is optional and ships **commented out**: it does
something only on a server running EdTools. Each child key is an EdTools currency id, or one of
`enchants`, `enchant`, `global-enchants`, `encantamientos` for the global enchant multiplier, and
both numbers are percentages exactly like the `buff:` block above. It has the same seeded-once
caveat as `hologram:` - the companion files you already have are never merged again, so you add the block
to them by hand. See the `edtools` band of `config.yml` above for the one-decimal rule that decides
what those percentages actually grant.

Each entry of that block also accepts an optional `max:`, a ceiling in percentage
points:

```yaml
edtools-boosts:
  money:
    initial: 0.4
    per-level: 0.4
    max: 30.0
```

Three things about where the ceiling sits, because they are what make it useful:

- It applies to the value **this companion** produced at its current level. A companion whose ramp
  reached 45 with `max: 30` contributes 30.
- It applies **before** the equipped companions are summed, so it caps one companion and never the player's
  total. Two equipped companions that each declare `max: 10` still grant 20 together. There is no cap on
  a player's total, deliberately.
- The one-decimal rounding still happens **afterwards**, on the sum. Capping first and rounding
  after is what lets three companions capped at 4 grant +10% instead of nothing.

Absent, `0` and any negative number all mean **no ceiling**, so a companion file that declares none
grants its full ramp. A negative value is logged once on load,
naming the companion and the entry, and then read as no ceiling. The key exists for a per-level ramp
running on a level cap it was not sized for: `per-level: 0.4` is +300% at level 750.

The `{buff}` and `{buff-value}` placeholders show this block too. Everywhere a
companion is described - the menus, an extracted companion item, a hologram line - those two placeholders name
the companion's EFFECT: a companion that declares a vanilla `buff:` shows that buff, and a companion
whose buff grants nothing resolves them from its `edtools-boosts` block instead. `{buff}` becomes
the boosted currencies joined in file order and `{buff-value}` the live value at the companion's current
level, with the per-entry `max:` already applied - the same
number, from the same formula, that the booster sum uses. A companion boosting several currencies at
different values shows the highest one.

Currency display names come from the language file: one `messages.edtools-currency-<id>` entry per
currency you want renamed (e.g. `edtools-currency-money: "&6Money"`), with
`menus.edtools-buff-separator` as the separator between two names. Only `enchant` ships named -
every other id is invented on your server - and an id without an entry shows as the raw id.

### `buff-display`: a companion whose effect lives in another plugin

The third and last thing those two placeholders can resolve from, added in 1.9.0. Some companions
exist for a plugin that is not this one: a plugin reads a player's equipped companions through the
SnCompanions API and grants a boost of its own for some of them, so the companion declares no
`buff:` and no `edtools-boosts:` because SnCompanions has nothing to apply for it. Without this
block such a companion renders as `Damage 0.0%` on every menu, item and hologram.

```yaml
buff-display:
  name: "&dBattle Pass XP"
  initial: 0.4
  per-level: 0.4
  max: 20.0
```

`{buff}` becomes `name` and `{buff-value}` becomes `initial + (level - 1) x per-level`, capped by
`max` (absent or `0` = no ceiling). **SnCompanions never applies this block** - it only shows it -
and it is the LAST fallback, after the vanilla buff and after `edtools-boosts`, so no companion
that showed something before 1.9.0 shows anything different now. The name is the switch: a block
with a name and no numbers is a legitimate declaration with no figure to it, while a block with
numbers and no name is ignored with a console warning, because `{buff}` would have nothing to
print. Nothing in this plugin widens the ramp and nothing validates it against anything, so write
the same numbers the other plugin uses: the figure a player reads here is then the figure they are
paid there.

Same seeded-once caveat as the two blocks above: the example ships commented out in
`ember_fox.yml` for a fresh install, and you add the block to the companion files you already have
by hand.

Here is the seeded `companions/ember_fox.yml` in full, as a working reference:

```yaml
# ============================================================
#  SnCompanions - companion definition: ember_fox
#  One file per companion. The file name is the companion id used by commands, by the
#  fusion targets of other companions and by the database rows.
#  Seeded once and never merged again: this file is yours. Copy it to add a
#  companion, delete it to remove one. A companion id still stored on a player whose file
#  is gone is never deleted; it renders as a placeholder that cannot be
#  equipped or fused until the file comes back.
#  Every companion is independent: it declares its own level cap, curve, buff and
#  fusion target. There is no rarity table.
# ============================================================

# Name shown on the companion item.
display-name: "&6&lEmber Fox"

# Lore shown on the companion item, above the progress lines the menus add.
lore:
  - "&7A restless fox that burns brighter"
  - "&7the longer you fight beside it."

# Head texture shown by the renderer and on the companion item. Accepts a raw base64
# payload, a basehead-/texture- prefixed value or an http skin URL.
head-texture: "eyJ0ZXh0dXJlcyI6eyJTS0lOIjp7InVybCI6Imh0dHA6Ly90ZXh0dXJlcy5taW5lY3JhZnQubmV0L3RleHR1cmUvZjdkOWE4ZDg5NTY1MmU4N2JmNmZkNTBmYmE2ZmFkMzljY2QwYmY4Y2E5NzgwZmE2ZDkxMTUzMjZkOWFkODNhYSJ9fX0="

# Optional BetterModel model. Declare it to render an animated model instead of
# the head; the head above stays the fallback when BetterModel is absent or the
# model id is gone.
# model:
#   # Model id registered in BetterModel.
#   id: ember_fox
#   # Animation played while the owner stands still.
#   animation-idle: idle
#   # Animation played while the owner moves.
#   animation-move: walk
#   # Size multiplier of the model.
#   scale: 1.0

# ------------------------------------------------------------
#  Hologram. The text drawn above this companion, riding the companion itself so the two
#  move as one thing. Leave the whole block out and the companion uses
#  holograms.default-lines from config.yml; the look (spacing, scale, shadow,
#  background) always comes from the holograms band there.
# ------------------------------------------------------------
hologram:
  # Draw a label above this companion. False silences this companion alone.
  enabled: true
  # This companion's own lines, top first. Every menu placeholder works here, plus
  # {owner}; PlaceholderAPI tokens resolve against the owner.
  lines:
    - "{group-color}{companion}"
    - "&7Lv. &f{level}&7/&f{level-cap}"
    - "&8{owner}"
  # Blocks above the companion the top line sits at. Leave it out to follow
  # holograms.height-offset.
  # height-offset: 0.9

# This companion's own level cap.
max-level: 50

# ------------------------------------------------------------
#  Experience.
# ------------------------------------------------------------
experience:
  # Experience needed to reach level 2.
  base: 500
  # Added to that requirement by each further level.
  per-level: 500
  # VANILLA_XP, BLOCK_BREAK, EDTOOLS_BLOCK_BREAK, MOB_KILL, DAMAGE_DEALT or
  # PLAYTIME.
  #
  # EDTOOLS_BLOCK_BREAK is NOT the same source as BLOCK_BREAK: EdTools consumes
  # the blocks its omnitools break without ever firing a vanilla
  # BlockBreakEvent, so on a farming server the two count completely different
  # things and this companion has to name the one your server actually produces. It
  # needs EdTools installed; without it a companion on this source simply earns
  # nothing.
  source: VANILLA_XP
  # Gain per unit of the source; here, per point of vanilla XP picked up.
  ratio: 1.0
  # Only for EDTOOLS_BLOCK_BREAK: count only the breaks made with these EdTools
  # tool ids, spelled as EdTools itself names them. Left out or empty, every
  # omnitool counts. A break EdTools reports with no tool at all never matches a
  # filter, so a companion listed here earns from that tool and nothing else.
  #
  # tools:
  #   - crop-tool
  #   - mining-tool

# ------------------------------------------------------------
#  Buff. Only equipped companions apply it, and the totals of every equipped companion are
#  summed before the per-buff cap is applied.
# ------------------------------------------------------------
buff:
  # DAMAGE, RESISTANCE or SPEED.
  type: DAMAGE
  # Percent granted at level 1.
  initial: 2.0
  # Percent added by each level above 1.
  per-level: 0.1

# ------------------------------------------------------------
#  EdTools boosters. Optional, and left commented out on purpose: uncomment it
#  only on a server that runs EdTools. Only equipped companions grant these, the
#  values of every equipped companion are summed per currency; nothing widens them,
#  so the number you write here is the number that is granted.
#
#  Each child key is an EdTools currency id, or one of "enchants", "enchant",
#  "global-enchants", "encantamientos" for the GLOBAL enchant multiplier. There
#  is no way to boost one named enchant: the global multiplier is all EdTools
#  exposes to other plugins.
#
#  READ THIS BEFORE PICKING NUMBERS. The percentages here are summed and then
#  handed to EdTools rounded to ONE decimal of the fraction it wants, which
#  means the granted boost moves in steps of 10%: a total below 5% grants no
#  booster at all, 5% to 14% grants +10%, 15% to 24% grants +20%. The rounding
#  is a hard rule of this integration, so write your companions in tens.
#
#  The optional "max" below feeds that same rule from the other end: it is
#  applied FIRST, to this companion alone, and the rounding happens afterwards on the
#  sum. It does NOT bound the player's total - two equipped companions that each cap
#  at 10 still add up to 20.
# ------------------------------------------------------------
# edtools-boosts:
#   money:
#     # Percent granted at level 1.
#     initial: 10.0
#     # Percent added by each level above 1.
#     per-level: 1.0
#     # Optional ceiling, in percentage points, applied to the value THIS companion
#     # produced, before the totals of your equipped companions are summed.
#     # Absent or 0 = no ceiling.
#     # Use it when a per-level ramp would run away at a high level cap.
#     max: 30.0
#   enchants:
#     initial: 10.0
#     per-level: 0.0

# ------------------------------------------------------------
#  Declared effect (1.9.0). Optional, and for ONE situation only: a companion
#  whose effect is paid by ANOTHER plugin, which reads this companion through
#  the SnCompanions API and grants something of its own for it. Such a
#  companion declares no "buff:" and no "edtools-boosts:", so without this
#  block {buff} and {buff-value} would read "Damage 0.0%" on every menu,
#  item and hologram.
#
#  SnCompanions NEVER applies this. It only shows it: {buff} becomes "name"
#  and {buff-value} becomes initial + (level - 1) x per-level, capped by
#  "max". It is the LAST fallback - a companion that declares a real "buff:"
#  or an "edtools-boosts:" block keeps showing that, and this block is
#  ignored. Write the same numbers the other plugin uses, so the figure a
#  player reads here is the figure they are paid there. Nothing in this
#  plugin widens this ramp: SnCompanions cannot know whether the other
#  plugin does. The name accepts & and &#RRGGBB codes like display-name.
# ------------------------------------------------------------
# buff-display:
#   # Shown as {buff}.
#   name: "&dBattle Pass XP"
#   # Percent shown at level 1.
#   initial: 0.4
#   # Percent added by each level above 1.
#   per-level: 0.4
#   # Optional ceiling in percentage points. Absent or 0 = no ceiling.
#   max: 20.0

# Group label declared in config.yml; drives the storage order, the bulk delete
# buttons and the lore, nothing else.
group: common

# ------------------------------------------------------------
#  Fusion. Two of THIS companion produce the companion named below. Remove the section to
#  make this companion unfusable.
# ------------------------------------------------------------
fusion:
  # Companion id produced by a successful fusion.
  into: stone_golem
  # Success chance in percent; on failure both parents are lost.
  chance: 50.0
  # Money charged per attempt; 0 is free.
  cost: 0
  # Announce a successful fusion of two of THIS companion to the whole server. Leave the
  # key out to follow config.yml fusion.broadcast.
  broadcast: false

# Companion ids that cannot be equipped alongside this one.
incompatible:
  - gale_sprite
```

## eggs.yml

**Every** companion egg lives in this one file, one top-level key per egg. The key is the egg id the
menu and `/companions admin openegg` take. One example is seeded, `basic_egg`. Add, rename and
delete keys freely; an id may not contain a dot, which is the yml path separator.

The file is seed-only and its header carries `# sn:extensible-root`, so a key you delete stays
deleted and no key ever gains a sub-key from an update. Deleting the whole FILE re-seeds the
example on the next reload; delete the KEYS you do not want instead.

{% hint style="info" %}
**An egg is a purchase, not an item.** Nothing is minted, traded or right-clicked: a player buys an
egg from the egg menu, its table is rolled straight away and the companions go into their storage.
That is why an egg has no `item:` block - what the menu draws lives in `guis/eggs.yml`, beside every
other button of that screen.
{% endhint %}

### The keys

| Key | Meaning |
|-----|---------|
| `display-name` | The name the menu and every `{egg}` placeholder show. `[rgb]`, `[small]` and `[noprefix]` are applied when the file is read, so the name renders the same in the menu and spliced into the middle of a chat line |
| `price.currency` | `vault` for the server economy, or `edtools:<id>` for one of your EdTools currencies. Anything else is refused with a console warning and charged as `vault`. See [What an egg costs](#what-an-egg-costs) |
| `opens` | The buttons of the egg menu: a LIST, one entry per bundle, each with its own `amount` and its own explicit `price`. There is no unit price multiplied by an amount - an owner who wants "10 for the price of 9" writes that number. An amount with no button of its own costs the FIRST button's price in proportion. An egg with no `opens` cannot be bought and stays admin-only |
| `drops` | The weighted table, one entry per companion id, each with an `amount` and a `weight`. The numbers are WEIGHTS, not percentages: they are normalized over whatever the table holds, so 60/30/10 and 6/3/1 behave identically. A row naming a companion with no `companions/<id>.yml` file is dropped when the egg loads, with one console warning, and the remaining weights renormalize on their own |
| `animation` | The hatch show of THIS egg. `enabled` switches it off for everybody; `shake-ticks` is clamped to 4-600 and `reveal-ticks` to 0-600, and the three sound keys take `"SOUND_ID [volume] [pitch]"` or `none`. The pitch of `step-sound` is ignored - it climbs from 0.8 to 2.0 across the shake. Each player can also switch the show off for themselves; see [The egg animation switch](#the-egg-animation-switch) |
| `feedback.message-key` | Lang key sent to the opener of ONE egg that produced ONE companion. Anything larger sends `messages.egg-opened-bulk` instead, which summarizes the whole open. Empty sends nothing |
| `feedback.broadcast-key` | Lang key announced to the whole server, once per DISTINCT companion won. Empty announces nothing |
| `cooldown-seconds` | Wait enforced between two PAID opens of this egg. `0` disables it entirely and costs nothing at runtime. It is armed only after the money really moved, so a refused purchase never starts one. An admin open ignores it: it costs nobody anything |

An egg that opens **always** produces a companion. There is no failure roll: an open is either
refused before anything is spent - the master switch is off, the player's row is still loading, the
table has no rollable row left, their storage cannot take the whole roll, or they cannot pay - or it
goes through.

### What an egg costs

`price.currency` picks the wallet, and it is the only thing that does:

| Value | Charged through |
|-------|-----------------|
| `vault` (the default) | The server economy, through Vault or whatever backend SnLib is configured with |
| `edtools:<id>` | The EdTools currency with that id, through the EdTools currency API |

{% hint style="warning" %}
**A missing provider refuses the purchase; it never makes the egg free.** With no economy plugin
installed, a `vault` egg answers `messages.egg-no-economy` and opens nothing. With EdTools absent,
or with an `<id>` EdTools does not serve, an `edtools:` egg answers `messages.egg-no-currency` -
and the console names that egg when `eggs.yml` loads, so a typo is visible before a player finds
it. This is deliberately unlike a costed fusion, which is free when there is no economy: a fusion
consumes companions the player already owns, while an egg PRODUCES them, so a free egg would hand
out companions nobody paid for.
{% endhint %}

**EdTools egg prices do not depend on `edtools.enabled`.** That switch owns the companion BOOSTERS
and nothing else. An egg priced in an EdTools currency is charged whenever EdTools itself is
reachable, even on a server that has turned companion boosters off.

The order money moves in is: the creative gate, the egg's cooldown, the price, the storage check,
the cancellable `CompanionEggOpenEvent`, the storage reservation, the charge, the grant. The
reservation is taken **before** the charge because the server economy settles a tick later, and if
the charge then fails those places are handed straight back. If the companions cannot be WRITTEN
after the money was taken - the one failure no gate can foresee - the whole price is refunded
automatically, in the currency it was paid in, and the player is told both things. A PARTIAL loss
is not refunded: the companions that did land were bought, and an `opens:` button prices a bundle
rather than one egg.

`/companions admin openegg` is a **gift**: it skips the creative gate, the cooldown and the price
entirely, which is also why it can open an egg that declares no `opens:` button at all.

### The shipped file in full

```yaml
# ============================================================
#  SnCompanions - companion eggs
#  ONE file for every egg. Every top-level key below is an egg id: it is the id
#  the egg menu and /companions admin openegg take. Add, rename and delete
#  freely; an id may not contain a dot, which is the yml path separator.
#  Seeded once and never merged again: this file is yours. Deleting the FILE
#  re-seeds the example below on the next reload; delete the KEYS you do not
#  want, not the file.
#
#  An egg is a PURCHASE, not an item. Nothing is minted, traded or right-clicked:
#  a player buys an egg from the menu, its table is rolled straight away and the
#  companions go into their storage. That is why an egg has no "item:" block -
#  what the menu draws lives in guis/eggs.yml, beside every other button.
#
#  The numbers under "drops" are WEIGHTS, not percentages: they are normalized
#  over whatever the table holds, so 60/30/10 and 6/3/1 behave identically and
#  the total does not have to add up to anything. An egg that opens ALWAYS
#  produces a companion; there is no failure roll. A row naming a companion with
#  no companions/<id>.yml file is dropped when the egg loads, with one console
#  warning, and the remaining weights renormalize on their own.
#
#  The entries under "opens" are the BUTTONS the egg menu shows, each with its
#  own explicit price. There is no unit price multiplied by an amount: an owner
#  who wants "10 for the price of 9" writes that number. An egg with no opens
#  cannot be bought at all and stays admin-only.
#
#  "price.currency" is either "vault" (the server economy) or "edtools:<id>",
#  which names one of your EdTools currencies. Anything else is refused with a
#  console warning and charged as vault.
#
#  "animation" is the hatch show played when the egg is opened. It delays only
#  the FEEDBACK: the companions are granted and saved before the first frame is
#  drawn, so quitting mid-hatch costs the show and nothing else.
# sn:extensible-root
# ============================================================

# ------------------------------------------------------------
#  basic_egg - the shipped example: the three companions this plugin ships,
#  four price tiers, a full hatch animation and no cooldown.
# ------------------------------------------------------------
basic_egg:

  # Name shown in the menu and spliced into the {egg} placeholder of every egg
  # message. [rgb], [small] and [noprefix] work here and are applied on load.
  display-name: "&d&lBasic Egg"

  # ----------------------------------------------------------
  #  What it costs, and in what.
  # ----------------------------------------------------------
  price:
    # "vault" for the server economy, or "edtools:<id>" for an EdTools currency.
    currency: vault

  # ----------------------------------------------------------
  #  The buttons of the egg menu. One entry per bundle: how many eggs it opens
  #  and what the whole bundle costs. Order is the order they are drawn in.
  # ----------------------------------------------------------
  opens:
    - amount: 1
      price: 1000
    - amount: 3
      price: 2900
    - amount: 10
      price: 9500
    - amount: 50
      price: 45000

  # ----------------------------------------------------------
  #  The drop table. One entry per companion id, which is a companions/<id>.yml
  #  file name. The numbers are WEIGHTS, not percentages.
  # ----------------------------------------------------------
  drops:
    ember_fox:
      # How many of this companion one winning roll grants.
      amount: 1
      # Relative weight of this row.
      weight: 60
    stone_golem:
      amount: 1
      weight: 30
    gale_sprite:
      amount: 1
      weight: 10

  # ----------------------------------------------------------
  #  The hatch animation. It delays only the FEEDBACK.
  #  A Dragon Egg appears about 1.5 blocks in front of the opener, at their
  #  feet, shakes faster and faster, breaks in a burst of its own block
  #  particles and reveals the companion they won, rising out of the wreckage
  #  with its name above it. Nothing about it is a real entity and nothing
  #  about it can be walked into: it is drawn with packets and only the players
  #  who may already see that owner's companions ever see or hear it.
  #  The companions are granted and SAVED before the first frame, so a player
  #  who quits, changes world or crashes mid-hatch loses the show and nothing
  #  else. Buying a second egg while one is still hatching is refused before
  #  anything is charged (messages.egg-animating); an admin "openegg" is never
  #  refused - it lands and reports in chat.
  #  Each player can switch the show off for themselves from the button in slot
  #  36 of the eggs menu. That choice is saved in the database (the
  #  egg_animation column of sncompanions_players, added automatically on the
  #  first boot that finds it missing) and survives a relog and a restart. With it off, the
  #  eggs give exactly the same companions and the summary goes straight to
  #  chat.
  # ----------------------------------------------------------
  animation:
    # Play the hatch show instead of naming the companion instantly. This is the
    # SERVER's switch: off here, nobody sees the show whatever they chose.
    enabled: true
    # Ticks the egg shakes before it breaks. 20 ticks is one second.
    # Clamped to 4-600.
    shake-ticks: 60
    # Ticks between the break and the companion being named. 0 reveals in the
    # same frame the egg breaks. Clamped to 0-600. The revealed head rises one
    # block over the first HALF of this, then holds still.
    reveal-ticks: 40
    # Sound played on each shake frame, as "SOUND_ID [volume] [pitch]".
    # "none" plays nothing.
    # The PITCH written here is ignored: the whole point of the shake is that it
    # climbs, so the plugin plays this sound from pitch 0.8 on the first frame
    # to 2.0 on the last. The id and the volume are used as written. Frames get
    # closer together as the shake goes on, from one every 10 ticks to one every
    # 2, so a longer shake-ticks is a longer build-up, not a slower one.
    step-sound: "BLOCK_NOTE_BLOCK_HAT 1.0 1.2"
    # Sound played when the egg cracks open, together with the block particles.
    # "none" plays nothing.
    break-sound: "ENTITY_ENDER_DRAGON_GROWL 0.6 1.4"
    # Sound played when the companion is revealed. "none" plays nothing.
    reveal-sound: "ENTITY_PLAYER_LEVELUP 1.0 1.4"

  # ----------------------------------------------------------
  #  Feedback. Every key points at an entry of lang/messages_<code>.yml, so the
  #  wording is translated and restyled with every other message rather than
  #  living here. Leave one empty to send nothing.
  #  Placeholders: {player} {egg} {companion} {amount}
  # ----------------------------------------------------------
  feedback:
    # Line sent to the player who opened the egg. Applies to a SINGLE egg that
    # produced ONE companion; anything larger sends messages.egg-opened-bulk
    # instead, which summarizes the whole open.
    message-key: "messages.egg-opened"
    # Line announced to the whole server, once per DISTINCT companion won.
    # Empty announces nothing.
    broadcast-key: ""

  # Seconds a player must wait between two PAID opens of THIS egg. 0 disables
  # the cooldown entirely and costs nothing at runtime. An admin open ignores
  # it: it costs nobody anything.
  cooldown-seconds: 0
```

## guis/

Four menu layouts, all managed and all re-skinnable without touching code.

| File | Menu |
|------|------|
| `main.yml` | Companion storage, the screen bare `/companions` opens |
| `eggs.yml` | The companion eggs shop, opened by `/companions eggs [egg]` and by the egg button of `main.yml`. ONE file for every egg you declare |
| `fusion.yml` | Fusion, including the Fuse All bulk path |
| `bulk_delete.yml` | Bulk deletion by group |

{% hint style="warning" %}
A menu file defines a fixed number of cells. If you raise a player's equip slots above the
cell count of `main.yml`, the extra companions cannot be shown. The plugin warns about this at
runtime, and `/companions admin unequip` always recovers a companion that ended up out of reach.
{% endhint %}

### The eggs menu

`guis/eggs.yml` is ONE file for every egg `eggs.yml` declares. Which egg is on screen travels with
the player rather than with the file, so adding a key to `eggs.yml` gives it a shop page with
nothing to write here, and deleting one leaves no orphan menu behind.

The shipped mask is five rows:

```yaml
layout:
  - "    i    "
  - "  p p p  "
  - "   p p   "
  - "         "
  - "  oo<oo  "

regions:
  pool: p
  opens: o
```

| Cells | What |
|---|---|
| 4 (`i`) | `egg-info`: the egg on screen and the money it is priced in |
| 11, 13, 15, 21, 23 (`p`) | the `pool` region: one companion this egg can give per cell |
| 36 | `anim-on` / `anim-off`: the viewer's own hatch animation switch. It names slot 36 itself, which is why the mask leaves that cell blank |
| 38, 39, 41, 42 (`o`) | the `opens` region: one price button per `opens:` entry of the egg |
| 40 (`<`) | `back`, which returns to `main.yml` |

Both regions are drawn in the order `eggs.yml` writes them and both are sized by the mask alone:
the plugin never names a slot. An egg with more drop rows than `p` cells shows the first ones and
the rest are simply not drawn - that is configuration, not a mistake. **Widen the run to show
more**, and the same holds for the `o` run and the price buttons.

The templates the plugin binds:

| Template | Placeholders |
|---|---|
| `egg-info` | `{egg}` `{currency}` |
| `pool-entry` | `{id}` `{companion}` `{companion-lore}` `{icon}` `{texture}` `{group}` `{group-color}` `{chance}` `{amount}` |
| `open-button`, `open-button-locked` | `{egg}` `{index}` `{amount}` `{price}` `{currency}` |
| `anim-on`, `anim-off` | none - exactly ONE of the two is drawn per open, whichever matches the viewer's saved preference |

`{chance}` is the row's weight **normalized over the whole table**, in percentage points with one
decimal, never the raw weight you wrote: 60/30/10 and 6/3/1 both read 60.0/30.0/10.0. A row dropped
because its companion file is gone does not dilute the survivors. `{amount}` means two different
things on purpose - how many COMPANIONS a pool row grants, and how many EGGS a price button buys.

`open-button-locked` is drawn instead of `open-button` when the player cannot pay for that bundle.
Its click is deliberately left enabled: pressing it runs exactly the same purchase, which refuses
with the message naming the price and the currency. The menu decides nothing about money - the
master switch, the creative rule, the egg's cooldown, the price, the storage decision, the charge
and the refund are all re-taken when the button is pressed.

`{currency}` is `menus.eggs.currency-<id>` from the lang file, the same word the chat lines use.

{% hint style="info" %}
**On an existing install the egg button of `main.yml` does not appear by itself.** That file is
managed, so the merge adds the new `eggs-button` entry but keeps YOUR `layout:`, which has no
letter for it. Put an `e` where you want it - the shipped mask uses slot 45, the first cell of the
bottom row, as `"e  < > Fd"`. `/companions eggs` works either way.
{% endhint %}

### The egg animation switch

Slot 36 of `guis/eggs.yml` is the player's own switch for the [hatch
show](#eggsyml). It is the one button on this screen that has nothing to do with the egg in front
of it: it is a preference of the VIEWER, saved on their row, and it applies to every egg they ever
open.

```yaml
  anim-on:
    material: ITEM_FRAME
    slots: [36]
    glow: true
    display-name: "&a&lAnimation: &aOn"
    lore:
      - "&7Whether a hatch show plays when you open"
      - "&7one of your companion eggs."
      - ""
      - "&2[CLICK TO TURN THE ANIMATION OFF]"
    click-actions:
      - "[companions-toggle-egg-anim]"
      - "[sound] UI_BUTTON_CLICK"

  # The same switch while the show is off.
  anim-off:
    material: ITEM_FRAME
    slots: [36]
    display-name: "&a&lAnimation: &cOff"
    lore:
      - "&7Whether a hatch show plays when you open"
      - "&7one of your companion eggs."
      - ""
      - "&2[CLICK TO TURN THE ANIMATION ON]"
    click-actions:
      - "[companions-toggle-egg-anim]"
      - "[sound] UI_BUTTON_CLICK"
```

Both templates declare `slots: [36]` rather than a `layout:` letter, which is why the shipped mask
has a blank there and why moving the switch means editing those two `slots:` lines, not the mask.
The plugin draws exactly **one** of them per open - `anim-on` when the show is on for that player,
`anim-off` when it is off - so restyling one and forgetting the other shows through immediately.

`[companions-toggle-egg-anim]` takes no argument. It flips the viewer's preference, tells them which
way it went (`messages.egg-animation-on` / `messages.egg-animation-off`) and redraws the screen. The
choice is **saved**: it lives in the `egg_animation` column of the `sncompanions_players` table and
survives a relog and a restart. A player who has never touched it watches the show.

{% hint style="info" %}
**The two switches are independent, and the server's wins.** `animation.enabled` in `eggs.yml` is
the owner's per-egg switch; the button is each player's switch for all eggs. With `enabled: false`
nobody sees that egg hatch, whatever the button says - and the button is still drawn, because it is
about the player rather than about the egg on screen.
{% endhint %}

With the show off, an egg gives exactly the same companions: only the presentation changes, and the
summary goes straight to chat with no egg drawn at all.

{% hint style="info" %}
**The menu closes when the show starts.** The animation happens in the world, and the shop is a
54-slot inventory sitting on top of it. It closes only when a show actually begins - a player with
the animation off keeps the menu open and gets their summary with the balance already redrawn.

Two more cases end the show before it starts, on purpose: a player who cannot see their own
companions at all (the visibility toggle, or a world outside `worlds.render`) gets the summary at
once rather than several seconds of nothing, and `holograms.enabled: false` reveals the companion
with no name plate, because that switch means the same thing here as it does above a companion.
{% endhint %}

### Colouring a companion by its group

Every template that draws a companion binds `{group-color}`, the colour its group declares under
`groups:` in `config.yml`. Use it wherever you want the group to tint a line:

```yaml
  companion-entry:
    material: "{texture}"
    display-name: "{group-color}{companion}"
    lore:
      - "&8Group: &r{group-color}{group}"
```

It is available in `main.yml`, `eggs.yml`, `fusion.yml` and `bulk_delete.yml`. The value is inserted before SnLib's text pipeline runs, so a legacy code, a
hex code and the `[rgb]` gradient tag all work. `[rgb]` is a PREFIX tag: it only applies when
`{group-color}` is the first thing on the line.

None of the shipped templates use it, so nothing changes appearance until you add it yourself.

Where it does **not** work, and why:

| Place | Why |
|---|---|
| `menus.group-separator` | joins group names on the information clock; no single companion |
| the `lore:` of a `companions/<id>.yml` file | that lore is itself the value of `{companion-lore}`, and a placeholder value is never re-scanned for further placeholders. No plugin placeholder resolves there, only PlaceholderAPI tokens. Put the colour on the menu line that carries `{companion-lore}` |

### Prefix tags in a display name

`[rgb]`, `[small]` and `[center]` are PREFIX tags: SnLib reads them at the start of a finished
line and nowhere else. That is why a `{group-color}` carrying `[rgb]` has to be first on its line,
and it used to bite display names too - an egg in `eggs.yml` or a companion in `companions/<id>.yml` whose
`display-name` began with `[rgb]` drew its gradient where it was the whole line but printed the
literal tag in chat, because a name spliced into a message as `{egg}` or `{companion}` lands in the
middle of the line:

```
SnCompanions | You opened [rgb]Basic Egg and got Ember Fox x1.
```

**That is why both files have their display-name prefix tags applied when the file is
read**, so the name looks the same wherever it is printed. Nothing in your
files changes: `eggs.yml` and `companions/` are seed-only and untouched, and the value is stored
already expanded, so a name is never expanded twice. `[center]` is the one tag dropped from the
chat copy - centering a fragment that lives inside somebody else's line means nothing.

{% hint style="info" %}
`groups:` is marked `# sn:extensible`, so a `color:` key is never inserted into a group of a
config that already exists. Until you write one, `{group-color}` falls back to the colour codes
that group's `display` already starts with (`"&9Rare"` yields `&9`, `"&#ff00aa&lEpic"` yields
`&#ff00aa&l`), so the placeholder is correct either way.
{% endhint %}

### The bulk delete icons

`bulk_delete.yml` draws a group with two templates, `group-button` and `group-empty`, and both
take their material from the same `{icon}` the group names under `menus.bulk-delete.icons` in
`config.yml`. A group the player stores nothing of therefore keeps its own icon and only greys
its name and lore, so the row never changes shape under the cursor.

{% hint style="info" %}
A `guis/bulk_delete.yml` whose `group-empty` is still a `GRAY_STAINED_GLASS_PANE` keeps it: SnLib
never overwrites a value your file already has. Set `templates.group-empty.material` to
`"{icon}"` by hand, or delete the file and restart to have it reseeded.
{% endhint %}

### Taking a companion out: the two triggers

`templates.companion-entry` in `main.yml` declares two lists that both run the same action, so a stored
companion leaves the storage either way:

```yaml
    shift-right-click-actions:
      - "[companions-extract] {instance}"
    drop-click-actions:
      - "[companions-extract] {instance}"
```

`drop-click-actions` is the Q key, Ctrl+Q with it, and it needs SnLib 1.31.0. Declaring it is also what lets Q
reach the cell at all: `main.yml` runs with `strict-clicks: true`, which discards every key
outside the four basic mouse clicks *unless the item under the cursor declares that key itself*.
So Q takes a companion out over a storage cell and does nothing anywhere else in the menu, and the
hotbar numbers, F and the offhand swap stay inert everywhere.

The equipped slot markers on the top row deliberately declare neither list. Taking an equipped companion
out is refused anyway - it would strand its slot and its buff - so the key is simply absent there
rather than present and rejected.

{% hint style="warning" %}
**Deleting either list does not stick.** `guis/main.yml` is managed and carries no extensible
marker, so the next boot merges any key you delete straight back. A deletion there is never an
opt-out. What holds instead:

- `companion-items.enabled: false` in `config.yml` - the intended switch, and it stops both triggers.
- `update-configs: false` in `config.yml` - freezes merging for **every** file, so you take on
  adding future keys by hand.
- a `# sn:extensible` comment line written directly above `companion-entry:` in your own `main.yml` -
  SnLib treats a marker you type as your decision and stops inserting anything under that
  template, including keys a future SnCompanions version adds there, logging one warning naming what it
  withholds.

The last one is how you keep exactly one of the two triggers: delete the list you do not want and
mark the template.
{% endhint %}

## lang/

Every user facing string lives in `lang/messages_<code>.yml`. English and Spanish ship with
the plugin. Pick the active one with the `lang` key at the top of `config.yml`, which falls
back to English when the named file is missing. To add a language, copy `messages_en.yml` to
`messages_<code>.yml`, translate the values, and point `lang` at the new code.

New keys are merged into your existing file on boot, with your values and your comments left
alone, so an update never overwrites a line you restyled.

**Five keys belong to the [hatch show](#the-egg-animation-switch):**

| Key | Sent when | Placeholders |
|---|---|---|
| `messages.egg-animation-on` | the player turned the show on from the slot-36 button | none |
| `messages.egg-animation-off` | the player turned it off from the same button | none |
| `messages.egg-animating` | the player tried to BUY an egg while their previous one is still hatching. Refused before anything is charged; a free `/companions admin openegg` is never refused for this | none |
| `menus.egg-reveal` | not sent - it is the floating LINE drawn above the companion the show reveals | `{companion}` |
| `menus.egg-reveal-bulk` | the same line when the open produced more than one companion | `{companion}` `{more}` |

The last two are labels rather than messages: they are drawn above the revealed companion for the
length of `reveal-ticks`, carry no prefix and are best kept short. `{more}` is how many OTHER
companions the same open produced; the full list still reaches chat when the show ends.

**One key belongs to the [eggs menu](#the-eggs-menu):**

| Key | Sent when | Placeholders |
|---|---|---|
| `messages.egg-unknown` | the shop was asked for an egg `eggs.yml` does not declare - an `eggs.default` that names a renamed or deleted key, or a price button clicked after that same edit | none |

Typing an id on `/companions eggs <egg>` cannot reach it: that argument only accepts ids that exist
right now. When it fires on the way IN, both doors - the command and the egg button of `main.yml` -
refuse to open rather than showing a blank window; the player's cursor still points at the id they
asked for, so fixing `eggs.yml` and reloading is enough.

**One key is sent when a player right clicks a companion head that belongs to somebody else,**
described under [companion-items](#a-companion-item-remembers-who-took-it-out):

| Key | Sent to | Placeholders |
|---|---|---|
| `messages.companion-item-not-yours` | a player redeeming a companion item stamped with another player's UUID | `{owner}` |

**One key is sent to the player who RECEIVES something from an admin command:**

| Key | Sent by | Placeholders |
|---|---|---|
| `messages.companion-received` | `/companions admin give` | `{amount}` `{companion}` `{level}` |

Blank the value to switch that notification off for everyone, or suppress it per command with
the `-s` flag (see [Commands](commands.md#silent-flags)). Offline players are never messaged.
`/companions admin openegg` does not use it: what its target reads is the egg's own reward line,
announced by the egg engine exactly as it would be for a bought egg.

**Eight keys belong to BUYING an egg.** None of them is ever seen by
`/companions admin openegg`, which is a gift:

| Key | Sent when | Placeholders |
|---|---|---|
| `messages.egg-creative` | a creative-mode player tries to buy while `eggs.allow-creative` is off | none |
| `messages.egg-cooldown` | the egg's own `cooldown-seconds` is still running for that player | `{time}` |
| `messages.egg-not-for-sale` | the egg declares no `opens:` button, so it has no price at all | `{egg}` |
| `messages.egg-no-economy` | the egg is priced in `vault` and the server has no economy plugin | none |
| `messages.egg-no-currency` | the egg is priced in an EdTools currency EdTools is not serving | `{currency}` |
| `messages.egg-no-money` | the player cannot pay, or their balance moved between the check and the charge | `{price}` `{currency}` |
| `messages.egg-charged` | the receipt, the moment the money is taken and just before the companions land | `{price}` `{currency}` |
| `messages.egg-refunded` | right after `messages.egg-failed`, when a failed open was a purchase | `{price}` `{currency}` |

`{price}` is the cost written with thousands separators. `{currency}` is the money word, and it
comes from the **`menus.eggs` block** at the end of the same file:

```yaml
menus:
  eggs:
    currency-vault: "Coins"
    # currency-orbs: "Orbs"
```

`currency-vault` is the only key that ships with a value, because every install has a server
economy to name. An egg priced with `price.currency: edtools:<id>` reads `currency-<id>` instead,
and those ids are invented in your own EdTools configuration - so none of them can ship. Add one
key per id you want renamed; an id with no key simply shows itself. The same block feeds the egg
menu, so renaming a currency renames it everywhere at once.
