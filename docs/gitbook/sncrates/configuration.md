# Configuration

Five kinds of file live under `plugins/SnCrates/`:

| File | What it holds | Merged on update? |
|---|---|---|
| `config.yml` | Defaults, keys, animations, effects, limits, database | Yes |
| `crates/*.yml` | The crates themselves: rewards, weights, limits, key items | **No** |
| `guis/*.yml` | The layout of every menu | Yes |
| `lang/messages_en.yml` | Every line the plugin sends | Yes |
| `opening.log` | An append-only record of every open | n/a |

"Merged" means SnLib adds new keys on boot and keeps your values, your comments and any extra keys
you added. There is no `config-version` anywhere and you never lose a file to an update. Set
`update-configs: false` to freeze the merge, after which SnLib only warns about missing keys.

Crate files are the exception on purpose: they are content, not configuration. `crates/example.yml`
is seeded once and never merged, re-seeded or rewritten. Delete it and it stays deleted; omit a key
inside a crate and it stays omitted, inheriting the documented default.

---

## config.yml

### Command

```yaml
command:
  aliases: [crate, snc]
  user-open-aliases: []
```

`aliases` are full aliases of `/crates`: the entire tree, admin subcommands included. They are
re-read on `/crates reload`.

`user-open-aliases` registers extra **standalone** commands that only open the key balance menu -
`/llaves` does exactly what a bare `/crates` does for a player and exposes no subcommand. They are
built while the server starts, so a change here needs a restart, not a reload. See
[Commands](commands.md#user-open-aliases).

### Database

```yaml
database:
  type: sqlite
  host: localhost
  port: 3306
  database: sncrates
  username: root
  password: ""
  connect-timeout-seconds: 10
  socket-timeout-seconds: 30
```

`type: sqlite` needs nothing else and writes `data.db` in the plugin folder. `type: mysql` reads the
connection block.

`connect-timeout-seconds` (1-3600) caps a single attempt to **open** a connection.
`socket-timeout-seconds` (0-3600, where 0 is unlimited) caps a read on an already-open connection.
Both apply to MySQL; SQLite is local and only uses the connect budget.

{% hint style="warning" %}
`socket-timeout-seconds: 0` is the driver's own default and can hang forever if the host stops
answering mid-query. Leave it at 30 unless you have a reason.
{% endhint %}

Seven `sncrate_`-prefixed tables store balances, limit counters, statistics, history, filters and
block locations. Item data is never stored there.

### Defaults

Values a crate falls back to when its own file omits them, and the seed values a brand-new crate is
created with in the editor.

```yaml
defaults:
  animation: NONE
  accepted-key-types: [PHYSICAL]
  preview-id: preview
  key-material: TRIPWIRE_HOOK
  key-name: "&f{crate}"
  display-name: "&f{crate}"
  reward-weight: 10.0
```

| Key | Meaning |
|---|---|
| `animation` | `NONE`, `CSGO`, `WHEEL`, `REVEAL` or `QUICK`. `NONE` opens instantly with no menu. |
| `accepted-key-types` | Any non-empty subset of `PHYSICAL`, `VIRTUAL`, `PERMISSION` (see the note below) |
| `preview-id` | Which file under `guis/` a preview opens |
| `key-material` | Material of a new crate's key item |
| `key-name` / `display-name` | `{crate}` is the crate id |
| `reward-weight` | Weight a reward is created with in the editor |

{% hint style="info" %}
The editor's **Accepted Keys** button cycles three combinations: `PHYSICAL`, `VIRTUAL`, and both.
`PERMISSION` is set here or in a crate file, not from that button - it is a rank-gated crate rather
than a key, and it consumes nothing when it opens. Such a crate keeps `PERMISSION` until somebody
clicks the button, at which point the cycle restarts at `PHYSICAL`, so leave that button alone on a
rank-gated crate. See [Rank-gated crates](faq.md) in the FAQ.
{% endhint %}

### Keys

```yaml
keys:
  restrict-usage: true
  allow-redeem: true
```

**Always on, whatever this says**: a key can never be placed as a block, put in an item frame, put
on an armour stand or eaten. Those four destroy the key, so they are refused for everyone,
operators included, with no bypass node.

`restrict-usage: true` adds the rest: the key becomes inert in crafting grids, anvils, enchanting
tables, furnaces, blast furnaces, smokers, brewing stands, grindstones, smithing tables, looms,
cartography tables, stonecutters and beacons, and a plain right-click does nothing at all - so a key
whose material is a snowball or an ender pearl cannot be thrown away.

Storing keys in containers (chests, barrels, shulker boxes, ender chests, hoppers, dispensers,
droppers), trading them to a villager and dropping them stay **allowed** on purpose. Players may
store and trade their keys; they may not destroy them.

{% hint style="info" %}
The world interaction itself is never blocked. A player holding a key can still open a door or a
chest - only the item's own use is denied, not the block they clicked. The 1.21 Crafter is
deliberately not in the blocked list.
{% endhint %}

`allow-redeem: true` enables the sneak-redeem: **sneak + right-click away from a crate block** while
holding a physical key, and every key for that crate in the inventory becomes a virtual balance. The
natural gesture is a right-click into empty air, and that is exactly what it is built for.

A key whose crate no longer exists is left alone - nothing is destroyed over a crate an admin may
recreate. A crate that does not accept `VIRTUAL` refuses the redeem, because there is no balance for
the keys to become.

### Filter and withdraw

```yaml
filter:
  enabled: true

withdraw:
  enabled: true
  stack-amount: 64
```

The **filter** puts a toggle in a crate's preview. A player can deactivate a reward for themselves:
it can still be rolled and still burns its limits, it is simply never handed to them. Nobody's
chances change either way. Deactivated rewards carry `preview.deactivated-tag` in the preview.

**Withdraw** turns a virtual balance back into the key item, from a button in the preview. The
button only appears on crates that accept **both** virtual and physical keys, since a withdrawn key
would otherwise be unusable.

| Click | Withdraws |
|---|---|
| Left | 1 |
| Right | `stack-amount` (64) |
| **Q** | as many as fit |

Only the keys that fit are handed over, and exactly that many are deducted.

### Editor

```yaml
editor:
  chat-input-timeout-seconds: 60
  preview-layouts: [preview, preview-compact]
```

| Key | Meaning |
|---|---|
| `chat-input-timeout-seconds` | How long an admin has to answer a chat prompt |
| `preview-layouts` | The layouts the crate panel's **Preview Layout** button cycles, in this order |

On expiry the prompt is cancelled and the editor screen reopens, exactly as typing `cancel` does.
Every prompt says both things underneath it - the cancel word and the seconds left - so an admin
never has to know them in advance.

Each `preview-layouts` entry is the name of a file under `guis/`, written without the `.yml`. An
entry naming a file that is not there is **skipped rather than offered**, so deleting a layout can
never leave a crate pointing at a preview that will not open. Add your own layout to this list after
copying `preview.yml` under a new name, and give it a label under `previews:` in the language file so
the button reads as a name instead of a file name.

### Opening log

```yaml
opening-log:
  enabled: true
  timestamp-format: "yyyy-MM-dd HH:mm:ss"
```

One line per open, appended to `plugins/SnCrates/opening.log`, with the reward's **display name**,
colour codes stripped, and every field sanitised of the `|` separator. A reward whose item carries
no custom name falls back to the reward id. A [failed](#fail) open writes its line too, with
`status.failed` from the language file (`failed` by default) where the reward's name goes.

{% hint style="warning" %}
The file grows forever. Prune it by hand when it gets large.
{% endhint %}

### Access

```yaml
access:
  allowed-worlds: []
  blocked-worlds: []
  allow-mass-open: true
  mass-open-max: 64
  mass-open-per-tick: 10
  mass-open-cooldown-ticks: 10
```

Both world lists gate the **physical crate block only**. `/crates open`, `/crates` and the key
balance menu do not consult them, and a key or a permission is still required there. When
`allowed-worlds` is non-empty, only those worlds allow using a crate block. `blocked-worlds` always
denies, even for a world named in `allowed-worlds`.

`allow-mass-open` decides whether a crate that declares nothing lets players open several at once:
sneak + right-click a crate block, or press Q on a crate in `/crates`.

`mass-open-max` caps one mass-open action. Both `-1` and `0` mean **unlimited**: the whole balance
goes in one action.

`mass-open-per-tick` is how many crates of that action are settled per server tick. The first batch
runs in the click itself and the rest follow one tick apart, so a balance of 500 keys takes about
2.5 seconds instead of running 500 rewards' worth of console commands in one tick. Every crate of the
batch is still a complete open - roll, one key, delivery - so a batch that stops early (the player
leaves, the crate is deleted, the server stops) has spent exactly the keys of the crates it opened
and the rest are still in the balance.

`mass-open-cooldown-ticks` is the wait between two mass-open actions of the same player, so a held Q
on the `/crates` menu does not fire a batch on every click. `0` disables it. The refusal is silent
unless `messages.open.mass-cooldown` in the language file says something.

{% hint style="info" %}
Before 2.3.0 an unlimited cap opened the whole balance in one tick. It no longer does; `64` stays
the shipped cap because a summary of 4,000 opens is still not something a player wants to read.
{% endhint %}

### Fail

```yaml
fail:
  chance: 0.0
  item:
    material: BARRIER
    display-name: "&c&lNothing"
    lore:
      - "&7The key was spent and nothing came out."
```

`chance` is the percentage of opens that **fail**: the key is spent and nothing is won. `0` never
fails, `100` always fails, decimals work (`2.5`). It is the value a crate that declares nothing
inherits; a crate overrides it with `settings.fail-chance` in its file or from the crate panel in the
editor.

What a failed open does, and does not do:

| Does | Does not |
|---|---|
| Spends the key | Deliver an item or run commands |
| Runs the animation and lands on `fail.item` | Burn any reward limit |
| Plays `effects.fail-sound` | Fire the particle burst or the broadcast |
| Counts as an open (`%sncrates_opens_<crate>%`) | Write a history row |
| Writes a line in `opening.log`, with `status.failed` where the reward's name goes | Fire `CrateRewardEvent` (`CrateOpenEvent` fires) |
| Sends `messages.open.fail`, or counts towards the mass-open summary | |

The fail is decided in the same roll as the winner, **after** the check that something is winnable:
a crate whose every reward is on limit still refuses the open with the key unspent, whatever its
fail chance.

`item` uses the [item format](#the-item-format) of the crate files. The animation's strip shows it at
the fail rate too, so the landing is a possibility the player watched scroll past.

{% hint style="info" %}
The reward chances in the preview are **not** reduced by the fail chance. They stay each reward's
share of the opens that pay out, and the preview's info icon says `Fails N% of the time` beside them
(`messages.preview.fail-tag`). `%sncrates_failchance_<crate>%` exposes the number.
{% endhint %}

### Stats

```yaml
stats:
  history-enabled: true
  history-cap: 50
```

`history-enabled` records every win in the player's rolling history; `history-cap` is how many rows
are kept per player, oldest pruned first.

{% hint style="info" %}
The history is written but never read back by the plugin. It exists so you can query it yourself.
Per-player open counts, which the placeholders do read, are separate and are not affected by
`history-enabled`.
{% endhint %}

### Animations

Every timing is in ticks; 20 ticks is one second. Which slots each animation paints is declared by
its layout under `guis/`.

```yaml
animations:
  csgo:
    duration-ticks: 120
    interval-ticks: 2
    strip-size: 27
  wheel:
    duration-ticks: 100
    interval-ticks: 2
    strip-size: 27
  reveal:
    reveal-interval-ticks: 6
  quick:
    duration-ticks: 20
    interval-ticks: 2
  final-visual-delay-ticks: 30
  tick-sound-interval-ticks: 4
```

| Animation | What it looks like |
|---|---|
| `CSGO` | A horizontal strip scrolls, decelerates and settles on the winner |
| `WHEEL` | Several rows spin at once and settle together |
| `REVEAL` | A grid of face-down cards flips one at a time; the last is the winner |
| `QUICK` | A short flourish, then the reward |
| `NONE` | No menu at all: the reward and the win message, instantly |

`strip-size` is how many items the scrolling strip is built from - for the wheel, per spinning row.
Both are **capped at 256**, and the strip is rebuilt on the main thread on every open, so a large
value costs real time.

`final-visual-delay-ticks` is the pause between the settled frame and the delivery, so the result is
readable. `tick-sound-interval-ticks` throttles the spin tick sound.

### Effects

```yaml
effects:
  open-sound: "BLOCK_CHEST_OPEN"
  win-sound: "ENTITY_PLAYER_LEVELUP"
  fail-sound: "ENTITY_VILLAGER_NO"
  complete-particle: "HAPPY_VILLAGER"
  complete-particle-count: 40
  rare-win-particle: "CRIT"
  tick-sound: "UI_BUTTON_CLICK"
  tick-sound-volume: 0.4
  glow-winner: true
```

A sound is `SOUND_ID [volume] [pitch]`, so `BLOCK_CHEST_OPEN 1.0 1.2` is valid. The id is either the
enum name or a `minecraft:entity.player.levelup` key. A particle is the Bukkit enum name. `""`
disables that effect.

`tick-sound` is global only, never per crate, and only its **id** is read here: the volume comes
from `tick-sound-volume` and the pitch is computed from the spin, so a volume or pitch written on
that line is ignored.

`fail-sound` is global only too. It replaces `win-sound` when the spin lands on a [fail](#fail), and
no particle burst goes with it.

{% hint style="warning" %}
`HAPPY_VILLAGER` is the 1.20.5+ spelling. On 1.20.1 to 1.20.4 the constant is `VILLAGER_HAPPY` - put
that here instead, or the completion burst stays off and the console logs one warning per load.
`CRIT` is the same on every version.
{% endhint %}

### Broadcast

```yaml
broadcast:
  enabled: true
  mode: BROADCAST
  only-flagged: true
```

`mode` is `BROADCAST` (the whole server), `PLAYER` (only the opener) or `NONE`. `only-flagged: true`
announces only rewards carrying `broadcast: true`; `false` announces every win. The line itself is
`messages.open.broadcast` in the language file.

### API events

```yaml
api-events:
  enabled: true
```

Whether `CrateOpenEvent` and `CrateRewardEvent` are fired. Turn it off only if nothing on the server
listens: both are built and dispatched once per crate opened, so a mass open fires two per crate in
the batch. See [Developer API](api.md).

### Language and debug

```yaml
lang: en
update-configs: true

debug:
  enabled: false
  level: DEBUG
  categories: []
```

`lang` loads `lang/messages_<code>.yml`, falling back to `en`. `debug` is also toggleable live with
`/crates debug`; `level` is `OFF`, `INFO`, `DEBUG` or `TRACE`, and an empty `categories` list lets
every category through.

---

## Crate files

Every `.yml` under `crates/` is a catalogue. **Each top-level key is a crate id**, so one file can
hold as many crates as you like. The in-game editor writes one file per crate, named after its id.

A crate id is lowercase letters, digits, underscore and hyphen, and cannot contain a dot. It is what
commands, permissions and placeholders refer to.

```yaml
example:
  display-name: "&#8354f2&lExample Crate"
  accepted-key-types: [PHYSICAL, VIRTUAL]
  animation: CSGO
  preview-id: "preview"
  open-permission: ""

  settings:
    mass-open-max: 10

  effects:
    complete-particle: "FLAME"
    complete-particle-count: 25
    rare-win-particle: ""

  key-item:
    material: TRIPWIRE_HOOK
    display-name: "&#8354f2&lExample Key"
    lore:
      - "&7A key to the &f&lExample Crate&7."
    amount: 1
    glow: true

  rewards:
    diamonds:
      weight: 70.0
      enabled: true
      material: DIAMOND
      display-name: "&b16x Diamonds"
      amount: 16
      per-player-limit: 0
      global-limit: 0
      cooldown-seconds: 0
      broadcast: false
      give-item: true
      commands: []
```

### Inheritance: the three states

Under `settings:` and `effects:`, every key has **three** states and the difference matters:

| State | Written as | Means |
|---|---|---|
| Inherit | the key is absent | Follow the `config.yml` value, including later changes to it |
| Override | the key has a value | Use this value for this crate only |
| Off | the key is `""` | That effect is off for this crate, even when `config.yml` names one |

All three are **remembered**. Editing a crate in game writes back the state each key is in, so an
inherited key is not quietly frozen to whatever the global value was that day, and a blanked key
does not turn itself back on.

`settings:` accepts `allow-mass-open`, `mass-open-max` and `fail-chance`. `effects:` accepts
`open-sound`, `win-sound`, `complete-particle`, `complete-particle-count` and `rare-win-particle`.

{% hint style="warning" %}
`complete-particle-count: 0` means exactly zero particles, **not** "fall back to the global 40".
Delete the key to inherit. `fail-chance: 0` is the same shape: it means this crate never fails,
even when `fail.chance` says otherwise, and deleting the key is what inherits.
{% endhint %}

### The item format

Used by `key-item` and by every hand-written reward. Only `material` is required.

| Key | Meaning |
|---|---|
| `material` | Item id, or `basehead-<base64>` for a textured player head |
| `display-name` | Empty leaves the vanilla name |
| `lore` | List of lines |
| `amount` | Stack size |
| `glow` | Enchantment glint |
| `custom-model-data` | Resource-pack model id |
| `item-model` | Base item model key, 1.21.2+ (e.g. `nexo:2d_player_head`) |
| `enchantments` | List of `"ENCHANT LEVEL"`, e.g. `"SHARPNESS 5"` |
| `flags` | List of `ItemFlag` names, or `HIDE_ALL` |
| `color` | `"R, G, B"` or `"#RRGGBB"`, for leather armour and potions |
| `trim-pattern` / `trim-material` | Armour trim, used together |
| `potion-effects` | List of `"EFFECT LEVEL DURATION_TICKS"` |
| `attributes` | List of `"ATTRIBUTE OPERATION AMOUNT [SLOT_GROUP]"` |
| `unbreakable` | Vanilla unbreakable flag |
| `max-stack-size` | 1.20.5+ stack cap |
| `skull-owner` | Player name or UUID, for a player head |
| `damage` | Spent vanilla durability |

Colour codes work in `display-name` and `lore`, both `&a` and `&#RRGGBB`.

{% hint style="warning" %}
PlaceholderAPI placeholders in an item's text resolve **once**, when crates load, against no
particular player. `%server_name%` works here; `%player_name%` does not. Menu button lore under
`guis/` is different - that resolves per viewer.
{% endhint %}

A reward can also carry `nbt: "<base64>"`, which is what the editor writes when it captures the item
from your hand. It carries enchants, custom heads, components and its own stack size, and when
present it **replaces the whole item spec above**.

### Reward keys

| Key | Default | Meaning |
|---|---|---|
| `weight` | `defaults.reward-weight` | Relative draw weight. Must be greater than zero. |
| `enabled` | `true` | `false` keeps the reward visible in the preview and the animation but it can never be won, and it stops counting towards the other odds |
| `per-player-limit` | `0` | Max wins per player. `0` is unlimited. |
| `global-limit` | `0` | Max wins across the whole server. `0` is unlimited. |
| `cooldown-seconds` | `0` | Length of the limit window in seconds. `0` makes both limits lifetime caps. |
| `broadcast` | `false` | Announce this win; `broadcast.*` in `config.yml` decides who hears it |
| `give-item` | `true` | Whether the item is actually handed over |
| `commands` | `[]` | Console commands run on win |

{% hint style="info" %}
`cooldown-seconds` is the **limit window**, not a cooldown between opens. There is no per-crate open
cooldown in SnCrates.
{% endhint %}

`give-item: false` is how a reward becomes command-only: the item stays as the icon players see in
the preview and the spin, and the winner gets whatever the commands grant instead.

### Win commands

Run from the console when the reward is won.

```yaml
commands:
  - "lp user {player} parent add legendary"
  - "eco give {player} 5000"
```

`{player}`, `{amount}` and `{crate}` are substituted first, then PlaceholderAPI resolves the whole
line against the winner, then it is dispatched. So a nested placeholder works:

```yaml
commands:
  - "/nivel give {player} %math_5*{edtools_leveling_level_level}/100%"
```

{% hint style="warning" %}
Commands are never colour-translated. Writing `&a` or a hex code in a command corrupts the command
text rather than colouring anything.
{% endhint %}

A command that fails is logged and the rest still run. In a mass open, one reward failing to settle
does not abort the batch.

### Editing a crate by hand

It works, but the editor rewrites a crate's whole entry when it saves, so **comments inside a crate
you edit in game do not survive**. Your values do, and so does the inherit/override/off shape of
every settings and effects key.

---

## The in-game editor

`/crates editor`, with `sncrates.admin.editor`.

| Screen | What it does |
|---|---|
| Crate list | Create a crate, page through the existing ones |
| Crate panel | Display name, animation, accepted keys, open permission, preview layout, key item, mass-open, fail chance, the five per-crate effects, crate block, duplicate, delete |
| Reward list | Add a reward, reorder, mass creation, page through |
| Reward panel | The item, weight, amount, per-player limit, global limit, limit window, win commands, can-be-won, announce, give-item, duplicate, delete |
| Item capture | Copies the item from your main hand |

Values that need typing are asked in chat, and every prompt tells you how to get out of it: the
cancel word (`cancel` by default) and how many seconds are left. Either way the editor screen
reopens. Animation, accepted keys, preview layout and the on/off controls are clicks, not prompts.

The **Fail Chance** control on the crate panel has two states: inherited, showing what `fail.chance`
currently says, and a value of the crate's own. Left-click types a percentage in chat (`0` never
fails, `100` always fails, decimals and a comma both work); shift-left-click goes back to inheriting.
A typed value outside 0 to 100 is clamped, and the confirmation says what was stored.

The reward panel's **Win Commands** icon lists the commands themselves, numbered, and marks in red
any line that will not run as written:

| Marked when | Why |
|---|---|
| The line is empty | Nothing to dispatch |
| It uses a placeholder that is not `{player}`, `{amount}` or `{crate}` | Those three are the only ones filled in, and they are case-sensitive |
| It uses a `%papi%` token and PlaceholderAPI is not installed | The token reaches the console as written |

The check never asks the server which commands exist: a command-map lookup would call a vanilla
command unknown on one server and known on the next, and a wrong mark teaches you to ignore the
marks. A misspelt command name is still yours to spot.

### The item capture

**Hold the item in your main hand and click the capture cell.** The crate gets a copy, and you keep
what you were holding - so the same item can be captured into as many crates as you like.

There is nothing to drop into and no cell you can put an item in. Every click in that window is
cancelled, exactly like every other menu in the plugin, and closing it changes nothing.

The stack size comes from your hand: holding 16 of something makes a reward of 16. The amount is
also editable on the reward's own panel afterwards, which is the easier way to change it.

One window does three jobs, depending on which button opened it: replace a reward's item, create a
new reward, or set a crate's physical key item.

### Crate blocks

The crate panel's **Crate Block** button arms the next block click:

- Left-click it to arm a **bind**, then click any block in the world to bind it to this crate.
- Right-click it to arm an **unbind**, then click a bound block to release it.

Either mouse button consumes an armed gesture, and it stays armed until it is used or you log out. A
block that already belongs to another crate is refused rather than repointed - unbind it first.

A bound block cannot be mined and survives explosions. Deleting the crate unbinds every block bound
to it, and a block left bound to a crate that no longer exists breaks normally the next time
somebody swings at it.

### What deleting removes

| Deleting | Also clears | Deliberately keeps |
|---|---|---|
| A crate | Its bound blocks, global limits, per-player limits and reward filters | Key balances, statistics and history |
| A reward | That reward's limits and filters | - |

Both force-close and settle any opening still running on the target first, so nobody loses a paid
spin to an admin's edit. Balances and statistics survive a crate delete because they are entitlement
and record, not limits.

---

## Menus

Each file under `guis/` is one menu: a title, a `layout` of rows where every character is a slot,
and the items each character maps to. Editing them is how you restyle the plugin.

| File | Menu |
|---|---|
| `key-balance.yml` | What a bare `/crates` opens |
| `preview.yml`, `preview-compact.yml` | Reward previews; a crate picks one with `preview-id` |
| `animation-csgo.yml`, `animation-wheel.yml`, `animation-reveal.yml`, `animation-quick.yml` | Which slots each animation paints |
| `editor-main.yml`, `editor-crate.yml`, `editor-rewards.yml`, `editor-reward.yml` | The editor screens |
| `reward-deposit.yml` | The item capture |

Names and lore resolve colour codes, HEX, `[center]` and PlaceholderAPI **per viewer**, so
`%player_name%` in a button's lore renders the name of whoever is looking.

A window title can name what it shows: `{player}` on `key-balance.yml`, `{crate}` and `{crate-id}`
on every preview layout (2.3.1) and on the animations, `{crate}` / `{reward}` / `{mode}` on the
editor screens. A placeholder a menu does not seed renders as its own literal text.

### Icons drawn from the crate's key item

Three icons are drawn from the crate's own physical key, so the player sees the key they are being
told they own - custom head, model data and all: the crate icons in `key-balance.yml`, and the
`info` and `withdraw` buttons of every preview layout. The `material` and `glow` on those templates
apply only to a crate that has **no** physical key.

The key item's own **lore** is not shown on them (2.4.0). Those lines are written for a player
holding the key ("sneak + right-click to redeem") and on a button they sat above what the button
was actually saying, with no way to move or remove them. Write `{lore}` in the template's `lore:`
to bring them back, wherever you want them:

```yaml
templates:
  default:
    display-name: "&#8354f2&l{crate}"
    lore:
      - "&7You own &f{keys} &7key(s)."
      - ""
      - "{lore}"          # the key item's own lore, one lore line per line it has
      - ""
      - "&8» &aLeft-click &7to open one"
```

A crate with no physical key, or a key with no lore, renders `{lore}` as one blank line. Reward
icons in a preview are unaffected: a reward's own lore is shown on its own icon, as it always was.

The placeholders each icon carries:

| Icon | Placeholders |
|---|---|
| A crate in `key-balance.yml` | `{crate}` `{keys}` `{fail-chance}` `{lore}` |
| `info` in a preview layout | `{crate}` `{rewards}` `{fail}` `{fail-chance}` `{lore}` |
| `withdraw` in a preview layout | `{crate}` `{keys}` `{lore}` |
| A reward in a preview layout | `{crate}` `{chance}` `{limit}` `{tags}` |

Add your own preview layout by copying `preview.yml` under a new name. Three places make it a
first-class layout: list its file name under `editor.preview-layouts` in `config.yml` so the editor
button can reach it, add a matching entry under `previews:` in the language file so the button shows
a readable name, and point a crate at it from the crate panel.

{% hint style="warning" %}
The `click-actions` lines (`[crates-editor-crate] delete` and friends) are the button's behaviour.
Change materials, names, lore and slots freely; changing or removing an action line changes what the
button does.
{% endhint %}

---

## Language

`lang/messages_en.yml` holds every line the plugin sends. `&0-&f`, `&l&o&m&n`, `&#RRGGBB`,
`[center]` and `%papi%` all work, and a value set to `""` is not sent at all - which is how a line
is hidden.

Four blocks sit outside `messages:` because they are spliced **into** other lines rather than sent:

| Block | Holds |
|---|---|
| `snlib:` | SnLib's shared 11-key command contract. Ship it complete: SnLib merges neutral defaults for anything you omit, which leaks unbranded lines |
| `status:` | The short state words: `on`, `off`, `enabled`, `disabled`, `none`, `unlimited`, `failed` |
| `animations:`, `key-types:`, `previews:` | Display labels for the editor. The keys are what crate files use - do not rename them |
| `format:` | The thousands separator and the list separator |

{% hint style="warning" %}
Never write `{prefix}` in a value. SnLib auto-prepends the `prefix:` at the top of the file to every
single-line message, and a literal `{prefix}` renders verbatim.
{% endhint %}

## Automatic updates and backups

SnLib merges new keys into `config.yml`, `guis/*.yml` and the language file on every boot, keeping
your values, your comments and any key you added, and takes a backup before it writes. Freeze a
section with a `# sn:extensible` comment and entries you delete there stay deleted.

Crate files are never touched by the merge.
