# Configuration

Four things are yours to edit:

| File | Holds |
|---|---|
| `config.yml` | Bulk claim, conditions, sounds, aliases, debug |
| `lang/messages_en.yml` | Every message the plugin sends |
| `guis/categories.yml`, `guis/vouchers.yml` | The admin menu layouts, items and click actions |
| `vouchers/*.yml` | Your vouchers, one file each |

New keys from future versions merge into your files **without touching your values or your comments**.

## Voucher files

One file per voucher, in `plugins/SnSimpleVouchers/vouchers/`.

- The **id** is the filename without `.yml`. `keys.yml` is the voucher `keys`.
- A **subfolder is a category**: `vouchers/crates/keys.yml` is in category `crates`. Exactly one level of subfolders is scanned.
- A file directly in `vouchers/` is **loose** and has no category.
- Files named `example.yml` are always skipped, even with `enabled: true`.
- `enabled` defaults to **true**: a file without the key loads. Set `enabled: false` to park one.
- A bad file skips itself with a warning naming the voucher and never takes the rest down.
- Duplicate ids across folders warn, and the last one wins.

### Full schema

```yaml
enabled: true

item:
  material: DIAMOND_SWORD        # any Material that exists as an ITEM
  display-name: "&#FFAA00&lExample Voucher"
  lore:
    - "&7Right-click to claim"
  custom-model-data: 12345
  glow: true
  color: "#FF0000"               # leather armour and potions
  trim:                          # armour only
    pattern: SENTRY
    material: GOLD

mode: WEIGHTED                   # LIST (default), WEIGHTED, RANDOM
amount: 5
auto-claim: false

commands:
  - command: "eco give {player} {amount}"
    weight: 75
    condition: "%player_level% >= 10"
  - command: "crate key give {player} rare 1"
    weight: 25
  - "broadcast &e{player} won something!"
```

A command entry can be a plain string, or a map with `command`, `weight` and `condition`.

### Item fields

| Field | Applies to |
|---|---|
| `material` | Any Material that exists as an item. `AIR`, `WATER`, `FIRE` and other block-only states are refused at load with a warning, because a voucher built from one is handed out as nothing. A name this server version does not know falls back to a placeholder instead of failing, so one file can target 1.20 and 1.21 alike |
| `display-name`, `lore` | Support `&` codes and `&#RRGGBB` HEX |
| `custom-model-data` | Integer |
| `glow` | Boolean |
| `color` | Leather armour and potions |
| `trim` | Armour only, `pattern` + `material` |

For a custom-texture head, set the material to `basehead-<base64>`.

{% hint style="info" %}
Write booleans unquoted. `enabled: "false"` and `auto-claim: "true"` are strings, not booleans; the plugin now parses them correctly and warns, but unquoted is the right form.
{% endhint %}

### `mode`

| Mode | Behaviour |
|---|---|
| `LIST` | Runs **every** eligible reward. The default |
| `WEIGHTED` | Picks **one** eligible reward, by `weight` |
| `RANDOM` | Picks **one** eligible reward, evenly |

Only entries whose `condition` passes take part. If no entry is eligible, nothing is dispatched, the voucher is **not** consumed, and the player sees `messages.claim.no-eligible-reward`.

### `amount` and bulk claim

With an `amount:`, a main-hand right-click consolidates every matching voucher in the player's main inventory into **one** command execution. Shift is not required. The offhand and the cursor slot are not scanned.

`{amount}` is the **sum** over everything the click consumed. Decimals work and are handled as exact decimals, not floating point. An amount too wide to write out (more than 64 characters) is refused at load.

Without an `amount:`, a right-click consumes exactly one voucher.

### `auto-claim`

`auto-claim: true` skips the physical item entirely. `/voucher give` and the menu's give-on-click resolve and dispatch the rewards straight to the player. Useful for triggering voucher rewards from another plugin without touching an inventory slot.

### Command placeholders

`{player}` and `{amount}` are substituted into each dispatched command. Commands run from the console.

## config.yml

```yaml
bulk:
  limit: 0                       # cap per click, 0 = unlimited
  aggregate-by-category: true

conditions:
  fail-closed: true

sounds:
  claim: ENTITY_PLAYER_LEVELUP
  gui-click: UI_BUTTON_CLICK
```

### `bulk.aggregate-by-category`

Ships **on**. When on, a bulk claim also consumes vouchers of **other ids in the same category folder**, as long as they declare the exact same reward list (same mode, commands, weights, conditions, in the same order). Their amounts are summed into one `{amount}` and the command runs once.

So `money_100` and `money_500` in one folder, both running `eco give {player} {amount}`, are aggregated: clicking either consumes both and pays the full total. Siblings whose reward list differs are left alone. Loose vouchers always match by id only.

Set it to `false` to consume only the clicked voucher's own id.

### `conditions.fail-closed`

Ships **true**. If a condition cannot be resolved - PlaceholderAPI missing, an unknown placeholder, a malformed expression - the reward is **not** eligible. Set it to `false` to let unresolvable conditions pass instead.

### Sounds

`claim` plays on a successful claim. `gui-click` plays when a menu click opens a category, returns to the root, or gives a voucher. The close and page buttons carry their own sound inside `guis/*.yml`. An empty value disables the sound.

## Conditions

Each reward entry can carry a `condition:` resolved against the claiming player at claim time.

```yaml
commands:
  - command: "crate key give {player} legendary 1"
    condition: "%player_level% >= 40"
  - command: "crate key give {player} rare 1"
    condition: "%player_level% >= 20 && %player_level% < 40"
  - command: "crate key give {player} common 1"
    condition: "%player_level% < 20"
```

- Comparators: `==` `!=` `>` `<` `>=` `<=`
- Logic: `&&` `||` `!`, with `( )` for grouping
- Operands: numbers, `"quoted strings"`, or raw placeholder output
- A condition with no comparator is read as a plain boolean
- String comparison is case-sensitive

{% hint style="warning" %}
`!` binds **looser** than the comparators: `!a == b` means `!(a == b)`. Use brackets if you want the other reading.
{% endhint %}

An unquoted operand cannot contain spaces or any of `& | ! = < > ( )`. Quote a value that might, such as a player name.

## Messages

`lang/messages_en.yml` holds every message. Five placeholders are available:

`{voucher}` `{category}` `{amount}` `{count}` `{player}`

**Blank a value to disable that message.** A missing key renders a visible marker instead, so you can tell the two apart.

Which claim message fires:

| Key | When |
|---|---|
| `messages.claim.single` | One voucher consumed |
| `messages.claim.bulk` | Several of the **same** id |
| `messages.claim.bulk-category` | Several **different** ids from one folder |

Do not write `{prefix}` in any value: the prefix is prepended automatically.
