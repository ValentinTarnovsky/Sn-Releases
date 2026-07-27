# FAQ

### How do I update SnBans?
Download the newer `snbans-v*` release, replace the jar and restart the server or proxy. `config.yml`, the active lang file and `webhooks.yml` auto-merge on boot, so your values and comments survive. `templates.yml` is the exception: it is seeded once and never merged again, so a template a newer version ships only reaches a fresh file. Copy it in by hand if you want it. Setting `update-configs: false` freezes the lang file and `webhooks.yml`, and SnLib then only warns about the keys it would have inserted. `config.yml` itself keeps merging, because it is the file that key arrives in.

### Does it support Folia?
No, SnBans is not Folia-compatible: it declares no Folia support. It runs on Paper 1.20.5 and newer (1.21.x included) and on Velocity 3.4.0 and newer, from the same jar.

### Do I install it on the proxy or on the backends?
One or the other, never both. Two deployments are supported and the same jar covers each:

- Every Paper backend, sharing one MySQL (`database.type: mysql`). The server that issues a punishment enforces it at once, and its peers pick the row up within `sync.interval-seconds`. Give each backend its own `server-name`.
- The Velocity proxy alone, which works standalone on SQLite. Logins, chat and commands are enforced at the proxy.

Installing on the proxy and on its backends at the same time announces every punishment twice, once from each side.

### Which deployment enforces mutes better?
The backends. Clients from 1.19.1 onward with enforced secure chat may not accept a chat message the proxy denies, so a proxy install is the weaker option for mutes. Blocked commands, bans, IP bans and blacklists are enforced exactly on either deployment.

### Where are punishments stored?
In SQLite by default, which needs no setup. Switch to MySQL under `database` in `config.yml` and point every backend at the same schema to make punishments network-wide. That MySQL user needs CREATE, SELECT, INSERT, UPDATE and DELETE on the schema. The cross-server poller exists only on a shared MySQL install, because a SQLite install has no peer to hear from.

### What happens without LuckPerms?
The staff hierarchy check stays off, whatever `hierarchy.enabled` says, and every rank can punish every rank. It is never an error path. SnBans logs one warning line, `hierarchy.enabled is true but LuckPerms is not installed; the staff weight check stays off and every rank can punish every rank`, and carries on. LuckPerms is optional on Paper and on Velocity alike. Availability is asked live, so installing or re-enabling LuckPerms later starts the check without restarting SnBans. Once it is active, a staff member needs strictly more primary-group weight than the target, so two members of one rank cannot act on each other. The console always passes.

### How do templates escalate?
A template id in `templates.yml` is matched against the full reason, ignoring case, so `/ban Notch hacks` uses the `hacks` template. The step comes from how many punishments of that same template the player already collected. Step one is the first offence, and the last step repeats for every further offence, so the shipped `hacks` ladder gives `1d`, then `5d`, then `permanent`. Offences a rollback reverted do not count toward the step; expired and manually lifted ones do. A `type: blacklist` template cannot shorten anything, because a blacklist is always permanent.

### Why did a template reason ban permanently instead of using its ladder?
Because the template's `type` did not match the command. A template only applies to a command issuing the same type, so `/ban Notch spam` ignores the shipped `spam` template (`type: mute`). With no duration typed either, the result is a manual permanent ban. Give the word its own ban template under a different id, or type a duration on the command. A malformed duration such as `5x` is refused outright rather than stored as permanent.

### How do I undo a wrong rollback?
There is no un-rollback command, so reverting a sweep is manual. A rollback lifts only the punishments that were still active inside the window and marks them as rollback removals, so nothing is deleted: the rows stay in `/history` and `/staffhistory` with status `Lifted`, with their id, reason, staff and duration. Read them back with `/staffhistory <staff>` and re-issue the ones that should stand. Type the duration explicitly when you do, because a rolled-back offence no longer counts toward escalation and a template reason would land on a lower ladder step than the original. Keeping `rollback.require-confirm: true` is what prevents the situation: the first invocation only counts the matches and prints the confirm command to run.

### Why is my rollback refused for being too wide?
The sweep has a safety ceiling of 5000 punishments per invocation, and a window above it is refused with a "narrow the window" line before anything is written. It is almost always a typo in the window token, such as `3650d`. Narrow the window and run the command again.

### Why does SnBans say the player is unknown?
The login table is the only source of accounts SnBans knows, and there is no Mojang lookup, on purpose: a name the network has never seen is a mistake rather than a punishable stranger. The account has to have connected at least once to this install, or to any server sharing the same MySQL. `retention.days` purges login records, so an account absent for longer than that window stops resolving by name. Raise the value if your staff need to name accounts that have not connected in a long time. Punishment rows themselves are never purged.

### Can I keep my LiteBans history?
Yes. `/snbans import litebans <host:port> <database> <user> <password>` reads a LiteBans database and writes its bans, mutes and login history into SnBans; see [Commands](commands.md) for the full mapping. Run it from the console, and run it without `confirm` first - that is a dry run which counts what it would write and writes nothing. It can only be run once: a second run would duplicate every punishment, so it refuses.

LiteBans kicks and warnings are not imported. SnBans stores no kick row and has no warning type, so there is nowhere for either to go.

### Why did an imported punishment show up as Expired instead of Lifted?
Because it expired rather than being lifted. LiteBans stamps a removal date on both, so the importer uses the recorded *remover* to tell them apart: a row nobody reverted keeps its original expiry and reads as Expired, with no staff member wrongly credited for ending it.

### Why do imported punishments not count towards my template ladders?
LiteBans identifies templates by number and SnBans by `templates.yml` id, and there is no mapping between the two. Rather than guess, the importer stores no template on an imported row, so escalation counts nothing from them and a player's next templated punishment starts at step one.

### Why can nobody remove a blacklist in game?
`/unblacklist` is console-only at runtime. The `snbans.unblacklist` node only makes the command grantable and tab-visible; the flow refuses any other sender before the target name is even looked up. Run it from the server console.

### My punishments are not reaching the other servers. Why?
Check three things in this order. Every backend must use `database.type: mysql` against the same schema. Each backend needs its own `server-name`, because the poller tells a peer's row from its own by that name and a copied default silently disables peer announcements. `sync.interval-seconds` is the delay you should expect. A SQLite install never syncs at all. On a peer, an arriving punishment is enforced and announced under that server's own `broadcasts` rules, while an arriving revert only drops the local enforcement, so a lift is never announced twice.

### Nothing arrives in my Discord channel. Why?
Every block of `webhooks.yml` ships with `enabled: false` and an empty `url`, so the file is inert until you fill both in. A silent punishment is only posted when that block's `include-silent` is `true`. The `include-silent` key of the `rollback` block does nothing: a rollback carries no visibility flag, and `broadcasts.rollback` alone decides how public it is.

### Does it need PlaceholderAPI, and is there a developer API?
No to both. SnBans registers no PlaceholderAPI expansion and ships no public developer API yet. The `{player}` style tokens you see belong to `lang/messages_en.yml` and `webhooks.yml` and are internal to SnBans. There are no GUIs either: every listing is chat-paginated. LuckPerms is the only optional integration.

### Where do I put my license key?
In `plugins/.Sn-License/license.yml`, which SnBans creates on its first start on a Paper server. It is one shared file for every Sn bundle plugin on that server, so a single key unlocks the whole pack. Paste your key over the placeholder line and restart. The key is validated once at startup against the Sn license backend, so the machine needs outbound HTTPS. Without a valid key SnBans logs `[Sn] License: FAIL` and disables itself. A standalone Velocity install creates and reads no license file, and has no key step.
