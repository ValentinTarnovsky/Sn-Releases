# Configuration

Everything lives in `plugins/SnJoinItem/`:

| File | What it holds |
|---|---|
| `config.yml` | When and where the item is delivered, the slot it is pinned to, the command aliases, the language and the debug switches |
| `items.yml` | The item itself: material, name, lore, lock and click actions |
| `lang/messages_en.yml` | Every line the plugin sends to a player or an admin |
| `data.yml` | Written by the plugin. The per-UUID opt-out list from `/joinitem disable` |

There is no `messages.yml` and no `config-version` key anywhere. `data.yml` is never seeded from
the jar and never merged - it is player state, and touching it would hand the item back to
everyone who had opted out.

Run `/joinitem reload` after editing `config.yml`, `items.yml` or the language file.

## config.yml

| Key | Default | What it does |
|---|---|---|
| `lang` | `en` | Loads `lang/messages_<code>.yml`. A missing or corrupt file falls back to `messages_en.yml` with a console line |
| `update-configs` | `true` | Master gate for the automatic key merge. `false` freezes your files and the plugin only warns about missing keys |
| `debug.enabled` | `false` | Runtime debug output to the server log |
| `debug.level` | `DEBUG` | Verbosity ladder: `OFF`, `INFO`, `DEBUG`, `TRACE` |
| `debug.categories` | `[]` | Empty lets every category through; naming categories narrows the output to those |
| `command.aliases` | `[jit]` | Aliases of `/joinitem`. Re-read on `/joinitem reload` |
| `delivery.give-on-join` | `true` | Master switch for **all automatic delivery**, not only joining |
| `delivery.give-delay-ticks` | `20` | Ticks to wait after join before giving the item |
| `delivery.restore-interval-ticks` | `40` | How often to re-check players and restore the item. `0` disables the sweep |
| `delivery.worlds` | `[]` | Worlds where the item is **given**. Empty means every world. Case-insensitive |
| `placement.slot` | `4` | Full-inventory index the item is pinned to |

The rest of this section explains the ones with consequences.

### Delivery

```yaml
delivery:
  give-on-join: true
  give-delay-ticks: 20
  restore-interval-ticks: 40
  worlds: []
```

**`give-on-join`** is read first by the single eligibility check that join, respawn and the restore
sweep all share. Setting it to `false` therefore does more than its name suggests: the respawn
re-give stops, and the restore sweep stops re-placing for every player. `/joinitem give` becomes the
only way in, and an external `/clear` then removes the item permanently.

**`give-delay-ticks`** is not cosmetic. Lobby, login/auth and inventory-restore plugins populate or
empty the inventory right after a player joins, and an item handed out inside the join event is
wiped by whichever of them runs a moment later. If the item does not appear on join, raise this to
40 or 80.

{% hint style="info" %}
`give-delay-ticks: 0` does not mean "inside the join event". The plugin floors the delay at one
tick, so `0` means the next tick - as early as the item is ever given. There is no setting that
gives it any sooner.
{% endhint %}

**`restore-interval-ticks`** is what makes the item genuinely un-removable. The lock cancels player
actions; it cannot see another plugin emptying the inventory, so this repeating sweep is what
survives Essentials `/ci`, `/clear` and any plugin that writes to the inventory directly. It is also
the retry that lets placement be skipped safely when an inventory is full. `0` disables both halves:
no restore after a clear, and no second attempt for a player whose inventory was full.

The sweep re-places when the target slot has lost the item **or** when a copy exists anywhere else,
which is how stray duplicates get cleaned up rather than accumulating as locked items the player
cannot remove.

**`worlds`** gates giving only. A player who receives the item in `lobby` and walks into `survival`
keeps it - the plugin never takes it away for being in the wrong world. Leave the list empty for
every world.

### Placement

```yaml
placement:
  slot: 4
```

A full-inventory index, not a hotbar index:

| Range | Meaning |
|---|---|
| `0-8` | Hotbar |
| `9-35` | Main inventory |
| `36-39` | Armour |
| `40` | Off-hand |

Anything outside `0-40` falls back to `4` with a warning in the console.

The plugin never destroys what is already in that slot. An occupant is moved to the first free slot;
if there is none the placement is skipped entirely and retried by the restore sweep once a slot
frees. Note that the relocation target is always a normal inventory slot (`0-35`), so if you pin the
item to an armour or off-hand slot whose occupant cannot be relocated, placement is skipped rather
than the occupant being destroyed.

### Command aliases

```yaml
command:
  aliases: [jit]
```

The config list is authoritative for the dynamic aliases and is re-read by `/joinitem reload`, so an
alias you add works without a restart. `jit` itself is also declared in the plugin's `plugin.yml`, so
removing it from this list does not unregister `/jit`. An alias that collides with an existing
command is skipped and reported in the console.

### Language

Set `lang: es` and drop `lang/messages_es.yml` next to the English file. Keys missing from your
language file fall back to the English value, with a console line naming the key.

{% hint style="info" %}
Never write `{prefix}` in a message value. The `prefix` line is prepended automatically.
{% endhint %}

The `snlib.*` block is SnLib's shared command contract (permission errors, usage, help layout). It
is shipped so the whole fleet looks alike; restyle it freely.

There is deliberately no `status:` block. The only state SnJoinItem exposes is through the two
placeholders below, which return the literal strings `true` and `false`.

### Debug

```yaml
debug:
  enabled: false
  level: DEBUG
  categories: []
```

SnJoinItem itself does not emit debug lines; what this switch surfaces is SnLib's own runtime
detail, which is mostly useful for action lines that silently do nothing (a positional guard skipped
for lack of a click surface, for example). There is no `/joinitem debug` subcommand, and unlike every
other key on this page **the debug block is read once at startup** - `/joinitem reload` does not
re-read it, so a change here needs a server restart.

## items.yml

One section, `join-item`. The id is referenced from Java: do not rename it. This file is merged from
the jar on every boot, so a renamed section is not replaced - the default `join-item` section is
simply re-inserted alongside yours, and the plugin keeps using that one while silently ignoring your
copy. Change the fields, not the id.

### Appearance

| Key | Default | What it does |
|---|---|---|
| `material` | `COMPASS` | Any Bukkit material that is a real **item** |
| `display-name` | `"&aMenu &7(click)"` | Supports `&` codes, `&#RRGGBB` hex and PlaceholderAPI, resolved per player |
| `lore` | two lines | Same formatting and placeholder support as `display-name` |
| `glow` | `false` | Enchantment glint without an enchantment |
| `unbreakable` | `true` | Stops the item losing durability |
| `flags` | six `HIDE_*` flags | Hides tooltip extras |
| `custom-model-data` | *not shipped* | Resource-pack model data. Add the key only if you need it |

{% hint style="warning" %}
**`material` must be a real item.** A block-only material - `POTATOES` (the crop) instead of
`POTATO`, `WATER`, `FIRE`, any `*_WALL_SIGN` - and `AIR` look valid, resolve happily, and then
produce a stack that vanishes when it is written into an inventory. The plugin catches this at boot,
disables the item and names the offending material in the console rather than letting the restore
sweep re-place nothing forever.
{% endhint %}

{% hint style="danger" %}
**Do not put a PlaceholderAPI token in `material`.** The name and lore are resolved per viewer, but
the material is cached once at boot to keep the hot paths cheap. A per-player material breaks the
item-frame guard and kills the inventory-click actions.
{% endhint %}

Choose the material with the lock in mind. The plugin re-creates the item whenever it leaves its
slot, so any material that can leave the inventory becomes a duplicator. Two classes to avoid: things
the player **spends** (food, potions, buckets, ender pearls, snowballs, eggs, fireworks, discs,
books) and things an **entity takes from the hand** (wheat and seeds for breeding, bone for taming,
name tags, dye, lead, saddle). Item frames and armour stands are blocked by the plugin; other entity
interactions are deliberately left alone so trading with a villager or clicking an NPC while holding
the item still works.

`custom-model-data` is deliberately shipped commented out. SnLib gates on the key being **present**,
not on its value, so `custom-model-data: 0` stamps a real model data of `0` onto the item rather than
meaning "none". Add the line when your pack needs it.

### Lock

| Key | Default | What it does |
|---|---|---|
| `locked` | `true` | Pins the item: inventory click, drag, manual equip, hand swap, drop, death drops and hopper transfer are all denied by SnLib |
| `placeable` | `false` | Stops the item being placed as a block |

SnJoinItem closes what SnLib's lock does not reach: handing the item to an item frame or an armour
stand, and the client resync that has to follow a cancelled creative-mode edit. Creative clicks
themselves are already covered, because Bukkit routes them through the ordinary inventory-click
event.

The lock applies to **everyone** holding the item, staff included. `snjoinitem.bypass` only controls
whether a player *receives* it; it does not exempt anyone from the lock or from the click. To take
the item off someone, use `/joinitem disable <player>`.

{% hint style="danger" %}
**Never set `locked: false` while `delivery.restore-interval-ticks` is above 0.** The player can then
drop or store the item and the sweep mints a replacement on the next interval - that is an item
duplicator. The plugin warns about this combination on boot. Set `locked: true`, or set
`restore-interval-ticks: 0`.
{% endhint %}

`placeable: false` only matters if you change `material` to a block, but it is left on deliberately:
a placeable join item is re-created by the restore sweep after every placement.

### Click actions

```yaml
  right-click-actions:
    - "[player] menu"

  left-click-actions:
    - "[player] menu"

  cooldown: 4
```

One action per line, each starting with a prefix. The most used ones:

| Prefix | Runs |
|---|---|
| `[player]` | The command as the player |
| `[console]` | The command from the console |
| `[message]` | A chat message (also the default when a line has no prefix) |
| `[actionbar]` | An action bar message |
| `[title]` | A title |
| `[sound]` | A sound for the clicking player |
| `[close]` | Closes the open inventory |
| `[open]` | Opens a SnLib GUI |
| `[connect]` | Sends the player to another server through the proxy |
| `[broadcastmessage]` | A message to the whole server |

A line may also carry leading guards, such as `[right-click]`, `[shift-left-click]`,
`[chance=25]` or `[click=RIGHT,SHIFT_RIGHT]`, before its real prefix.

Optional keys, not shipped because they are off by default:

| Key | What it does |
|---|---|
| `shift-right-click-actions` | Runs instead of the base right list when the click is shifted |
| `shift-left-click-actions` | Same for the left side |
| `interact-requirements` | Conditions that must pass before any action runs |
| `deny-actions` | Runs when `interact-requirements` fail |

`interact-requirements` are placeholder comparisons: `left OP right` with `>`, `<`, `>=`, `<=`, `=`,
`==` and `!=`, joined with `&&` and `||` and groupable with parentheses. Lines in the list are joined
with an implicit AND.

**`cooldown` is in ticks**, per player: `20` is one second. It is shared between the two click
surfaces, so one cooldown covers both. `0` disables it and leaves the click completely unthrottled.

{% hint style="warning" %}
**`%player%` resolves for a click inside an inventory but not for a click in the world.** The
inventory path binds it; SnLib builds the world-click context itself and binds nothing. Use
`%player_name%` (PlaceholderAPI) whenever an action has to work from both surfaces.
{% endhint %}

#### Declare both sides

The two sides never cross. A right click runs `right-click-actions` and a left click runs
`left-click-actions`, and neither ever falls back to the other, in the world or inside an open
inventory. Declare only one side and the opposite click does nothing at all, on both surfaces, with
nothing logged.

Declare both `right-click-actions` and `left-click-actions` if both clicks have to act. Declaring
them with different contents is fine and fully supported - each click then runs its own list on both
surfaces.

Not every click reaches these lists either. Middle click, drop, number keys, off-hand swap and double
click never fire on either surface, and the positional guards `[click-air]` and `[click-block]` never
fire on an inventory click because there is no clicked block. A line guarded by one of those is
skipped silently, with no console error.

## PlaceholderAPI

Optional. When PlaceholderAPI is installed, placeholders resolve in the item name and lore, in the
messages and inside the click actions, and this expansion is registered (a console line confirms it):

| Placeholder | Returns |
|---|---|
| `%snjoinitem_enabled%` | `true` unless the player was opted out with `/joinitem disable` |
| `%snjoinitem_disabled%` | `true` if the player was opted out |

Both resolve by UUID, so they answer for offline players too, and both return the literal strings
`true` and `false`.

{% hint style="warning" %}
They reflect the **opt-out list only**. They do not account for `snjoinitem.bypass`,
`delivery.worlds` or `delivery.give-on-join`, so `%snjoinitem_enabled%` can read `true` for a player
who will never receive the item.
{% endhint %}

The identifiers and their `true`/`false` output are a wire contract: other plugins' configs and
scoreboard lines compare against them, so they are not renameable and not restyleable.

## Automatic updates and backups

There is no `config-version` key. On every boot SnJoinItem compares each of its YML files against the
copy bundled in the jar and inserts the keys you are missing, in place, with their comments. Your
values, your comments, your ordering and any extra keys you added are all preserved - nothing is
deleted, nothing is reformatted. You never have to reconfigure after an update.

Before writing a merge, the file on disk is copied next to itself as
`old-<name>-<yyyyMMdd-HHmmss>.yml`, and the last three of those are kept. If a file is so broken it
does not parse as YAML at all, it is moved aside to `<name>.backup-N` and regenerated from the jar
rather than crashing the plugin, with a warning naming the backup.

`update-configs: false` freezes the merge: instead of inserting anything, the console reports how
many keys are missing in each file. Two things stay outside the gate by design - `config.yml` itself,
so the `update-configs` key can arrive through a merge in the first place, and SnLib's own `snlib.*`
message block, which is the library's command contract rather than your content. `data.yml` is
outside all of this: it is never seeded and never merged.

Every merge line in the console is prefixed `[update-configs]`, so a search for that tag tells you
exactly what happened to your files on the last boot.
