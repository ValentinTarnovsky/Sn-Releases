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

### Can I see when a banned or muted player keeps trying?
Yes, that is what `attempt-notices` in `config.yml` is for. With it on, the console and every `snbans.notify` holder are told when a ban, IP ban or blacklist refuses a login, and when a mute cancels a chat message or one of `mute.blocked-commands`. Each of the three is its own toggle, and the wording lives in the `messages.attempt` block of the lang file. There is no public form of the notice: announcing to the whole server that a banned player is trying to get in is an invitation to bait them.

### Why did I only get one notice when a muted player sent ten messages?
Because of `attempt-notices.cooldown-seconds`, which defaults to 60. The same account's same kind of attempt is reported once per window, and that window is what keeps the feature usable: a banned client reconnects every few seconds by itself and a muted player types faster the less they are heard, so without it the notice would be a flood aimed at staff chat. Set it to `0` if you really want every attempt. Note that only the staff notice is throttled - the muted player was answered every single time, because somebody who sees nothing concludes the server is broken and keeps trying.

### An attempt notice named an account I never punished. Why?
Because an IP ban and an IP mute reach every account on the address, and the notice names the account that **tried** rather than the account on the punishment. That is the point: a ban-evading alt is visible the moment it knocks. The `{id}` in the same line is the row that stopped them, so `/history` on the original account is one lookup away. `{server}` follows the same rule - it is the server the attempt was made against, not the one the punishment was issued on.

### Staff got an attempt notice but the player got in anyway. How?
A notice reports the decision SnBans made, and two setups can override it afterwards. Another plugin can allow a login SnBans denied, if it acts at a later event priority - an appeal or bypass plugin does exactly that. And on a proxy-only install a chat message from a 1.19.1+ client with enforced secure chat may be delivered even though the proxy denied it, which is the same caveat that makes backend installs the recommended path for mutes. Neither is SnBans changing its mind: the punishment is still in force and the next attempt is refused the same way.

### Where are punishments stored?
In SQLite by default, which needs no setup. Switch to MySQL under `database` in `config.yml` and point every backend at the same schema to make punishments network-wide. That MySQL user needs CREATE, SELECT, INSERT, UPDATE and DELETE on the schema. The cross-server poller exists only on a shared MySQL install, because a SQLite install has no peer to hear from.

### What happens without LuckPerms?
The staff hierarchy check stays off, whatever `hierarchy.enabled` says, and every rank can punish every rank. It is never an error path. SnBans logs one warning line, `hierarchy.enabled is true but LuckPerms is not installed; the staff weight check stays off and every rank can punish every rank`, and carries on. LuckPerms is optional on Paper and on Velocity alike. Availability is asked live, so installing or re-enabling LuckPerms later starts the check without restarting SnBans. Once it is active, a staff member needs strictly more primary-group weight than the target, so two members of one rank cannot act on each other. The console always passes.

### How do templates escalate?
A template id in `templates.yml` is matched against the full reason, ignoring case, so `/ban Notch hacks` uses the `hacks` template. The step comes from how many punishments of that same template the player already collected. Step one is the first offence, and the last step repeats for every further offence, so the shipped `hacks` ladder gives `1d`, then `5d`, then `permanent`. Offences a rollback reverted do not count toward the step; expired and manually lifted ones do. A `type: blacklist` template cannot shorten anything, because a blacklist is always permanent.

### Why did a template reason ban permanently instead of using its ladder?
Because the template's `type` did not match the command. A template only applies to a command issuing the same type, so `/ban Notch spam` ignores the shipped `spam` template (`type: mute`). With no duration typed either, the result is a manual permanent ban. Give the word its own ban template under a different id, or type a duration on the command. A malformed duration such as `5x` is refused outright rather than stored as permanent.

### How do I undo a wrong rollback?
You cannot, and the same is true of a wipe. Both **delete** the punishments they sweep rather than marking them lifted, so the rows leave `/history` and `/staffhistory` entirely and there is nothing to read back and re-issue. Restore from a database backup, or re-issue from whatever record you kept outside SnBans. What prevents the situation is the dry run: the first invocation of either command only counts the matches and prints the confirm command to run, and for `/snbans wipe` that step cannot be turned off.

### Why is my rollback refused for being too wide?
The sweep has a safety ceiling of 5000 punishments per invocation, and a window above it is refused with a "narrow the window" line before anything is written. It is almost always a typo in the window token, such as `3650d`. Narrow the window and run the command again. `/snbans wipe` has no such ceiling and needs none: it never reads the rows it erases, so there is no batch to bound.

### How do I unban everyone at once?
`/snbans wipe ban confirm`, from the console. It erases every ban that is still in force in one statement, which is what you want after a bad mass-ban or for a season amnesty. `mute` and `blacklist` do the same for theirs and `all` does the three together. Without `confirm` it is a dry run that counts and writes nothing.

### Why can only the console run `/snbans wipe`?
Because the smallest thing it does is lift every ban on the network at once, and a permission node can be granted by mistake while being the console cannot. Granting `snbans.admin.wipe` to a rank makes the subcommand visible in `/snbans help` and nothing more: a player who runs it is answered `messages.console-only` whatever they hold.

### Does a wipe erase old punishments too?
No. It only erases what is still **in force**. A ban somebody already served out, or one a staff member already lifted, is a record of something that happened and stays in `/history` untouched. That is what makes `/snbans wipe ban` the bulk `/unban` it is meant to be rather than a history purge.

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

### Does `/alts` see accounts on my other servers?
Yes, and it always has: login history carries no server column, so on a shared MySQL every backend sees every account of the network. Since 1.6.0 the **Online** marker is network-wide too - each server publishes its connected players into `snbans_presence` and refreshes them on a heartbeat, so an alt connected to another backend renders as online instead of offline. There is nothing to configure: it is on wherever the backend can be shared and absent on SQLite, exactly like cross-server punishment sync. The table is created on boot, so an existing install needs no migration.

### An account still shows as offline even though they are on another server. Why?
Three things to check, in this order. The other server must be running SnBans 1.6.0 or newer against the *same* MySQL - a peer on an older build publishes nothing. Each server needs its own `server-name`, since that is what tells its presence rows from a peer's. And a server that was killed without a clean shutdown keeps its rows for under a minute before they stop counting, so give it that long after a crash. If the peer was restarted while players were already connected, its own boot pass publishes them, so no reconnect is needed.

### How do I keep a staff member's alt off `/alts`?
Add their name (or UUID) to `alts.hidden` in `config.yml` and reload. No player sees that account in an alt scan afterwards, in either direction: it is dropped from everybody else's listing, and `/alts <that name>` answers as if the address carried nothing - hiding only one end would leak the same link from the other. The console still sees the whole scan, and there is deliberately no permission node for it, because a node can be mis-granted and being the console cannot. Nothing about enforcement changes: a hidden account is still banned, muted and kicked like any other, and `/snbans match` is not filtered.

### Does it need PlaceholderAPI, and is there a developer API?
No PlaceholderAPI: SnBans registers no expansion, and the `{player}` style tokens you see belong to `lang/messages_en.yml` and `webhooks.yml`, internal to SnBans. There **is** a public developer API on Paper - see [Developer API](api.md). There are no GUIs either: every listing is chat-paginated. LuckPerms is the only optional integration.

### Where do I put my license key?
In `plugins/.Sn-License/license.yml`, which SnBans creates on its first start on a Paper server. It is one shared file for every Sn bundle plugin on that server, so a single key unlocks the whole pack. Paste your key over the placeholder line and restart. The key is validated once at startup against the Sn license backend, so the machine needs outbound HTTPS. Without a valid key SnBans logs `[Sn] License: FAIL` and disables itself. A standalone Velocity install creates and reads no license file, and has no key step.
