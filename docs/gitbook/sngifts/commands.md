# Commands

Everything lives under `/gifts`. The alias `/regalos` ships by default and is configurable under
`command.aliases` in `config.yml`, which is re-read on `/gifts reload`.

Players never need a subcommand. A bare `/gifts` opens the menu, and that is the whole player
facing surface. The console gets the generated help instead, because the console has no menu to
open.

| Command | Permission | Description |
|---------|-----------|-------------|
| `/gifts` | `sngifts.use` | Open the daily gifts menu |
| `/gifts reset <player>` | `sngifts.admin.reset` | Clear one player's claims for today |
| `/gifts resetall` | `sngifts.admin.resetall` | Reset every player's claims and playtime |
| `/gifts resetgifts` | `sngifts.admin.resetgifts` | Reroll the rewards every gift hands out today |
| `/gifts bypass` | `sngifts.admin.bypass` | Toggle your own playtime requirement |
| `/gifts reload` | `sngifts.admin.reload` | Reload the config, the language file and the menu |
| `/gifts debug` | `sngifts.admin.debug` | Toggle the runtime debug output |
| `/gifts help` | none | Paginated help, filtered to what you may run |

`/gifts help`, `/gifts reload` and `/gifts debug` are provided by SnLib.

{% hint style="warning" %}
`sngifts.use` gates the whole tree, admin subcommands included, because it sits on the root. If you
restrict it to a rank, grant it to your staff group as well.
{% endhint %}

## What each reset does

The three reset commands are deliberately not interchangeable.

| | Claims | Playtime | Rewards | Per IP counters | Bypasses |
|---|---|---|---|---|---|
| `/gifts reset <player>` | Cleared, one player | Untouched | Untouched | Refunded for that player | Untouched |
| `/gifts resetall` | Cleared, everybody | Zeroed | Untouched | Cleared | Cleared |
| `/gifts resetgifts` | Untouched | Untouched | Redrawn | Untouched | Untouched |
| The daily reset | Cleared, everybody | Zeroed | Redrawn | Cleared | Cleared |

`/gifts reset <player>` also works on an offline target. The address that paid for a claim is
recorded on the claim itself, so the per IP slots are given back without a live connection to read.

## The bypass toggle

`/gifts bypass` skips the playtime gate for you and nothing else. A gift you already claimed today
stays claimed, and the per IP limit still applies. The toggle is session state: it lasts until you
toggle it back, quit, or the day resets.

{% hint style="danger" %}
`/gifts reset <player>`, `/gifts resetall` and `/gifts resetgifts` cannot be undone. `resetall`
wipes the stored playtime of every player on the server, and `resetgifts` replaces today's draw for
everybody, including players who have not claimed yet.
{% endhint %}
