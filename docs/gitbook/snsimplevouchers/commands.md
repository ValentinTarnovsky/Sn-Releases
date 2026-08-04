# Commands

The root command is `/voucher`, with `/vouchers` and `/sv` as default aliases. The alias list lives in `config.yml` under `command.aliases` and is authoritative once set.

{% hint style="warning" %}
`/voucher` itself requires `snsimplevouchers.admin`. Every subcommand is staff-facing, so a player without that node loses access to the command entirely, including its help. See [Permissions](permissions.md).
{% endhint %}

| Command | Permission | Description |
|---|---|---|
| `/voucher help [page]` | `snsimplevouchers.admin` | Paginated, permission-filtered help, rendered under whichever alias you typed |
| `/voucher reload` | `snsimplevouchers.admin.reload` | Reloads config, language, menus and rescans `vouchers/` |
| `/voucher debug` | `snsimplevouchers.admin.debug` | Toggles debug logging |
| `/voucher open` | `snsimplevouchers.admin.open` | Opens the paginated admin menu |
| `/voucher give <player> <voucher> [amount] [-s]` | `snsimplevouchers.admin.give` | Gives a voucher to a player |
| `/voucher giveall <voucher> [amount] [-s]` | `snsimplevouchers.admin.giveall` | Gives a voucher to every online player |

`help`, `reload` and `debug` are provided by SnLib and behave the same across every Sn plugin.

## `/voucher give`

```
/voucher give <player> <voucher> [amount] [-s]
```

`[amount]` and `-s` are read in **either order**, and the last numeric token wins. All of these are the same invocation:

```
/voucher give Steve keys 5 -s
/voucher give Steve keys -s 5
```

`[amount]` floors at 1 and has no upper bound.

### `-s` (silent)

Suppresses the sender message, the receiver message **and** the no-eligible-reward notice. It exists so other plugins and command blocks can call `/voucher give` from their own reward systems without double-messaging the player.

### What `[amount]` means

This differs by voucher type, on purpose:

| Voucher | `[amount]` means |
|---|---|
| Ordinary | How many physical items to hand over |
| `auto-claim: true` | A **claim multiplier**: the resolved `{amount}` becomes `voucher.amount * n`, dispatched **once** |

The admin menu's give-on-click always uses a multiplier of 1.

### Unknown voucher

An unknown id reaches the handler and answers with `messages.voucher-not-found`, naming the id you typed, rather than a generic rejection. Tab completion suggests loaded ids but does not restrict what you can type.

### What happens when the inventory is full

By default the overflow drops at the receiver's feet, so nothing is lost. Set `give.drop-overflow: false` in `config.yml` and the give becomes all-or-nothing instead: a receiver without room for the whole stack gets nothing, and both sides are told. See [Configuration](configuration.md).

## `/voucher giveall`

```
/voucher giveall <voucher> [amount] [-s]
```

The same give, run once per online player. The grammar is `/voucher give`'s minus the target, so the trailing `[amount] [-s]` pair is read in either order here too.

{% hint style="info" %}
`[amount]` is the amount **per player**, not a pool split between them. `/voucher giveall keys 3` with 20 players online hands out 60 vouchers.
{% endhint %}

An `auto-claim` voucher dispatches its rewards to every online player instead of minting items, exactly as `/voucher give` does for one.

### The pass never aborts part way

A player who cannot be served is counted and skipped, and the rest of the server is still served. Two things can cause a skip:

| Situation | Result |
|---|---|
| No inventory room, with `give.drop-overflow: false` | That player receives nothing; everyone else is unaffected |
| An `auto-claim` voucher whose rewards are all ineligible for that player | That player gets no reward; everyone else is unaffected |

When the pass is over you get `messages.giveall.sender` naming how many players received, plus a second line naming how many were skipped when any were. The skipped line differs by cause, so you always know whether it was inventory space or a failed `condition:`.

`-s` suppresses everything: the receivers' messages and your own summary.

### Nobody online

`/voucher giveall` answers `messages.giveall.no-players` and does nothing.

## `/voucher open`

Opens the root menu: every category folder first, then every loose voucher. Clicking a category opens that folder's menu; clicking a voucher gives it to you.

Clicking a voucher in the menu requires `snsimplevouchers.admin.give`, the same node the command requires. Opening the menu and handing out of it are separate entitlements.

The menus are defined in `guis/categories.yml` and `guis/vouchers.yml` and are yours to edit: layout, items, click actions, sounds and pagination.

## `/voucher reload`

Re-reads `config.yml`, `lang/messages_en.yml`, both menu files and every file under `vouchers/`. Deleting a voucher file and reloading really unloads it.

Online staff holding `snsimplevouchers.admin.reload` receive the new voucher count. The console additionally logs `Loaded N vouchers (M skipped).`, which carries the skipped count.
