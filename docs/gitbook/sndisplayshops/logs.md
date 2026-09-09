# Trade log

Every stock movement is written to `plugins/SnDisplayShops/logs/<date>.log`, one file per day, and
read back in game with `/dshop logs`.

This is the page to reach for when a player says their stock disappeared.

## What gets recorded

| Action | What it is |
|---|---|
| `BUY` | A player bought from a shop. Stock went **down**. |
| `SELL` | A player sold to a shop. Stock went **up**. |
| `DEPOSIT` | The owner put stock in - shift-click, or the `+` button. |
| `WITHDRAW` | The owner took stock out - a grid cell, or the withdraw button. |
| `PICKUP` | The shop was picked up and its whole stock handed back. |
| `DESTROY` | A shop was removed while it still held stock, and the stock was destroyed with it. |
| `EXTERNAL` | Another plugin took stock through the developer API - a sell wand, for instance. |

{% hint style="warning" %}
`BUY` and `SELL` name the **player's** side, not the shop's. A shop set to `BUY` mode buys from
players, so it produces `SELL` lines. That is the same wording every message a buyer reads uses,
and reading it the other way round makes an investigation come out backwards.
{% endhint %}

`DESTROY` is the one to know about. Only the SuperiorSkyblock teardowns produce it - an island
disbanded, a member kicked, banned, or quitting - because they are the only paths that remove a
shop without handing its stock back first. Both are switches under
`integrations.superiorskyblock` in `config.yml`.

`EXTERNAL` carries no player. The API gives the plugin no way to say who swung the wand.

## Reading it in game

```
/dshop logs [filters...]
```

Needs `sndisplayshops.admin.logs` (default op). With no filters it shows today.

Filters are `key:value` tokens in any order - press <kbd>TAB</kbd> for the list.

```
/dshop logs days:7 action:withdraw,pickup,destroy player:Steve
/dshop logs shop:2d9e near:100 sort:oldest
/dshop logs item:"Espada de fuego" days:30 stats
/dshop logs min-unit:1 max-unit:1 days:30
```

### Every filter

| Filter | What it takes |
|---|---|
| `action:` | One or more of `buy,sell,deposit,withdraw,pickup,destroy,external`, comma separated. `all` clears it. |
| `player:` | The actor's name **or** uuid. |
| `owner:` | The shop owner's name **or** uuid. |
| `shop:` | A shop uuid, or just the start of one - eight characters is plenty. |
| `world:` | World name. |
| `item:` | Any part of the item's display name, ignoring case. Quote it if it has spaces. |
| `material:` | An exact material, comma separated for several. `material:hand` uses what you are holding. |
| `currency:` | A currency id from `config.yml`. |
| `days:` | The last N days. Bounded by `trade-log.query.max-days`. |
| `from:` / `to:` | `2026-09-01`, or `2026-09-01 14:30`. Both ends included. |
| `min-qty:` / `max-qty:` | How many items moved. |
| `min-unit:` / `max-unit:` | Price per unit. **This is the one that finds a mispriced shop.** |
| `min-total:` / `max-total:` | What the whole trade came to. |
| `near:` | A radius in blocks around you. Players only. |
| `sort:` | `newest` (the default) or `oldest`. |
| `page:` / `per-page:` | Paging. |
| `stats` | On its own, with no colon: a summary instead of a listing. |

Numbers are plain digits; `1_000_000` is allowed for readability. `1m` is not - a filter that
silently read a typo as zero would be worse than one that refuses it.

### `stats`

Add the bare word `stats` to any query and you get totals instead of lines: how many movements of
each action and how many items they moved, money in and out per currency, the five busiest items
and the five busiest players. The totals cover **every** match, not just the page.

```
/dshop logs player:Steve days:7 stats
```

## Things it will tell you that are easy to miss

**Filters stop the query rather than being ignored.** An unknown key or an unreadable value names
the offending token and runs nothing. A filter that was quietly dropped would let you read a result
set as the answer to a question the plugin never asked.

**`days:` and `from:`/`to:` together are refused.** Two disagreeing time filters resolved by a
hidden precedence rule is how an investigation gets a confidently wrong answer.

**Skipped older lines are reported.** Filtering on `material:` against files written before 2.8.0
finds nothing, because those files have no material column. The reply says how many lines were
skipped for that reason, so "no results" is never mistaken for "nothing happened".

**Truncation is reported.** If the scan hits `max-results` or `max-scanned-lines`, the reply says
the totals are a lower bound. Narrow the query, or raise the ceilings in `config.yml`.

## The file format

```
[2026-09-09 14:03:11] BUY player=Steve player-uuid=0a1b… owner=Alex owner-uuid=7f3c… shop=2d9e…
  loc=world:120:64:-338 item="Diamond Sword" material=DIAMOND_SWORD qty=3 unit=1000 total=3000
  currency=okicoins
```

(one line in the file; wrapped here to fit). The uuids, `loc`, `shop`, `material` and `currency`
are the machine columns; the names beside them are for reading. `qty`, `unit` and `total` are what
actually moved, never what the shop advertises.

A stock movement carries no `unit`, `total` or `currency` - no money changed hands - and a
`DESTROY` or `EXTERNAL` line carries `player=-`, because there was no actor.

A `PICKUP` or `DESTROY` covering more variants than `trade-log.max-rows-per-event` allows folds the
rest into one line carrying `rows=` and their summed `qty`. Nothing is counted twice, so adding up
the `qty` column across an event is always the true total.

{% hint style="warning" %}
**Changed in 2.8.0: the actor column was renamed.** It is `player=` / `player-uuid=` now; before
2.8.0 it was `buyer=` / `buyer-uuid=`, which stopped being true once the column also carried people
depositing and withdrawing. `material=` is new.

`/dshop logs` reads **both** spellings, so your existing files stay fully searchable. A script of
your own that greps for `buyer=` needs updating.

Files written from 2.8.0 on open with a `# SnDisplayShops log format 2` line, so the two
generations can be told apart at a glance.
{% endhint %}

## Restyling

The words `BUY`, `DEPOSIT` and the rest are shown from `status.action-*` in
`lang/messages_<code>.yml`, so you can translate or recolour them. They style the **listing** only -
what goes into the file is a fixed ASCII token that never changes, so a restyle can never break the
file format, your own greps, or the `action:` filter.

Every line `/dshop logs` prints comes from `messages.logs-*` in the same file, including the
hover on each entry. The `{item}` token is a name a shop owner chose, so it is neutralised before it
reaches the message: it cannot colour the rest of the row or turn it into a button. Do not wrap it
in a `<hover:...>` or `<click:...>` of your own and expect the quoting to hold - put the machine
values (`{shop}`, `{world}`, `{x}`, `{y}`, `{z}`, `{material}`, `{currency}`) inside tags instead.

## Housekeeping

Old files are never deleted or compressed. Prune the folder yourself if the server is busy enough
for it to matter - `/dshop logs` simply reads whatever is there.

`trade-log.enabled: false` stops new lines. `/dshop logs` still reads the files already written.
