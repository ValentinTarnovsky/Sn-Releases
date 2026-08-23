# Configuration

SnPets ships with the YAML files below. `config.yml`, the language files and the menu layouts
are managed by SnLib: new keys are auto-merged on boot, and your values and comments are
preserved. Set `update-configs: false` to freeze them, in which case SnLib only warns about
missing keys instead of inserting them.

`traits.yml`, `boost-grades.yml`, `boxes.yml` and the `pets/` folder are seeded once, on a
fresh install, and never written to again. A pet file you delete stays deleted, and a file you
add is picked up on the next reload; one file is one pet, and the file name is its id. Boxes
work the same way but live as keys inside `boxes.yml` rather than as separate files - if you are
upgrading from a version that used a `boxes/` folder, see [boxes.yml](#boxesyml), which is
migrated for you automatically.

## config.yml
```yaml
# ============================================================
#  SnPets - configuration
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

# Runtime debug output (also toggleable live via /pets debug).
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
  # Aliases of /pets. Re-read on /pets reload.
  aliases: [pet]

  # Pets listed per page by /pets admin list. Raising it makes one command
  # print more lines at once and pushes more of the sender's chat history
  # off screen; 1 to 50, values outside that are clamped.
  list-page-size: 8

# ------------------------------------------------------------
#  Database. type=sqlite needs nothing else; type=mysql reads host/port/etc.
#  ONE SERVER PER DATABASE: SnPets loads a player on join, caches them in
#  memory and saves on quit, so two servers pointing at the same MySQL will
#  overwrite each other and silently destroy pets. Cross-server sharing is
#  not supported.
# ------------------------------------------------------------
database:
  # sqlite or mysql
  type: sqlite
  # MySQL connection (ignored when type is sqlite).
  host: localhost
  port: 3306
  database: snpets
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
#  Groups. A group is a free-text label a pet file points at with its
#  "group" key. It is not a rarity system: the plugin uses it for the
#  storage sort order, the bulk delete buttons, the pet lore and the colour
#  the menus draw that pet's lines in, nothing else. A pet whose group is
#  not listed here sorts last and gets no bulk delete button.
# ------------------------------------------------------------
# sn:extensible
groups:
  common:
    # Name shown in the pet lore and on the bulk delete button.
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
#  Pet storage.
# ------------------------------------------------------------
storage:
  # Pets a player may keep before permissions or purchases raise it.
  base-capacity: 54

  # Ceiling of the PURCHASED storage that "/pets admin storage give|set" may
  # leave on a player. 0 disables it, which is the shipped value: storage is
  # paginated, so unlike the equip slots it has no visual limit to agree with.
  # A command that would go past the ceiling is NOT cancelled - it is clamped to
  # the ceiling and says so. This does not limit the "snpets.storage.<n>"
  # permission grants, which are a separate half of the same total.
  max-capacity: 0

# ------------------------------------------------------------
#  Equip slots.
# ------------------------------------------------------------
slots:
  # Pets a player may equip at once before permissions or purchases raise it.
  base-count: 1

  # Ceiling of the PURCHASED slots that "/pets admin slots give|set" may leave on
  # a player. 0 disables it. Keep it equal to the number of 's' cells in the
  # layout of guis/main.yml (7 in the shipped menu): the menu can only draw that
  # many, so slots past it are bought and never usable. The two files are not
  # read from each other on purpose - this comment is the link.
  # A command that would go past the ceiling is NOT cancelled - it is clamped to
  # the ceiling and says so. This does not limit the "snpets.slots.<n>"
  # permission grants, which are a separate half of the same total.
  max-count: 7

  # Forbid equipping two pets that belong to the same group. Off by default: a
  # player may equip as many pets of one group as they have slots for. Pets that
  # declare no group are never blocked by this rule.
  unique-group: false

# ------------------------------------------------------------
#  Formation. The equipped pets stand on an arc around their owner that turns
#  with the owner's camera. The arc is always used whole, end to end: the pets
#  spread evenly across it and slide over whenever one is equipped or unequipped.
#  The arc is cut from a circle or from an oval, see "shape" below.
#  "Evenly" means an even share of the ANGLE, which is an even spacing on the
#  ground only on a CIRCLE. On an OVAL the pets near the ends sit closer together
#  than the ones near the middle - about 23% closer at 4 pets and 28% at 6, at the
#  values below. 1, 2 and 3 pets are unaffected.
# ------------------------------------------------------------
formation:
  # Blocks between the owner and the arc at their SIDES. Under shape CIRCLE this
  # is the distance in every direction; under shape OVAL the back of the arc uses
  # back-radius instead.
  radius: 2.0
  # CIRCLE keeps every pet at "radius" blocks. OVAL keeps "radius" at the owner's
  # sides and "back-radius" straight behind (and in front), so the arc hugs the
  # owner's back and opens out at the flanks.
  # Delete this key and the plugin uses CIRCLE, not the OVAL shipped here: that is
  # what keeps a config written before 1.6.0 on the circle it always had. A value
  # that is neither logs one warning and falls back to CIRCLE too.
  shape: OVAL
  # Blocks between the owner and the arc straight behind them. Read on every load
  # but only USED when shape is OVAL; a CIRCLE ignores it in favour of radius.
  back-radius: 1.4
  # Blocks above the owner's feet the pets float at.
  height-offset: 1.9
  # Total width of the arc in degrees, capped at 360. Read together with
  # arc-center-offset below: at the default centre of 180, an arc of 180 puts the
  # first pet exactly at the owner's right and the second exactly at their left,
  # and going past 180 swings both ends a little in FRONT of the shoulders (at the
  # 190 shipped here, 0.12 blocks). A LONE pet takes the right END of the arc, not
  # the middle, so a single pet moves forward with them. At a different
  # arc-center-offset "past 180" swings the ends somewhere else entirely.
  arc-degrees: 190.0
  # Degrees from the owner's facing to the centre of the arc. 180 puts the arc
  # behind the owner, 0 in front of them, 90 at their right.
  arc-center-offset: 180.0
  # Where a pet looks: OWNER_YAW (the same way as the owner), OUTWARD (away from
  # the owner) or CENTER (at the owner).
  facing: OWNER_YAW
  # Degrees added to every pet's facing, to correct a head texture that is not
  # drawn looking forward. Under the default facing of OWNER_YAW, 180 points every
  # head OPPOSITE the owner's yaw - which is what faces them toward the camera in
  # third person, and toward anyone standing behind the owner. (Turning around
  # never shows you your own pets: the arc is camera-relative and comes with you.)
  # Note that 180 also swaps OUTWARD and CENTER.
  facing-offset: 180.0

# ------------------------------------------------------------
#  Animation. ONE shared task moves every pet of every player; there is no task
#  per pet and no task per player.
# ------------------------------------------------------------
animation:
  # Ticks between formation updates, minimum 2. The client interpolates between
  # updates, so 2 already looks fluid and a lower value would only cost tick.
  interval-ticks: 2
  # Uniform scale of a floating pet head.
  head-size: 1.3
  # Blocks a client keeps rendering a pet for, and the radius the plugin scans
  # for viewers.
  view-range: 48
  # Hands the movement packet write to a dedicated thread instead of the server
  # thread. Turn it off to send inline.
  async-packet-send: true
  # Vertical bob of a pet.
  bounce:
    # Blocks the pet rises and falls; 0 keeps the pets perfectly still.
    height: 0.06
    # Radians of bob phase advanced per animation tick; higher bobs faster.
    speed: 0.1

# ------------------------------------------------------------
#  BetterModel models. A pet whose file declares a "model" block is drawn as an
#  animated model instead of a floating head, with one animation while it holds
#  still and another while it moves. BetterModel is optional: without it, with
#  the switch below off, or when the engine does not know the model id, the pet
#  falls back to its head and nothing breaks. Pets that fell back pick their
#  model up on their own when BetterModel enables or reloads its models.
#  The vertical bob under "animation" is never applied to a model: a model
#  animates itself, so use height-offset below to place it instead.
# ------------------------------------------------------------
models:
  # Draw pets that declare a model as models. Off renders every pet as its head
  # even with BetterModel installed.
  enabled: true
  # Blocks per second of pet movement above which the moving animation plays.
  # The pets also travel when the owner only turns the camera, because the whole
  # arc swings with it, so looking around counts as moving.
  move-threshold: 0.8
  # Ticks the pet must stay below the threshold before the idle animation comes
  # back. Keeps a single step from flickering between the two animations.
  idle-delay-ticks: 8
  # Blocks added to a model pet's height, on top of formation.height-offset.
  # Heads float; a model usually wants to sit lower or stand on the ground, so a
  # negative value here lowers only the pets drawn as models. The two are SUMMED,
  # so with formation.height-offset at 1.9 a model that should walk on the floor
  # needs roughly -1.9 here.
  height-offset: 0.0
  # There is nothing to smooth here, which is why no setting for it exists: a
  # model pet's bones ride an invisible carrier that moves exactly like the heads
  # beside them, and the client interpolates that carrier on its own.

# ------------------------------------------------------------
#  Holograms. Text drawn above every pet, riding the pet itself: the client
#  moves it with the pet, so it never lags behind. One line of text is one
#  packet entity; nothing is spawned for a pet whose lines are empty.
#  A pet file may declare its own "hologram:" block (enabled, lines,
#  height-offset); a pet that declares none uses default-lines below.
#  Placeholders: every pet placeholder of the menus ({pet} {level} {level-cap}
#  {exp} {exp-next} {percent} {group} {group-color} {trait} {buff} {buff-value})
#  plus {owner}. PlaceholderAPI tokens resolve against the OWNER.
#  The text is only rewritten when something on the pet changes - a level up, a
#  trait or boost roll, an admin edit - never on the animation tick.
# ------------------------------------------------------------
holograms:
  # Master switch. Off spawns nothing at all.
  enabled: true
  # Blocks above the pet's origin the FIRST (top) line sits at. Careful: the
  # origin is not the same for both kinds of pet. A head pet's is the head
  # itself (formation.height-offset above the owner's feet); a model pet's is
  # its carrier, which also carries models.height-offset - so a model pushed
  # down to stand on the ground needs a LARGER offset here, or the text lands
  # inside it. A pet file can override this per pet, which is what the golem
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
  # Lines of a pet whose file declares no hologram block. Empty draws nothing.
  default-lines:
    - "{pet}"
    - "&7Lv. &f{level}&7/&f{level-cap}"

# ------------------------------------------------------------
#  Visibility. Two independent switches per player, stored in the database and
#  kept across relogs, so all four combinations are valid:
#    /pets toggle -> hides the player's OWN pets, from themselves AND from
#                    everyone else (permission snpets.toggle)
#    /pets hide   -> hides EVERY other player's pets, for that player only
#                    (permission snpets.hide)
# ------------------------------------------------------------
visibility:
  # Seconds a player must wait between two uses of /pets toggle. 0 disables the
  # cooldown. It survives a relog, so reconnecting does not reset it.
  own-cooldown-seconds: 3
  # Seconds a player must wait between two uses of /pets hide. Counted
  # separately from the one above, so one toggle never blocks the other.
  others-cooldown-seconds: 3

# ------------------------------------------------------------
#  Experience. Every pet declares its own source and its own ratio in its
#  pets/<id>.yml file; this section only decides which sources are live at all,
#  how often the playtime one pays and what a level up feels like. All equipped
#  pets gain from the same event, each one from its own source. Stored pets
#  never gain anything, and a pet at its level cap freezes: it keeps what it had
#  and resumes exactly there if the cap ever rises.
# ------------------------------------------------------------
experience:
  # Master switch of every experience source. Off freezes every pet where it is.
  enabled: true

  # Per-source switches. Turning one off only stops the gain: no level is ever
  # touched and no pet loses anything.
  sources:
    # Pet experience per point of vanilla XP the owner picks up. The owner's XP
    # is observed, never consumed.
    vanilla-xp: true
    # Pet experience per block broken. A pet may declare a material whitelist in
    # its own file; without one every block counts, which on a mine or generator
    # world is thousands of events a second.
    block-break: true
    # Pet experience per mob killed. A dead player is not a mob kill: PvP feeds
    # the damage-dealt source instead.
    mob-kill: true
    # Pet experience per point of damage the owner deals, melee and projectile,
    # counted after armor and after the pets' own damage buff.
    damage-dealt: true
    # Pet experience per minute the owner stays connected.
    playtime: true

  playtime:
    # Seconds between playtime grants. Every pet's playtime ratio is read per
    # MINUTE whatever this says, so changing it only changes how often the grant
    # is paid, never what a pet earns per minute of play.
    interval-seconds: 60

  level-up:
    # Sends messages.pet-level-up to the owner when one of their equipped pets
    # gains a level.
    message: true
    # Sound played to the owner on level up. "none" plays nothing.
    sound: "ENTITY_PLAYER_LEVELUP 1.0 1.6"

# ------------------------------------------------------------
#  Buffs. A pet declares which of the three it grants, what it gives at level 1
#  and what each level adds. The engine then computes, per buff:
#
#    base       = initial + (level - 1) x per-level
#    pet buff   = base x (1 + buff boost% + trait buff%)
#    player     = the sum of every equipped pet
#    effect     = that sum capped by the "cap" below
#
#  The two modifiers are added to each other and multiply the base once; the cap
#  applies LAST, to the summed total, never per pet. A total that comes out
#  negative is treated as zero.
# ------------------------------------------------------------
buffs:
  # Master switch of all three buffs.
  enabled: true

  damage:
    # Whether pets grant the damage buff at all.
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
    # Whether pets grant the resistance buff at all.
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
    # Whether pets grant the speed buff at all.
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
#  Traits. ONE global table in traits.yml: any trait can land on any pet, and a
#  pet carries exactly one at a time. Rolling again replaces it and always
#  spends the ticket, because the roll normalizes over the table and can never
#  come up empty. The numbers in traits.yml are WEIGHTS, not percentages.
#  A trait grants up to three things at once: percentage points of pet
#  experience, flat levels on the pet's cap, and percentage points on ONE named
#  buff - which only pays out on a pet that actually grants that buff.
# ------------------------------------------------------------
traits:
  # Master switch of the trait system. Off refuses every trait roll and makes
  # every trait worth 0; nothing is cleared, so switching it back on restores
  # every pet's trait exactly as it was.
  enabled: true

  # The roulette the traits menu plays before it reveals what was rolled. Same
  # four knobs, same names and same limits as a box's "animation:" block: both
  # run on the ONE shared spinner, so a number typed here and the same number
  # typed under a box's key in boxes.yml behave identically.
  roll:
    # Play the roulette at all. Off reveals the trait immediately.
    enabled: true
    # Ticks one spin lasts, clamped to 4-600.
    duration-ticks: 30
    # Ticks between two frames of the spin, minimum 2; lower spins faster.
    step-ticks: 2
    # Sound played on every frame of the spin. "none" plays nothing.
    step-sound: "UI_BUTTON_CLICK 0.6 1.6"
    # Sound played once the rolled trait is revealed. "none" plays nothing.
    reveal-sound: "ENTITY_PLAYER_LEVELUP 1.0 1.4"

# ------------------------------------------------------------
#  Boosts. ONE global grade ladder in boost-grades.yml, shared by the three
#  stats every pet carries. Same rule as the traits: the numbers there are
#  WEIGHTS, the roll always produces a grade and the dice is always spent.
#  A grade is worth a value inside its own min/max range, picked once per pet
#  and stable for that pet's whole life, and either end may be negative.
#    experience -> exp x (1 + experience grade% + trait exp%)
#    level      -> floor(pet level cap x (1 + level grade%)) + trait levels
#    buff       -> pet buff base x (1 + buff grade% + trait buff%)
#  Rolling all three stats at once costs a normal dice; rolling one chosen stat
#  costs a special dice.
# ------------------------------------------------------------
boosts:
  # Master switch of the boost system. Off refuses every boost roll and makes
  # every grade worth 0; nothing is cleared, so switching it back on restores
  # every pet's three grades exactly as they were.
  enabled: true

  # The roulette the boosts menu plays before it reveals what was rolled. Same
  # four knobs, same names and same limits as a box's "animation:" block.
  roll:
    # Play the roulette at all. Off reveals the grades immediately.
    enabled: true
    # Ticks one spin lasts, clamped to 4-600.
    duration-ticks: 30
    # Ticks between two frames of the spin, minimum 2; lower spins faster.
    step-ticks: 2
    # Sound played on every frame of the spin. "none" plays nothing.
    step-sound: "UI_BUTTON_CLICK 0.6 1.6"
    # Sound played once the rolled grades are revealed. "none" plays nothing.
    reveal-sound: "ENTITY_PLAYER_LEVELUP 1.0 1.4"

# ------------------------------------------------------------
#  Fusion. Two pets of the SAME id become the pet that their own
#  pets/<id>.yml names as its fusion target: it is a pet-to-pet map, never a
#  rarity ladder, and a pet whose file names no target cannot be fused at all.
#  The target, the success chance and the price of one attempt are declared per
#  pet in that file; this section holds only the rules every fusion obeys.
#  On a FAILED roll both parents are lost and the price is still paid. That is
#  a normal outcome of a legal attempt, not an error.
# ------------------------------------------------------------
fusion:
  # Master switch of the whole feature. Off refuses every fusion and every
  # Fuse All; nothing is deleted and no pet loses its fusion target.
  enabled: true

  # Which parent the result takes each preserved field from.
  #   BEST   - per FIELD, whichever parent holds the better value: the higher
  #            level, the higher experience, the rarer trait and the better
  #            grade of each boost stat. The result may take its level from one
  #            parent and its trait from the other.
  #   FIRST  - everything comes from the pet in the left input slot.
  #   SECOND - everything comes from the pet in the right input slot.
  keep-from: BEST

  # What survives a fusion at all. A field switched off here is not taken from
  # either parent: the result carries what a brand new pet carries, which is
  # level 1, no experience, no trait and no boost grades.
  keep:
    # Carry over a level. It is capped by the TARGET pet's own max-level, since
    # every pet declares its own ceiling.
    level: true
    # Carry over the experience banked toward the next level.
    exp: true
    # Carry over a trait. Under BEST the result keeps a trait when EITHER
    # parent had one, and the rarer of the two when both did.
    trait: true
    # Carry over the experience boost grade.
    boost-experience: true
    # Carry over the level boost grade.
    boost-level: true
    # Carry over the buff boost grade.
    boost-buff: true

  cost:
    # Charge the `fusion.cost` each pet file declares. The price is money, taken
    # through Vault or through whatever economy backend SnLib is configured
    # with. A server with no economy plugin at all fuses for FREE and says so
    # once in the console: refusing instead would make every pet that names a
    # price unfusable. Off makes every attempt free without editing pet files.
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

  # Default for a pet whose pets/<id>.yml fusion block declares no broadcast key
  # of its own; a pet file overrides it either way, so you can announce the rare
  # fusions and stay quiet about the common ones. Fuse All never broadcasts,
  # whatever any pet says: it rolls per pair, so a good run would post one line
  # per winning pair.
  # HEADS UP: two shipped pet files already override this - stone_golem.yml sets
  # broadcast: true and ember_fox.yml sets false - and pets/ is seed-only, so
  # those stay whatever they are no matter what you put here. Turning this on will
  # NOT make Ember Fox announce; edit or delete the key in its file to do that.
  broadcast: false

  # Sound played to the owner when a fusion produces a pet. "none" plays nothing.
  success-sound: "ENTITY_PLAYER_LEVELUP 1.0 1.2"
  # Sound played to the owner when a fusion fails and both parents are lost.
  # "none" plays nothing.
  failure-sound: "ENTITY_ITEM_BREAK 1.0 0.8"

# ------------------------------------------------------------
#  Pet boxes. A box is a physical item declared in boxes.yml, one key per box,
#  and opened with a right click. What a box contains, how it looks, which
#  sound it plays and what it says on opening lives in that file; this section
#  holds only the rules every box obeys.
#  Success chance: a box carries one, drawn from its own chance.min/chance.max
#  when it is handed out - or forced with the [chance] argument of
#  /pets admin givebox - and written onto the item itself. A failed open
#  CONSUMES the box and grants nothing; a successful one rolls the box's
#  weighted table, which always produces a pet.
#  Storage full never blocks RECEIVING a box, only OPENING it: a single open
#  refuses and hands the box straight back with its percentage intact, a bulk
#  open opens what fits and returns the rest. Nothing is ever lost.
# ------------------------------------------------------------
boxes:
  # Master switch of box opening. Off leaves every box item in place and in
  # inventories, but a right click refuses and hands the box straight back.
  enabled: true

  # Let a creative-mode player open boxes. Off by default: creative
  # middle-click copies any stack with its NBT intact, so one box would mint
  # pets without limit.
  allow-creative: false

  # Blocks a box right click steps aside for, so the block wins the click.
  # Without them, clicking a chest with a box in hand would open the box
  # instead of the chest. An empty list makes every block openable.
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

  bulk:
    # Let shift + right click open the whole held stack at once. A box whose
    # own file turns its reveal animation on can never be bulk opened: that
    # click opens one box and says so, because 64 spinners firing together is
    # what this rule exists to prevent.
    enabled: true
    # Most boxes one shift + right click may open. A vanilla stack holds 64, so
    # raising this above 64 only matters for a box with a custom
    # max-stack-size. Minimum 2.
    max-per-click: 64
    # Announce a bulk open too, once per DISTINCT pet won, using the box's own
    # feedback.broadcast-key. Off by default: a wide table would otherwise turn
    # one stack into a wall of chat.
    broadcast: false

# ------------------------------------------------------------
#  Pet items. A stored pet can be taken out of the storage as a physical head
#  (shift + right click on it in the main menu) and redeemed back by anyone
#  with a right click, which is how pets change hands. The item carries the
#  pet's whole state - level, experience, trait and the three boost grades - so
#  a pet that comes back is the pet that left. A redeem into a full storage
#  hands the item straight back, and so does every other refusal: the item is
#  the pet, and it is never destroyed by a refusal.
#  An EQUIPPED pet cannot be taken out (unequip it first) and neither can one
#  whose pets/<id>.yml is gone, because it has no face left to travel with.
#  The boxes.blocked-blocks list above applies to the redeem click too, so a
#  pet item steps aside for a chest exactly like a box does.
# ------------------------------------------------------------
pet-items:
  # Master switch of extracting and redeeming. Off refuses both and hands a
  # clicked item back. Items already in circulation stay in inventories.
  enabled: true

  # Let a creative-mode player redeem pet items. Off by default: creative
  # middle-click copies any stack with its NBT intact, so one pet item would
  # mint pets without limit.
  allow-creative: false

  # Look of the extracted item. Its MATERIAL is always the pet's own head and
  # cannot be set here; the name and lore below are yours.
  # Every pet placeholder the menus use works here:
  #   {pet} {pet-lore} {group} {group-color} {level} {level-cap} {exp}
  #   {exp-next} {percent} {bar} {trait} {buff} {buff-value} {boosts}
  # {pet-lore} and {boosts} are multi-line and expand to one lore line each.
  item:
    display-name: "{pet}"
    lore:
      - "{pet-lore}"
      - ""
      - "&8Group: &r{group-color}{group}"
      - "&7Level &f{level}&7/&f{level-cap}"
      - "&7Exp &f{exp}&7/&f{exp-next} &8({percent}%)"
      - "&7Trait: &r{trait}"
      - "{boosts}"
      - ""
      - "&a&lRIGHT CLICK"
      - "&2Redeem this pet into your storage"

# ------------------------------------------------------------
#  Worlds. TWO separate lists, on purpose: where pets are DRAWN and where their
#  buffs APPLY are different questions, and mixing them would turn a cosmetic
#  decision into a combat-balance one.
# ------------------------------------------------------------
worlds:
  # Whitelist of the worlds pets are drawn in. An EMPTY list means every world.
  # Outside this list the pets stay equipped and their buffs keep applying;
  # only the rendering stops. Use worlds.buff-disabled below to switch a buff
  # off - this list never does.
  render: []
  # Narrows the list above for OTHER players' pets only. An EMPTY list means
  # "wherever render allows". Name a world here to let players see their OWN
  # pets there while everyone else's stay hidden, as in a lobby where a crowd of
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
#  Menus. Everything the seven menus look like lives in guis/<id>.yml: the
#  titles, the masks, every material, every name, every lore line and every
#  click action. These two keys are the exceptions, because neither can be
#  written as an item field.
#  There is deliberately no page-size key: the number of cells a menu gives its
#  paged region IS its page size, so widening the letter run in that menu's
#  layout widens the page and the arrows follow. A second knob could only ever
#  disagree with the picture.
# ------------------------------------------------------------
menus:

  # The experience bar spliced into a pet's {bar} lore line.
  progress-bar:
    # Cells the bar is drawn with, 1 to 64.
    length: 20
    # Glyph of a cell the pet already earned.
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
#  PlaceholderAPI. SnPets exposes %snpets_...% when PlaceholderAPI is
#  installed; without it nothing is registered and no key here does
#  anything. The tokens themselves are not configurable - another plugin's
#  scoreboard refers to them by name - so this section only decides how the
#  NUMBERS are written.
#
#  The full list:
#    %snpets_pets_total%      %snpets_pets_stored%     %snpets_pets_equipped%
#    %snpets_storage_used%    %snpets_storage_total%   %snpets_storage_free%
#    %snpets_slots_used%      %snpets_slots_total%     %snpets_slots_free%
#    %snpets_buff_damage%     %snpets_buff_resistance% %snpets_buff_speed%
#    %snpets_currency_trait-ticket%  %snpets_currency_dice-normal%
#    %snpets_currency_dice-special%
#    %snpets_equipped_<n>%    the pet in equip slot <n>, or the status.none word
#    %snpets_equipped_<n>_level%   _exp%   _exp_next%   _cap%
#
#  Every one of them is answered from memory: a player who is not loaded
#  reads as zero rather than making the server wait for a database read,
#  which is what lets a scoreboard use them every tick.
# ------------------------------------------------------------
placeholders:
  # Decimal places of the three buff percentages (%snpets_buff_*%). 0 prints
  # whole numbers, 2 prints 12.50. Written with a dot on every server no
  # matter its language, so a scoreboard condition comparing the value to a
  # number keeps working. 0 to 6.
  #
  # The default is 1 because that is what the MENUS print: raise it and the
  # same buff reads 12.50 on your scoreboard and 12.5 in the pet lore at the
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

### Capacity ceilings

Added in 1.9.0. `slots.max-count` and `storage.max-capacity` cap what the two admin capacity
commands may leave on a player.

| Key | Default | Caps |
|---|---|---|
| `slots.max-count` | `7` | the purchased equip slots `/pets admin slots give\|set` may leave |
| `storage.max-capacity` | `0` (off) | the purchased storage `/pets admin storage give\|set` may leave |

A command that would go past its ceiling is **not cancelled**. It writes the ceiling and tells the
admin it did, so on a stock install `/pets admin slots give Snopeyy 100` leaves 7 purchased slots
and prints `messages.admin-slots-clamped` before the usual confirmation. That notice is shown even
to an admin who typed `-sf`: that flag mutes success confirmations, and a number other than the one
typed is corrective information. Set a key to `0` to remove its ceiling.

{% hint style="info" %}
Keep `slots.max-count` equal to the number of `s` cells in the layout of `guis/main.yml` - 7 in the
shipped menu. The menu can only draw that many, so slots past it are bought and never usable.
SnPets does not read `guis/main.yml` to derive the ceiling, on purpose: two files you edit
independently should not silently depend on each other.
{% endhint %}

{% hint style="warning" %}
The ceiling bounds **purchases only**. It does not limit the `snpets.slots.<n>` /
`snpets.storage.<n>` permission grants, so a rank may still grant more than an admin command can -
the effective total a player ends up with is the highest of the config base, what their rank grants
and what they bought.

It also never lowers a row on its own. If you reduce `slots.max-count` on a live server, players
already above it keep what they have until the next `slots give`/`set` on them, which then clamps
them down to the new ceiling.
{% endhint %}

## traits.yml

```yaml
# ============================================================
#  SnPets - traits
#  ONE global table for the whole server: any trait here can land on any pet,
#  and a pet carries exactly one at a time. Rolling again replaces it.
#  Seeded once and never merged again: this file is yours. Every top-level key
#  below is a trait id. Add, rename and delete freely; a trait id still stored
#  on a pet whose entry is gone is never cleared, it renders marked as unknown
#  and is worth 0 until the entry comes back.
#  The "weight" numbers are WEIGHTS, not percentages: the roll normalizes over
#  the whole table, always produces a trait and always spends the ticket. They
#  do not have to add up to 100.
# sn:extensible-root
# ============================================================

# The default pack is deliberately small: five common traits and one rare one.
# Copy an entry to add your own.

studious:
  # Name shown in the trait index and on the pet.
  display-name: "&aStudious"
  # Lore shown under the name in the trait index.
  lore:
    - "&7Learns faster than the rest."
  # Slot of the trait index menu this entry renders at. Remove the key to let
  # the menu place it in the next free slot.
  slot: 11
  # Material of the index icon.
  icon: EXPERIENCE_BOTTLE
  # Draws the index icon as a player head instead of the material above. Accepts
  # a raw base64 payload, a basehead-/texture- prefixed value or an http skin
  # URL; an unreadable value falls back to the material with one warning.
  # head-texture: ""
  # Roll weight relative to every other entry. 0 retires the trait: pets that
  # already carry it keep it, it simply stops being rolled.
  weight: 30.0
  effects:
    # Percentage points added to the experience the pet gains. Negative allowed.
    exp-percent: 15.0
    # Flat levels added to the pet's level cap.
    level-bonus: 0
    buff:
      # Buff this trait pushes: DAMAGE, RESISTANCE or SPEED. Leave it empty for
      # a trait that touches no buff. A trait only pays out on a pet that grants
      # that same buff.
      type: ""
      # Percentage points added to the buff named above.
      percent: 0.0

veteran:
  display-name: "&eVeteran"
  lore:
    - "&7Has room to grow past its limit."
  slot: 13
  icon: GOLDEN_APPLE
  weight: 20.0
  effects:
    exp-percent: 0.0
    level-bonus: 5
    buff:
      type: ""
      percent: 0.0

swift:
  display-name: "&bSwift"
  lore:
    - "&7Only helps a pet that grants Speed."
  slot: 15
  icon: FEATHER
  weight: 25.0
  effects:
    exp-percent: 0.0
    level-bonus: 0
    buff:
      type: SPEED
      percent: 10.0

fierce:
  display-name: "&cFierce"
  lore:
    - "&7Only helps a pet that grants Damage."
  slot: 20
  icon: IRON_SWORD
  weight: 25.0
  effects:
    exp-percent: 0.0
    level-bonus: 0
    buff:
      type: DAMAGE
      percent: 10.0

stalwart:
  display-name: "&9Stalwart"
  lore:
    - "&7Only helps a pet that grants Resistance."
  slot: 22
  icon: SHIELD
  weight: 25.0
  effects:
    exp-percent: 0.0
    level-bonus: 0
    buff:
      type: RESISTANCE
      percent: 10.0

prodigy:
  display-name: "&6&lProdigy"
  lore:
    - "&7Rare. Learns fast and grows tall."
  slot: 24
  icon: NETHER_STAR
  weight: 5.0
  effects:
    exp-percent: 25.0
    level-bonus: 3
    buff:
      type: ""
      percent: 0.0
```

## boost-grades.yml

```yaml
# ============================================================
#  SnPets - boost grades
#  ONE global ladder for the whole server, shared by all three boost stats
#  (experience, level, buff). This file exists so a grade table is never copied
#  into a per-pet file again.
#  Seeded once and never merged again: this file is yours. Every top-level key
#  below is a grade id. A grade id still stored on a pet whose entry is gone is
#  never cleared, it renders marked as unknown and is worth 0 until the entry
#  comes back.
#  The "weight" numbers are WEIGHTS, not percentages: the roll normalizes over
#  the ladder, always produces a grade and always spends the dice. They do not
#  have to add up to 100.
#  A grade is worth a value inside its min/max range, picked once per pet and
#  stable for that pet's whole life. Both ends may be NEGATIVE: a penalty grade
#  is content, not a mistake.
#  What each stat does with the percentage:
#    experience -> exp x (1 + experience grade% + trait exp%)
#    level      -> floor(pet level cap x (1 + level grade%)) + trait levels
#    buff       -> pet buff base x (1 + buff grade% + trait buff%)
# sn:extensible-root
# ============================================================

# The default ladder is deliberately short: five rungs, one of them a penalty.

flawed:
  # Name shown wherever the grade is rendered.
  display-name: "&8Flawed"
  # Roll weight relative to every other rung. 0 retires the rung: pets that
  # already carry it keep it, it simply stops being rolled.
  weight: 25.0
  # Low end of what this rung is worth, in percentage points. May be negative.
  min-percent: -5.0
  # High end. Set it equal to min-percent for a fixed-value rung.
  max-percent: -1.0
  # Stats this rung may be rolled for: experience, level, buff. An EMPTY list
  # means all three, which is what makes the file read as one shared ladder.
  stats: []

common:
  display-name: "&7Common"
  weight: 40.0
  min-percent: 0.0
  max-percent: 3.0
  stats: []

fine:
  display-name: "&aFine"
  weight: 20.0
  min-percent: 4.0
  max-percent: 8.0
  stats: []

superior:
  display-name: "&9Superior"
  weight: 12.0
  min-percent: 9.0
  max-percent: 15.0
  stats: []

mythic:
  display-name: "&6&lMythic"
  weight: 3.0
  min-percent: 16.0
  max-percent: 25.0
  stats: []
```

## pets/

One file per pet type, named after its id. Three examples are seeded on a fresh install:
`ember_fox.yml`, `gale_sprite.yml` and `stone_golem.yml`. A pet file declares the display
name, the group it belongs to, the head or BetterModel it renders as, its level cap and
experience curve, the buffs it grants per level and, since 1.8.0, the hologram drawn above
it. Copy one of the examples to add your own.

The `hologram:` block is optional and, because this folder is seeded once and never merged
again, it is never added to the pet files you already have. That is what
`holograms.default-lines` in `config.yml` is for: a pet that declares no block uses those
lines, so you only write a block for the pets that should differ. Only an explicit
`enabled: false` silences a pet - `gale_sprite.yml` ships that way as the mute example, and
`stone_golem.yml` ships its own `height-offset` because a model pet's label is measured from
the carrier its bones ride, which also carries `models.height-offset`.

Here is the seeded `pets/ember_fox.yml` in full, as a working reference:

```yaml
# ============================================================
#  SnPets - pet definition: ember_fox
#  One file per pet. The file name is the pet id used by commands, by the
#  fusion targets of other pets and by the database rows.
#  Seeded once and never merged again: this file is yours. Copy it to add a
#  pet, delete it to remove one. A pet id still stored on a player whose file
#  is gone is never deleted; it renders as a placeholder that cannot be
#  equipped or fused until the file comes back.
#  Every pet is independent: it declares its own level cap, curve, buff and
#  fusion target. There is no rarity table.
# ============================================================

# Name shown on the pet item.
display-name: "&6&lEmber Fox"

# Lore shown on the pet item, above the progress lines the menus add.
lore:
  - "&7A restless fox that burns brighter"
  - "&7the longer you fight beside it."

# Head texture shown by the renderer and on the pet item. Accepts a raw base64
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
#  Hologram. The text drawn above this pet, riding the pet itself so the two
#  move as one thing. Leave the whole block out and the pet uses
#  holograms.default-lines from config.yml; the look (spacing, scale, shadow,
#  background) always comes from the holograms band there.
# ------------------------------------------------------------
hologram:
  # Draw a label above this pet. False silences this pet alone.
  enabled: true
  # This pet's own lines, top first. Every menu placeholder works here, plus
  # {owner}; PlaceholderAPI tokens resolve against the owner.
  lines:
    - "{group-color}{pet}"
    - "&7Lv. &f{level}&7/&f{level-cap}"
    - "&8{owner}"
  # Blocks above the pet the top line sits at. Leave it out to follow
  # holograms.height-offset.
  # height-offset: 0.9

# This pet's own level cap, before trait and boost bonuses.
max-level: 50

# ------------------------------------------------------------
#  Experience.
# ------------------------------------------------------------
experience:
  # Experience needed to reach level 2.
  base: 500
  # Added to that requirement by each further level.
  per-level: 500
  # VANILLA_XP, BLOCK_BREAK, MOB_KILL, DAMAGE_DEALT or PLAYTIME.
  source: VANILLA_XP
  # Gain per unit of the source; here, per point of vanilla XP picked up.
  ratio: 1.0

# ------------------------------------------------------------
#  Buff. Only equipped pets apply it, and the totals of every equipped pet are
#  summed before the per-buff cap is applied.
# ------------------------------------------------------------
buff:
  # DAMAGE, RESISTANCE or SPEED.
  type: DAMAGE
  # Percent granted at level 1.
  initial: 2.0
  # Percent added by each level above 1.
  per-level: 0.1

# Group label declared in config.yml; drives the storage order, the bulk delete
# buttons and the lore, nothing else.
group: common

# ------------------------------------------------------------
#  Fusion. Two of THIS pet produce the pet named below. Remove the section to
#  make this pet unfusable.
# ------------------------------------------------------------
fusion:
  # Pet id produced by a successful fusion.
  into: stone_golem
  # Success chance in percent; on failure both parents are lost.
  chance: 50.0
  # Money charged per attempt; 0 is free.
  cost: 0
  # Announce a successful fusion of two of THIS pet to the whole server. Leave the
  # key out to follow config.yml fusion.broadcast.
  broadcast: false

# Pet ids that cannot be equipped alongside this one.
incompatible:
  - gale_sprite
```

## boxes.yml

**Every** pet box lives in this one file, one top-level key per box. The key is the box id the
admin give commands take, and the physical item is registered as `box_<id>`. Two examples are
seeded, `basic` and `rare`. Add, rename and delete keys freely.

The file is seed-only and its header carries `# sn:extensible-root`, so a key you delete stays
deleted and no key ever gains a sub-key from an update. Deleting the whole FILE re-seeds the two
examples on the next reload; delete the KEYS you do not want instead.

### Upgrading from the boxes/ folder

Before 1.4.0 each box was its own `boxes/<id>.yml` file. On the first boot after updating, the
plugin folds that folder into `boxes.yml` and renames it to `boxes-migrated/`. Nothing is
deleted, every box id is kept, and the box items already in players' inventories keep opening.

Three things worth knowing:

- A file name containing a dot cannot be a yml key, so `my.box.yml` migrates as `my_box` with a
  loud console warning. Its item id changes, so the copies of THAT box already handed out stop
  opening. Every other box is unaffected.
- Because `boxes.yml` is seed-only, a migrated box does **not** receive the keys 1.4.0 added.
  `chance` defaults to 100/100 and `feedback.fail-broadcast-key` to empty, which is exactly the
  behaviour those boxes had before. Add them by hand to any box you want to turn into a gamble,
  and add `{chance}` to its lore if you want the odds shown on the item.
- The per-box comments do not survive the move (a YAML reader cannot carry a file header onto a
  key). Each migrated key gets a `# Migrated from boxes/<file>` line, and the originals are all
  still there in `boxes-migrated/`.

If the migration cannot finish - an unreadable file, a full disk, a read-only data folder - the
plugin refuses to start instead of continuing with the two example boxes, and the console names
the cause. Your folder is left untouched; fix it and start the server again.

### The success chance

A box can fail to open. Each one declares a band:

```yaml
rare:
  chance:
    min: 60
    max: 90
```

When a box is **handed out**, a percentage is drawn in that band, rendered into the item's
`{chance}` name and lore, and written onto the stack itself. **Opening rolls the number on the
stack**, not the box's current band. That means lowering the band never devalues the boxes
already in circulation, a shop can sell guaranteed boxes beside gambled ones, and two stacks of
the same box at different odds never merge.

`/pets admin givebox <player> <box> [amount] [chance]` forces an exact percentage instead of
drawing one - `100` is what a shop wants. `giveallbox` draws once, so everybody gets the same
number.

A failed open **consumes the box** and grants nothing. The player gets
`messages.box-chance-failed` (or `messages.box-chance-failed-bulk` for a stack), the box's
cooldown starts as if it had opened, and if the box declares `feedback.fail-broadcast-key` the
loss is announced to the server. Leave both band ends at `100` for a box that always opens.

A box handed out before 1.4.0 carries no percentage and counts as 100%.

### The shipped file in full

```yaml
# ============================================================
#  SnPets - pet boxes
#  ONE file for every box. Every top-level key below is a box id: it is the id
#  the admin give commands take, and the physical item is registered as
#  "box_<id>". Add, rename and delete freely.
#  Seeded once and never merged again: this file is yours. Deleting the FILE
#  re-seeds the two examples on the next reload; delete the KEYS you do not
#  want, not the file. A box item already in a player's inventory whose key is
#  gone keeps its look but no longer opens.
#  Right click opens one box. Shift + right click opens the whole held stack,
#  which requires "animation.enabled: false" on that box.
#  Upgrading from an older version: a boxes/ folder is migrated into this file
#  automatically on the first boot and the folder is renamed to boxes-migrated/.
#  Nothing is deleted and the box items already handed out keep working.
# sn:extensible-root
# ============================================================

# ------------------------------------------------------------
#  basic - the plain example: no animation, no broadcast, no cooldown, and it
#  always opens.
# ------------------------------------------------------------
basic:

  # ----------------------------------------------------------
  #  The physical item. Every appearance field of the SnLib item spec works
  #  here: material, display-name, lore, glow, custom-model-data, enchantments,
  #  flags, max-stack-size and the rest.
  #  {chance} in the name or the lore is replaced, when the box is HANDED OUT,
  #  by the success chance that box copy was stamped with. It is written into
  #  the item itself, so two stacks handed out at different percentages never
  #  merge and never lose their odds.
  # ----------------------------------------------------------
  item:
    # Item material, or a head texture written as "texture-<base64>",
    # "basehead-<base64>" or an http skin URL, which makes it a player head.
    material: CHEST
    # Name shown on the box item, and the {box} placeholder of every box message.
    display-name: "&f&lBasic Pet Box"
    # Lore shown on the box item.
    lore:
      - "&7Right click to open."
      - "&7Shift + right click opens the whole stack."
      - ""
      - "&7Success chance: &f{chance}%"
      - ""
      - "&7Contains one pet:"
      - "&f60% &7Ember Fox"
      - "&f30% &7Stone Golem"
      - "&f10% &7Gale Sprite"
    # Adds the enchantment shimmer without an enchantment.
    glow: false

  # ----------------------------------------------------------
  #  Chance of the box OPENING at all, in percent.
  #  A box handed out without an explicit percentage is stamped with a value
  #  drawn uniformly between min and max, and that value is what it rolls when
  #  a player opens it. A failed open CONSUMES the box and grants nothing.
  #  /pets admin givebox <player> <box> [amount] [chance] overrides the draw
  #  with an exact number, which is how a shop sells guaranteed boxes.
  #  Leave both at 100 for a box that always opens. min > max is swapped, and
  #  both are clamped to 0-100.
  #  A box handed out by an older version carries no percentage and counts as
  #  100, so nothing already in circulation changes.
  # ----------------------------------------------------------
  chance:
    min: 100
    max: 100

  # ----------------------------------------------------------
  #  The drop table. One entry per pet id, which is a pets/<id>.yml file name.
  #  The numbers are WEIGHTS, not percentages: they are normalized over whatever
  #  the table holds, so 60/30/10 and 6/3/1 behave identically and the total does
  #  not have to add up to anything. A box that OPENS always produces a pet;
  #  the only way to get nothing is a failed chance roll above.
  #  An entry naming a pet with no file is dropped when the box loads, with one
  #  console warning, and the remaining weights renormalize on their own.
  # ----------------------------------------------------------
  drops:
    ember_fox:
      # How many of this pet one winning roll grants.
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
  #  Reveal animation. It delays only the FEEDBACK: the pet is granted the moment
  #  the box opens, so quitting mid-spinner loses nothing.
  #  An animated box CANNOT be bulk opened. A shift + right click on one opens a
  #  single box instead and says so, because 64 spinners at once is what the rule
  #  against it exists to prevent.
  # ----------------------------------------------------------
  animation:
    # Delay the reveal behind a short spinner instead of showing it instantly.
    enabled: false
    # Ticks between the open and the reveal. 20 ticks is one second.
    duration-ticks: 40
    # Ticks between two spinner frames.
    step-ticks: 4
    # Sound played on each spinner frame, as "SOUND_ID [volume] [pitch]".
    # "none" plays nothing.
    step-sound: "BLOCK_NOTE_BLOCK_HAT 1.0 1.6"
    # Sound played when the pet is revealed. "none" plays nothing.
    reveal-sound: "ENTITY_PLAYER_LEVELUP 1.0 1.4"

  # ----------------------------------------------------------
  #  Feedback. Every key points at an entry of lang/messages_<code>.yml, so the
  #  wording is translated and restyled with every other message rather than
  #  living here. Leave one empty to send nothing.
  #  Placeholders: {player} {box} {pet} {amount} {chance}
  # ----------------------------------------------------------
  feedback:
    # Line sent to the player who opened the box. Applies to a SINGLE open; a bulk
    # open sends messages.box-opened-bulk instead, which summarizes the stack.
    message-key: "messages.box-opened"
    # Line announced to the whole server. Empty announces nothing.
    broadcast-key: ""
    # Line announced to the whole server when an open FAILS its chance roll.
    # Empty announces nothing. A bulk open only announces when
    # boxes.bulk.broadcast is on in config.yml.
    # Placeholders: {player} {box} {amount} {chance}
    fail-broadcast-key: ""

  # Seconds a player must wait between two opens of THIS box. 0 disables the
  # cooldown entirely and costs nothing at runtime. A refused open never starts
  # it; a FAILED open does, because the box was consumed all the same.
  cooldown-seconds: 0

# ------------------------------------------------------------
#  rare - the other half of the feature set: a reveal animation, a server
#  broadcast, a cooldown, a row worth more than one pet, and a chance band that
#  makes an open a gamble.
# ------------------------------------------------------------
rare:

  # ----------------------------------------------------------
  #  The physical item.
  # ----------------------------------------------------------
  item:
    # Item material, or a head texture written as "texture-<base64>",
    # "basehead-<base64>" or an http skin URL, which makes it a player head.
    material: ENDER_CHEST
    # Name shown on the box item, and the {box} placeholder of every box message.
    display-name: "&5&lRare Pet Box"
    # Lore shown on the box item.
    lore:
      - "&7Right click to open."
      - ""
      - "&7Success chance: &f{chance}%"
      - "&8A failed open destroys the box."
      - ""
      - "&7Contains:"
      - "&f50% &7Stone Golem"
      - "&f30% &7Gale Sprite"
      - "&f20% &7Ember Fox &8x2"
    # Adds the enchantment shimmer without an enchantment.
    glow: true

  # ----------------------------------------------------------
  #  Chance of the box OPENING at all, in percent. Handed out without an
  #  explicit percentage, this box is stamped somewhere between 60 and 90 and
  #  keeps that number for good.
  # ----------------------------------------------------------
  chance:
    min: 60
    max: 90

  # ----------------------------------------------------------
  #  The drop table. The numbers are WEIGHTS, not percentages: they are
  #  normalized over whatever the table holds. A box that OPENS always produces
  #  a pet. A row naming a pet with no file is dropped when the box loads and
  #  the remaining weights renormalize on their own.
  # ----------------------------------------------------------
  drops:
    stone_golem:
      # How many of this pet one winning roll grants.
      amount: 1
      # Relative weight of this row.
      weight: 50
    gale_sprite:
      amount: 1
      weight: 30
    ember_fox:
      # A row worth more than one pet. The box is opened only when the WHOLE row
      # fits in the storage, so a bulk open never splits one box in half.
      amount: 2
      weight: 20

  # ----------------------------------------------------------
  #  Reveal animation. It delays only the FEEDBACK: the pet is granted the moment
  #  the box opens, so quitting mid-spinner loses nothing.
  #  An animated box CANNOT be bulk opened. A shift + right click on this one
  #  opens a single box instead and says so.
  # ----------------------------------------------------------
  animation:
    # Delay the reveal behind a short spinner instead of showing it instantly.
    enabled: true
    # Ticks between the open and the reveal. 20 ticks is one second.
    duration-ticks: 50
    # Ticks between two spinner frames.
    step-ticks: 4
    # Sound played on each spinner frame, as "SOUND_ID [volume] [pitch]".
    # "none" plays nothing.
    step-sound: "BLOCK_NOTE_BLOCK_HAT 1.0 1.8"
    # Sound played when the pet is revealed. "none" plays nothing.
    reveal-sound: "ENTITY_PLAYER_LEVELUP 1.0 1.2"

  # ----------------------------------------------------------
  #  Feedback. Every key points at an entry of lang/messages_<code>.yml.
  #  Placeholders: {player} {box} {pet} {amount} {chance}
  # ----------------------------------------------------------
  feedback:
    # Line sent to the player who opened the box. Applies to a SINGLE open.
    message-key: "messages.box-opened"
    # Line announced to the whole server. A bulk open only announces when
    # boxes.bulk.broadcast is on in config.yml, and this box cannot be bulk
    # opened anyway, because its animation is on.
    broadcast-key: "messages.box-broadcast"
    # Line announced to the whole server when an open FAILS its chance roll.
    fail-broadcast-key: "messages.box-fail-broadcast"

  # Seconds a player must wait between two opens of THIS box. 0 disables the
  # cooldown entirely and costs nothing at runtime. A refused open never starts
  # it; a FAILED open does, because the box was consumed all the same.
  cooldown-seconds: 3
```

## guis/

Seven menu layouts, all managed and all re-skinnable without touching code.

| File | Menu |
|------|------|
| `main.yml` | Pet storage, the screen bare `/pets` opens |
| `selector.yml` | Pet picker used when a screen needs one pet chosen |
| `traits.yml` | Trait rolling for one pet |
| `traits_index.yml` | Read-only catalogue of every trait and its odds |
| `boosts.yml` | Boost rolling for one pet |
| `fusion.yml` | Fusion, including the Fuse All bulk path |
| `bulk_delete.yml` | Bulk deletion by group |

{% hint style="warning" %}
A menu file defines a fixed number of cells. If you raise a player's equip slots above the
cell count of `main.yml`, the extra pets cannot be shown. The plugin warns about this at
runtime, and `/pets admin unequip` always recovers a pet that ended up out of reach.
{% endhint %}

### close-actions

All seven files carry the same top-level key, and it is what makes the pet a player picked in
Boosts or Traits last exactly one visit to the menus:

```yaml
# Runs on the natural close of this menu (ESC). Navigating to another SnPets
# menu keeps the selected pet; leaving the menus entirely forgets it.
close-actions:
  - "[pets-forget-selection]"
```

`[pets-forget-selection]` only clears the pick when no SnPets menu is open any more, so walking
Boosts to the selector and back, or Traits to the trait index and back, keeps it. Remove the key
from a file if you want a pick made there to survive closing that screen.

### Skipping the roulette

`boosts.yml` and `traits.yml` draw the cell of the pet being rolled with a `spinning` template
while the animation runs, and that template is clickable:

```yaml
  spinning:
    material: ENDER_EYE
    key: p
    display-name: "&e&lRolling..."
    lore:
      - "&7{candidate}"
      - ""
      - "&eClick to skip the animation"
    glow: true
    click-actions:
      - "[pets-skip-spin]"
```

`[pets-skip-spin]` reveals the result immediately: the same message, the same reveal sound and
the same redraw the animation would have produced when it ended on its own. It can never change
what came out, because the roll is decided and saved before the first frame is ever drawn. The
spinner is shared with pet box openings, so a skip also finishes a box reveal of the same player.

Delete the `click-actions` block if you want the animation to always run to the end.

{% hint style="info" %}
Upgrading from 1.2.1 or earlier: the `click-actions` key is a new key, so SnLib inserts it into
your `boosts.yml` and `traits.yml` on the next boot and the skip starts working. The
`&eClick to skip the animation` line is a new element of an existing `lore` list, and the
updater never appends to a list you already own, so add that line by hand if you want the hint
shown.
{% endhint %}

### Colouring a pet by its group

Every template that draws a pet binds `{group-color}`, the colour its group declares under
`groups:` in `config.yml`. Use it wherever you want the group to tint a line:

```yaml
  pet-entry:
    material: "{texture}"
    display-name: "{group-color}{pet}"
    lore:
      - "&8Group: &r{group-color}{group}"
```

It is available in `main.yml`, `selector.yml`, `fusion.yml`, `boosts.yml`, `traits.yml` and
`bulk_delete.yml`. The value is inserted before SnLib's text pipeline runs, so a legacy code, a
hex code and the `[rgb]` gradient tag all work. `[rgb]` is a PREFIX tag: it only applies when
`{group-color}` is the first thing on the line.

None of the shipped templates use it, so nothing changes appearance until you add it yourself.

Since **1.8.1** it also reaches two more places: `menus.boost-line` in the language file, so each
of a pet's three boost lines can start with its group colour, and the three stat cells and four
roll buttons of `boosts.yml`, which the file's own header had been promising since 1.3.0. On the
roll buttons it is empty while no pet is selected, so a template that uses it never shows a
literal token.

Where it does **not** work, and why:

| Place | Why |
|---|---|
| `menus.grade-row` | describes a ladder rung, not a pet: there is no group in scope |
| `menus.group-separator` | joins group names on the information clock; no single pet |
| `menus.trait-effect-exp` / `-level` / `-buff` | describe a trait in the index, not a pet |
| the `lore:` of a `pets/<id>.yml` file | that lore is itself the value of `{pet-lore}`, and a placeholder value is never re-scanned for further placeholders. No plugin placeholder resolves there, only PlaceholderAPI tokens. Put the colour on the menu line that carries `{pet-lore}` |

### Prefix tags in a display name

`[rgb]`, `[small]` and `[center]` are PREFIX tags: SnLib reads them at the start of a finished
line and nowhere else. That is why a `{group-color}` carrying `[rgb]` has to be first on its line,
and it used to bite display names too - a box in `boxes.yml` or a pet in `pets/<id>.yml` whose
`display-name` began with `[rgb]` drew its gradient on the ITEM but printed the literal tag in
chat, because a name spliced into a message as `{box}` or `{pet}` lands in the middle of the line:

```
SnPets | The [rgb]Basic Pet Box failed to open (1% chance)...
```

**1.8.1 fixes this.** Both files have their display-name prefix tags applied when the file is
read, so the name looks the same on the item and in every message that names it. Nothing in your
files changes: `boxes.yml` and `pets/` are seed-only and untouched, and the item is still built
from the raw yml, so a name is never expanded twice. `[center]` is the one tag dropped from the
chat copy - centering a fragment that lives inside somebody else's line means nothing.

{% hint style="info" %}
`groups:` is marked `# sn:extensible`, so the `color:` key added in 1.3.0 never reaches a
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
Upgrading from 1.2.0 or earlier: `group-empty` used to be a `GRAY_STAINED_GLASS_PANE`, and SnLib
never overwrites a value your file already has. Set `templates.group-empty.material` to
`"{icon}"` by hand, or delete `guis/bulk_delete.yml` and restart to have it reseeded.
{% endhint %}

## lang/

Every user facing string lives in `lang/messages_<code>.yml`. English and Spanish ship with
the plugin. Pick the active one with the `lang` key at the top of `config.yml`, which falls
back to English when the named file is missing. To add a language, copy `messages_en.yml` to
`messages_<code>.yml`, translate the values, and point `lang` at the new code.

New keys are merged into your existing file on boot, with your values and your comments left
alone, so an update never overwrites a line you restyled. **1.5.0 adds two**, both sent to the
player who RECEIVES something from an admin command:

| Key | Sent by | Placeholders |
|---|---|---|
| `messages.pet-received` | `/pets admin give` | `{amount}` `{pet}` `{level}` |
| `messages.box-received` | `/pets admin givebox`, `/pets admin giveallbox` | `{amount}` `{box}` `{chance}` |

Blank either value to switch that notification off for everyone, or suppress it per command with
the `-s` flag (see [Commands](commands.md#silent-flags)). Offline players are never messaged.
