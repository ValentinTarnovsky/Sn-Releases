# Configuration

Four things are yours to edit:

| File | Holds |
|---|---|
| `config.yml` | Bulk claim, giving, conditions, sounds, aliases, debug |
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
multi-claim: false               # mass claim for vouchers with no amount

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

Without an `amount:`, a right-click consumes exactly one voucher - unless the voucher sets `multi-claim`.

### `multi-claim` - mass claim for crates

`amount` can bulk-claim because it is a **multiplier**: 64 vouchers worth 100 each pay the same as one command carrying 6400. A crate has nothing to multiply. `pets admin givebox {player} Box_Basica 1` is not worth 64 times more written once, and a `RANDOM` crate has to draw a separate prize per copy or one click hands out 64 copies of the same thing.

So a crate cannot use `amount`, and without this key a player holding 200 of them right-clicks 200 times.

`multi-claim: true` makes **one** main-hand right-click consume the whole stack and roll the rewards **once per voucher consumed**. 200 vouchers, 200 independent draws, one click. Shift is not required.

```yaml
mode: RANDOM
multi-claim: true
commands:
  - "pets admin givebox {player} Box_Basica 1 -sf"
  - "gens addmax {player} 1"
  - "booster give {player} money 0.5 10m"
```

| Detail | Behaviour |
|---|---|
| Cap | `bulk.multi-limit` in `config.yml`, ships at **128**. Vouchers above the cap stay in the inventory for the next click. The same cap bounds `/voucher give` and `/voucher giveall` on an auto-claim crate |
| `{amount}` in the commands | Always `1` - each execution is worth one voucher |
| `{amount}` in the message | The number of vouchers the click consumed |
| Tick cost | The rewards do **not** all fire in one tick. `bulk.commands-per-tick` spreads the overflow over the following ticks |
| Aggregation | `bulk.aggregate-by-category` applies, and only ever matches other `multi-claim` vouchers with an identical reward list. A folder holding both crates and `amount` vouchers keeps the two apart |
| With `amount` | Contradictory. `amount` wins, and the console says so at load. Use one or the other |
| With `auto-claim` | `/voucher give <player> <id> 10` becomes 10 draws instead of one, matching what claiming 10 items does. Clamped to `bulk.multi-limit`; the sender's message reports the number actually handed over |

Claim messages for this shape are `messages.claim.multi` and `messages.claim.multi-category`, separate from the `bulk` pair because a crate has no summed amount to report.

### `auto-claim`

`auto-claim: true` skips the physical item entirely. `/voucher give` and the menu's give-on-click resolve and dispatch the rewards straight to the player. Useful for triggering voucher rewards from another plugin without touching an inventory slot.

### Command placeholders

`{player}` and `{amount}` are substituted into each dispatched command. Commands run from the console.

## config.yml

```yaml
bulk:
  limit: 0                       # cap per click for 'amount' vouchers, 0 = unlimited
  multi-limit: 128               # cap per click for 'multi-claim' vouchers
  commands-per-tick: 20          # dispatch budget per tick, 0 = no spreading
  aggregate-by-category: true

give:
  drop-overflow: true

conditions:
  fail-closed: true

sounds:
  claim: ENTITY_PLAYER_LEVELUP
  gui-click: UI_BUTTON_CLICK
```

### `bulk.multi-limit`

Ships at **128**. Caps how many vouchers one click consumes on a `multi-claim` voucher. Unlike `bulk.limit`, this is not the number of items folded into a single execution - it is the number of **executions** one click queues, so it is the knob that decides what a mass claim costs your server. Raise it for 256-crate claims, lower it if a click is doing too much at once.

On a `mode: LIST` crate every roll runs the whole command list, so the real command count is this number times the length of that list. Keep the cap lower for those.

It also bounds `/voucher give` and `/voucher giveall` on an auto-claim crate, so no command argument can queue an unbounded number of rewards.

`0` means unlimited. A full inventory is 2304 vouchers.

### `bulk.commands-per-tick`

Ships at **20**. The dispatch budget **per claim**, per tick. Everything up to this many reward commands runs immediately inside the click; the overflow of a big `multi-claim` is spread over the following ticks, so 128 crate rewards never land in a single tick. At 20/tick that worst case is under half a second and the player just watches the prizes arrive.

It is per claim, not server-wide: one player mass-claiming crates never puts anybody else's ordinary claim behind their queue.

Only a `multi-claim` can ever exceed this - every other claim dispatches at most one command per reward entry - so on a server without one this key changes nothing. `0` turns the spreading off.

A queued reward is dispatched by player **name**, so a player who logs out while their own overflow is still draining can miss the tail of it. Do not set this so low that a claim takes many seconds to deliver.

Anything still queued when the server stops is dispatched during shutdown rather than dropped: those vouchers were consumed already.

### `bulk.aggregate-by-category`

Ships **on**. When on, a bulk claim also consumes vouchers of **other ids in the same category folder**, as long as they declare the exact same reward list (same mode, commands, weights, conditions, in the same order). Their amounts are summed into one `{amount}` and the command runs once.

So `money_100` and `money_500` in one folder, both running `eco give {player} {amount}`, are aggregated: clicking either consumes both and pays the full total. Siblings whose reward list differs are left alone. Loose vouchers always match by id only.

Set it to `false` to consume only the clicked voucher's own id.

### `give.drop-overflow`

Ships **true**. Decides what happens when the voucher items do not fit in the receiver's inventory.

| Value | Behaviour |
|---|---|
| `true` | The receiver keeps what fits and the rest drops at their feet. Nothing is ever lost |
| `false` | All-or-nothing. A receiver without room for the **whole** stack gets nothing, and both sides are told |

A partial delivery never happens either way, so nobody is quietly handed 4 of the 10 vouchers they were promised.

The key covers all three ways a voucher item reaches a player: `/voucher give`, `/voucher giveall` and the give-on-click of `/voucher open`. `auto-claim` vouchers are unaffected, since they dispatch rewards and never produce an item that could overflow.

{% hint style="info" %}
`false` is the setting to use with `/voucher giveall` when handing a voucher to a full server: nobody ends up with vouchers scattered on the floor of wherever they happened to be standing. The summary line tells you how many players were skipped so you can follow up.
{% endhint %}

The receiver sees `messages.inventory-full` and the giver sees `messages.give.inventory-full`.

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
| `messages.claim.bulk` | Several of the **same** id, consolidated by `amount` |
| `messages.claim.bulk-category` | Several **different** ids from one folder, consolidated by `amount` |
| `messages.claim.multi` | Several of the **same** id on a `multi-claim` voucher |
| `messages.claim.multi-category` | Several **different** ids from one folder on a `multi-claim` voucher |

Do not write `{prefix}` in any value: the prefix is prepended automatically.
