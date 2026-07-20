# Configuration

SnMiniGames ships with the following YAML files. `config.yml` and `lang/messages_en.yml` are managed: new keys are auto-merged on boot and your edits and comments are preserved. `games/parkour.yml` is seeded once and never auto-merged, so the setup wand and your edits fully own it.

Every minigame gets its own `games/<game>.yml` holding its `enabled` flag, queue settings, maps and rewards. Settings shared by every game, such as the leave item appearance, live in `config.yml` instead.

## config.yml

```yaml
# ============================================================
#  SnMiniGames - configuration
#  Managed by SnLib: new keys are auto-merged on boot; your values and
#  comments are preserved. Do NOT add a config-version key (retired).
#  Set update-configs: false to freeze this file (SnLib only warns about
#  missing keys instead of inserting them).
# ============================================================

# Active language code; loads lang/messages_<code>.yml (falls back to en).
lang: en

# Master switch of the SnLib auto-updater for this plugin's managed files.
update-configs: true

# Runtime debug output (also toggleable live via /minigames debug).
debug:
  enabled: false

# ------------------------------------------------------------
#  Main command.
# ------------------------------------------------------------
command:
  # Aliases of /minigames. Re-read on /minigames reload.
  aliases: [mg]

# ------------------------------------------------------------
#  Player state snapshots (crash-safe restore).
# ------------------------------------------------------------
snapshots:
  # Folder inside the plugin data folder where durable per-player state
  # snapshots are written before any in-game mutation.
  folder: snapshots

# ------------------------------------------------------------
#  Admin map setup wand.
# ------------------------------------------------------------
setup-wand:
  # Material of the setup wand item.
  material: BLAZE_ROD
  # Display name of the setup wand item.
  name: "&#8354f2&lSetup Wand"

# ------------------------------------------------------------
#  Leave item (hotbar item that leaves the round on right-click).
#  Material and name are framework-level: a single item is registered for
#  every minigame. Whether each game hands it out and in which hotbar slot
#  is per-game, under leave-item in that game's file (games/<game>.yml).
# ------------------------------------------------------------
leave-item:
  # Material of the leave item.
  material: BARRIER
  # Display name of the leave item.
  name: "&cLeave"

# ------------------------------------------------------------
#  Queue / round start.
# ------------------------------------------------------------
queue:
  # 3-2-1 countdown sound spec: "SOUND_ID [volume] [pitch]" (blank = none).
  countdown-sound: "BLOCK_NOTE_BLOCK_PLING 1.0 1.0"

# ------------------------------------------------------------
#  In-game restrictions.
# ------------------------------------------------------------
in-game:
  # Commands an in-game player may still run (lowercase, without the slash).
  # The leave command is ALWAYS allowed regardless of this list.
  allowed-commands: []
```

## games/parkour.yml

```yaml
# ============================================================
#  SnMiniGames - Parkour game
#  Seeded once by SnLib and NEVER auto-merged: your structure and values
#  are kept. The setup wand (/minigames admin parkour ...) writes maps
#  back into this file.
# ============================================================

# Whether the Parkour game is active. When false it is not registered:
# it will not appear in /minigames list, /minigames join or PlaceholderAPI.
enabled: true

# Display name of the game, shown wherever {game} appears. Supports color
# codes and the full SnLib text pipeline.
display-name: "&#8354f2&lParkour"

# ------------------------------------------------------------
#  Queue and round staging (shared framework settings).
# ------------------------------------------------------------
queue:
  # Seconds between automatic rounds. 0 disables auto-start (rounds then open
  # only via /minigames admin start parkour).
  auto-start-interval: 120
  # Seconds the queue stays open before the round begins.
  countdown: 30
  # Minimum queued players for the round to begin; below this at countdown end
  # the round is cancelled.
  min-players: 2
  # Maximum queued players; reaching it starts the round early.
  max-players: 12
  # Broadcast the "starts in" line every N seconds while the queue counts down.
  announce-interval: 10
  # Mini-lobby players are teleported to while they wait (world;x;y;z;yaw;pitch).
  waiting-spawn: "world;0.5;65.0;0.5;0.0;0.0"
  # Seconds a player must wait before rejoining after leaving or losing. 0 = none.
  rejoin-cooldown: 15

# ------------------------------------------------------------
#  Join gate.
# ------------------------------------------------------------
join:
  # Require an empty inventory, armor and hands to join (prevents item smuggling).
  require-empty-inventory: true

# ------------------------------------------------------------
#  Start countdown (the 3-2-1 GO freeze before the race).
# ------------------------------------------------------------
start-countdown:
  # Seconds of the pre-race freeze. 0 skips the freeze.
  seconds: 3

# ------------------------------------------------------------
#  Leave item (hotbar item that leaves the round on right-click).
#  Material and name are framework-level and live in config.yml under
#  leave-item; here you only choose whether this game hands it out and where.
# ------------------------------------------------------------
leave-item:
  # Whether to give the leave item while in the round.
  enabled: true
  # Hotbar slot (0-8) the leave item occupies.
  slot: 8

# ------------------------------------------------------------
#  Map rotation across rounds: RANDOM (a random map each round) or
#  SEQUENTIAL (maps picked in file order).
# ------------------------------------------------------------
map-rotation: RANDOM

# ------------------------------------------------------------
#  Maps. Each entry is one parkour course, created in-game with
#  /minigames admin parkour create <name>.
#  Wand commands (select a block with the setup wand first):
#    /minigames admin parkour setstart|addwaypoint|setwin <map>
#  Standing-position commands (run where you stand):
#    /minigames admin parkour setwaiting|setend|sethold <map>
#  Also: delete <map>, setwinners <map> <count>, clearwaypoints <map>, list.
#    Full command list: /minigames admin parkour help
# ------------------------------------------------------------
maps:
  arena1:
    # Where players spawn to begin the race (world;x;y;z;yaw;pitch).
    start-spawn: "world;0.5;70.0;0.5;0.0;0.0"
    # Ordered checkpoints. Stepping on one (its block or the block underfoot)
    # sets it as the respawn point; only forward checkpoints count.
    waypoints:
      - "world;10.5;72.0;0.5;0.0;0.0"
      - "world;20.5;75.0;3.5;0.0;0.0"
    # Stepping on this block (or the block underfoot) finishes the race.
    win-block: "world;30.5;80.0;5.5;0.0;0.0"
    # Falling this many blocks below the last checkpoint's height sends the
    # player back to that checkpoint.
    fall-blocks: 5
    # How many players must finish for the round to end. 1 ends at the first
    # finisher; higher freezes each finisher at the hold spot until N finish
    # (or the time limit).
    winners: 1
    # Maximum round length in seconds; 0 disables the limit.
    time-limit: 300
    # Everyone is teleported here when the round ends. Leave blank to send each
    # player back to where they were before joining.
    end-teleport: "world;0.5;64.0;0.5;0.0;0.0"
    # (winners > 1) where each finisher is frozen while waiting for the rest.
    finish-hold: "world;30.5;81.0;5.5;0.0;0.0"
    # How far (blocks) a frozen finisher may drift from finish-hold before being
    # pinned back. Minimum 0.5.
    finish-hold-radius: 3.0
    # Rewards by finish position (1 = first). Each position is a list of action
    # strings run for that finisher, with placeholders {player}, {position},
    # {game} and {map}. Rewards do NOT fire for a player who left before the
    # round ended (offline finishers are skipped).
    rewards:
      1:
        - "[console] eco give {player} 1000"
        - "[broadcastmessage] &a{player}&a finished 1st in Parkour!"
      2:
        - "[console] eco give {player} 500"
      3:
        - "[console] eco give {player} 250"

# ------------------------------------------------------------
#  Sounds for parkour events. Format: "SOUND_ID [volume] [pitch]"
#  (blank or "none" = silent). The 3-2-1 start countdown sound is
#  configured globally in config.yml (queue.countdown-sound).
# ------------------------------------------------------------
sounds:
  # Played when a player joins the parkour queue.
  join: "ENTITY_EXPERIENCE_ORB_PICKUP 1.0 1.0"
  # Played when a player reaches a checkpoint.
  checkpoint: "ENTITY_PLAYER_LEVELUP 1.0 1.2"
  # Played when a player falls and is returned to their checkpoint.
  fell: "ENTITY_VILLAGER_NO 1.0 1.0"
  # Played to a player when they finish the course.
  win: "UI_TOAST_CHALLENGE_COMPLETE 1.0 1.0"
```

## lang/messages_en.yml

```yaml
# ============================================================
#  SnMiniGames - language file (English)
#  Managed by SnLib: new keys merge in on boot, your values are kept.
#  Your edits survive updates.
# ============================================================

# Prefix prepended to every single-line message sent via sn.lang().send(...)
prefix: "&#8354f2&lSnMiniGames &8| &7"

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
  player-not-found: "&cPlayer not found: &e{value}"
  unknown-subcommand: "&cUnknown subcommand: &e{value}"
  reload-done: "&aConfiguration reloaded."
  help:
    header: "&8&m----------&r &#8354f2&lSnMiniGames &8&m----------"
    entry: "&#8354f2{usage} &7{description}"
    footer: "&7Page &f{page}&7/&f{total} &8- &#8354f2/{command} help <page>"

# Plugin messages. Do NOT write {prefix} in any value: SnLib auto-prepends
# the prefix above to every single-line message sent via sn.lang().send(...).
messages:
  players-only: "&cThis command can only be used by players."

  # Command-tree feedback (the /mg command). Placeholders: {game} display name,
  # {id} game id, {map} map id, {player} target name, {state} round-state word,
  # {count}/{max} player counts, {reason} force-join failure token. The list.*
  # lines are rendered with getList and sent WITHOUT the prefix.
  commands:
    # Sent when a resolved game id no longer maps to a registered game (defensive).
    game-not-found: "&cThere is no game called &e{game}&c."
    # Sent when /mg leave is used by a player who is not in a game.
    not-in-game: "&cYou are not in a minigame."
    list:
      # Header printed before the per-game list entries.
      header: "&8&m---------&r &#8354f2&lMinigames &8&m---------"
      # One line per registered game.
      entry: "&8- &#8354f2{game} &8(&7{state}&8) &7{count}&8/&7{max}"
    admin:
      start:
        # Sent when a round was force-opened.
        opened: "&aOpened a {game}&a round."
        # Sent when a round for that game is already live.
        already-running: "&cA {game}&c round is already running."
        # Sent when the round could not be opened (waiting spawn unset or unavailable).
        failed: "&cCould not open a {game}&c round; check its waiting spawn."
        # Sent when the optional [map] token names no map of that game.
        map-not-found: "&cThere is no map called &e{map}&c for {game}&c."
      forcestart:
        # Sent when the waiting round was force-started, skipping the countdown.
        started: "&aForce-started the {game}&a round, skipping the countdown."
        # Sent when there is no open round to force-start; points to /minigames admin start.
        no-round: "&cThere is no {game}&c round open. Open one first with &f/minigames admin start {id}&c."
        # Sent when the round has already left the queue (past WAITING).
        already-started: "&cThe {game}&c round has already started."
        # Sent when the waiting queue is empty, so forcing would start an empty round.
        empty: "&cNobody is queued for the {game}&c round yet."
      stop:
        # Sent when the live round was stopped and every player restored.
        stopped: "&aStopped the {game}&a round and restored every player."
        # Sent when there is no live round to stop.
        no-round: "&cThere is no {game}&c round running."
      forcejoin:
        # Sent when the player was forced into the round (the target got the queue message).
        joined: "&aForced &f{player}&a into the {game}&a round."
        # Sent when the force-join failed; {reason} is the failure token.
        failed: "&cCould not force &f{player}&c into {game}&c (&e{reason}&c); they were told why."
        # Sent when no game was given and the open round is ambiguous or absent.
        specify-game: "&cSpecify a game: there is not exactly one open round."
      # Scoped per-game setup help, shown by /minigames admin <game> help. Each game
      # hides its own setup subcommands from the root help and exposes only that one
      # command, so /minigames help stays readable as more minigames ship.
      # Placeholders: {game} display name, {id} game id, {page}, {total}, {usage},
      # {description}.
      game-help:
        # Header printed before the entries.
        header: "&8&m---------&r &#8354f2&l{game} &7setup &8(&7{page}&8/&7{total}&8)"
        # One line per setup subcommand.
        entry: "&e{usage} &7{description}"
        # Footer, sent only when there is more than one page.
        footer: "&7Page &f{page}&7/&f{total} &8- &7/minigames admin {id} help <page>"
        # Sent when the game declares no setup subcommands at all.
        empty: "&7{game}&7 has no setup commands."

  # Queue and round-lifecycle messages, namespaced under queue.*.
  # Placeholders: {game} display name, {id} game id, {time} seconds,
  # {count} queued players, {max} max players, {n} countdown number.
  queue:
    # Broadcast when a round opens. The clickable [JOIN] runs the ROOT command
    # (/minigames, never a config-removable alias). Keep the <click>/<hover> tags
    # when editing, and keep the plain typed command at the end: it is the only
    # way Bedrock (Geyser) players can join - their chat cannot click.
    announce: "&7A {game}&7 round is starting! <click:run_command:'/minigames join {id}'><hover:show_text:'&aClick to join'>&a&l[JOIN]</hover></click> &8- &7or type &f/minigames join {id}"
    # Broadcast instead of announce when the game id is not click-safe (no button).
    announce-plain: "&7A {game}&7 round is starting! Type &f/minigames join {id}&7 to play."
    # Broadcast every announce-interval seconds while the queue counts down.
    starts-in: "&7{game}&7 starts in &e{time}s &8({count}/{max})"
    # Broadcast when a player joins the queue.
    player-joined: "&f{player}&7 joined the queue &8({count}/{max})"
    # Broadcast when a player leaves the queue or the round.
    player-left: "&f{player}&7 left the queue &8({count}/{max})"
    # Broadcast when the round leaves the queue and begins.
    round-starting: "&a{game}&7 is starting!"
    # Broadcast when the round is cancelled for too few players.
    round-cancelled: "&cNot enough players - the {game}&c round was cancelled."
    # Sent when the queue is already at max players.
    full: "&cThat round is already full."
    # Sent when there is no open round for the game.
    no-round: "&cThere is no {game}&c round open right now."
    # Sent when the round has already left the queue.
    queue-closed: "&cThe {game}&c round has already started."
    # Sent when the player is already in a game.
    already-in-game: "&cYou are already in a game."
    # Sent when the player is still on rejoin cooldown.
    on-cooldown: "&cYou must wait &e{time}s &cbefore rejoining."
    # Sent when the player's inventory, armor or hands are not empty.
    inventory-not-empty: "&cEmpty your inventory, armor and hands before joining."
    # Sent when the player is inside a vehicle.
    in-vehicle: "&cLeave your vehicle before joining."
    # Sent when the player tries to join while dead.
    not-allowed-dead: "&cYou cannot join while dead."
    # Sent when the round could not be joined (spawn unset or snapshot failed).
    join-error: "&cCould not join the round right now. Please try again."
    # 3-2-1 countdown title shown each second (title;subtitle;fadeIn;stay;fadeOut in ticks).
    countdown-title: "&#8354f2&l{n};&7Get ready!;0;20;0"
    # GO title shown when the freeze ends (title;subtitle;fadeIn;stay;fadeOut in ticks).
    countdown-go: "&a&lGO!;;0;20;10"
  # In-game protection feedback (players in a round).
  protection:
    # Sent when an in-game player runs a command that is not whitelisted.
    command-blocked: "&cYou cannot use that command while in a minigame."

  # Parkour game feedback (players in a parkour round).
  # Placeholders: {checkpoint} 1-based checkpoint number, {total} checkpoint
  # count, {position} finish position, {player} finisher name, {game} display name.
  parkour:
    # Sent when a player reaches a checkpoint.
    checkpoint: "&aCheckpoint &f{checkpoint}&7/&f{total}&a reached!"
    # Title on reaching a checkpoint (title;subtitle;fadeIn;stay;fadeOut in ticks).
    checkpoint-title: "&#8354f2&lCHECKPOINT;&7{checkpoint}/{total};0;20;10"
    # Sent when a player falls and is returned to their last checkpoint.
    fell: "&cYou fell - back to your last checkpoint."
    # Title on falling (title;subtitle;fadeIn;stay;fadeOut in ticks).
    fell-title: "&c&lYOU FELL;&7Back to your checkpoint;0;20;10"
    # Sent to a player when they finish the course.
    won: "&6You finished the parkour!"
    # Title on finishing (title;subtitle;fadeIn;stay;fadeOut in ticks).
    win-title: "&6&lFINISHED;&ePosition {position};0;40;10"
    # Broadcast when a player finishes the course.
    winner-broadcast: "&f{player}&7 finished &6{game}&7 in position &f{position}&7!"
    # Sent to a finisher who must wait at the hold spot for the other winners.
    finish-hold-wait: "&aYou finished! Waiting for the other players to arrive..."
    # Broadcast when the round ends because it hit its time limit.
    time-limit: "&c{game}&c round ended - the time limit was reached."

  # Map setup feedback (the /mg admin wand + /mg admin parkour ... commands).
  # Placeholders: {map} map id, {x}/{y}/{z}/{world} selected block, {count} map
  # count, {waypoints} checkpoint count, {winners} winner count, {index} the new
  # checkpoint number.
  setup:
    # Sent when the setup wand is given (the wand command toggles: given/removed).
    wand-given: "&aReceived the setup wand. Left- or right-click a block to select it; run the command again to remove it."
    # Sent when the wand command removes the wand the admin already had.
    wand-removed: "&aSetup wand removed from your inventory."
    # Sent when a block is selected with the wand.
    block-selected: "&7Selected block &f{x}&7, &f{y}&7, &f{z}&7 in &f{world}&7."
    # Sent when a wand-consuming command runs with no block selected yet.
    no-selection: "&cSelect a block with the setup wand first."
    # Sent when a create map id is not a valid token (letters, digits, - and _ only).
    invalid-map-name: "&cInvalid map name &e{map}&c; use letters, digits, - or _ (max 32)."
    # Sent when a map is created; names the follow-up commands (wand vs standing).
    map-created: "&aCreated the map &f{map}&a. Wand: &fsetstart&a, &faddwaypoint&a, &fsetwin&a; where you stand: &fsetwaiting&a, &fsetend&a, &fsethold&a - all under &f/minigames admin parkour <sub> {map}&a. Full list: &f/minigames admin parkour help&a."
    # Sent when create is used with an id that already exists.
    map-exists: "&cA map called &e{map}&c already exists."
    # Sent when a map is deleted.
    map-deleted: "&aDeleted the map &f{map}&a."
    # Sent when a command targets a map id that does not exist.
    map-not-found: "&cThere is no map called &e{map}&c."
    # Sent when trying to delete a map that a live round is currently playing.
    cannot-delete-active: "&cThe map &e{map}&c is in a running round; stop it first."
    # Sent by list when no maps exist yet.
    list-empty: "&7No parkour maps yet. Create one with &f/minigames admin parkour create <name>&7."
    # List header, printed before the per-map entries.
    list-header: "&8&m---------&r &#8354f2&lParkour maps &8(&7{count}&8)"
    # One line per map.
    list-entry: "&8- &#8354f2{map} &8- &7{waypoints}&8 checkpoints, &7{winners}&8 winners"
    # Sent when the start spawn is set.
    start-set: "&aStart of &f{map}&a set to your wand selection."
    # Sent when a checkpoint is added; {index} is the new checkpoint number.
    waypoint-added: "&aCheckpoint &f{index}&a added to &f{map}&a."
    # Sent when every checkpoint of a map is removed.
    waypoints-cleared: "&aRemoved every checkpoint of &f{map}&a."
    # Sent when the win block is set.
    win-set: "&aWin block of &f{map}&a set to your wand selection."
    # Sent when the waiting spawn is set (shared by every parkour map).
    waiting-set: "&aWaiting spawn set to your location."
    # Sent when the end teleport is set.
    end-set: "&aEnd teleport of &f{map}&a set to your location."
    # Sent when the finish-hold spot is set.
    hold-set: "&aFinish-hold spot of &f{map}&a set to your location."
    # Sent when the winner count is set.
    winners-set: "&aWinners of &f{map}&a set to &f{winners}&a."
```
