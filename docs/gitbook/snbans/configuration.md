# Configuration

SnBans ships four YAML files. New keys are auto-merged on boot; your edits and comments are preserved. The one exception is `templates.yml`, which is seeded once and never touched again.

## Where the files live

Paper reads them from `plugins/SnBans/`, Velocity from `plugins/snbans/`. The keys are identical on both platforms, so one edited file can be copied from a backend to the proxy.

| File | Holds | Update mode |
|------|-------|-------------|
| `config.yml` | Database, `server-name`, per-type IP and silent defaults, hierarchy, alts, history paging, rollback, sync, retention, mute blocked commands, broadcast toggles, language code, debug and `/snbans` aliases. | Managed: new keys merge in on boot, values and comments preserved. |
| `lang/messages_en.yml` | Every line SnBans sends, including the per-type announcement, staff-notify and disconnect-screen blocks. | Managed, same merge. |
| `templates.yml` | Your punishment escalation ladders. | Seeded once and never auto-updated: entries you rename or delete stay that way. |
| `webhooks.yml` | One Discord block per event: toggle, URL, content and embed. | Managed, same merge. |

Set `update-configs: false` to freeze the merge. SnLib then only warns in console about the keys it would have inserted.

{% hint style="info" %}
The freeze covers `lang/messages_en.yml` and `webhooks.yml`. `config.yml` itself is exempt on both platforms, because it is the file the `update-configs` key arrives in, so a new key still reaches it. `templates.yml` is never merged either way.
{% endhint %}

## Reloading

`/snbans reload` re-reads `config.yml`, the active lang file, `templates.yml` and `webhooks.yml` on both platforms.

{% hint style="warning" %}
`command.aliases` is re-sourced on both platforms as well, and names you removed are unregistered. On Velocity a new name only tab-completes for players who are already connected after they reconnect, because the proxy does not resend its command tree. Typed in full it works right away.
{% endhint %}

## config.yml

```yaml
# ============================================================
#  SnBans - configuration
#  Managed by SnLib: new keys are auto-merged on boot; your values and
#  comments are preserved. Do NOT add a config-version key (retired).
#  Set update-configs: false to freeze this file (SnLib only warns about
#  missing keys instead of inserting them).
#  Sections marked "# sn:extensible" are yours: entries you delete there
#  stay deleted.
# ============================================================

# Active language code; loads lang/messages_<code>.yml (falls back to en).
lang: en

# Master switch of the managed-file updater, honoured on a backend and on the
# proxy alike: true merges any key a new SnBans version added into config.yml,
# the lang file and webhooks.yml while preserving your values and comments,
# false freezes all three (SnLib then only warns about the keys it would have
# inserted). templates.yml is never touched either way - it is seeded once.
update-configs: true

# Runtime debug output (also toggleable live via /snbans debug).
debug:
  # Master toggle of the debug output.
  enabled: false
  # Verbosity threshold: OFF, INFO, DEBUG or TRACE.
  level: DEBUG
  # Category filter; an empty list lets every category through.
  categories: []

# ------------------------------------------------------------
#  Main command.
# ------------------------------------------------------------
command:
  # Extra names for /snbans, and ONLY for /snbans: the punishment and lookup
  # roots (/ban, /history, ...) are already the short words staff type, so an
  # alias for them would collide with another plugin rather than save anybody a
  # keystroke.
  #
  # Shipped EMPTY on purpose, which is not an oversight: this list is
  # authoritative when present, so any name written here is the complete set and
  # /snbans keeps working alongside it. An empty list means "no extra names",
  # which is what a punishment plugin wants by default - a stolen root (/b, /s)
  # is a support ticket on a network that already runs something using it.
  #
  # Re-read on /snbans reload; on Velocity they are applied at proxy start (a
  # reload re-reads every other key, but not command names). A name that is
  # already a command on the proxy is refused with a warning naming it, because
  # Velocity's registrar would REPLACE the original.
  aliases: []

# ------------------------------------------------------------
#  Database. type=sqlite needs nothing else; type=mysql reads host/port/etc.
#  Several servers sharing the same MySQL make every punishment network-wide.
# ------------------------------------------------------------
database:
  # sqlite or mysql
  type: sqlite
  # MySQL connection (ignored when type is sqlite).
  host: localhost
  # Port the MySQL server listens on.
  port: 3306
  # Name of the schema the punishment tables are created in.
  database: snbans
  # MySQL user; it needs CREATE, SELECT, INSERT, UPDATE and DELETE on that schema.
  username: root
  # Password of that user; an empty value means none.
  password: ""

# ------------------------------------------------------------
#  Model. What a punishment is on this network.
# ------------------------------------------------------------
# Name of this server; stored on every punishment and shown as {server}.
# MUST be unique per install: cross-server sync tells a peer's row from its own
# by this name, so copying the same value to every backend silently disables
# peer announcements. Also install SnBans on the backends OR the proxy, never
# both, or every punishment is announced twice.
server-name: "Server"

# Default scope and visibility of each punishment type.
punishments:
  # /ban and /ipban.
  ban:
    # Also ban the target's last known IP when /ban is used instead of /ipban.
    ip-by-default: false
    # Issue bans silently, so only snbans.notify holders see them.
    silent-by-default: false
  # /mute and /ipmute.
  mute:
    # Also mute the target's last known IP when /mute is used instead of /ipmute.
    ip-by-default: false
    # Issue mutes silently, so only snbans.notify holders see them.
    silent-by-default: false
  # A blacklist is always permanent and always covers the IP, so it has no scope key.
  blacklist:
    # Issue blacklists silently, so only snbans.notify holders see them.
    silent-by-default: false
  # /kick and /ipkick. A kick is NOT stored: it disconnects the player, announces
  # and posts its webhook, and writes nothing to the database - so it never shows
  # up in /history or /staffhistory and never counts towards a template ladder.
  # There is no ip-by-default key here: /kick clears one account and /ipkick
  # clears the address, and a kick you did not ask to be network-wide should not
  # silently become one.
  kick:
    # Kick silently, so only snbans.notify holders see it.
    silent-by-default: false

# Staff hierarchy, read from the LuckPerms primary group weight.
hierarchy:
  # Master toggle; hierarchy stays off while LuckPerms is not installed.
  enabled: true
  # Also check the weight on /unban, /unmute and /unblacklist.
  applies-to-unsanction: false
  # Also check the weight on the two commands that read alt data: /alts and /snbans match.
  applies-to-alts: true

# ------------------------------------------------------------
#  Features. Alt scanning, history paging, rollback and cross-server sync.
# ------------------------------------------------------------
# Accounts sharing the current IP of a player, listed by /alts.
alts:
  # Scan the current IP of every joining player and warn snbans.notify holders.
  auto-scan-on-join: true
  # Punishment states of an alt that trigger the join warning. Accepted tokens:
  # BLACKLISTED, IPBANNED, BANNED, MUTED, ONLINE, OFFLINE; an unknown one is
  # logged and ignored, and an empty list turns the warning off. ONLINE and
  # OFFLINE match every account, so listing either warns on every join.
  notify-states: [BANNED, IPBANNED, BLACKLISTED, MUTED]
  # Accounts that never appear in an alt scan shown to a PLAYER. A name (case
  # insensitive) or a UUID; both forms may be mixed, and a UUID keeps working
  # after the account is renamed. The rule runs BOTH ways, because hiding only
  # one of them would leak the same link from the other end: a listed account
  # is dropped from everybody else's /alts and join warning, AND a scan whose
  # target is listed answers as if the address carried nothing.
  # The console is the only exception - it always sees the whole scan, and so
  # does the developer API, which is server-side code with the same trust.
  # There is deliberately no permission for it: a node can be mis-granted,
  # being the console cannot.
  # Nothing about ENFORCEMENT changes. A hidden account is still scanned,
  # stored, banned, muted and kicked exactly like any other, and /snbans match
  # still reports the addresses it shares with a named account.
  hidden: []

# Chat pagination of /history and /staffhistory.
history:
  # Entries shown per page by /history and /staffhistory.
  page-size: 8

# Bulk revert of the punishments a staff member issued inside a time window.
rollback:
  # Require the confirm token before /snbans rollback <staff> <time> reverts anything.
  require-confirm: true

# /snbans wipe - the bulk amnesty - has NO section here on purpose, and in
# particular no require-confirm key. It erases every punishment of a kind that
# is still in force, so the smallest thing it can do is lift every ban on the
# network at once; a key to skip its dry run would be a key whose only purpose
# is to remove the guard. It is also refused for anybody but the console. What
# you can configure about it is broadcasts.wipe, at the bottom of this file.

# Cross-server propagation; only does something when servers share one MySQL.
sync:
  # Seconds between cross-server punishment polls.
  interval-seconds: 5

# ------------------------------------------------------------
#  Limits. Data retention and what a mute forbids.
# ------------------------------------------------------------
# Login history retention; the daily purge deletes records older than this.
retention:
  # Days a login record is kept; punishments are never purged.
  days: 90

# What an active mute forbids besides chatting.
mute:
  # Commands a muted player cannot run, without the leading slash.
  blocked-commands: [msg, tell, w, me, r]

# ------------------------------------------------------------
#  Feedback. Which events are announced to the whole server.
# ------------------------------------------------------------
# Public announcements, one key per announced event; snbans.notify holders are
# told either way. An unban covers both a ban and an IP ban, and so on.
broadcasts:
  # Announce bans; false leaves them to snbans.notify holders.
  ban: true
  # Announce IP bans; false leaves them to snbans.notify holders.
  ipban: true
  # Announce unbans; false leaves them to snbans.notify holders.
  unban: true
  # Announce mutes; false leaves them to snbans.notify holders.
  mute: true
  # Announce IP mutes; false leaves them to snbans.notify holders.
  ipmute: true
  # Announce unmutes; false leaves them to snbans.notify holders.
  unmute: true
  # Announce blacklists; false leaves them to snbans.notify holders.
  blacklist: true
  # Announce unblacklists; false leaves them to snbans.notify holders.
  unblacklist: true
  # Announce kicks; false leaves them to snbans.notify holders.
  kick: true
  # Announce IP kicks; false leaves them to snbans.notify holders.
  ipkick: true
  # Announce rollbacks; false leaves them to snbans.notify holders.
  rollback: true
  # Announce wipes; false leaves them to snbans.notify holders. A wipe that
  # erased nothing is never announced either way - there was nothing to tell.
  wipe: true

# Staff notices for the punishments a player RUNS INTO, as opposed to the ones
# staff hand out: a banned account knocking on the door and a muted player still
# trying to talk. They go to the console and to snbans.notify holders only, and
# there is no public form of them at all - telling the whole server that a banned
# player is trying to get in is an invitation to bait them. The refusal the player
# themselves sees is unchanged and never mentions that staff were told. Wording
# lives in the messages.attempt block of the lang file.
attempt-notices:
  # Report a login a ban, an IP ban or a blacklist refused. A mute never denies a
  # login, so a muted player joining is not an attempt at anything: the automatic
  # join alt scan is what reports them (alts.notify-states above).
  login: true
  # Report a chat message a mute cancelled.
  chat: true
  # Report one of mute.blocked-commands a mute cancelled.
  command: true
  # Seconds before the SAME player's SAME kind of attempt is reported again.
  # This is the key that decides whether the feature is useful or unusable: the
  # two players it reports are the two who repeat themselves - a banned client
  # reconnects every few seconds by itself, and a muted player types faster the
  # less they are heard - so without a window a staff notice becomes a flood
  # aimed at staff chat. Only the STAFF notice is throttled; the refusal the
  # player themselves sees is never withheld. 0 reports every single attempt.
  cooldown-seconds: 60
```

## Notable settings

A few keys deserve a closer note on how they behave at runtime.

### database.type

`sqlite` needs nothing else and is what a single Velocity proxy runs on. `mysql` reads the host, port, schema, user and password below it. Several servers pointing at one MySQL is what makes punishments network-wide.

{% hint style="warning" %}
The MySQL user needs CREATE, SELECT, INSERT, UPDATE and DELETE on the configured schema. SnBans creates its own tables on first boot.
{% endhint %}

### server-name

Stored on every punishment and shown wherever `{server}` appears. It must be unique per install. Cross-server sync tells a peer's row from its own by this name, so the same value on every backend silently disables peer announcements.

{% hint style="danger" %}
Install SnBans on the backends or on the proxy, never both. Both halves announce the same event, so every punishment is announced twice.
{% endhint %}

### command.aliases

Extra names for `/snbans`, and only for `/snbans`. The eleven flat roots take no configurable aliases. The list is authoritative when present, so the names you write are the complete set, and `/snbans` keeps working alongside them.

Entries are normalized into command literals: trimmed, leading slashes stripped, lower-cased and de-duplicated, with the file order kept. Writing `["/Punish"]` therefore yields `punish` on both platforms.

Both platforms re-source the list on `/snbans reload`, and Velocity unregisters the names you removed. Velocity also refuses a name that is already a proxy command, with a warning naming it, because its registrar would replace the original. A new name reaches an already connected player's tab completion only after they reconnect.

### punishments.ban and punishments.mute

`ip-by-default: true` makes a plain `/ban` or `/mute` cover the target's last known IP, exactly as the explicit `/ipban` and `/ipmute` do. `silent-by-default: true` issues the type silently, so only `snbans.notify` holders see it.

`punishments.blacklist` carries no `ip-by-default` key. A blacklist is always permanent and always covers the IP, so the flag would never be read.

### punishments.kick

`silent-by-default: true` makes `/kick` and `/ipkick` silent, so only `snbans.notify` holders see them. One key covers both commands, exactly as `punishments.ban` covers `/ban` and `/ipban`.

There is no `ip-by-default` key. Reaching the whole address is what `/ipkick` is for, and a kick you did not ask to be address-wide should not silently become one. There is no `silent-by-default` per command either, and no duration anywhere: a kick cannot expire.

### hierarchy

Reads the weight of the LuckPerms primary group and refuses a punishment against an equal or higher rank. `applies-to-unsanction` extends the check to `/unban`, `/unmute` and `/unblacklist`. `applies-to-alts` extends it to the two commands that read alt data, `/alts` and `/snbans match`.

{% hint style="info" %}
LuckPerms is optional on Paper and on Velocity. Without it the hierarchy check stays off with a single console warning, and every rank can punish every rank, whatever `enabled` says.
{% endhint %}

### alts.notify-states

The alt states that trigger the join warning. Accepted tokens are `BLACKLISTED`, `IPBANNED`, `BANNED`, `MUTED`, `ONLINE` and `OFFLINE`. An unknown token is warned about in console and skipped, and an empty list turns the warning off.

`ONLINE` and `OFFLINE` match every account, so listing either one warns on every single join.

### alts.hidden

Accounts that never appear in an alt scan shown to a **player**. Entries are names (matched
case-insensitively) or UUIDs, and the two forms may be mixed in one list - a UUID keeps working
after the account is renamed, which is the one way a name list silently stops hiding what it was
written to hide.

```yaml
alts:
  hidden:
    - Snopeyy
    - 069a79f4-44e9-4726-a5be-fca90e38aaf5
```

The rule runs **both ways**, because hiding only one end would leak the identical link from the
other:

* a listed account is dropped from everybody else's `/alts` and from the join warning;
* a scan whose **target** is listed answers as if the address carried nothing, so `/alts <hidden>`
  reads exactly like an account nobody shares an address with.

{% hint style="info" %}
The console is the only exception - it always sees the whole scan, and so does the developer API,
which is server-side code with the same trust. There is deliberately no permission node for this:
a node can be mis-granted, being the console cannot.
{% endhint %}

Nothing about **enforcement** changes. A hidden account is still scanned, stored, banned, muted and
kicked exactly like any other, so an IP ban placed on one of its alts still reaches it.
`/snbans match` is not filtered either: it takes two explicit names and reports shared addresses
rather than listing accounts.

### history.page-size

Entries per page of `/history` and `/staffhistory`. The value is clamped to the range 1 to 50. A value outside it is moved and warned about once per reload, since a silently clamped number is a setting you believe is in effect.

The ceiling exists because the value binds straight into a SQL `LIMIT`, and each returned row becomes a token map plus a rendered chat line.

### rollback.require-confirm

While `true`, `/snbans rollback <staff> <time>` only counts the matches and prints the confirm command to run. Nothing is reverted until `confirm` is appended.

{% hint style="danger" %}
Setting this to `false` removes the dry run, so the first invocation reverts immediately.
{% endhint %}

### There is no `wipe.require-confirm`

`/snbans wipe` has no configuration section at all, and that is deliberate rather than an omission. Its dry run cannot be turned off and it cannot be granted to a player: the command is console only at runtime, and the smallest thing it does is lift every ban on the network at once. The only thing you configure about it is `broadcasts.wipe` below, plus its lang block and its webhook block.

### sync.interval-seconds

Seconds between cross-server punishment polls, clamped to a minimum of 1. It only does something when several servers share one MySQL. The issuing server enforces a punishment immediately, and its peers pick the row up within this interval.

### retention.days

Days a login record is kept, clamped to a minimum of 1. The daily purge deletes older records. Punishment rows are never purged, whatever this value is.

### mute.blocked-commands

Commands a muted player cannot run, written without the leading slash. Entries are normalized like the aliases above: trimmed, slashes stripped, lower-cased and de-duplicated, in file order.

### broadcasts

One key per announced event. `false` leaves that event to `snbans.notify` holders instead of the whole server. An unban covers both a ban and an IP ban, and the same pairing applies to the other reverts.

`broadcasts.rollback` and `broadcasts.wipe` alone decide how visible those two bulk actions are: neither `/snbans rollback` nor `/snbans wipe` declares an `-s` or `-p` flag, so neither is ever silent. A wipe that erased nothing is not announced at all, whatever the toggle says.

### attempt-notices

The counterpart of `broadcasts`: that section is about the punishments staff hand out, this one about the punishments players run into. Three toggles pick which refusals are reported to the console and to `snbans.notify` holders.

| Key | Reports |
|-----|---------|
| `login` | A ban, an IP ban or a blacklist refused a login |
| `chat` | A mute cancelled a chat message |
| `command` | A mute cancelled one of `mute.blocked-commands` |

`cooldown-seconds` (default 60) is the key that decides whether the feature is useful. The two players it reports are the two who repeat themselves: a banned client reconnects every few seconds on its own, and a muted player types faster the less they are heard. The same account's same kind of attempt is reported once per window; `0` reports every single attempt. Only the staff notice is throttled - the refusal the player gets is unchanged, because somebody who sees nothing concludes the server is broken and keeps trying.

`{player}` is the account that **tried**, which is not always the account named on the punishment: an IP ban and an IP mute reach every account on the address, so the notice names the alt that ran into the row while `{id}` names the row that stopped them. That is what makes a ban-evading alt visible the moment it knocks.

{% hint style="info" %}
There is no public form of these notices and no webhook block for them: an attempt is not something anybody did, and announcing to the whole server that a banned player is trying to get in is an invitation to bait them. A muted player joining is not an attempt either, since a mute does not deny logins - the join alt scan reports them through `alts.notify-states`.
{% endhint %}

### debug

`enabled` is the master toggle, `level` the verbosity threshold (`OFF`, `INFO`, `DEBUG` or `TRACE`) and `categories` a filter that lets everything through while empty.

{% hint style="info" %}
The live `/snbans debug` toggle is Paper-only, as is the `snbans.admin.debug` node. The proxy has no counterpart, so edit `debug.enabled` and reload instead.
{% endhint %}

### There is no config-version key

SnLib merges by key, so no version marker is needed. Do not add one.

## templates.yml

```yaml
# ============================================================
#  SnBans - punishment templates
#  Seeded once by SnLib and never auto-updated afterwards: entries you rename
#  or delete stay that way.
#
#  A template turns a reason into an escalating punishment:
#    - The template id is matched against the FULL reason of the command,
#      case-insensitively, so "/ban Notch hacks" uses the "hacks" template.
#      The ids of the matching type are also offered on TAB, so staff can see
#      which words this server has ladders for without leaving the chat box.
#    - reason is the text actually STORED on the punishment and shown by every
#      broadcast, history line, Discord embed and disconnect screen. The id is
#      the short word staff type; the reason is the sentence the player reads.
#      Leave it out and the id is used as the reason, which is how templates
#      behaved before this key existed.
#    - type is ban, mute or blacklist.
#    - THE TYPE HAS TO MATCH THE COMMAND. A template only applies to a command
#      that issues the same type, so the "spam" template below (type: mute) is
#      used by "/mute Notch spam" and IGNORED by "/ban Notch spam" - the ban
#      then has no template and no ladder at all, and becomes a manual
#      PERMANENT ban, because no duration was typed either. That is deliberate:
#      a /ban must never silently borrow a mute's ladder. If staff need to ban
#      for a reason word a mute template owns, give the word its own ban
#      template (a different id) or type a duration on the command.
#    - ladder is the escalation list. Step 1 is the first offence, step 2 the
#      second, and the last step repeats for every further offence.
#    - Each step is a duration token (30s, 30m, 12h, 5d) or permanent.
#    - Offences reverted by /snbans rollback do not count: a rollback DELETES
#      the punishments it undoes, so they leave the player's history entirely.
#      Expired and manually removed ones do count.
#    - A reason matching no template - or matching one of another type - is a
#      manual punishment: the duration typed on the command, or permanent when
#      none was given.
# ============================================================

# Example template: /ban <player> hacks escalates 1d, then 5d, then permanent.
hacks:
  # Punishment type this template issues: ban, mute or blacklist.
  type: ban
  # Text stored and shown as the reason; the id above is only what staff type.
  reason: "Using unfair advantages"
  # Escalation steps; the last one repeats for every further offence.
  ladder: [1d, 5d, permanent]

# Example template: /mute <player> spam escalates 30m, then 2h, then 1d.
spam:
  # Punishment type this template issues: ban, mute or blacklist.
  type: mute
  # Text stored and shown as the reason; the id above is only what staff type.
  reason: "Spamming the chat"
  # Escalation steps; the last one repeats for every further offence.
  ladder: [30m, 2h, 1d]
```

### Writing a template

Each top-level key is a template id, matched against the full reason of the command and case-insensitively. `type` is `ban`, `mute` or `blacklist`. `ladder` is the escalation list, where every step is a duration token such as `30s`, `5m`, `2h` or `7d`, or the literal `permanent`.

`reason` is the text stored on the punishment and rendered as `{reason}` everywhere: the broadcast, the staff notice, the history entry, the Discord embed and the disconnect screen. It exists so the two halves of a template can be different things - the id is the short word staff have to type and tab-complete (`hacks`), while the reason is the sentence a player is entitled to read ("Using unfair advantages"). The key is optional: a template without one keeps its id as its reason, which is exactly how templates behaved before the key existed.

Step 1 applies to the first offence, step 2 to the second, and the last step repeats for every further offence. Offences reverted by `/snbans rollback` do not count toward the step - a rollback deletes them - while expired and manually removed ones do.

The ids of the command's own type are offered on tab, so `/ban Notch <TAB>` lists the ban ladders and `/mute Notch <TAB>` the mute ones. The suggestion is a convenience only: any reason is still accepted, and one matching no template is a manual punishment.

{% hint style="warning" %}
A template only applies to a command that issues the same type. `/ban Notch spam` against a `type: mute` template gets no template and no ladder, and with no typed duration becomes a permanent ban. Give the word its own ban template with a different id, or type a duration.
{% endhint %}

{% hint style="info" %}
This file is seeded once, so keys added by a future version never appear in an existing copy. **`reason:` is such a key**: an existing `templates.yml` will not grow it on its own, and its templates keep storing their id as the reason until you add the line by hand. Delete your file to get the shipped examples back, and read the release notes when a version changes the format.
{% endhint %}

## webhooks.yml

```yaml
# ============================================================
#  SnBans - Discord webhooks
#  Managed by SnLib: new keys are auto-merged on boot; your values and
#  comments are preserved. Set update-configs: false in config.yml to
#  freeze every merge.
#
#  One block per event. A block sends nothing while enabled is false or
#  url is empty, so the file is inert until you fill a URL in.
#
#  Placeholders usable in content, embed title and embed description:
#    {player}    punished player name       {staff}     staff member name
#    {reason}    reason, see below          {duration}  duration, or Permanent
#    {server}    server that issued it      {template}  template id, or None
#    {id}        punishment id              {date}      date and time
#    {type}      translated word naming the event ("ban", "unban", ...)
#    {status}    whether the punishment still applies (Active/Expired/Lifted)
#
#  {reason} is the only staff-TYPED value, and it arrives neutralized for
#  Discord: colour codes and %placeholder% markers are gone, @everyone, @here
#  and raw id mentions cannot ping, and the markdown characters are escaped so a
#  reason cannot restyle the embed around it. Your own markdown below (the **
#  pairs) is untouched - only the value is neutralized, never this file.
#
#  The rollback event has no punishment row of its own, so it carries {staff},
#  {total} (how many punishments were reverted), {duration} (the window that was
#  swept, humanized), {window} (that same window as the single token the command
#  accepts) and {type}. A placeholder with no value is left as written.
# ============================================================

ban:
  # Send this webhook when a ban is issued.
  enabled: false
  # Discord webhook URL; an empty URL never sends.
  url: ""
  # Plain text posted above the embed; empty sends the embed alone.
  content: ""
  embed:
    # Title line of the embed.
    title: "Player banned"
    # Side color of the embed, as a hex string.
    color: "#FF5555"
    # Body of the embed, one entry per line.
    description:
      - "**Player:** {player}"
      - "**Staff:** {staff}"
      - "**Reason:** {reason}"
      - "**Duration:** {duration}"
      - "**Server:** {server}"
  # Also send when the punishment was issued silently.
  include-silent: false

ipban:
  # Send this webhook when an IP ban is issued.
  enabled: false
  # Discord webhook URL; an empty URL never sends.
  url: ""
  # Plain text posted above the embed; empty sends the embed alone.
  content: ""
  embed:
    # Title line of the embed.
    title: "Player IP banned"
    # Side color of the embed, as a hex string.
    color: "#FF5555"
    # Body of the embed, one entry per line.
    description:
      - "**Player:** {player}"
      - "**Staff:** {staff}"
      - "**Reason:** {reason}"
      - "**Duration:** {duration}"
      - "**Server:** {server}"
  # Also send when the punishment was issued silently.
  include-silent: false

unban:
  # Send this webhook when a ban is lifted.
  enabled: false
  # Discord webhook URL; an empty URL never sends.
  url: ""
  # Plain text posted above the embed; empty sends the embed alone.
  content: ""
  embed:
    # Title line of the embed.
    title: "Player unbanned"
    # Side color of the embed, as a hex string.
    color: "#55FF55"
    # Body of the embed, one entry per line.
    description:
      - "**Player:** {player}"
      - "**Staff:** {staff}"
      - "**Server:** {server}"
  # Also send when the revert was issued silently.
  include-silent: false

mute:
  # Send this webhook when a mute is issued.
  enabled: false
  # Discord webhook URL; an empty URL never sends.
  url: ""
  # Plain text posted above the embed; empty sends the embed alone.
  content: ""
  embed:
    # Title line of the embed.
    title: "Player muted"
    # Side color of the embed, as a hex string.
    color: "#FFAA00"
    # Body of the embed, one entry per line.
    description:
      - "**Player:** {player}"
      - "**Staff:** {staff}"
      - "**Reason:** {reason}"
      - "**Duration:** {duration}"
      - "**Server:** {server}"
  # Also send when the punishment was issued silently.
  include-silent: false

ipmute:
  # Send this webhook when an IP mute is issued.
  enabled: false
  # Discord webhook URL; an empty URL never sends.
  url: ""
  # Plain text posted above the embed; empty sends the embed alone.
  content: ""
  embed:
    # Title line of the embed.
    title: "Player IP muted"
    # Side color of the embed, as a hex string.
    color: "#FFAA00"
    # Body of the embed, one entry per line.
    description:
      - "**Player:** {player}"
      - "**Staff:** {staff}"
      - "**Reason:** {reason}"
      - "**Duration:** {duration}"
      - "**Server:** {server}"
  # Also send when the punishment was issued silently.
  include-silent: false

unmute:
  # Send this webhook when a mute is lifted.
  enabled: false
  # Discord webhook URL; an empty URL never sends.
  url: ""
  # Plain text posted above the embed; empty sends the embed alone.
  content: ""
  embed:
    # Title line of the embed.
    title: "Player unmuted"
    # Side color of the embed, as a hex string.
    color: "#55FF55"
    # Body of the embed, one entry per line.
    description:
      - "**Player:** {player}"
      - "**Staff:** {staff}"
      - "**Server:** {server}"
  # Also send when the revert was issued silently.
  include-silent: false

blacklist:
  # Send this webhook when a blacklist is issued.
  enabled: false
  # Discord webhook URL; an empty URL never sends.
  url: ""
  # Plain text posted above the embed; empty sends the embed alone.
  content: ""
  embed:
    # Title line of the embed.
    title: "Player blacklisted"
    # Side color of the embed, as a hex string.
    color: "#FF0000"
    # Body of the embed, one entry per line.
    description:
      - "**Player:** {player}"
      - "**Staff:** {staff}"
      - "**Reason:** {reason}"
      - "**Server:** {server}"
  # Also send when the punishment was issued silently.
  include-silent: false

unblacklist:
  # Send this webhook when a blacklist is lifted.
  enabled: false
  # Discord webhook URL; an empty URL never sends.
  url: ""
  # Plain text posted above the embed; empty sends the embed alone.
  content: ""
  embed:
    # Title line of the embed.
    title: "Player unblacklisted"
    # Side color of the embed, as a hex string.
    color: "#55FF55"
    # Body of the embed, one entry per line.
    description:
      - "**Player:** {player}"
      - "**Staff:** {staff}"
      - "**Server:** {server}"
  # Also send when the revert was issued silently.
  include-silent: false

kick:
  # Send this webhook when a player is kicked.
  enabled: false
  # Discord webhook URL; an empty URL never sends.
  url: ""
  # Plain text posted above the embed; empty sends the embed alone.
  content: ""
  embed:
    # Title line of the embed.
    title: "Player kicked"
    # Side color of the embed, as a hex string.
    color: "#FFAA00"
    # Body of the embed, one entry per line. A kick stores no punishment, so
    # there is no {duration} and no {id} to report here.
    description:
      - "**Player:** {player}"
      - "**Staff:** {staff}"
      - "**Reason:** {reason}"
      - "**Server:** {server}"
  # Also send when the kick was issued silently.
  include-silent: false

ipkick:
  # Send this webhook when every account on an address is kicked.
  enabled: false
  # Discord webhook URL; an empty URL never sends.
  url: ""
  # Plain text posted above the embed; empty sends the embed alone.
  content: ""
  embed:
    # Title line of the embed.
    title: "IP kicked"
    # Side color of the embed, as a hex string.
    color: "#FFAA00"
    # Body of the embed, one entry per line. {total} is how many accounts were
    # actually disconnected.
    description:
      - "**Player:** {player}"
      - "**Staff:** {staff}"
      - "**Accounts:** {total}"
      - "**Reason:** {reason}"
      - "**Server:** {server}"
  # Also send when the kick was issued silently.
  include-silent: false

rollback:
  # Send this webhook when /snbans rollback reverts a batch of punishments.
  enabled: false
  # Discord webhook URL; an empty URL never sends.
  url: ""
  # Plain text posted above the embed; empty sends the embed alone.
  content: ""
  embed:
    # Title line of the embed.
    title: "Punishments rolled back"
    # Side color of the embed, as a hex string.
    color: "#8354f2"
    # Body of the embed, one entry per line.
    description:
      - "**Staff:** {staff}"
      - "**Reverted:** {total}"
      - "**Window:** {duration}"
  # INERT in this one block, and read anyway so every block has the same shape:
  # /snbans rollback declares no -s / -p flag and defers to broadcasts.rollback
  # alone, so a sweep is never silent and this key can never decide anything.
  # Changing it has no effect; it is kept so the block stays complete if a
  # future version does give the command a visibility flag.
  include-silent: false

wipe:
  # Send this webhook when /snbans wipe erases a batch of punishments. A wipe
  # that erased nothing sends nothing.
  enabled: false
  # Discord webhook URL; an empty URL never sends.
  url: ""
  # Plain text posted above the embed; empty sends the embed alone.
  content: ""
  embed:
    # Title line of the embed.
    title: "Punishments wiped"
    # Side color of the embed, as a hex string.
    color: "#FF5555"
    # Body of the embed, one entry per line. A wipe reads none of the rows it
    # erases, so it has no {player}, {reason}, {duration} or {id} to report:
    # {kind} is what it covered ("bans", "punishments"), {total} how many.
    description:
      - "**Wiped:** {total} {kind}"
      - "**Staff:** {staff}"
  # INERT here for the reason it is inert on rollback just above: /snbans wipe
  # declares no -s / -p flag, so a wipe is never silent.
  include-silent: false
```

### Enabling a webhook

There are twelve blocks, one per event: `ban`, `ipban`, `unban`, `mute`, `ipmute`, `unmute`, `blacklist`, `unblacklist`, `kick`, `ipkick`, `rollback` and `wipe`. Each carries `enabled`, `url`, `content`, an `embed` section and `include-silent`.

A block sends nothing while `enabled` is `false` or `url` is empty, so the shipped file is inert until you paste a Discord webhook URL in. Set `include-silent: true` on a block to post silent punishments of that type too.

### Placeholders

Usable in `content`, `embed.title` and every `embed.description` line of the eight punishment blocks.

| Placeholder | Description |
|-------------|-------------|
| `{player}` | Punished player name. |
| `{staff}` | Staff member name. |
| `{reason}` | Reason typed on the command, neutralized for Discord. |
| `{duration}` | Duration, or `Permanent`. |
| `{server}` | Server that issued the punishment. |
| `{template}` | Template id, or `None`. |
| `{id}` | Punishment id. |
| `{date}` | Date and time. |
| `{type}` | Translated word naming the event, such as `ban` or `unban`. |
| `{status}` | Whether the punishment still applies: `Active`, `Expired` or `Lifted`. |

The `rollback` block has no punishment row of its own, so it carries a different set.

| Placeholder | Description |
|-------------|-------------|
| `{staff}` | Staff member whose punishments were swept. |
| `{total}` | How many punishments were reverted. |
| `{duration}` | The window that was swept, humanized. |
| `{window}` | That same window as the single token the command accepts. |
| `{type}` | Translated word naming the event. |

The `wipe` block has none either, and it reads none of the rows it erases, so its set is smaller again.

| Placeholder | Description |
|-------------|-------------|
| `{staff}` | Who ran the wipe, which is always the console. |
| `{total}` | How many punishments were erased. |
| `{kind}` | What was covered, as a translated plural: `bans`, `mutes`, `blacklists` or `punishments`. |
| `{target}` | That same kind as the raw token the command accepts: `ban`, `mute`, `blacklist` or `all`. |
| `{type}` | Translated word naming the event. |

A placeholder with no value is left as written.

{% hint style="info" %}
`{reason}` is the only staff-typed value, and it arrives neutralized. Color codes and `%placeholder%` markers are stripped, `@everyone`, `@here` and raw id mentions cannot ping, and markdown characters are escaped. Your own markdown in the file is untouched.
{% endhint %}

{% hint style="warning" %}
`include-silent` is inert in the `rollback` and `wipe` blocks. Neither command declares an `-s` or `-p` flag - each defers to its own `broadcasts` toggle alone - so changing the key there has no effect. It exists only to keep every block the same shape.
{% endhint %}

## Language file

`lang/messages_en.yml` holds every line SnBans sends, on both platforms. It is managed, so new keys merge in on boot and your edits survive updates. It is long, so it is not reproduced here in full; its header states the contract:

```yaml
# ============================================================
#  SnBans - language file (English)
#  Managed by SnLib: new keys merge in on boot; your edits survive updates.
# ============================================================
```

Copy the file to `messages_<code>.yml` and set `lang` in `config.yml` to add a language. The `lang` key selects the file on boot and falls back to `en`.

{% hint style="info" %}
On Velocity, a `lang` code naming a file that is neither bundled nor already in the data directory falls back to English with one console warning. English is also mounted alongside a translation as a per-key fallback there, so a key your translation is missing still renders, disconnect screens included.
{% endhint %}

### Key families

| Family | Holds |
|--------|-------|
| `prefix` | The single value SnLib prepends to every single-line message. It ships as the SnBans brand tag. |
| `snlib` | SnLib's shared command contract, 12 keys: permission, usage, number and value validation, out-of-range, number-too-small, player-not-found, unknown subcommand, reload confirmation, and the help header, entry and footer. |
| `messages` (errors) | Refusals and errors: `console-only`, `unknown-player`, `hierarchy-denied`, `already-punished`, `not-punished`, `invalid-duration`, `internal-error`, `match-self`, `self-target`, `reload-busy`, plus `muted` and `muted-command` for a muted player. |
| `messages.format` | The words other messages splice in as placeholder values: `permanent`, `no-template`, `no-reason`, `console`, the three `status-*` words behind `{status}`, the twelve `type-*` words behind `{type}`, and the four `wipe-*` plurals behind `{kind}`. `no-reason` is the odd one out: it is WRITTEN to the database as the reason of a punishment a `snbans.noreason` holder issued bare, so retranslating it changes what new punishments record and leaves the stored ones reading as they did. |
| `messages.<event>` | One block per event (`ban`, `ipban`, `mute`, `ipmute`, `blacklist`, `unban`, `unmute`, `unblacklist`, `kick`, `ipkick`), each with `announce` for the public broadcast and `notify` for `snbans.notify` holders. Five carry `screen`, the disconnect screen the player sees: `ban`, `ipban` and `blacklist` because they deny a login, plus `kick` and `ipkick` because they disconnect somebody already in. `kick` additionally carries `not-online`, the answer when the target is not connected to this server. A kick block has no `{duration}`, `{id}`, `{template}` or `{status}` to render, since a kick has no length, no row, no ladder and no state; `{total}` is how many accounts an `ipkick` disconnected and is not available in a `screen`. |
| `messages.attempt` | The three attempt notices of `attempt-notices`: `login`, `chat` and `command`. They share the audience of a `notify` block and are sent the same way, so they are lists and carry their own inline tag. `{player}` is the account that tried, not the account the punishment names. |
| `messages.alts` | The `/alts` listing and the join scan: `header`, `scan-header`, `legend`, `entry`, `none`, `footer` and the five `status-*` color prefixes behind `{color}`. |
| `messages.history` and `messages.staffhistory` | The two paged listings: `header`, `entry` and a `footer` carrying the clickable page arrows. |
| `messages.match` | The `/snbans match` listing: `header`, `entry`, `none` and `footer`. |
| `messages.rollback` | The sweep: `announce` and `notify` for the network, plus `dry-run`, `done`, `none` and `too-wide` for the staff member who ran it. |
| `messages.wipe` | The bulk amnesty: `announce` and `notify` for the network, plus `dry-run`, `done`, `none` and `unknown-target` for whoever ran it. `dry-run` prints a runnable confirm command, so keep `{label}`, `{target}` and `{confirm}` in it as tokens rather than spelling them out. |
| `messages.import` | The LiteBans import: `started`, `progress`, `dry-run`, `done` and `empty` for a run that worked, plus `already-imported`, `unknown-source`, `bad-address`, `no-driver`, `failed` and `busy` for one that did not. |

### Editing rules

Never write `{prefix}` in a value. SnLib prepends the configured prefix to every single-line message, so a literal one renders twice.

A value written as a YAML list is sent line by line without the prefix. The single exception is an `announce` list holding one line, which is broadcast as a message and so gets it. A `notify` list never does, which is why every shipped `notify` value carries its own inline tag. Keep that tag if you shorten one.

{% hint style="danger" %}
Do not delete or blank one of the five `screen` blocks. A screen key with nothing in it disconnects the player with an empty screen and no log line saying why.
{% endhint %}

{% hint style="warning" %}
Keep the `<click>` and `<hover>` tags in the two history footers, or the arrows still render but clicking does nothing. Keep `{label}` there too, so the buttons always dispatch the listing they belong to. In `rollback.dry-run`, keep both `{label}` and `{confirm}`, or you print a command the plugin refuses.
{% endhint %}

Player-typed values are sanitized before they reach a line. `{reason}`, `{value}` and the `{player}` of `unknown-player` have legacy color codes and `%placeholder%` markers removed, and MiniMessage tags escaped. A typed `<rainbow>` or `%player_name%` is shown as the text it was typed as.
