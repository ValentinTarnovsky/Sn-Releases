# Configuration

SnDTC has two files you edit. `config.yml` holds the settings, and `dtcs.yml` holds the cores.

Both are re-read by `/sndtc reload`, so nothing here needs a restart.

## config.yml

### `defaults` - what a new core IS

These are stamped into a core at the moment `/sndtc create` runs. Changing them never touches a
core that already exists.

| Key | Default | Meaning |
|-----|---------|---------|
| `active-block` | `DIAMOND_BLOCK` | Block shown while the core is alive. **Must be breakable** |
| `inactive-block` | `BEDROCK` | Block shown while the core is down. `AIR` is allowed |
| `max-health` | `100` | Health pool. One completed break removes 1 point |
| `cron` | `5m` | Regeneration schedule - see [Schedules](#schedules) |
| `display-range` | `50` | `-1` every world, `0` same world, `N` a radius in blocks |

### `limits`

| Key | Default | Meaning |
|-----|---------|---------|
| `hit-cooldown-ms` | `250` | Minimum **milliseconds** between two counted hits from the same player |

The cooldown is per player and global across cores, a rejected hit does not extend it, and a
player's first hit is never blocked. `0` or less is honoured and turns the limit off entirely,
which is said in the console rather than done silently. The accepted upper bound is `60000`.

### `display`

| Key | Default | Meaning |
|-----|---------|---------|
| `update-interval` | `20` | Ticks between refreshes of the bar, action bar and hologram. Range 5 to 1200 |
| `bossbar.enabled` | `true` | Master switch |
| `bossbar.color` | `RED` | `RED`, `GREEN`, `BLUE`, `YELLOW`, `PINK`, `PURPLE`, `WHITE` |
| `bossbar.style` | `SOLID` | `SOLID`, `SEGMENTED_6`, `SEGMENTED_10`, `SEGMENTED_12`, `SEGMENTED_20` |
| `bossbar.title` | | Local tokens `{core}` `{health}` `{max_health}` `{percent}` |
| `actionbar.enabled` | `true` | Master switch |
| `actionbar.message` | | Same local tokens |

{% hint style="info" %}
The boss bar and the action bar resolve PlaceholderAPI **differently**, and it is not a bug. A
boss bar carries one title for everyone who sees it, so `%tokens%` there resolve against the
server - `%player_name%` will not work. The action bar is rendered separately for each player,
so `%tokens%` there do resolve per viewer.
{% endhint %}

### `hologram`

| Key | Default | Meaning |
|-----|---------|---------|
| `enabled` | `true` | Master switch. Turning it off stops new holograms and takes existing ones down |
| `offset-y` | `2.0` | Height above the core block. Range -64.0 to 64.0 |
| `lines` | | Default lines. Local tokens `{core}` `{health}` `{max_health}` `{percent}` `{status}` `{time}` |

`%tokens%` from other plugins work in hologram lines and resolve against the server, since one
hologram is shown to everyone. SnDTC's own `%sndtc_*%` tokens do **not** resolve there - they need
a viewer - so use the local `{...}` tokens instead.

### `rewards`

| Key | Default | Meaning |
|-----|---------|---------|
| `top-positions` | `3` | How many ranked players get position-specific commands |
| `positions` | | Console commands per finishing position. Tokens `{player}` `{damage}` `{position}` `{core}` |
| `default` | | Console commands for everyone ranked below `top-positions` |
| `max-paid` | `100` | Upper bound on how many below the top positions are paid in one destruction. `0` for no bound |

A position with no list is **skipped**: its winner gets nothing, not the default reward. Raising
`top-positions` above the number of position lists you configured therefore announces winners who
are paid nothing, and SnDTC names that in the console when it happens.

Set `default` to `[]` to pay the also-rans nothing. Deleting the key is not the way to switch the
payout off - SnLib merges missing keys back in on the next boot.

{% hint style="danger" %}
**How often a payout fires is not set in this block.** It is decided by `defaults.max-health` and
`defaults.cron` together. With the shipped values - 100 health, a `5m` schedule - one core can be
ground down by a single player in about 25 seconds and comes back 288 times a day. Multiply that
by your position-1 list before you go live.
{% endhint %}

## dtcs.yml

This file is yours. Add, edit and remove cores by hand and run `/sndtc reload` to apply.

`current-health` and `active` are runtime state that SnDTC writes back for you, at every state
change and once more at shutdown. Individual hits are **not** written as they land, so while a
fight is running the file still shows the health the core had when the fight began.

On reload the file wins for everything you edit, with one exception: an **active** core that is
part-way through a fight is refilled to full health and SnDTC says so in the console. Its damage
totals live only in memory and cannot survive the reload, so leaving it damaged would hand the
whole reward to whoever landed the next hit. To set the health of a live core, use
`/sndtc sethealth`.

### Fields

| Field | Falls back to |
|-------|---------------|
| `world` | `"world"` |
| `x`, `y`, `z` | **Required** |
| `active-block` / `inactive-block` | `defaults.*` in `config.yml` |
| `max-health` | `defaults.max-health` |
| `current-health` | `max-health` |
| `cron` | `defaults.cron` |
| `display-range` | `defaults.display-range` |
| `active` / `hologram-enabled` | `true` |

A core is skipped, **by name in the console**, when a coordinate is missing or unreadable, when a
material does not exist or names an item rather than a block, and when `max-health` is zero,
negative or not a number at all.

{% hint style="warning" %}
A core that fails to load is not protected. Its block can be mined, exploded or pushed like any
other block, and it keeps its drops. The console says so on every refusal - fix the file and
reload.
{% endhint %}

Quoting still works: `"500"` reads as 500 and `"false"` as false, with a note asking you to drop
the quotes. Only a value that is not a number at all - `5,000`, `fivethousand` - is refused.

### Per-core overrides

Each one **replaces** the `config.yml` value entirely rather than merging with it, and the three
reward settings resolve independently of each other. Nothing SnDTC writes back ever touches them.

```yaml
cores:
  SpawnCore:
    world: world
    x: 100
    y: 64
    z: -200
    active-block: DIAMOND_BLOCK
    inactive-block: BEDROCK
    max-health: 500
    current-health: 500
    cron: "4h"
    display-range: 80
    active: true
    hologram-enabled: true
    hologram-lines:
      - "&#8354f2&lSPAWN CORE"
      - "&7HP: &c{health}&7/&c{max_health}"
      - "&7{time}"
    rewards:
      top-positions: 2
      positions:
        1:
          - "give {player} netherite_ingot 1"
        2:
          - "eco give {player} 2500"
      default:
        - "eco give {player} 250"
```

## Schedules

### Simple interval

One or more `<number><unit>` parts with no spaces: `30s`, `5m`, `1h`, `1d`, `1h30m`, `1d12h`.
Units are `s`, `m`, `h`, `d`.

Intervals run on a **fixed grid**, not from the moment the core died, so `4h` means 00:00 / 04:00
/ 08:00 whenever it was destroyed. A core destroyed exactly on a boundary waits a full interval
rather than zero.

The grid lines up with midnight for any interval that divides evenly into 24 hours - `30s`, `5m`,
`1h`, `2h`, `3h`, `4h`, `6h`, `8h`, `12h`, `1d`, `2d`. One that does not, like `5h` or `2h30m`,
still fires exactly that often but walks across the day: `5h` lands at 00:00, then 01:00 the next
day, then 02:00.

### 5-field cron

`minute hour day-of-month month day-of-week`. Use this when an event has to happen at a set
wall-clock time.

```
0 */2 * * *     every two hours, on the hour
30 8 * * *      08:30 every day
0 0 * * 1       midnight every Monday
0 12,18 * * *   noon and 18:00 every day
```

Each field accepts `*`, a number, `a-b` ranges, `a,b` lists and `*/step`. Day-of-week is 0-7 with
both 0 and 7 meaning Sunday.

### No schedule

An empty `cron` means the core never comes back on its own and waits for `/sndtc start`.

## Language

Every string a player sees lives in `lang/messages_en.yml`, including the state words that the
chat lines, the hologram `{status}` slot and the `%sndtc_<core>_status%` placeholder all share:
`status.active`, `status.inactive` and `status.no-schedule`. Editing one changes it everywhere at
once.
