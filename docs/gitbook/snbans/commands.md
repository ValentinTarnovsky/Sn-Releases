# Commands

SnBans has no player-facing commands. Every command below is a staff command, and every permission node defaults to `op`. Granting `snbans.admin` alone grants all of them, because its children map is exhaustive. There is no `snbans.use` node: each root carries its own leaf node instead. See [Permissions](permissions.md) for the whole tree.

`/snbans` is the admin root, and the only command that takes configurable aliases, listed under `command.aliases` in `config.yml`. That list is authoritative: the names you write there are the complete set, and `/snbans` keeps working alongside them. The eleven flat roots (`/ban`, `/history`, and the rest) take no configurable aliases. On Paper the alias list is re-read on `/snbans reload`. On Velocity a reload re-registers the aliases too, but the proxy does not resend its command tree to players who are already connected, so a new alias only tab-completes for them after they reconnect. Typed in full it works right away.

Every command exists on both platforms with the same syntax, permissions and messages, except where a row or a hint below says otherwise. Usage lines render under the label you actually typed, so `/punish rollback ...` is what an alias shows you. Run `/snbans` with no arguments for the help listing: SnLib generates it on Paper, and the proxy shell renders its own on Velocity.

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
| `/alts <player>` | `snbans.alts` | Lists the accounts sharing the target's current IP |
| `/history <player> [page]` | `snbans.history` | Shows a player's punishment history |
| `/staffhistory <staff> [page]` | `snbans.staffhistory` | Shows the punishments issued by a staff member |
| `/snbans` | `snbans.admin` | Main admin command of SnBans; a bare call shows the help listing |
| `/snbans match <player> <other>` | `snbans.admin.match` | Lists the IPs two accounts have ever shared |
| `/snbans rollback <staff> <time> [confirm]` | `snbans.admin.rollback` | Reverts the punishments a staff member issued in a window |
| `/snbans reload` | `snbans.admin.reload` | Reloads the plugin configuration: config, lang, templates and webhooks |
| `/snbans help [page]` | `snbans.admin` | Shows the available commands |
| `/snbans debug` | `snbans.admin.debug` | Toggles runtime debug output (Paper only) |

{% hint style="info" %}
The `-s` (silent) and `-p` (public) flags work on every punishment and every revert, and both need `snbans.silent`. On an issuing command you type the flag inside the reason (`/ban Notch -s hacks`), anywhere in it, and the last flag wins. On a revert it is an explicit trailing token. Without a flag the per-type `silent-by-default` key decides. A silent punishment still reaches every holder of `snbans.notify`.
{% endhint %}

{% hint style="info" %}
When the reason matches a template id in `templates.yml`, case-insensitively and as the full reason, the duration comes from that template's ladder. The step is chosen by how many punishments of that template the player already collected, and the last step repeats. The template `type` must match the command's type, so `/ban Notch spam` against a `type: mute` template gets no ladder at all. A free reason is a manual punishment.
{% endhint %}

{% hint style="warning" %}
`/unblacklist` is console-only at runtime. Holding `snbans.unblacklist` makes the command grantable and tab-visible, but the flow refuses any non-console sender before the target name is even looked up. Tab completion therefore offers a player nothing for either of its arguments.
{% endhint %}

{% hint style="info" %}
`/history` and `/staffhistory` page in chat at `history.page-size` entries per page, clamped to 1-50. A page past the end renders the last real page instead of an empty listing. The footer arrows are clickable and jump between pages.
{% endhint %}

{% hint style="info" %}
`/alts` asks about the target's current IP; `/snbans match` asks which IPs two accounts have ever shared. Both are read-only, both are subject to `hierarchy.applies-to-alts`, and `/snbans match` refuses two names that resolve to the same account. Naming your own account is refused on every punishment command and on `/snbans rollback`.
{% endhint %}

{% hint style="info" %}
Two platform differences. `/snbans debug` exists on Paper only, because SnLib injects it there and the proxy has no counterpart; on Velocity the debug channel is the proxy logger's own level. The flat-root descriptions render on Paper only, where Bukkit seeds them into `/help`. On the proxy, `/snbans help` still lists `match`, `rollback`, `reload` and `help` with their descriptions.
{% endhint %}

## Tab completion

Both platforms suggest real values as you type:

- Online player names for every target and staff argument.
- Nothing at all for either argument of `/unblacklist` when a player types it.
- `-s` and `-p` on a revert, but only for holders of `snbans.silent`.
- The duration examples `30m`, `1d`, `1h`, `30s`, and `5m` for the `/snbans rollback` window.
- The literal `confirm` for the rollback confirm token, and nothing else.
- The subcommand names of `/snbans`, alphabetically and permission-filtered.
- An angle-bracket hint such as `<reason>` or `<page>` for free-form arguments.

{% hint style="info" %}
Suggestions are a convenience only. Any name is accepted and resolved against the login table, so offline accounts and accounts this server has never seen stay typable, and a name the network has never seen is answered with the unknown-player line. At most 100 names are offered: Paper caps the online list before filtering it, while Velocity filters first and caps the matches.
{% endhint %}

{% hint style="danger" %}
`/snbans rollback <staff> <time>` is destructive in bulk and cannot be undone: it reverts every punishment that staff member issued inside the window. It is a dry run by default, so the first call only counts the matches and prints the confirm command to run. Setting `rollback.require-confirm` to `false` removes the dry run and makes the first call destructive. A rollback takes no `-s` / `-p` flag, so `broadcasts.rollback` alone decides who sees it, and a window matching more than 5000 punishments is refused with a request to narrow it.
{% endhint %}
