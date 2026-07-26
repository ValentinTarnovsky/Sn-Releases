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

# Chat pagination of /history and /staffhistory.
history:
  # Entries shown per page by /history and /staffhistory.
  page-size: 8

# Bulk revert of the punishments a staff member issued inside a time window.
rollback:
  # Require the confirm token before /snbans rollback <staff> <time> reverts anything.
  require-confirm: true

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
  # Announce rollbacks; false leaves them to snbans.notify holders.
  rollback: true
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

### hierarchy

Reads the weight of the LuckPerms primary group and refuses a punishment against an equal or higher rank. `applies-to-unsanction` extends the check to `/unban`, `/unmute` and `/unblacklist`. `applies-to-alts` extends it to the two commands that read alt data, `/alts` and `/snbans match`.

{% hint style="info" %}
LuckPerms is optional on Paper and on Velocity. Without it the hierarchy check stays off with a single console warning, and every rank can punish every rank, whatever `enabled` says.
{% endhint %}

### alts.notify-states

The alt states that trigger the join warning. Accepted tokens are `BLACKLISTED`, `IPBANNED`, `BANNED`, `MUTED`, `ONLINE` and `OFFLINE`. An unknown token is warned about in console and skipped, and an empty list turns the warning off.

`ONLINE` and `OFFLINE` match every account, so listing either one warns on every single join.

### history.page-size

Entries per page of `/history` and `/staffhistory`. The value is clamped to the range 1 to 50. A value outside it is moved and warned about once per reload, since a silently clamped number is a setting you believe is in effect.

The ceiling exists because the value binds straight into a SQL `LIMIT`, and each returned row becomes a token map plus a rendered chat line.

### rollback.require-confirm

While `true`, `/snbans rollback <staff> <time>` only counts the matches and prints the confirm command to run. Nothing is reverted until `confirm` is appended.

{% hint style="danger" %}
Setting this to `false` removes the dry run, so the first invocation reverts immediately.
{% endhint %}

### sync.interval-seconds

Seconds between cross-server punishment polls, clamped to a minimum of 1. It only does something when several servers share one MySQL. The issuing server enforces a punishment immediately, and its peers pick the row up within this interval.

### retention.days

Days a login record is kept, clamped to a minimum of 1. The daily purge deletes older records. Punishment rows are never purged, whatever this value is.

### mute.blocked-commands

Commands a muted player cannot run, written without the leading slash. Entries are normalized like the aliases above: trimmed, slashes stripped, lower-cased and de-duplicated, in file order.

### broadcasts

One key per announced event. `false` leaves that event to `snbans.notify` holders instead of the whole server. An unban covers both a ban and an IP ban, and the same pairing applies to the other reverts.

`broadcasts.rollback` alone decides how visible a sweep is: `/snbans rollback` declares no `-s` or `-p` flag, so a sweep is never silent.

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
#    - Offences reverted by /snbans rollback do not count; expired and
#      manually removed ones do.
#    - A reason matching no template - or matching one of another type - is a
#      manual punishment: the duration typed on the command, or permanent when
#      none was given.
# ============================================================

# Example template: /ban <player> hacks escalates 1d, then 5d, then permanent.
hacks:
  # Punishment type this template issues: ban, mute or blacklist.
  type: ban
  # Escalation steps; the last one repeats for every further offence.
  ladder: [1d, 5d, permanent]

# Example template: /mute <player> spam escalates 30m, then 2h, then 1d.
spam:
  # Punishment type this template issues: ban, mute or blacklist.
  type: mute
  # Escalation steps; the last one repeats for every further offence.
  ladder: [30m, 2h, 1d]
```

### Writing a template

Each top-level key is a template id, matched against the full reason of the command and case-insensitively. `type` is `ban`, `mute` or `blacklist`. `ladder` is the escalation list, where every step is a duration token such as `30s`, `5m`, `2h` or `7d`, or the literal `permanent`.

Step 1 applies to the first offence, step 2 to the second, and the last step repeats for every further offence. Offences reverted by `/snbans rollback` do not count toward the step, while expired and manually removed ones do.

{% hint style="warning" %}
A template only applies to a command that issues the same type. `/ban Notch spam` against a `type: mute` template gets no template and no ladder, and with no typed duration becomes a permanent ban. Give the word its own ban template with a different id, or type a duration.
{% endhint %}

{% hint style="info" %}
This file is seeded once, so keys added by a future version never appear in an existing copy. Delete your file to get the shipped examples back, and read the release notes when a version changes the format.
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
```

### Enabling a webhook

There are nine blocks, one per event: `ban`, `ipban`, `unban`, `mute`, `ipmute`, `unmute`, `blacklist`, `unblacklist` and `rollback`. Each carries `enabled`, `url`, `content`, an `embed` section and `include-silent`.

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

A placeholder with no value is left as written.

{% hint style="info" %}
`{reason}` is the only staff-typed value, and it arrives neutralized. Color codes and `%placeholder%` markers are stripped, `@everyone`, `@here` and raw id mentions cannot ping, and markdown characters are escaped. Your own markdown in the file is untouched.
{% endhint %}

{% hint style="warning" %}
`include-silent` in the `rollback` block is inert. `/snbans rollback` declares no `-s` or `-p` flag and defers to `broadcasts.rollback` alone, so changing this key has no effect. It exists only to keep every block the same shape.
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
| `messages.format` | The words other messages splice in as placeholder values: `permanent`, `no-template`, `console`, the three `status-*` words behind `{status}`, and the nine `type-*` words behind `{type}`. |
| `messages.<event>` | One block per event (`ban`, `ipban`, `mute`, `ipmute`, `blacklist`, `unban`, `unmute`, `unblacklist`), each with `announce` for the public broadcast and `notify` for `snbans.notify` holders. `ban`, `ipban` and `blacklist` also carry `screen`, the disconnect screen the punished player sees. |
| `messages.alts` | The `/alts` listing and the join scan: `header`, `scan-header`, `legend`, `entry`, `none`, `footer` and the five `status-*` color prefixes behind `{color}`. |
| `messages.history` and `messages.staffhistory` | The two paged listings: `header`, `entry` and a `footer` carrying the clickable page arrows. |
| `messages.match` | The `/snbans match` listing: `header`, `entry`, `none` and `footer`. |
| `messages.rollback` | The sweep: `announce` and `notify` for the network, plus `dry-run`, `done`, `none` and `too-wide` for the staff member who ran it. |

### Editing rules

Never write `{prefix}` in a value. SnLib prepends the configured prefix to every single-line message, so a literal one renders twice.

A value written as a YAML list is sent line by line without the prefix. The single exception is an `announce` list holding one line, which is broadcast as a message and so gets it. A `notify` list never does, which is why every shipped `notify` value carries its own inline tag. Keep that tag if you shorten one.

{% hint style="danger" %}
Do not delete or blank one of the three `screen` blocks. A screen key with nothing in it refuses the player with an empty disconnect screen and no log line saying why.
{% endhint %}

{% hint style="warning" %}
Keep the `<click>` and `<hover>` tags in the two history footers, or the arrows still render but clicking does nothing. Keep `{label}` there too, so the buttons always dispatch the listing they belong to. In `rollback.dry-run`, keep both `{label}` and `{confirm}`, or you print a command the plugin refuses.
{% endhint %}

Player-typed values are sanitized before they reach a line. `{reason}`, `{value}` and the `{player}` of `unknown-player` have legacy color codes and `%placeholder%` markers removed, and MiniMessage tags escaped. A typed `<rainbow>` or `%player_name%` is shown as the text it was typed as.
