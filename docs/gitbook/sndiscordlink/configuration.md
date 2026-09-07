# Configuration

SnDiscordLink ships with the following YAML files. New keys are auto-merged on boot, and your edits and comments are preserved.

{% hint style="warning" %}
On a network, every server must ship the SAME `config.yml`. The only per-server value is the `SNDISCORDLINK_SERVER` environment variable, used by `rewards.claim-on`.
{% endhint %}

## config.yml

```yaml
# ============================================================
#  SnDiscordLink - configuration
#  Managed by SnLib: new keys are auto-merged on boot; your values and
#  comments are preserved. Do NOT add a config-version key (retired).
#  Set update-configs: false to freeze this file (SnLib only warns about
#  missing keys instead of inserting them).
#  Sections marked "# sn:extensible" are yours: entries you delete there
#  stay deleted.
# ============================================================

# Active language code; loads lang/messages_<code>.yml (falls back to en).
lang: en

# Master switch of the SnLib auto-updater for this plugin's managed files.
update-configs: true

# Runtime debug output (also toggleable live via /discord debug).
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
  # Aliases of /discord. Re-read on /discord reload.
  aliases: [dl, discordlink]

# ------------------------------------------------------------
#  Database. type=sqlite needs nothing else; type=mysql reads host/port/etc.
# ------------------------------------------------------------
database:
  # sqlite or mysql
  type: sqlite
  # MySQL connection (ignored when type is sqlite).
  host: localhost
  port: 3306
  database: sndiscordlink
  username: root
  password: ""

# ------------------------------------------------------------
#  Discord bot. One bot per network: servers sharing the same MySQL share
#  every link, code, reward and cooldown and elect ONE of them to run the
#  bot through a database lease; the others only read the links.
# ------------------------------------------------------------
discord:
  # Bot token: paste it here, or write env:VARIABLE_NAME to read it from that environment variable.
  token: ""
  # Id of the Discord server (guild) the bot serves, kept in quotes; slash commands register there.
  guild-id: ""
  # auto elects ONE server through the shared database lease; always forces this server to run
  # the bot (single-server installs only: two `always` = two bots on one token); never disables
  # the bot here while this server still issues codes and runs rewards.
  mode: auto
  # Names of the slash commands the bot registers in Discord.
  commands:
    # Links a Minecraft account with a code, or issues a code when run without one.
    link: link
    # Unlinks the caller's Minecraft account.
    unlink: unlink
    # Staff group (info, unlink, accounts, stats); needs Manage Server in Discord.
    admin: dlink

# Rules of the linking flow itself.
linking:
  # How long a code stays valid (5m, 30s, 1h30m, 7d).
  code-ttl: 5m
  # Minimum time between two code requests (in game or in Discord) and between two
  # /discord verify attempts by the same player; 0 disables it.
  request-cooldown: 30s
  # Minecraft accounts one Discord user may link.
  max-accounts-per-discord: 1
  # Wrong codes one Discord user may try inside failed-attempts-window (below) before being
  # blocked for the rest of that window.
  max-failed-attempts: 5
  # Length of that window; wrong attempts older than this are forgotten.
  failed-attempts-window: 5m

# ------------------------------------------------------------
#  Features built on the link: the linked role, in-game claims, rank
#  sync, role rewards and the action lists run around a link.
# ------------------------------------------------------------
linked-role:
  # Discord role given on link and removed on unlink; empty disables it.
  role-id: ""
  # Set the Minecraft name as the Discord nickname on link, and CLEAR it on unlink, which
  # leaves the member showing their plain Discord username. A nickname they had chosen before
  # linking is overwritten and not remembered, so leave this off if your members set their own.
  nickname-sync: false

# Rewards a linked player claims in game with /discord claim <claim>; /booster is the
# shortcut for the booster entry. Each entry names what the member must have
# (requires), how often it can be claimed (cooldown) and the actions to run.
# The entry key is the claim id (at most 32 characters, never starting with $).
# sn:extensible
claims:
  booster:
    # booster = the member boosts the Discord server, or a Discord role id the member must hold.
    requires: booster
    # Time between two claims of this entry by the same player; 0 allows claiming again at once.
    cooldown: 7d
    # Actions run on the claiming player; {player} is the Minecraft name.
    actions:
      - "[console] give {player} diamond 8"
      - "[message] &aThanks for boosting! Your reward was delivered."

# LuckPerms group -> Discord role id (in quotes). Only PERMANENT groups count, and only the
# roles mapped here are ever added or removed by the bot. Never map the same group/role pair
# here AND in role-rewards: the two would feed each other forever.
# sn:extensible
rank-roles:
  vip: "000000000000000000"
  mvp: "000000000000000000"

# Discord role id (in quotes) -> actions run in game when a linked member gains or loses the
# role. Never mirror a rank-roles pair here (loop guard).
# sn:extensible
role-rewards:
  "000000000000000000":
    # Run once when the member gains the role, the next time the player is online.
    on-gain:
      - "[console] lp user {player} parent add booster"
    # Run once when the member loses the role, the next time the player is online.
    on-lose:
      - "[console] lp user {player} parent remove booster"

# Action lists run around the link itself; {player} is the Minecraft name.
rewards:
  # Run once per account, the first time it links (never again on a relink).
  link:
    - "[console] eco give {player} 1000"
  # Run every time the account gets unlinked (after the on-lose lists of the role-rewards
  # the member had been granted).
  unlink: []
  # When a linked member leaves the Discord server.
  leave:
    # Remove the link automatically when the member leaves (the unlink actions run too).
    auto-unlink: false
    # Actions run on the player when the member leaves.
    actions: []
  # Server names allowed to deliver these rewards, matched against the SNDISCORDLINK_SERVER
  # environment variable of each server; empty = any server. Rows (rewards AND notices)
  # wait until the player is on an allowed server. Prefer network-wide actions such as
  # LuckPerms or a shared economy when empty.
  claim-on: []

# ------------------------------------------------------------
#  Sync intervals. Lower values react faster and query the database more.
# ------------------------------------------------------------
sync:
  # Seconds between checks for a code redeemed in Discord while the player waits after /link.
  watcher-seconds: 5
  # Seconds between refreshes of the Discord roles of the players online on this server.
  refresh-seconds: 30
  # Seconds between passes of the bot host over rank changes waiting to reach Discord.
  dirty-seconds: 10
  # Minutes between full reconciles of every link by the bot host.
  full-minutes: 60
  # Seconds between refreshes of the global link count behind %sndiscordlink_linked_count%.
  count-seconds: 60

# ------------------------------------------------------------
#  Integrations.
# ------------------------------------------------------------
luckperms:
  # Read LuckPerms groups for rank-roles; off degrades the plugin to linking only.
  enabled: true

# Discord webhook log of link events. The bot host posts all of them except claimed, which
# the backend the player claimed on posts, so keep webhook.url identical on every server.
webhook:
  # Webhook URL; empty disables the log.
  url: ""
  # Events posted to the webhook.
  events:
    # A player linked an account.
    linked: true
    # A player or a staff member unlinked an account.
    unlinked: true
    # A player claimed a reward.
    claimed: true
    # A linked member left the Discord server.
    left: true

# ------------------------------------------------------------
#  Feedback.
# ------------------------------------------------------------
broadcasts:
  # Announce to the whole server the FIRST time an account ever links (lang key
  # broadcast-linked). It shares that once-per-account marker with rewards.link, so an
  # unlink and relink is not announced again.
  linked: true
```

## Notable settings

### database.type

`sqlite` keeps everything in a local file and means a single server. `mysql` is what a network needs: links, codes, rewards, cooldowns and the bot lease all live in that one database, and the servers coordinate through it.

### discord.mode

`auto` lets the servers elect one bot host through a database lease, with failover in about 30 seconds. `always` forces this server to run the bot and belongs to single-server installs only. `never` keeps this server from ever hosting, while it still issues codes and delivers rewards.

### linking.code-ttl

How long a generated code stays valid, written as `5m`, `30s` or `1h30m`. A code is single use and is consumed the moment it is redeemed.

### linking.max-accounts-per-discord

How many Minecraft accounts one Discord user may link. Raise it above `1` and the member keeps a mapped role while any of the linked accounts justifies it.

### linking.max-failed-attempts

How many wrong codes one Discord user may try inside `failed-attempts-window` before being blocked for the rest of that window. It is the brute-force guard on the 6-digit code space.

### linked-role.nickname-sync

Sets the Minecraft name as the member's Discord nickname on link, and clears it on unlink. A nickname the member had chosen before linking is overwritten and not remembered, so leave this off when your members set their own.

### claims

Each entry is one reward claimed in game with `/discord claim <entry>`. `requires` is either `booster` or a Discord role id the member must hold, `cooldown` is the time between two claims by the same player, and `actions` is what runs. The entry key is the claim id.

### rank-roles

Maps a LuckPerms group to a Discord role id. Only PERMANENT group memberships count, and only the roles listed here are ever added or removed by the bot.

{% hint style="danger" %}
Never map the same group and role pair in both `rank-roles` and `role-rewards`. The two would feed each other in a loop.
{% endhint %}

### role-rewards

Maps a Discord role id to `on-gain` and `on-lose` action lists. Each list runs once, the next time the player is online, on the server where they are online.

### rewards.link

Runs once per account, the first time it ever links. A later unlink and relink does not run it again, and `broadcasts.linked` shares that same once-per-account marker.

### rewards.leave.auto-unlink

When a linked member leaves your Discord server, `true` removes the link and runs the unlink actions too. `false` keeps the link and runs only `rewards.leave.actions`.

### rewards.claim-on

The server names allowed to deliver rewards and notices, matched against the `SNDISCORDLINK_SERVER` environment variable. Leave it empty and any server delivers, which is the right choice when your actions are network-wide, such as LuckPerms or a shared economy.

### sync

Lower values react faster and query the database more often. `watcher-seconds` is how quickly a code redeemed in Discord reaches the waiting player, `refresh-seconds` how often the Discord roles of online players are refreshed, and `full-minutes` how often the bot host reconciles every link.

### webhook.url

An embed log of link events posted to a Discord webhook. Keep this URL identical on every server: the bot host posts every event except `claimed`, which is posted by the server the player claimed on.

## Other managed YAML

This file is also auto-merged on boot, so your edits and comments survive updates.

- `lang/messages_en.yml`: every player-facing message. Copy it to `messages_<code>.yml` and set `lang` in `config.yml` to add a language.

### State words

The `status:` block of `lang/messages_en.yml` holds the state words: linked yes and no, boosting yes and no, `none` and `unknown`. One block feeds `/discord status`, `/discord info`, the Discord replies and the placeholders, so editing a word once changes it everywhere.

### Owner-owned sections

`claims`, `rank-roles` and `role-rewards` are marked `# sn:extensible` in `config.yml`. Entries you delete there stay deleted, and the auto-merge never puts the sample entries back.
