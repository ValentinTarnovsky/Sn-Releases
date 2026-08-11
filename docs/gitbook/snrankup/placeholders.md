# Placeholders

All placeholders below require [PlaceholderAPI](https://www.spigotmc.org/resources/placeholderapi.6245/)
and use the `%snrankup_` prefix. SnRankUp detects PlaceholderAPI at startup and registers the
expansion only when it is present. Without it the plugin is fully functional.

| Placeholder | Description |
|-------------|-------------|
| `%snrankup_prefix%` | The display prefix of the rank the player holds |
| `%snrankup_num%` | The number of that rank |
| `%snrankup_next_prefix%` | The display prefix of the rank that comes next |
| `%snrankup_next_num%` | The number of the next rank, or the max rank word when there is none |
| `%snrankup_top_name_<N>%` | The name at position N of the leaderboard |
| `%snrankup_top_value_<N>%` | The rank display at position N of the leaderboard |

```
%snrankup_prefix%
%snrankup_next_num%
%snrankup_top_name_1%
%snrankup_top_value_1%
```

A rank's number is its key read as an integer when the key is one, so a rank keyed `'3'` is number
3. When the key is a word, the plugin falls back to `order` minus one.

{% hint style="info" %}
`%snrankup_top_name_<N>%` and `%snrankup_top_value_<N>%` answer even when nothing asks on behalf of
a player. Holograms, signs and Discord bridges get a real value, and `/papi parse --null` does too.
A position the board cannot fill renders empty.
{% endhint %}

{% hint style="warning" %}
At the highest rank there is no next rank, so `%snrankup_next_prefix%` renders empty while
`%snrankup_next_num%` renders the max rank word. If your scoreboard or menu writes `Next:` in front
of the prefix, guard that line or use the number variant.
{% endhint %}

## Inside SnRankUp itself

Rank lore in `rankup.yml` carries tokens that need no PlaceholderAPI at all:

| Token | Description |
|-------|-------------|
| `{req_hours}` | The playtime this rank costs |
| `{missing_hours}` | The playtime the viewer still lacks, 0 once it is met |
| `{current_hours}` | The viewer's playtime |
| `{req_<currency>}` | The amount of that currency this rank costs |
| `{missing_<currency>}` | The amount of it the viewer still lacks, 0 once it is met |

`<currency>` is the id you declared in `config.yml`, always lowercased. A rank priced with
`Tokens: 500` produces `{req_tokens}`, not `{req_Tokens}`.

{% hint style="warning" %}
The `hours` pair exists on every rank. A currency pair exists only on a rank whose `requirements`
name that currency, so `{req_vault}` belongs on that rank's own lore and never on a template shared
by the whole ladder. A token with nothing behind it renders as literal text.
{% endhint %}

The rank display prefixes and the tokens above both support `&` colors, `&#RRGGBB` hex and
MiniMessage, and any PlaceholderAPI placeholder resolves inside them as well.
