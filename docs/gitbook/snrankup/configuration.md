# Configuration

SnRankUp ships four files, all of them managed:

| File | What it is |
|---|---|
| `config.yml` | Language, aliases, database, menu mode, leaderboard, currencies |
| `rankup.yml` | The ladder: one entry per rank |
| `lang/messages_en.yml` | Every line the plugin sends to a player or an admin |
| `guis/*.yml` | The layout of both menus, one file each |

Managed means new keys are merged into your file on boot. Your values, your comments and your own
additions are preserved, and example entries you delete stay deleted. Sections marked
`# sn:extensible` are the ones whose entries are yours: `currencies:` in `config.yml`, `rankups:` in
`rankup.yml`, and `items:` in both menus.

{% hint style="info" %}
There is no `config-version` key anywhere, and there is never anything to migrate by hand. Set
`update-configs: false` to freeze every managed file. SnLib then only warns about missing keys.
{% endhint %}

## config.yml

### Language, updates and debug

```yaml
# Active language code; loads lang/messages_<code>.yml (falls back to en).
lang: en

# Master switch of the SnLib auto-updater for this plugin's managed files.
update-configs: true

# Runtime debug output.
debug:
  # Master toggle of the debug output.
  enabled: false
  # Verbosity threshold: OFF, INFO, DEBUG or TRACE.
  level: DEBUG
  # Category filter; an empty list lets every category through.
  categories: []
```

`/rankup debug` toggles the same output without editing the file, which is what you want while
tracing a live rankup.

### Command aliases

```yaml
command:
  # Aliases of /rankup. Re-read on /rankup reload.
  aliases: [ru, rank]
```

Add an alias here and reload to have it answer. `ru` and `rank` are always registered.

### Database

```yaml
database:
  # sqlite or mysql
  type: sqlite
  # MySQL connection (ignored when type is sqlite).
  host: localhost
  port: 3306
  database: snrankup
  username: root
  password: ""
```

SQLite needs nothing: the file is created next to the config. Switch `type` to `mysql` and fill the
block to share ranks across a network. Every read and write happens off the server thread either
way.

### Menu mode

```yaml
menu:
  # single    -> guis/rankup-menu.yml: one dynamic button for the next rank.
  # paginated -> guis/rankup-list.yml: the whole ladder, one slot per rank,
  #              paginated, each tile showing claimed / ready / next / locked.
  mode: single
```

This is the switch that decides which of the two menus `/rankup` opens. Both files ship, so you can
change your mind with a reload.

### Currencies

```yaml
# sn:extensible
currencies:
  vault:
    type: vault
  # tokens:
  #   type: placeholder
  #   check: '%mycurrency_balance%'
  #   consume: 'mycurrency take %player% {amount}'
  #   deposit: 'mycurrency give %player% {amount}'
```

Two types exist:

| Type | What it reads | How it charges |
|---|---|---|
| `vault` | The Vault economy balance | Through Vault, so any economy plugin |
| `placeholder` | The `check` placeholder | Runs the `consume` console command |

`{amount}` and `%player%` are substituted in `consume` and `deposit`. `deposit` is what makes a
refund possible, so declare it whenever the currency can be given back.

{% hint style="warning" %}
A `placeholder` currency with no `consume` command cannot charge anything, so a rank priced in it
fails instead of being free.
{% endhint %}

{% hint style="warning" %}
A currency whose backing plugin is missing is skipped at load with a warning, and every requirement
naming it is dropped. That makes the rank cheaper, not blocked. The warning in the server log is
your only signal, so read the log after removing an economy plugin.
{% endhint %}

The shipped file declares `vault`, and the shipped ladder prices one rank in it. On a server with no
economy plugin you get that warning on every boot. Delete the `vault` entry if you do not want it:
the section is extensible, so it stays deleted.

### Leaderboard

```yaml
top:
  # Entries kept in the ranking snapshot.
  size: 10
  # Seconds between leaderboard refreshes. Values below 5 are raised to 5.
  refresh-seconds: 60
```

`size` bounds how many positions the placeholders can answer. The shipped `guis/rankup-menu.yml`
lists positions 1 to 10, so lowering `size` leaves the lines past it blank.

{% hint style="info" %}
Lowering `size` does not make the refresh read less. A rank's position lives in `rankup.yml` rather
than in a column, so every row is always read and ranked on the database thread. What `size` bounds
is how much of that crosses back to the server thread.
{% endhint %}

## rankup.yml

One entry per rank under `rankups:`. Two fields are required and three are optional.

| Field | Required | What it is |
|---|---|---|
| `order` | Yes | Progression order, lower is earlier. Values need not be contiguous |
| `display` | Yes | The prefix shown in messages and placeholders |
| `requirements` | No | What the player pays to REACH this rank |
| `rewards` | No | What fires when the player REACHES this rank |
| `menu-item` | No | How this rank looks in the menus |

```yaml
# sn:extensible
rankups:

  '0':
    order: 1
    display: '&8[&#8354f2&l0&8]'
    menu-item:
      material: BLAZE_POWDER
      display-name: '&#8354f2&lRank #0'
      lore:
        - '&7Where everyone starts.'

  '1':
    order: 2
    display: '&8[&#8354f2&l1&8]'
    requirements:
      hours: 2
      vault: 5000
    rewards:
      - '[broadcast] &a%player% &7reached rank &f1&7.'
      - '[console] give %player% diamond 1'
    menu-item:
      material: BLAZE_POWDER
      display-name: '&#8354f2&lRank #1'
      lore:
        - '&#8354f2&lREQUIREMENTS:'
        - '  &8| &7Hours: &f{req_hours}h'
        - '  &8| &7Money: &f${req_vault}'
```

{% hint style="info" %}
`requirements` and `rewards` describe reaching a rank, never leaving it. A rankup from one rank to
the next checks and charges the price written on the rank the player is going to, and fires the
rewards written there.
{% endhint %}

{% hint style="warning" %}
The lowest `order` is the starting rank. Everyone begins there, and because nobody ever reaches it,
its requirements are never evaluated and its rewards never fire. Writing either on it logs a
warning at load.
{% endhint %}

### Requirements

Keys are `hours`, always available, or any currency id you declared in `config.yml`. They are
checked and charged in the order you write them, and an amount of 0 or less is skipped.

`hours` is playtime, and it is the one requirement that is checked but never charged. It is read
from the player's own playtime statistic, so it counts across your whole server history.

If a later charge is refused, everything already taken in that rankup is handed back through the
`deposit` commands.

### Rewards

One line per entry, each an SnLib action:

| Tag | What it does |
|---|---|
| `[broadcast] <msg>` | Announces to everyone |
| `[message] <msg>` | Sends to the player |
| `[console] <cmd>` | Runs as the console |
| `[title]`, `[actionbar]`, `[sound]` | The rest of the SnLib action set |
| `[chance=50]` | Guard that fires the line half the time |

A line with no tag, and a line whose tag is not recognized, runs as a console command. `%player%`
and `{player}` become the player's name, and PlaceholderAPI is applied afterwards.

{% hint style="warning" %}
A reward line that ranks the same player up again has no base case and will keep going. Rewards
fire before the new rank is stored, so avoid `[console] rankup force %player%` and the rankup
action as reward lines.
{% endhint %}

### menu-item

| Field | What it is |
|---|---|
| `material` | Bukkit material, or `basehead-<base64>` for a custom head texture |
| `display-name` | The item title |
| `lore` | Your own lines for this rank |
| `glow` | true or false |
| `states` | Optional per state override used by the paginated menu |

`states` takes `claimed`, `ready`, `next` and `locked`, each accepting the same fields. Anything you
leave out of a state keeps what `menu-item` declares. Use it for the one rank that has to look
different in one state; to dress a whole state the same way on every rank, write `material` or
`glow` once on that state's template in `guis/rankup-list.yml` instead.

## guis/

Both menus are ordinary SnLib menu files: a `title`, `rows`, a `layout` grid of one character per
slot, `regions` naming the cells the plugin paints, `items` for the static buttons and `templates`
for the states.

| File | Used when | What it shows |
|---|---|---|
| `rankup-menu.yml` | `menu.mode: single` | One dynamic button for the next rank, your current rank, and the leaderboard |
| `rankup-list.yml` | `menu.mode: paginated` | The whole ladder, one slot per rank, opening on the page holding your next rank |

Move a button by moving its letter in the layout, and hide it by deleting the letter. The `items:`
section is extensible, so an entry you delete stays deleted, at the cost that a button a future
version ships is not inserted into your file either.

{% hint style="info" %}
On the paginated menu, exactly one tile is ever `ready` or `next`, never both. `ready` means this is
your next rank and you can afford it; `next` means it is your next rank and you cannot yet.
{% endhint %}

{% hint style="info" %}
A state template takes four appearance fields. `display-name` replaces the rank item's name and
`lore` appends after its own lore, while `material` (`basehead-<base64>` included) and `glow`
dress every tile of that state at once - one line instead of the same field repeated on every
rank. Leave a field out and each rank keeps what its own `menu-item` says. Anything else you
write on a template is ignored, because the rank's item already carries it.

One rank that has to look different in a single state still wins: `material` or `glow` under that
rank's `menu-item.states.<state>` in `rankup.yml` beats the template. An unknown material on a
template is reported once in the console and leaves that state showing each rank's own item.
{% endhint %}

{% hint style="info" %}
`strict-clicks: true` in both files means only plain left and right clicks reach a tile. Leave it
on: without it, drop, the number keys, swap offhand and middle click over the ready tile all buy a
rank, because Bukkit reports every one of them as a click on that slot.
{% endhint %}

## lang/messages_en.yml

Every player facing and admin facing line, including the `Next:` labels, the max rank word and the
requirement failure messages. `&` colors, `&#RRGGBB` hex and MiniMessage all work, and the file
merges on update like the rest.

## Applying changes

`/rankup reload` re-reads every managed file and rebuilds the ladder, the currencies and the
leaderboard. Command aliases are re-read too.

{% hint style="info" %}
A reload that cannot parse `rankup.yml` keeps the ladder currently in memory and reports the
failure. Your server keeps working on the last good ladder while you fix the file.
{% endhint %}
