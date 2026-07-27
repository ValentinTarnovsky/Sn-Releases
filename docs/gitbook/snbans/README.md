# SnBans

SnBans is a network-wide punishment suite: bans, mutes, permanent blacklists and alt scanning. One
jar runs on both Paper 1.20.5 or newer (1.21.x included) and Velocity 3.4.0 or newer. Every command
carries the same syntax, permissions and messages on both platforms. Punishment data lives in one
shared SQL schema, so several servers pointed at the same MySQL share a single punishment record.

## Features

- **Eight sanction commands**: `/ban`, `/ipban`, `/mute`, `/ipmute` and `/blacklist`, plus
  `/unban`, `/unmute` and `/unblacklist`. Bans and mutes each have an explicit IP variant and a
  per-type `ip-by-default` switch, so a plain `/ban` can cover the address too. A blacklist is
  always permanent and always covers the IP, so it takes no duration argument at all.
- **Two kick commands that store nothing**: `/kick` and `/ipkick` disconnect, announce and post
  their webhook, and write no row - so a kick never appears in `/history`, has nothing to undo and
  never counts towards a template ladder. `/ipkick` clears the target's last known address, which
  reaches their online alts even when the target is offline. Having no row is also the one
  limitation: on a multi-backend Paper install a kick reaches only the server it was run on, while
  a Velocity install is network-wide.
- **Escalating templates**: a reason matching a `templates.yml` id takes its duration from that
  template's ladder. The step is picked from how many punishments of that template the player
  already collected, and the last step repeats for every further offence. Offences reverted by a
  rollback do not count toward the escalation.
- **Alt detection on two axes**: `/alts` lists the accounts sharing the target's current IP, and
  `/snbans match` lists every IP two accounts have ever shared. An automatic join scan warns
  `snbans.notify` holders, filtered by the states listed in `alts.notify-states`.
- **Network-wide punishments**: on one shared MySQL the issuing server enforces immediately, and
  its peers pick the row up within `sync.interval-seconds`. No plugin messaging channel is
  involved, and a server skips its own rows so nothing is announced twice.
- **Three-layer visibility**: a `snbans.silent` gated `-s` / `-p` flag on every issue and every
  revert, a per-type `silent-by-default`, and a per-event `broadcasts.*` toggle. A silent
  punishment is still reported to `snbans.notify` holders.
- **Eleven Discord webhook blocks**, one per event, each with its own URL, embed and `include-silent`
  toggle. Staff-typed reasons arrive neutralized, so no reason can ping `@everyone` or restyle your
  embed. The shipped file is inert until you fill a URL in.
- **Bulk staff rollback**: `/snbans rollback <staff> <time>` counts the matches first and prints the
  confirm command to run. Nothing is reverted until `confirm` is appended.
- **LiteBans import**: `/snbans import litebans ...` reads an existing LiteBans database and writes
  its bans, mutes and login history into SnBans, so a network switching over keeps its record
  instead of starting blank. It is a dry run until `confirm` is appended, and it refuses to run
  twice.
- **Paginated lookups**: `/history <player>` and `/staffhistory <staff>` page punishment records in
  chat, at `history.page-size` entries per page.

## Paper or Velocity

The same jar covers two deployments, and the choice is yours:

| Deployment | Database | Notes |
|------------|----------|-------|
| Every Paper backend | One shared MySQL (`database.type: mysql`) | Punishments and reverts are network-wide. Each backend needs its own `server-name`. |
| The Velocity proxy alone | SQLite, standalone | Logins, chat and commands are enforced at the proxy. |

{% hint style="danger" %}
Pick exactly one. Installing SnBans on the proxy and on its backends double-announces every
punishment, because the backend and the proxy each announce the same event.
{% endhint %}

{% hint style="warning" %}
Backend installs stay the recommended path for **mutes**. Clients from 1.19.1 onward with enforced
secure chat may not accept a chat message the proxy denies. Blocked commands, bans, IP bans and
blacklists are enforced exactly on either deployment.
{% endhint %}

Give each backend its own `server-name`. It is what `{server}` resolves to, and it is how the sync
poller tells a peer's row from its own. Copying the default name to every backend silently disables
peer announcements.

Two behaviors differ by platform. `/snbans debug` and the `snbans.admin.debug` node are Paper only,
with no proxy counterpart. `command.aliases` is re-sourced on `/snbans reload` on both, but the proxy
does not resend its command tree to players who are already connected, so a new alias only
tab-completes for them after they reconnect.

{% hint style="info" %}
SnBans is not Folia-compatible: `plugin.yml` declares no `folia-supported` key.
{% endhint %}

## Optional integrations

SnBans runs on **SnLib** alone. One optional soft dependency changes its behavior when present:

- **LuckPerms**: enables the staff hierarchy, read from the primary group weight. A staff member
  needs a strictly greater weight than the target, so two members of the same rank cannot sanction
  each other, and the console always passes. Whether the check also covers reverts and alt lookups
  is configurable. Without LuckPerms the check stays off on both platforms, with a console warning
  rather than an error, and every rank can punish every rank.

## Links

- Source and releases: part of the shared [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases) repo
