# Configuration

`config.yml` is a managed file: a new version's keys are merged in on boot and your values and
comments are preserved. There is no `config-version` key. Set `update-configs: false` to freeze
the file, after which SnLib only warns about missing keys instead of inserting them.

## Top level

```yaml
lang: en
update-configs: true
debug:
  enabled: false
  level: DEBUG
  categories: []
command:
  aliases: [room, snrooms]
```

`lang` picks `lang/messages_<code>.yml`. Both `en` and `es` ship. `debug` is the same output
`/rooms debug` toggles live.

## `default-room` - the template

Copied into every new room by `/rooms create`. Changing it does **not** change rooms that already
exist; use `/rooms set` for those.

```yaml
default-room:
  teams: 2
  team-size: 1
  material: GLASS
  walls: true
  ceiling: true
  floor: false
  replaceable: [AIR, CAVE_AIR, VOID_AIR, WATER, LAVA, GRASS, SHORT_GRASS, TALL_GRASS, SNOW, FIRE]
  effects: []
  start-countdown: "5s"
  close-after-round: "10s"
  interior-build: false
```

`teams x team-size` is the exact number of players that starts a round.

`replaceable` is the list of blocks the shell may overwrite. Anything else on a shell position is
left exactly as it is. Both `GRASS` (the 1.20.x name) and `SHORT_GRASS` (1.20.3+) are listed so
one file works on either version; the name your server does not know is simply ignored.

Beyond this list, the shell also takes over anything a player could **walk through**, because
leaving one of those would be a hole in the finished room. It records those too, so they are put
back exactly. The one thing it will not touch is a walk-through block carrying data a restore
cannot reproduce, a sign being the obvious case.

`effects` are `TYPE:amplifier` strings, for example `["SPEED:1", "SLOWNESS:0"]`. An effect a
player already has is left alone and never removed at the end of the round, so the plugin cannot
take away a potion someone drank before walking in.

## `teams` - clan grouping

```yaml
teams:
  use-clans: true
  require-clan-for-team-rooms: true
  allies-count-as-one: false
```

Only consulted for rooms with `team-size` above 1. A room of one player per side never asks
SnClans anything, so a 1v1 room works whether or not SnClans is installed.

With SnClans absent (or `use-clans: false`), a room of bigger sides cannot form teams and stays
open, with one warning in the console naming the room.

## `round`

```yaml
round:
  countdown-display: actionbar
  leaving-counts-as-defeat: true
  max-duration: "10m"
```

`countdown-display` is `actionbar`, `bossbar`, `title` or `none`.

`leaving-counts-as-defeat` decides what walking out of a running round means. Left on, it is an
elimination - which is what stops a losing player from stepping out through a gap to deny the
win. Turned off, the player keeps their place and may come back; their round effects are taken
off on the way out and given back on the way in.

`max-duration` is the longest a round may run before it is ended as a draw and the room reopens.
**It is a safety net, not a game rule.** A round normally ends when one side is left standing, so
a side whose last member is alive but out of reach would otherwise keep the room sealed with
everyone else shut inside it. That is reachable with `leaving-counts-as-defeat: false`, since a
player can then walk out and simply stay out. `0s` removes the net; only do that if
`leaving-counts-as-defeat` stays on.

## `anti-glitch`

```yaml
anti-glitch:
  tracking-radius: 32
  message-cooldown: "3s"
```

`tracking-radius` is how far around a room a player's position outside it is remembered. That
remembered spot is where an intruder is sent back to, and it is preferred over everything else
because it is a place they reached by walking. A smaller radius is cheaper but makes the plugin
fall back to the room's exit, or to the nearest block outside the region, more often.

`message-cooldown` rate limits the refusal message and its sound, so a player pressed against a
sealed wall is not spammed on every step. `0s` disables the limit.

## `shell` - limits and smoothing

```yaml
shell:
  max-blocks: 60000
  batch-threshold: 2000
  batch-size: 500
```

`max-blocks` is the largest shell a room may build. A region needing more **refuses to seal and
stays open** rather than trying. For scale, a 20x20x20 room with the default faces is about 1844
blocks, so the ceiling is generous; it exists so a mistyped selection cannot ask the server to
place a million blocks.

`batch-threshold` is the size above which placement and removal are spread across ticks instead
of done in one pass, and `batch-size` is how many blocks per tick that spreading uses. A
20x20x20 room (1844 blocks) seals in a single pass; 21x21x21 (2041) starts being spread.

Raising `batch-threshold` makes more rooms seal in one pass, which is smoother for small rooms
and worse for large ones. It is capped internally so it cannot be used to switch the smoothing
off entirely.

## `feedback`

```yaml
feedback:
  sounds:
    round-start: "BLOCK_NOTE_BLOCK_PLING 1.0 1.5"
    round-end: "ENTITY_PLAYER_LEVELUP 1.0 1.0"
    seal: "BLOCK_GLASS_PLACE 1.0 1.0"
    evict: "ENTITY_ENDERMAN_TELEPORT 1.0 1.0"
  broadcast:
    round-start: false
    round-end: false
```

Sounds are `SOUND_ID [volume] [pitch]`; an empty value plays nothing. `round-start` and
`round-end` are played to each participant (the shell is closed, so a sound played into the
world would be heard by the wrong people). `seal` is played at the middle of the room, because a
shell going up is an event everyone nearby should hear.

The broadcasts are server-wide announcements and are **off by default**. The players inside a
room are always told what happens to them regardless.

## Selection wand

`selection-wand` controls the appearance of the wand and of the particle box drawn while
selecting: the item itself (material, name, lore, glow), the particle `type`, `color` and `size`,
the `step` between points, the refresh `interval-ticks`, the `render-distance`, whether the box
is shown to just the selecting admin or everyone nearby (`visibility`), and the safety limits
`particle-budget`, `max-render-volume`, `max-volume` and `timeout-ticks`.

The defaults are tuned to stay cheap on a busy server: a bigger box renders sparser rather than
heavier, and above `max-render-volume` only the eight corners are marked.
