# Commands

Root command: `/crates`, aliases `/crate` and `/snc`.

| Command | Description | Permission |
|---|---|---|
| `/crates` | Opens your key balance menu | `sncrates.use` |
| `/crates open <crate> [player]` | Opens a crate, spending a key | `sncrates.admin.open` |
| `/crates preview <crate> [player]` | Shows a crate's rewards, read-only | `sncrates.admin.preview` |
| `/crates balance <player>` | Shows another player's key balances, on your screen | `sncrates.admin.balance` |
| `/crates editor` | Opens the crate editor | `sncrates.admin.editor` |
| `/crates key give <player> <crate> [amount] [-s] [-sf]` | Adds virtual keys | `sncrates.admin.keys` |
| `/crates key giveall <crate> [amount] [-s] [-sf]` | Adds virtual keys to every online player | `sncrates.admin.keys` |
| `/crates key take <player> <crate> [amount] [-s] [-sf]` | Removes virtual keys | `sncrates.admin.keys` |
| `/crates key set <player> <crate> <amount> [-s] [-sf]` | Sets a balance to an exact number | `sncrates.admin.keys` |
| `/crates key wipe [crate] [confirm]` | Deletes **every** player's virtual key balances | `sncrates.admin.keys` + `sncrates.admin.wipekeys` |
| `/crates givekey <player> <crate> [amount] [-s] [-sf]` | Hands over the physical key item | `sncrates.admin.givekey` |
| `/crates reload` | Reloads config, language, menus and crates | `sncrates.admin.reload` |
| `/crates debug` | Toggles runtime debug output | `sncrates.admin.debug` |
| `/crates help [page]` | Paginated help, filtered by what you can run | none |

`reload`, `help` and `debug` are injected by SnLib and behave identically in every Sn plugin.

Extra aliases are read from `command.aliases` in `config.yml` (shipped as `[crate, snc]`) and are
re-read on `/crates reload`, so an alias you add works without a restart. `crate` and `snc` are also
declared in the plugin's `plugin.yml`, where Bukkit owns them, so emptying that list does not
unregister them.

Every `<player>` argument is an **online, exact** player name. An offline or misspelled name is
rejected with `Player not found`.

The crate slot tab-completes the loaded crate ids but accepts anything, so an unknown id answers
`Crate <what you typed> does not exist.` rather than a generic invalid-value error.

{% hint style="info" %}
The root carries no permission of its own, on purpose: a bare `/crates` is a player-facing command,
and gating the root would hide the whole tree - help included - from anyone without that node. Each
subcommand carries its own, and help lists only the ones the sender can actually run.
{% endhint %}

## `/crates` - the key balance menu

For a player, a bare `/crates` opens their key balance menu: one icon per crate they can hold a
balance in. It needs `sncrates.use`, which defaults to `true`.

Inside the menu:

| Click | Does |
|---|---|
| Left-click a crate | Opens one crate |
| Right-click a crate | Opens that crate's preview |
| **Q** (drop) on a crate | Mass-opens, up to `mass-open-max` |

A crate that accepts **only** physical keys has no virtual balance, so it is not shown here at all -
and it is skipped before a slot is consumed, so the layout never develops a hole.

Pressing Q on a crate whose mass-open is off falls back to a single animated open rather than doing
nothing.

A mass open is spread over ticks (`access.mass-open-per-tick` in `config.yml`), so a large balance
never runs every reward command in one tick. The menu **stays open** while the batch drains and the
balance on the icon updates as it goes. When it ends, one summary lists what came out:

```
Crates | Opened 10x Example Key:
   6x 16x Diamonds
   3x Legendary Rank
   1x failed - those keys were spent
```

The last line appears only when the crate has a [fail chance](configuration.md#fail). A second Q
within `access.mass-open-cooldown-ticks` is ignored, silently unless you give
`messages.open.mass-cooldown` a value.

From the console, a bare `/crates` renders the help page.

## `/crates open <crate> [player]`

Opens the crate, exactly as clicking a crate block does. With no player named it opens for you; from
the console the player argument is required.

{% hint style="warning" %}
This command does **not** bypass the key check. A target with no key is refused and a successful
open still costs one key. It is a convenience for admins, not a free open. To hand somebody a free
open, give them a key first.
{% endhint %}

The open runs through the same path a crate block does, so limits, filters, the animation and the
exactly-once delivery guarantee all apply. The only difference is that the event's source block is
`null`, because a command has none.

## `/crates preview <crate> [player]`

Opens the read-only preview. It costs nothing, needs no key and spends nothing. With a player named
it opens on their screen.

## `/crates balance <player>`

Renders the **target's** balances on **your** screen. The target is not disturbed and receives no
message; the menu title is what names them.

The icons in an inspection view are inert: each carries the subject's UUID and a click that does not
match the clicker is ignored, so an inspecting admin cannot spend the keys they are looking at.

Console cannot run it - the menu has to open on somebody's screen.

## `/crates editor`

Opens the crate editor. It takes no arguments. See [Configuration](configuration.md#the-in-game-editor).

## The key commands

All five sit under `/crates key` and share `sncrates.admin.keys`. `wipe` needs a second
permission on top of it - see [below](#crates-key-wipe-crate-confirm).

```
/crates key give <player> <crate> [amount]
/crates key giveall <crate> [amount]
/crates key take <player> <crate> [amount]
/crates key set <player> <crate> <amount>
/crates key wipe [crate] [confirm]
```

`[amount]` is **optional and defaults to 1** for `give`, `giveall` and `take`. `set` requires it,
and `set` is the only one that accepts `0`, because setting a balance to nothing is how you clear
it. `wipe` takes no amount at all - it clears balances outright, for everybody.

| Command | Executor is told | Target is told |
|---|---|---|
| `give` | `Gave Nx <key> key(s) to <player>.` | `You received Nx <key> key(s).` |
| `giveall` | `Gave Nx <key> key(s) to N player(s).` | each: `You received ...` |
| `take` | `Took Nx <key> key(s) from <player>.` | `You received Nx <key> key(s).` |
| `set` | `Set <player>'s <key> keys to N.` | `You received Nx <key> key(s).` |

{% hint style="warning" %}
`take` and `set` send the target the **received** line even when keys were removed. That is
long-standing behaviour, kept deliberately. Use `-s` to suppress the target's message when it would
be confusing.
{% endhint %}

### Crates with no virtual balance

A crate that accepts `PHYSICAL` but not `VIRTUAL` has no balance to operate on:

- `give` **falls back to handing over the physical key item** and reports it as such.
- `take` and `set` are **refused**, with `Crate <id> uses a physical key item - it has no virtual
  balance to change.` They do not silently succeed. Physical keys live in inventories, so they are
  removed by hand.

A crate that accepts `PERMISSION` but not `PHYSICAL` is not affected: all of them work on it.

`wipe` is the exception to this whole section - it is **never** refused for having no virtual
balance. It only removes stored numbers, and the rows most worth removing belong to exactly such a
crate: the leftovers of one that used to accept virtual keys and was later switched to a physical
key item.

### The two flags

| Flag | Suppresses |
|---|---|
| `-s` | the line sent to the **receiver** |
| `-sf` | the line sent to the **executor** |

They are independent, can be combined, and work on `give`, `giveall`, `take`, `set` and `givekey`.
They go after the amount:

```
/crates key giveall vote 1 -s
/crates key give Notch legendary 5 -s -sf
```

`wipe` takes neither flag: it notifies nobody but the admin who ran it, and there is no single
target to stay quiet about.

### `/crates key wipe [crate] [confirm]`

Deletes the virtual key balances of **every player on the server**, offline players included.

```
/crates key wipe            # every crate
/crates key wipe common     # only the crate 'common'
```

Physical key items are **not** touched. They live in inventories and ender chests across the
server, and no balance command reaches them - same as `take`.

#### It runs in two halves

There is no undo, so the first invocation never destroys anything. Without the confirmation word
the command only **counts**: it reports how many balances and how many keys are at stake, and
quotes back the exact line that would delete them.

```
/crates key wipe
> WARNING This deletes 4,120 virtual key(s) across 1,284 balance(s), for EVERY player and
> EVERY crate. It cannot be undone. Run /crates key wipe confirm to go through with it.

/crates key wipe confirm
> Wiped 1,284 virtual key balance(s), for every player and every crate.
```

{% hint style="warning" %}
The confirmation word is **never tab-completed**. Having to type it is the point: an admin who
could tab their way from `/crates key wipe` to a confirmed server-wide wipe has the same
protection as no confirmation at all. The warning line is the only place the word is offered.
{% endhint %}

The word itself is `messages.keys.wipe-confirm-word` in your language file (default `confirm`),
so it translates with everything else. The warning quotes it from there, so the two can never
disagree.

#### What it protects you from

- **A mistyped crate id is refused**, not widened. `/crates key wipe commmon confirm` reports
  `Crate commmon does not exist.` and deletes nothing - it does not fall through to wiping the
  whole server.
- **An empty scope is reported, not armed.** If nothing is stored you are told so instead of being
  asked to confirm a delete of zero rows. It is usually how you find out you named the wrong crate.
- **Online players are cleared too.** Their cached balances go with the stored ones, so nobody
  keeps opening crates against keys that no longer exist anywhere.
- The count in the warning and the count in the result can legitimately differ: keys handed out in
  between are deleted too. Both numbers were true when they were printed.

#### Permission

`wipe` needs `sncrates.admin.wipekeys` **on top of** the group's `sncrates.admin.keys`. Both
default to op and both are children of `sncrates.admin`, so granting the parent still grants
everything - but a moderator given only `sncrates.admin.keys` can run the other four key commands
and **not** this one. See [Permissions](permissions.md).

## `/crates givekey <player> <crate> [amount]`

Hands over the **physical key item**, not a balance. `[amount]` defaults to 1.

Delivery is fit-aware and never drops anything on the ground:

- A target whose inventory is completely full is refused with `<player>'s inventory is full.` -
  naming the target, not you.
- A target with partial room receives only what fits, and only that much is counted as given.

`giveall` behaves differently on purpose: a player whose inventory is full is **skipped**, the run
does not abort, and the summary reports how many players were actually reached.

## `/crates reload`

Re-reads `config.yml`, `lang/messages_en.yml`, every file under `guis/` and every crate file under
`crates/`, and re-sources the command aliases from config.

It does **not** re-register `command.user-open-aliases`. Those are top-level commands built while
the server starts; adding or removing one needs a restart.

## `/crates debug`

Toggles SnLib's runtime debug output for this plugin, matching `debug.enabled` in `config.yml`. Use
it to see which crate an open resolved to, which reward was pre-computed, and why a withdraw ended
the way it did.

## User-open aliases

`command.user-open-aliases` in `config.yml` registers extra **standalone** commands that do exactly
what a bare `/crates` does for a player, and expose nothing else:

```yaml
command:
  user-open-aliases: [llaves, keys]
```

`/llaves` opens the key balance menu and requires `sncrates.use`. `/llaves editor`,
`/llaves key giveall` and `/llaves reload` do not exist.

{% hint style="info" %}
This is not `command.aliases`. An alias there mirrors the **whole** `/crates` tree, admin
subcommands included, so putting `llaves` in that list would put `/llaves key giveall` on your
server. That is why they are two settings and not one.
{% endhint %}

A name already taken - the root's own name, one of the full aliases, or a duplicate inside the list
- is skipped silently. Bukkit would keep the first registration anyway.
