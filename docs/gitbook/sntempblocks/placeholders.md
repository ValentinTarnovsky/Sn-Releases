# Placeholders

All placeholders require [PlaceholderAPI](https://www.spigotmc.org/resources/placeholderapi.6245/) and use the `%sntempblocks_` prefix.

There are two families. The plain ones answer about the zone the viewer is standing in right now,
which is what a scoreboard or an action bar wants. The ones ending in a zone id name a zone
explicitly, which is what a hologram at the arena entrance wants.

## The viewer's own zone

| Placeholder | Description |
|-------------|-------------|
| `%sntempblocks_zone%` | Id of the zone the viewer is standing in |
| `%sntempblocks_mode%` | Its engine, as a word: `Per block` or `Interval` |
| `%sntempblocks_tracked%` | Blocks currently tracked in that zone |
| `%sntempblocks_next_wipe%` | Time left until that zone's next wipe, formatted |
| `%sntempblocks_next_wipe_seconds%` | The same, as a bare number of seconds |

## A named zone

| Placeholder | Description |
|-------------|-------------|
| `%sntempblocks_tracked_<zone>%` | Blocks currently tracked in that zone |
| `%sntempblocks_next_wipe_<zone>%` | Time left until that zone's next wipe, formatted |
| `%sntempblocks_next_wipe_seconds_<zone>%` | The same, as a bare number of seconds |

Replace `<zone>` with the zone id, which is the key you wrote under `zones:` in `zones.yml`. For
a zone called `pvp-arena` that is `%sntempblocks_next_wipe_pvp-arena%`.

## What they return when there is no answer

Nothing here invents a number.

| Situation | Output |
|-----------|--------|
| The viewer is standing in no zone | `None` |
| The zone id names no loaded zone | `Unknown` |
| The zone runs on `PER_BLOCK` | `n/a` for both next-wipe placeholders |

A `PER_BLOCK` zone has no single next-wipe instant, because every block carries its own deadline.
Returning `n/a` is why a countdown never ticks down to a wipe that will not happen.

{% hint style="warning" %}
`next_wipe_seconds` returns `n/a` rather than a number on a `PER_BLOCK` zone or an unknown zone.
If another plugin does arithmetic on that value, make sure it tolerates a non-numeric result.
{% endhint %}

{% hint style="info" %}
All three words are language values under `status:` in your messages file, so you can restyle or
translate them once and every command and placeholder follows.
{% endhint %}
