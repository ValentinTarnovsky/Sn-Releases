# Placeholders

All placeholders require [PlaceholderAPI](https://www.spigotmc.org/resources/placeholderapi.6245/)
and use the `%sncrates_` prefix. The expansion is registered by the plugin itself on boot:

```
[SnCrates] Registered PlaceholderAPI expansion 'sncrates'.
```

Without PlaceholderAPI installed the plugin works normally; the placeholders simply do not resolve.

| Placeholder | Description |
|---|---|
| `%sncrates_total_opens%` | How many crates the player has opened in total, every crate combined |
| `%sncrates_keys_<crateId>%` | The player's **virtual** key balance for that crate |
| `%sncrates_opens_<crateId>%` | How many times the player has opened that crate |
| `%sncrates_chance_<crateId>_<rewardId>%` | The computed chance of that reward, as shown in the preview |

`<crateId>` and `<rewardId>` are the raw ids - the top-level key of a crate file and the key under
its `rewards:` block - not display names.

```
%sncrates_keys_example%
%sncrates_opens_vipcrate%
%sncrates_chance_example_diamonds%
```

## What they report, exactly

**`keys_<crateId>` is the virtual balance only.** Physical keys sitting in an inventory are items,
not a balance, and are not counted. On a crate that accepts only `PHYSICAL` the value is always `0`,
which is correct: that crate has no balance.

**`chance_<crateId>_<rewardId>` is the same number the preview shows.** It is computed from the
weights of the rewards that can currently be won, so a reward with `enabled: false` is excluded from
the pool and does not dilute the others. A player's own filter does not change it - a deactivated
reward is still rolled and still burns its limits, it is simply never handed to that player.

**An unknown crate or reward id leaves the raw `%sncrates_...%` text on screen.** That is
deliberate: it is how you find out you typed an id that does not exist. Resolving a typo to `0`
would hide it behind a plausible number for ever.

**A player whose data is still loading resolves to `0`.** The slice loads asynchronously on join, so
a scoreboard that renders in the same tick as the join can briefly show zeros. It fills in a moment
later. A **known** crate always renders a number for exactly this reason - a scoreboard line never
flickers between a value and raw placeholder text.

So the two fallbacks say different things: raw text means "that id does not exist", `0` means "that
id exists and the answer is zero, or is not loaded yet".

## Ids containing underscores

`%sncrates_chance_<crateId>_<rewardId>%` has an ambiguity that the plugin resolves by asking the
crate catalogue: both ids may contain underscores, so `chance_epic_crate_vip_rank` could be crate
`epic` / reward `crate_vip_rank` or crate `epic_crate` / reward `vip_rank`.

The boundaries are scanned left to right and the first split where **both** the crate and the reward
resolve against your loaded crates wins. If you happen to have both interpretations configured, the
leftmost one is what renders. Avoiding underscores in crate ids sidesteps the question entirely.

## Server-context parses leave the raw text

A placeholder parsed with **no player** - `PlaceholderAPI.setPlaceholders(null, text)`, which is
what a few plugins use for server-wide text - leaves `%sncrates_...%` in place rather than resolving
to `0`.

Every scoreboard, tab, hologram and chat plugin passes a player, so this only shows up in that one
narrow case. If you hit it, give the parse a player.

## Placeholders inside SnCrates itself

Two other places accept placeholders, and they behave differently from each other:

| Where | Resolved | Against |
|---|---|---|
| Menu names and lore under `guis/` | Every time the item is built | The **viewer** |
| Crate and reward item text in `crates/*.yml` | **Once**, when crates load | Nobody in particular |
| Reward win commands | At win time, after `{player}`/`{amount}`/`{crate}` | The **winner** |

So `%player_name%` in a menu button's lore renders the name of whoever is looking, and the same
placeholder written into a reward's `display-name` in a crate file does not - it is baked in at load
time, when there is no player. Use `%server_name%`-style placeholders there, and put per-player text
in the menu layout instead.

Win commands are the useful case:

```yaml
commands:
  - "eco give {player} %vault_eco_balance%"
  - "/nivel give {player} %math_5*{edtools_leveling_level_level}/100%"
```

`{player}`, `{amount}` and `{crate}` are substituted first, then PlaceholderAPI resolves the line
against the winner, then it is dispatched to the console. Nested placeholders work because the
internal substitution runs first.

{% hint style="warning" %}
Win commands are never colour-translated. A `&a` or a hex code in a command corrupts the command
text.
{% endhint %}
