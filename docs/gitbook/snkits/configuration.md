# Configuration

SnKits ships four kinds of file:

| File | What it is | Managed |
|---|---|---|
| `config.yml` | Menus, auto-claim, multi-claim, sounds, database, editor strings | Yes |
| `lang/messages_en.yml` | Every line the plugin sends to a player or an admin | Yes |
| `guis/*.yml` | The layout of every menu, one file each | Yes |
| `kits/<id>.yml` | One kit | No, this is data |

Managed means new keys are merged into your file on boot. Your values, your comments and your own
additions are preserved, and example entries you delete stay deleted. Sections marked
`# sn:extensible` are the ones whose entries are yours.

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

### Command

```yaml
command:
  # Aliases of /kit. Re-read on /kit reload.
  aliases: [kits]
```

The aliases in `config.yml` are authoritative at runtime, and the `plugin.yml` list is only the
fallback used when the key is missing.

### Database

Player usage lives in the database: cooldown timestamps and spent one-time claims. SQLite needs no
setup. MySQL reads the connection block.

```yaml
database:
  # sqlite or mysql
  type: sqlite
  # MySQL connection (ignored when type is sqlite).
  host: localhost
  port: 3306
  database: snkits
  username: root
  password: ""
```

### Menus

```yaml
# Menu a bare /kit opens. Names a file in guis/, without the .yml.
default-gui: "kits-main"
```

### Items editor

The grid `/kit edit` opens is a real inventory you move items around in, so its strings live here
instead of in a `guis/` file.

```yaml
items-editor:
  # Title of the grid. {kit} is the kit id.
  title: "&#8354f2&lItems &8- &e{kit}"
  # Slot 53. Click it to drop a fresh command item into the first free cell.
  # Full Sn item spec: material, display-name, lore, glow, amount,
  # custom-model-data, enchantments, flags and the rest all work.
  add-command-button:
    material: "COMMAND_BLOCK"
    display-name: "&#8354f2Add Command Item"
    lore:
      - "&7Adds a command item to the grid."
      - "&7Shows in the preview, runs commands on"
      - "&7claim instead of being given."
      - ""
      - "&7Right-click it in the grid to configure."
    glow: true
  # Lore appended to a command item inside the editor only, and stripped back
  # off when the grid is read, so the stored item keeps exactly its own lore.
  # Set it to [] to append nothing.
  command-item-hint-lore:
    - ""
    - "&eRight-click &7to configure commands."
    - "&7Drag to move it within the grid."
  # Same, for the item marked as the kit's offhand item. Shift-right-click an
  # item in the grid to mark it; a kit has at most one.
  offhand-item-hint-lore:
    - ""
    - "&#8354f2>> Offhand item"
    - "&7Shift-right-click to unset."
```

Slots 0 to 52 are the kit contents, at the exact position the preview shows them. Slot 53 is the
add-command button. Closing the grid saves, including with Escape.

### Auto-claim

Hands kits to players when they join.

```yaml
auto-claim:
  # Master switch of the give-on-join.
  enabled: false
  # first-join gives only while the player has no usage recorded, so
  # /kit reset all confirm re-arms a whole season with no extra state.
  # every-join attempts a full claim on every join and lets the kit's own
  # cooldown and one-time flag decide. Anything else reads as first-join.
  mode: "first-join"
  # Kit ids handed out on join, in this order. An unknown id is skipped.
  # sn:extensible
  kits:
    - "default"
  # true gives the kit with no message and no sound.
  silent: false
```

Auto-claim runs the same claim path as everything else, so `auto-armor`, `auto-offhand`, the
inventory space check and `drop-items-when-full` all behave identically.

A denied give is always silent in both modes, so nobody is told about a cooldown on every login.
Only a success is announced, and only while `silent` is false.

{% hint style="success" %}
Season reset with `mode: first-join`: run `/kit reset all confirm` when the season starts. Every
player's usage is cleared, so the kit is handed out again on each player's next join.
{% endhint %}

### Multi-claim

Lets a player claim the same kit more than once before its cooldown is over, one extra claim per
extra copy they own.

```yaml
multi-claim:
  # false gives every player exactly one claim per cooldown and reads no
  # permission at all.
  enabled: false
  # Prefix of the amount node. A missing trailing dot is added.
  permission-prefix: "snkits.uses"
  # Ceiling applied to every resolved amount, whatever the permission says.
  # Clamped into 1..64: a claim past 64 cannot be tracked, and an untracked
  # claim cannot be counted as spent.
  max-uses: 10
```

The amount comes only from permissions, never from this file and never from the database, so a
shop sells another copy with one grant. See [Permissions](permissions.md).

Each claim carries its own cooldown, so copies recharge one by one rather than all at once. With a
one day cooldown and three copies:

| Time | Action | Left |
|------|--------|------|
| 10:00 | claim | 2 |
| 10:01 | claim | 1 |
| 10:02 | claim | 0, denied until 10:00 tomorrow |
| tomorrow 10:00 | recharge | 1 |
| tomorrow 10:01 | recharge | 2 |

One-time kits are multiplied too. Three copies is three claims ever, until a reset.

### Chat input and delete confirmation

```yaml
chat-input:
  # Word the admin types to abort a prompt.
  cancel-keyword: "cancel"
  # Seconds a prompt waits before cancelling itself.
  timeout-seconds: 60

delete-confirmation:
  # Seconds a pending /kit delete <id> stays valid. Repeating the command
  # inside the window deletes the kit; after it, the handshake starts over.
  timeout-seconds: 15
```

### Sounds

```yaml
sounds:
  # A kit was claimed.
  claim-success: "ENTITY_PLAYER_LEVELUP 1.0 1.0"
  # A claim was refused: disabled, permission, one-time, cooldown, no space, or
  # data not loaded yet.
  claim-denied: "ENTITY_VILLAGER_NO 1.0 1.0"
  # A menu click that is not a claim: navigation, preview, back.
  gui-click: "UI_BUTTON_CLICK 0.7 1.0"
  # An admin flipped a flag in the edit menu.
  toggle: "BLOCK_LEVER_CLICK 0.7 1.2"
```

Each value is `SOUND_ID [volume] [pitch]`. Set one to `none` to silence it. A sound id this server
does not know is reported once at load and then plays nothing.

## Kit files

One kit is one file, `kits/<id>.yml`. The id is the file name without the `.yml`, in lowercase
`a-z`, `0-9`, `_` and `-`. Every key below is a toggle or a field in `/kit edit`, so editing these
files by hand is optional.

| Key | Type | Effect |
|---|---|---|
| `enabled` | boolean | A disabled kit stays visible, is still previewable, and says so when clicked |
| `cooldown` | seconds | Time before the kit can be claimed again |
| `requires-permission` | boolean | Gates the kit behind `snkits.kit.<id>` |
| `one-time` | boolean | The kit can be claimed once ever, until a reset |
| `auto-armor` | boolean | Armor pieces equip themselves on claim |
| `auto-offhand` | boolean | The off-hand item equips itself on claim |
| `drop-items-when-full` | boolean | `true` drops the overflow, `false` refuses the claim |
| `offhand-slot` | slot | The grid slot holding the kit's off-hand item |
| `contents` | section | The items, one entry per grid slot |

{% hint style="warning" %}
The plugin rewrites a kit file whole every time the kit is saved. Comments you add to one are not
preserved. Edit kits with `/kit edit` where you can.
{% endhint %}

A claim never overwrites what a player is wearing or holding. If the helmet slot is taken, the
kit's helmet goes to the inventory instead.

With `drop-items-when-full: false`, a claim into an inventory without room is refused rather than
dropped on the ground, and the refusal costs the player nothing: no cooldown spent, no one-time
claim spent.

### Items

Entries under `contents` are keyed by grid slot and stored with Bukkit's own serialization, so
enchantments, custom names, lore, custom model data, leather colors, skull textures and shulker
contents all survive a round trip untouched.

An item's name and lore resolve placeholders per player, in the preview and on the item that lands
in the inventory. Both `%player_name%` style PlaceholderAPI tokens and `&` or `&#RRGGBB` colors
work.

### Command items

A command item is never given. It runs commands when the kit is claimed.

| Key | Type | Effect |
|---|---|---|
| `command` | boolean | Marks the entry as a command item |
| `display` | item | What the preview shows, or nothing at all |
| `run-as` | `console` or `player` | Who runs the commands |
| `commands` | list | The commands, without a leading slash |

Add one in the items editor with slot 53, then right-click it in the grid to configure it. A
command item with no display is invisible in the preview and still runs.

## Menu files

`guis/kits-main.yml` and `guis/kits-more.yml` ship as examples. Any other file you drop in `guis/`
becomes a menu you can open by name, and the name is the file name without the `.yml`, lowercase.

`guis/kit-preview.yml`, `guis/kit-edit.yml` and `guis/command-item.yml` are the preview and the
admin tools. They are reached from their own flows, never by name.

### Layout

A menu is a `title`, a `layout` of rows, an `items` section for decoration and navigation, and a
`templates` section for kits. Each entry claims a letter with `key`, and the letter's position in
the layout is where it is drawn.

```yaml
title: "&#8354f2&lKits"

# f filler   k the default kit   m go to page 2
layout:
  - "fffffffff"
  - "f   k  mf"
  - "fffffffff"
```

To move a button, move its letter. To remove one, delete its letter from the layout and leave the
entry in place.

{% hint style="warning" %}
Give every kit its own letter. Two kits sharing one cell means one is painted over by the other.
{% endhint %}

### The four states

Every kit shown in a menu is four entries under `templates`, all sharing one layout letter:

| Entry | When it is painted |
|---|---|
| `kit-<id>-available` | Claimable right now |
| `kit-<id>-on-cooldown` | Every copy the player owns is recharging |
| `kit-<id>-one-time-used` | Their one-time allowance is spent |
| `kit-<id>-disabled` | The kit is switched off |

The plugin paints exactly one of them per viewer, so the four never fight over the cell. Delete the
`-disabled` entry and a switched-off kit is drawn with its `-available` entry instead. Delete any
of the other three and the kit is simply not drawn while it is in that state.

To add a kit to a menu, give it a free letter in the layout, then copy the four `kit-default-*`
entries and rename them to your kit id.

### Placeholders in a menu

`{kit}`, `{cooldown}`, `{uses_left}` and `{uses_max}` resolve in `display-name` and in `lore`, in
all four states, with no PlaceholderAPI installed. PlaceholderAPI tokens resolve too, per viewer.

{% hint style="warning" %}
Use `&#RRGGBB` for hex colors. A bare `#RRGGBB` is not a color and renders as literal text.
{% endhint %}

### Click actions

On a kit cell, left-click claims and right-click previews. Shift-left claims and shift-right
previews as well. Middle, drop and hotbar clicks do nothing.

| Action | Where it goes |
|---|---|
| `[kits] claim` | Claims the kit of the cell |
| `[kits] preview` | Opens the read-only preview of the kit |
| `[kits] open <menu>` | Opens another file in `guis/`, by name |

There is no pagination. A second page is another file with buttons linking both ways, which is why
the layout of a full page is yours rather than the plugin's.

Add a `[sound] SOUND_ID` line to any click-actions list to layer another sound over the ones in
`config.yml`.

## lang/messages_en.yml

Every player-facing string lives here, including the state words the menus and placeholders use.
Change `lang` in `config.yml` to load a different `lang/messages_<code>.yml`.
