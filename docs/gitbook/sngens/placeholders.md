# Placeholders

All placeholders require [PlaceholderAPI](https://www.spigotmc.org/resources/placeholderapi.6245/)
and use the `%sngens_` prefix. The expansion is registered by the plugin itself, so there is
nothing to download from the PlaceholderAPI ecloud.

## Player stats

| Placeholder | Description |
|-------------|-------------|
| `%sngens_currentplaced%` | Generators this player currently has placed |
| `%sngens_max%` | This player's generator slot allowance |
| `%sngens_multiplier%` | Effective sell multiplier, exact decimal |
| `%sngens_multiplier_short%` | The same value, short formatted, for example `2,5` |
| `%sngens_multiplier_raw%` | The same value truncated to a whole number |

The multiplier placeholders resolve the full chain: permission nodes, the stored player
multiplier, an active sell event and equipment bonuses, clamped by
`player-multiplier-limit`. They do not include a sellwand, since a wand only exists at the
moment of a swing.

```
&7Generators: &f%sngens_currentplaced%&7/&f%sngens_max%
&7Sell multiplier: &f%sngens_multiplier_short%x
```

## Hidden variants

| Placeholder | Description |
|-------------|-------------|
| `%sngens_currentplaced_hide%` | Placed count, masked when the player hides their stats |
| `%sngens_max_hide%` | Slot allowance, masked the same way |
| `%sngens_multiplier_hide%` | Sell multiplier, masked the same way |

A player toggles the mask with `/gens hidestats`, and the state is stored per player. While it
is on these three return `hide-stats-placeholder` from the language file, which is `&7???` out
of the box. Use them on public scoreboards and leave the plain ones for the player's own menus.

## Server wide

| Placeholder | Description |
|-------------|-------------|
| `%sngens_total_generator%` | Generators currently active across the whole server |
| `%sngens_corrupt_time%` | Countdown to the next corruption wave, as `HH:MM:SS` |
| `%sngens_top_limit_material%` | Material name of the leaderboard cutoff generator, empty when no cutoff is set |

These four do not depend on a player, so they also resolve in console, proxy and offline
contexts.

## Server events

| Placeholder | Description |
|-------------|-------------|
| `%sngens_event_name%` | The active event's display name, or the idle text |
| `%sngens_event_time%` | Time left on the active event, or time until the next one |
| `%sngens_event_isactive%` | `true` or `false` |

The idle and active wording comes from `events.yml`, under `events.placeholders`:

```yaml
placeholders:
  no-event: '&7Waiting Event...'
  no-event-timer: '&7Event in: &f{timer}'
  active-event: '{display_name}'
  active-event-timer: '&7Time left: &f{timer}'
```

So a single scoreboard line works during an event and between events:

```
%sngens_event_name%
%sngens_event_time%
```

## Leaderboard

`<n>` is the rank, starting at 1. Anything above the configured board size returns an empty
string, so a hologram with ten lines does not break on a server with three players.

| Placeholder | Description |
|-------------|-------------|
| `%sngens_top_<n>_name%` | Display name at rank `<n>` |
| `%sngens_top_<n>_name_raw%` | The bare owner nick at rank `<n>` |
| `%sngens_top_<n>_value%` | Total generator value at rank `<n>`, exact |
| `%sngens_top_<n>_value_short%` | The same value, short formatted |
| `%sngens_rank%` | The requesting player's own rank, empty when unranked |
| `%sngens_value%` | The requesting player's own total generator value |
| `%sngens_value_short%` | The same value, short formatted |

`name` and `name_raw` differ only in island mode. There `name` is the island's name and
`name_raw` is the island leader's player name. In player mode both return the player name.

A leaderboard hologram usually looks like this:

```
&6&lTOP GENERATORS
&e#1 &f%sngens_top_1_name% &7- &a%sngens_top_1_value_short%
&e#2 &f%sngens_top_2_name% &7- &a%sngens_top_2_value_short%
&e#3 &f%sngens_top_3_name% &7- &a%sngens_top_3_value_short%
&7Your rank: &f%sngens_rank%
```

{% hint style="info" %}
These read the last computed snapshot, not the live database. The board recomputes every
`gens-top.refresh-interval-seconds`, and `/gens forcetop` recomputes it immediately.
{% endhint %}

## Equipment bonuses

Read from the player's currently equipped off-hand and armor set.

| Placeholder | Description |
|-------------|-------------|
| `%sngens_eq_speed%` | Generator speed bonus, in percent |
| `%sngens_eq_tier%` | Flat tier offset applied to the drop pool |
| `%sngens_eq_sell_multi%` | Sell multiplier bonus, in percent |
| `%sngens_eq_drop_multi%` | Drop multiplier bonus, in percent |
| `%sngens_eq_lucky%` | Chance to double a drop, in percent |

All five return `0` when nothing relevant is equipped, so they are safe to put on a permanent
HUD.

```
&bSpeed &f+%sngens_eq_speed%%   &6Tier &f+%sngens_eq_tier%
&aSell &f+%sngens_eq_sell_multi%%  &dLucky &f+%sngens_eq_lucky%%
```
