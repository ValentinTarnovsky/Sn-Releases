# Configuration

SnMiniGames ships with the following YAML files. `config.yml` and `lang/messages_en.yml` are managed: new keys are auto-merged on boot and your edits and comments are preserved. `games/parkour.yml` is seeded once and never auto-merged, so the setup wand and your edits fully own it.

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
# ------------------------------------------------------------
leave-item:
  # Whether to give the leave item while in the round.
  enabled: true
  # Material of the leave item.
  material: BARRIER
  # Display name of the leave item.
  name: "&cLeave"
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

All player-facing messages live here, including the shared `snlib.*` command block, the queue, parkour, protection and setup message groups. Every line supports the SnLib text pipeline (`&` codes, `&#RRGGBB` hex, MiniMessage click and hover tags).
