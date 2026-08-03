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

## `/voucher open`

Opens the root menu: every category folder first, then every loose voucher. Clicking a category opens that folder's menu; clicking a voucher gives it to you.

Clicking a voucher in the menu requires `snsimplevouchers.admin.give`, the same node the command requires. Opening the menu and handing out of it are separate entitlements.

The menus are defined in `guis/categories.yml` and `guis/vouchers.yml` and are yours to edit: layout, items, click actions, sounds and pagination.

## `/voucher reload`

Re-reads `config.yml`, `lang/messages_en.yml`, both menu files and every file under `vouchers/`. Deleting a voucher file and reloading really unloads it.

Online staff holding `snsimplevouchers.admin.reload` receive the new voucher count. The console additionally logs `Loaded N vouchers (M skipped).`, which carries the skipped count.
