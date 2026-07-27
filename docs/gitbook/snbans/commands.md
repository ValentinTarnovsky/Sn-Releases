# Commands

SnBans has no player-facing commands. Every command below is a staff command, and every permission node defaults to `op`. Granting `snbans.admin` alone grants all of them, because its children map is exhaustive. There is no `snbans.use` node: each root carries its own leaf node instead. See [Permissions](permissions.md) for the whole tree.

`/snbans` is the admin root, and the only command that takes configurable aliases, listed under `command.aliases` in `config.yml`. That list is authoritative: the names you write there are the complete set, and `/snbans` keeps working alongside them. The eleven flat roots (`/ban`, `/history`, and the rest) take no configurable aliases. On Paper the alias list is re-read on `/snbans reload`. On Velocity a reload re-registers the aliases too, but the proxy does not resend its command tree to players who are already connected, so a new alias only tab-completes for them after they reconnect. Typed in full it works right away.

Every command exists on both platforms with the same syntax, permissions and messages, except where a row or a hint below says otherwise. Usage lines render under the label you actually typed, so `/punish rollback ...` is what an alias shows you. Run `/snbans` with no arguments for the help listing: SnLib generates it on Paper, and the proxy shell renders its own on Velocity.

Every command is reachable twice: as its own root (`/ban Notch hacks`) and as a subcommand of the admin root (`/snbans ban Notch hacks`). The two are the same command - same permission, same arguments, same messages - and the second form is what lets `/snbans help` list the whole surface rather than only the admin subcommands. The listing is permission-filtered and paged, so `/snbans help 2` shows the rest.

Duration tokens are `30s`, `5m`, `2h`, `7d`, and the literal `permanent`. No duration and no matching template means permanent. A token that reads as nothing, such as `5x`, is refused rather than stored as a permanent punishment.

| Command | Permission | Description |
|---------|-----------|-------------|
| `/ban <player> [time] <reason...>` | `snbans.ban` | Bans a player |
| `/ipban <player> [time] <reason...>` | `snbans.ipban` | Bans a player and their last known IP |
| `/unban <player> [-s\|-p]` | `snbans.unban` | Removes an active ban |
| `/mute <player> [time] <reason...>` | `snbans.mute` | Mutes a player |
| `/ipmute <player> [time] <reason...>` | `snbans.ipmute` | Mutes a player and their last known IP |
| `/unmute <player> [-s\|-p]` | `snbans.unmute` | Removes an active mute |
| `/blacklist <player> <reason...>` | `snbans.blacklist` | Blacklists a player permanently (account and IP) |
| `/unblacklist <player> [-s\|-p]` | `snbans.unblacklist` | Removes a blacklist (console only) |
| `/kick <player> <reason...>` | `snbans.kick` | Kicks a player from the server; stores nothing |
| `/ipkick <player> <reason...>` | `snbans.ipkick` | Kicks every account connected from the player's IP |
| `/alts <player>` | `snbans.alts` | Lists the accounts sharing the target's current IP |
| `/history <player> [page]`<br>alias `/hist` | `snbans.history` | Shows a player's punishment history |
| `/staffhistory <staff> [page]` | `snbans.staffhistory` | Shows the punishments issued by a staff member |
| `/snbans` | `snbans.admin` | Main admin command of SnBans; a bare call shows the help listing |
| `/snbans match <player> <other>` | `snbans.admin.match` | Lists the IPs two accounts have ever shared |
| `/snbans rollback <staff> <time> [confirm]` | `snbans.admin.rollback` | Reverts the punishments a staff member issued in a window |
| `/snbans import <source> <host:port> <database> <user> <password> [prefix] [confirm]` | `snbans.admin.import` | Imports punishments from another plugin's database |
| `/snbans reload` | `snbans.admin.reload` | Reloads the plugin configuration: config, lang, templates and webhooks |
| `/snbans help [page]` | `snbans.admin` | Shows the available commands |
| `/snbans debug` | `snbans.admin.debug` | Toggles runtime debug output (Paper only) |

{% hint style="info" %}
The `-s` (silent) and `-p` (public) flags work on every punishment and every revert, and both need `snbans.silent`. On an issuing command you type the flag inside the reason (`/ban Notch -s hacks`), anywhere in it, and the last flag wins. On a revert it is an explicit trailing token. Without a flag the per-type `silent-by-default` key decides. A silent punishment still reaches every holder of `snbans.notify`.
{% endhint %}

{% hint style="warning" %}
**A kick stores nothing.** `/kick` and `/ipkick` disconnect the player, announce the action and post their webhook, and write no row at all: no id, no expiry, nothing to un-kick, nothing in `/history` or `/staffhistory`, and no contribution to a template ladder. They still obey the staff weight check, the `-s` / `-p` flags and `snbans.noreason`, and a kick that reaches nobody answers `messages.kick.not-online` instead of announcing a kick that never happened.

Because there is no row, a kick cannot travel between servers the way a ban does. On a **multi-backend Paper** install `/kick` only reaches players connected to the server it was run on. On a **Velocity** install it is network-wide, because the proxy holds every player.

`/ipkick` clears the target's LAST KNOWN address, so it reaches their online alts even when the target themselves is offline - which is what makes it useful against ban evasion. A kick takes no duration, so its whole tail is the reason and a reason may start with a digit here where a `/ban` reason may not.
{% endhint %}

{% hint style="info" %}
When the reason matches a template id in `templates.yml`, case-insensitively and as the full reason, the duration comes from that template's ladder and the STORED reason becomes that template's `reason:` text. The step is chosen by how many punishments of that template the player already collected, and the last step repeats. The template `type` must match the command's type, so `/ban Notch spam` against a `type: mute` template gets no ladder at all. A free reason is a manual punishment.
{% endhint %}

{% hint style="info" %}
A punishment command with no reason answers its usage line. Holders of `snbans.noreason` may run `/ban Notch` bare instead, and the row records the `messages.format.no-reason` text ("No reason" by default). The console always may. The node is separate from `snbans.ban` on purpose: the reason is what a history line, an appeal and a disconnect screen are read from, so the default stays "type one".
{% endhint %}

{% hint style="warning" %}
`/unblacklist` is console-only at runtime. Holding `snbans.unblacklist` makes the command grantable and tab-visible, but the flow refuses any non-console sender before the target name is even looked up. Tab completion therefore offers a player nothing for either of its arguments.
{% endhint %}

{% hint style="info" %}
`/history` and `/staffhistory` page in chat at `history.page-size` entries per page, clamped to 1-50. A page past the end renders the last real page instead of an empty listing. The footer arrows are clickable and jump between pages.
{% endhint %}

{% hint style="info" %}
`/alts` asks about the target's current IP; `/snbans match` asks which IPs two accounts have ever shared. Both are read-only, both are subject to `hierarchy.applies-to-alts`, and `/snbans match` refuses two names that resolve to the same account.

Naming your own account is allowed wherever it only affects you: `/alts <you>`, `/snbans match <you> <other>` and lifting your own punishment with `/unban` or `/unmute`. The staff weight check never refuses you your own account, because no rank can outrank itself. What stays refused is issuing a punishment on yourself - that would lock you out of the server you are moderating - and `/snbans rollback <you>`, which is the one way to erase a colleague's work to reach your own punishment. Both refusals hold whether or not LuckPerms is installed.
{% endhint %}

{% hint style="info" %}
One platform difference. `/snbans debug` exists on Paper only, because SnLib injects it there and the proxy has no counterpart; on Velocity the debug channel is the proxy logger's own level. Everything else, `/snbans help` included, lists the same commands on both sides.
{% endhint %}

## Tab completion

Both platforms suggest real values as you type:

- Online player names for every target and staff argument.
- Nothing at all for either argument of `/unblacklist` when a player types it.
- `-s` and `-p` on a revert, but only for holders of `snbans.silent`.
- The duration examples `30m`, `1d`, `1h`, `30s`, and `5m` for the `/snbans rollback` window.
- The literal `confirm` for the rollback confirm token, and nothing else.
- The subcommand names of `/snbans`, alphabetically and permission-filtered.
- The template ids of the command's own type on a reason argument, so `/ban Notch <TAB>` lists the ban ladders and `/mute Notch <TAB>` the mute ones.
- An angle-bracket hint such as `<reason>` or `<page>` for free-form arguments.

{% hint style="info" %}
Suggestions are a convenience only. Any name is accepted and resolved against the login table, so offline accounts and accounts this server has never seen stay typable, and a name the network has never seen is answered with the unknown-player line. At most 100 names are offered: Paper caps the online list before filtering it, while Velocity filters first and caps the matches.
{% endhint %}

{% hint style="danger" %}
`/snbans rollback <staff> <time>` is destructive in bulk and cannot be undone: it **deletes** every punishment that staff member issued inside the window. The rows are erased rather than marked as lifted, so they leave the sanctioned player's history entirely and can never push a template ladder up a step. It is a dry run by default, so the first call only counts the matches and prints the confirm command to run. Setting `rollback.require-confirm` to `false` removes the dry run and makes the first call destructive. A rollback takes no `-s` / `-p` flag, so `broadcasts.rollback` alone decides who sees it, and a window matching more than 5000 punishments is refused with a request to narrow it.
{% endhint %}

{% hint style="warning" %}
On a multi-server MySQL install, a rollback is the one action peers do not learn about through the sync poller: the poller reads rows whose removal was just recorded, and a deleted row is in no feed. The server that ran the sweep lifts its own mutes at once; other backends keep an erased mute in force until the player next connects there, at which point the login check finds nothing and the mute is gone. A single-server (SQLite) install has no peer and is unaffected.
{% endhint %}

## Importing from LiteBans

`/snbans import litebans <host:port> <database> <user> <password> [prefix] [confirm]` reads a LiteBans database and writes its punishments into SnBans, so a network switching over keeps its history instead of starting blank. `prefix` defaults to `litebans_`, and a bare `host` without a port is read as port 3306.

Without `confirm` it is a **dry run**: it connects, counts what a real run would write, reports the mapping decisions, and writes nothing. Run it that way first and check the numbers against what LiteBans reports.

{% hint style="danger" %}
**Run it from the console, and only once.** The password is a command argument, so it is written to the server log either way - a player running it also puts it in chat. Treat those credentials as disclosed afterwards.

A successful run records itself in the `snbans_imports` table, and every later run refuses. That guard is not a convenience: importing twice would insert a second copy of every punishment, and the copies are indistinguishable from the originals afterwards. A run that fails partway records nothing and can be retried, but whatever it already wrote is still there - check the tables before retrying.
{% endhint %}

### What is imported

| Source | Becomes | Notes |
|--------|---------|-------|
| `litebans_bans` | `BAN` punishments | Removals, expiry and silence carried over |
| `litebans_mutes` | `MUTE` punishments | Same |
| `litebans_history` | `snbans_logins` rows | Collapsed to one row per `(player, address)` pair |
| `litebans_kicks` | nothing | A SnBans kick stores no row, so there is nowhere for them to go |
| `litebans_warnings` | nothing | SnBans has no warning type |

Importing the login history is what makes `/history`, `/alts`, `/snbans match` and name resolution work from the first boot: without it SnBans cannot turn a typed name into an account. Pairs whose last connection is already older than `retention.days` are skipped, because the next daily purge would delete them anyway; the dry run reports how many that is.

### Mapping decisions

These are the places where LiteBans records something SnBans models differently. Each one is a deliberate choice, not a limitation of the reader.

- **Scope follows LiteBans' own IP flag.** A punishment it recorded as an IP ban becomes one here. Be aware that a LiteBans install running `punish_ip: true` sets that flag on *every* punishment it ever wrote, so on such a network every imported row covers the address as well - which is exactly what that network is already enforcing today, but it is worth knowing before you import.
- **Expired is not the same as lifted.** LiteBans stamps a removal date on a punishment that merely ran out as well as on one a staff member reverted. Only rows with a recorded remover are imported as lifted; the rest keep their original expiry and read as Expired in `/history`, with no reverter attributed to them.
- **Templates are not carried over.** LiteBans template numbers are integers and SnBans template ids are `templates.yml` strings, so there is no mapping between them. Imported punishments carry no template and therefore count towards no escalation ladder - a player's next templated punishment starts at step one.
- **Console punishments** keep their name and carry no staff UUID, which is how SnBans already represents the console.
- **Rows with no usable address.** LiteBans writes `#` where it knew no address. Those punishments import as account-only, and on the login side they are dropped entirely: treating that placeholder as an address would put every account behind it on one "shared IP" and make each of them look like an alt of every other.
- **Accounts the source never recorded a name for** are stored as `Unknown-<uuid prefix>`, which keeps them distinct from one another in `/history` and in name resolution.

### While it runs

A large history takes several minutes. The command answers immediately, reports progress as it goes, and posts a summary at the end with the same counts the dry run showed. Do not restart the server during it: a partial import leaves rows behind and records no guard row.

The new `snbans_imports` table is created on boot if missing, on both platforms, so an existing install gains it with no migration step.
