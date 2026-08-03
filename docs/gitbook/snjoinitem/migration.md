# Migrating from 1.x

SnJoinItem 2.0.0 was rebuilt from scratch on SnLib. The plugin does the same job it did in
1.0.7 - one configurable item, given on join, pinned to a slot, locked, running a command when
clicked - but every file it reads changed shape, and the item itself is now described in a file
that did not exist before.

Nothing migrates itself. Plan for ten minutes of re-typing values, not for a drop-in jar swap.

**Back up `plugins/SnJoinItem/` before you start.**

## 1. SnLib is required

`SnLib.jar` must be in `plugins/`. SnJoinItem declares `depend: [SnLib]` and simply will not
load without it. Updating SnLib itself always needs a full restart, never a `/reload`.

## 2. A licence key is required

SnJoinItem joined the licensed bundle. On first boot it creates
`plugins/.Sn-License/license.yml` with a placeholder line; paste your key there and restart.
The file is **shared** by every bundle plugin, so a server already running one needs nothing
new. Without a valid key the console prints `[Sn] License: FAIL` and the plugin disables
itself.

## 3. Start from a fresh `config.yml`

{% hint style="danger" %}
Do not leave your 1.x `config.yml` in place and let the new jar merge into it. Move it out of
`plugins/SnJoinItem/` first and let 2.0.0 write a clean one.
{% endhint %}

SnLib merges managed files instead of replacing them: missing keys are inserted, your values
and comments are kept. That is the right behaviour between 2.x versions, and the wrong one
here, because two 1.x keys changed from a single value into a section:

| 1.x | 2.0.0 |
|---|---|
| `debug: false` | `debug:` with `enabled`, `level`, `categories` |
| `command: "[player] menu"` | `command:` with `aliases` |

The merge inserts the new sub-keys indented under a line that already holds a value, and the
file stops being valid YAML. SnLib catches that on the following start-up, moves the broken
file to `config.yml.backup-1` and regenerates a clean one from the jar. Nothing of yours is
destroyed, but nothing is carried over either - you just spend two restarts arriving where
moving the file aside puts you immediately.

Everything else in 1.x `config.yml` was regrouped:

| 1.x key | 2.0.0 |
|---|---|
| `config-version` | retired, see below |
| `give-on-join` | `delivery.give-on-join` |
| `give-delay-ticks` | `delivery.give-delay-ticks` |
| `restore-interval-ticks` | `delivery.restore-interval-ticks` |
| `worlds` | `delivery.worlds` |
| `item.slot` | `placement.slot` |
| `item.*` (everything else) | moved to `items.yml`, see section 4 |
| `command` | moved to `items.yml` |
| `command-cooldown-ms` | moved to `items.yml`, and now in **ticks** |
| `debug` | `debug.enabled` |
| - | `lang` **(new)** - selects `lang/messages_<code>.yml` |
| - | `update-configs` **(new)** - master gate for the key merge above |
| - | `command.aliases` **(new)** - extra aliases for `/joinitem` |

### `config-version` is gone

1.x used a version number to decide when your file was outdated, and an outdated file was
renamed to `old-config.yml` and regenerated. That policy is retired. There is **no
`config-version` key anywhere** in 2.0.0 - delete the ones your old files carry, and never add
one. New keys are merged in on every boot instead, with a timestamped `old-config-*.yml`
backup taken before each merge (the last three are kept). Set `update-configs: false` to
freeze your files and have SnLib only warn about keys it would have added.

## 4. The item moved to `items.yml`

The whole `item:` block and the click command left `config.yml`. They now live in `items.yml`,
under a section called `join-item`.

{% hint style="warning" %}
Do not rename the `join-item` section. The id is referenced from the plugin's code, and
because the file is merged from the jar on every boot, a renamed section does not replace the
original - the default one is re-inserted beside yours and the plugin keeps using that.
{% endhint %}

| 1.x `config.yml` | 2.0.0 `items.yml` |
|---|---|
| `item.material` | `material` |
| `item.name` | `display-name` |
| `item.lore` | `lore` |
| `item.model-data: 0` | `custom-model-data`, **omit the key** for none |
| `item.glow` | `glow` |
| `item.hide-flags: true` | `flags:` - an explicit list of the flags to hide |
| `item.unbreakable` | `unbreakable` |
| `command: "[player] menu"` | `right-click-actions` and `left-click-actions`, one action per line |
| `command-cooldown-ms: 200` | `cooldown: 4` - **ticks**, not milliseconds |
| - | `locked` **(new)** - the lock is now a setting, and must stay `true` |
| - | `placeable` **(new)** |

Four of those rows bite in practice:

- **`custom-model-data` has no "none" value.** SnLib gates on the key being present, not on
  its value, so `custom-model-data: 0` stamps a real model data of 0 onto the item. If you had
  `model-data: 0` in 1.x, which meant "none", do not copy it across - leave the key out. It
  ships commented out for exactly this reason.
- **`cooldown` is in ticks.** Divide your old millisecond value by 50: the 1.x default of
  `200` is `4` ticks, which is what 2.0.0 ships.
- **`material` must be a real item.** 1.x fell back to `COMPASS` with a warning when the
  material was not an item. 2.0.0 refuses instead: `AIR`, and block-only materials such as
  `POTATOES` (the crop, rather than `POTATO`), disable the item entirely and say so in the
  console on boot. Fix the material and run `/joinitem reload`.
- **`locked: true` is load-bearing.** With `locked: false` a player can drop or store the
  item, and the restore sweep then mints a replacement every interval, which is an item
  duplicator. The plugin warns on boot if you combine `locked: false` with a restore interval
  above 0.

You also gain optional keys 1.x had no equivalent for: `shift-right-click-actions`,
`shift-left-click-actions`, `interact-requirements` and `deny-actions`.

### Declare both click lists

1.x had one `command:` line that ran on every use-click. 2.0.0 has one list per side. Declare
**both**, even if they run the same thing, which is what the shipped file does.

Declaring only one side leaves the two click surfaces disagreeing: a click inside an open
inventory falls back to the other side's list (kept deliberately, so a 1.x-shaped
configuration still behaves), while a click in the world runs nothing at all and logs nothing.

## 5. `%player%` no longer works everywhere

In 1.x, `%player%` was substituted by the plugin on both click surfaces. In 2.0.0 clicks in
the world are dispatched by SnLib, which does not bind it, so:

| Where the click happens | `%player%` |
|---|---|
| Inside an open inventory | resolves |
| In the world, item in hand | does **not** resolve, the token is dispatched literally |

If an action has to work from both, use `%player_name%` instead. That needs PlaceholderAPI
installed.

## 6. `snjoinitem.bypass` is narrower

In 1.x this node did two things: the holder never received the item, **and** every lock check
let them through, so staff could move and drop it freely.

In 2.0.0 it means only the first: **the player does not receive the item**. The lock is
enforced by SnLib across all its vectors with no per-player permission awareness, so it is
uniform for everyone actually holding the item, staff included.

The practical consequence: to take the item off someone, `/joinitem disable <player>` is now
the only way. It removes every copy they hold, which is the point - the item is locked, so
they cannot remove it themselves.

The node is still `default: false`, so nobody has it unless you grant it.

## 7. `give-delay-ticks: 0` now means the next tick

In 1.x, `0` gave the item inside the join event itself. In 2.0.0 the floor is one tick: the
item is never handed out inside `PlayerJoinEvent`, because the lobby, login and
inventory-restore plugins that run right after it are precisely what wipes an item given that
early.

If your item does not appear on join, the fix is unchanged: raise the delay, try `40` or `80`.

## 8. Items already in players' inventories are not recognised

1.x tagged every copy with its own persistent marker. 2.0.0 tags items through SnLib, under a
different key. A copy minted by 1.x is therefore an ordinary item to 2.0.0: not locked, not
counted, and clicking it runs nothing.

What players see after the update:

- On join they get a fresh, real copy in the pinned slot.
- If their old copy was sitting in that slot, it is **relocated to a free slot**, not
  destroyed - 2.0.0 never overwrites an item it does not own.
- So a player can briefly hold two identical-looking items. The old one is now droppable, and
  disappears the moment they drop it.

If you would rather not explain that to anyone, clear inventories once after the update
(`/clear`, or Essentials `/ci`). The restore sweep puts the real item back within
`delivery.restore-interval-ticks`.

## 9. `messages.yml` became `lang/messages_en.yml`

The old `plugins/SnJoinItem/messages.yml` is not read by anything in 2.0.0. Nothing deletes
it; keep it open beside the new file while you re-apply your wording, then delete it.

| 1.x `messages.yml` | 2.0.0 `lang/messages_en.yml` |
|---|---|
| `prefix` | `prefix` - same key, same meaning |
| `errors.no-permission` | `snlib.no-permission` |
| `errors.player-not-found` | `snlib.player-not-found` |
| `errors.specify-player` | `messages.give-console-needs-player` |
| `commands.unknown-subcommand` | `snlib.unknown-subcommand` |
| `commands.reload-success` | `snlib.reload-done` |
| `commands.give-success` | `messages.give-success` - now takes `{player}` |
| `commands.disable-success` / `disable-already` | `messages.disable-success` / `messages.disable-already` |
| `commands.enable-success` / `enable-already` | `messages.enable-success` / `messages.enable-already` |
| `commands.help-header`, `help-admin-header`, `help-reload`, `help-give`, `help-disable`, `help-enable` | gone - help is generated, see below |
| `config-version` | retired |

Two rules when you copy your lines across:

- **Drop every `{prefix}` token.** The prefix is prepended automatically now; a line that
  still contains `{prefix}` prints it literally.
- **Leave the `snlib:` block styled as shipped** unless you restyle your whole fleet. It is
  SnLib's shared command contract, used by every Sn plugin, and any key you delete falls back
  to an unbranded default.

Your hand-written help lines are gone: `/joinitem help` is generated from the command tree,
paginated, and filtered by permission, so it only ever lists what the sender can actually run.

Messages that did not exist in 1.x and are worth reading: `messages.give-failed`,
`messages.enable-pending`, `messages.disable-target` and `messages.enable-target` - the last
two are sent to the affected player, not to the admin.

## What does not change

- **Every command works exactly as before.** `/joinitem`, the `jit` alias, `give [player]`,
  `disable <player>`, `enable <player>`, `reload` and `help`. Your macros, signs and command
  blocks keep working. (There is still no `/joinitem update` subcommand.)
- **The permission nodes keep their names**: `snjoinitem.admin` and its `give`, `toggle` and
  `reload` children, plus `snjoinitem.bypass`. Nothing to re-grant in LuckPerms. One node is
  new, `snjoinitem.admin.update`, which is notify-only: it controls who sees the update notice
  on join, not a command.
- **Your opt-outs survive.** `data.yml` keeps the same `opted-out:` section keyed by UUID, and
  it is deliberately the one file SnLib never seeds, merges or regenerates. Leave it where it
  is and every `/joinitem disable` you have ever run still holds.
- **The placeholders are unchanged.** `%snjoinitem_enabled%` and `%snjoinitem_disabled%`,
  still resolved by UUID, still returning the literal strings `true` and `false`, still
  reflecting the opt-out list only.
- **The shipped defaults are the same item**: a compass named Menu in hotbar slot 4, both
  clicks running `/menu` as the player.

## Checklist

1. Back up `plugins/SnJoinItem/`.
2. Install `SnLib.jar` and put your key in `plugins/.Sn-License/license.yml`.
3. Move `config.yml` and `messages.yml` **out** of `plugins/SnJoinItem/`. Leave `data.yml`
   exactly where it is.
4. Drop in the new jar and start the server. Fresh `config.yml`, `items.yml` and
   `lang/messages_en.yml` are written.
5. Re-apply your delivery and placement values under `delivery:` and `placement:`.
6. Re-describe your item in `items.yml`: material, `display-name`, lore, flags. Convert the
   cooldown from milliseconds to ticks, and leave `custom-model-data` out unless you need it.
7. Split your old `command:` line into `right-click-actions` **and** `left-click-actions`, and
   swap any `%player%` an in-world click has to resolve for `%player_name%`.
8. Re-apply your wording in `lang/messages_en.yml`, dropping every `{prefix}`.
9. Run `/joinitem reload` and check the console: a bad material or an unlocked item is
   reported there, not in chat.
10. Join with a test account and confirm the item lands in the right slot and both clicks work.
