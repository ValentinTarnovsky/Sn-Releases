# SnGifts

SnGifts hands out a gift for every playtime milestone a player reaches that day. Players accrue
playtime while they are online, tiers unlock as they pass their thresholds, and each tier pays a
set of rewards drawn fresh every gift day. Everything resets on a clock you choose.

## Features

- **Playtime tiers.** Any number of gift tiers, each with its own `time-needed-minutes` and its own
  reward count. Order is by threshold ascending, so inserting a tier never renumbers the ones your
  players already know.
- **A menu you lay out yourself.** Placement lives in `guis/gifts.yml` as a layout mask plus named
  regions. Move, add or remove cells without ever touching a slot number.
- **One reward pool, one draw per day.** `rewards.yml` holds the pool. Each gift day every tier
  draws its own commands from it, and the draw is stored, so everyone who claims that tier on that
  day gets the same rewards.
- **Console commands and economy payouts.** A reward line is a console command with `{player}`
  substituted, or `vault:<amount>` to deposit through Vault instead.
- **A daily reset on your own clock.** Wall clock time plus any IANA timezone, with a server-wide
  announcement when the day rolls over.
- **Per address claim limits.** Cap how many times one IP can take the same gift each day, to blunt
  alt farming, with a bypass permission for staff.
- **Notifications.** Players are told the moment a gift unlocks, and reminded of unclaimed gifts
  every so many played minutes.
- **SQLite or MySQL.** SQLite needs no setup. All database and file work happens off the main
  thread.

{% hint style="warning" %}
SnGifts requires **SnLib 1.27.0 or newer** and refuses to enable on an older one. See
[Installation](installation.md).
{% endhint %}

## Optional integrations

- **Vault**: enables `vault:<amount>` reward lines, which deposit money instead of running a
  command. Without an economy those lines are skipped and the player is told, while the rest of the
  gift still runs.
- **PlaceholderAPI**: resolves `%placeholder%` tokens inside the menu's static items. SnGifts
  registers no placeholders of its own, and everything works without PlaceholderAPI installed.

## Links

- Source and releases: part of the shared [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases) repo
