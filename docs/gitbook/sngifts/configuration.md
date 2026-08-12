# Configuration

SnGifts ships four files:

| File | What it is | Managed |
|---|---|---|
| `config.yml` | Tiers, reset clock, claim limits, playtime, database, sounds | Yes |
| `lang/messages_en.yml` | Every line the plugin sends to a player or an admin | Yes |
| `guis/gifts.yml` | The gifts menu: layout, regions, templates | Yes |
| `rewards.yml` | The reward pool | No, this file is yours |

Managed means new keys are merged into your file on boot. Your values, your comments and your own
additions are preserved, and example entries you delete stay deleted. Sections marked
`# sn:extensible` are the ones whose entries are yours.

{% hint style="info" %}
There is no `config-version` key anywhere, and there is never anything to migrate by hand. Set
`update-configs: false` to freeze every managed file. SnLib then only warns about missing keys.
{% endhint %}

## config.yml

### Language, updates and debug

```yaml
# Active language code; loads lang/messages_<code>.yml (falls back to en).
lang: en

# Master switch of the SnLib auto-updater for this plugin's managed files.
update-configs: true

# Runtime debug output (also toggleable live via /gifts debug).
debug:
  # Master toggle of the debug output.
  enabled: false
  # Verbosity threshold: OFF, INFO, DEBUG or TRACE.
  level: DEBUG
  # Category filter; an empty list lets every category through.
  categories: []
```

### Command

```yaml
command:
  # Aliases of /gifts. Re-read on /gifts reload.
  aliases: [regalos]
```

The aliases here are authoritative at runtime, and the `plugin.yml` list is only the fallback used
when the key is missing. Rename `regalos`, add more, or empty the list entirely.

### Database

Five things survive a restart: a player's playtime, the tiers they claimed, the day's drawn
commands, the last reset date, and the per IP counters. SQLite needs no setup. MySQL reads the
connection block.

```yaml
database:
  # sqlite or mysql
  type: sqlite
  # MySQL connection (ignored when type is sqlite).
  host: localhost
  port: 3306
  database: sngifts
  username: root
  password: ""
```

{% hint style="info" %}
The connection pool is not configurable and never appears in this file. SnLib sizes it.
{% endhint %}

### Gift tiers

This is the section you will edit most. Each entry is one claimable gift.

```yaml
gifts:
  # sn:extensible
  tiers:
    gift1:
      # Minutes of playtime needed today before this gift can be claimed.
      time-needed-minutes: 60
      # How many commands are drawn from rewards.yml for this tier each day.
      amount-of-rewards: 1
    gift2:
      time-needed-minutes: 120
      amount-of-rewards: 2
    gift3:
      time-needed-minutes: 180
      amount-of-rewards: 2
```

Three rules are worth knowing before you rearrange anything:

- **Order is by `time-needed-minutes` ascending**, ties broken by id. It is never the order you
  write the entries in. That order decides both which menu cell a tier lands in and the
  `{gift_index}` number shown on the item.
- **The entry id is what is stored against a player's claim.** Renaming an entry makes it claimable
  again for everyone who had already taken it.
- **`gifts.tiers` is marked `# sn:extensible`.** The three shipped examples are yours to rename or
  delete, and a deleted entry never comes back on the next boot.

Add a fourth tier by adding a fourth entry, then give the menu a fourth `g` cell in
`guis/gifts.yml`.

### Daily reset

```yaml
reset:
  # Timezone of the reset clock (any IANA zone id). Invalid values fall back to UTC.
  timezone: UTC
  # Wall-clock time of the daily reset, HH:mm. Invalid values fall back to 00:00.
  time: "00:00"
```

When the day rolls over, claims clear, playtime returns to zero, a fresh reward set is drawn, and
`lists.reset-broadcast` goes out to the whole server. Set `timezone` to your own zone, for example
`Europe/Madrid` or `America/Argentina/Buenos_Aires`, so the reset lands where your players expect
it.

### Notifications

```yaml
notifications:
  # Remind a player of unclaimed gifts every N minutes of playtime. 0 disables.
  unclaimed-reminder-minutes: 60
```

The interval counts played minutes, not wall clock minutes, so an idle offline player is never
reminded. The unlock notice is separate and always fires the moment a tier becomes claimable.

### Claim limits

```yaml
claim:
  # Max claims of the SAME gift from one IP per day. 0 or less disables the limit.
  # Players with sngifts.claim.bypass-ip are never counted.
  ip-limit-per-gift: 1
```

{% hint style="warning" %}
This counts the address a player connects from, so everyone sharing one address shares one
allowance: siblings on a home connection, a school network, and mobile players behind carrier grade
NAT. At `1` the second person on such a connection cannot claim a gift the first one took.

Behind a BungeeCord or Velocity proxy it is only correct when the backend server has IP forwarding
enabled. Without forwarding every player appears to connect from the proxy itself, so the first
claim of a gift consumes the allowance for the whole network until the next daily reset. Raise the
limit or set `0` if you cannot enable forwarding.
{% endhint %}

### Playtime and storage

```yaml
playtime:
  # How often accrued playtime is counted, in seconds.
  tick-interval-seconds: 60

storage:
  # How often accrued playtime is flushed to the database, in minutes.
  # Claims are always written through immediately and never wait for this.
  autosave-interval-minutes: 10
```

A claim is never at risk from the autosave interval. It is written through the moment it happens.
The interval only decides how much accrued playtime a hard crash can cost.

### Sounds

```yaml
sounds:
  # Played when a claim is refused.
  locked: ENTITY_VILLAGER_NO
  # Played when a claim succeeds.
  claimed: ENTITY_EXPERIENCE_ORB_PICKUP
  volume: 1.0
  pitch: 1.0
```

Any Bukkit sound name works. An unknown name is ignored silently.

## rewards.yml

The pool every tier draws from. This file is not managed: SnGifts writes it once and never touches
it again.

```yaml
rewards:
  - "give {player} diamond 1"
  - "give {player} emerald 5"
  - "give {player} netherite_ingot 1"
  - "give {player} golden_apple 2"
  - "vault:500"
```

| Syntax | What it does |
|---|---|
| `{player}` | Replaced with the claiming player's name |
| `vault:<amount>` | Deposits that amount through Vault instead of running a command |
| anything else | Run as a console command |

How the draw works: once per day, each tier draws its own `amount-of-rewards` lines from this pool
at random. A tier never draws the same line twice, but two different tiers can draw the same line
on the same day, because the pool is shared. The draw is stored, so a mid-day restart keeps what
players already see. Only the daily reset and `/gifts resetgifts` reroll it.

{% hint style="warning" %}
Put more lines in the pool than your largest `amount-of-rewards`, or a tier cannot fill its draw.
Replace the shipped examples with commands your server actually has.
{% endhint %}

## guis/gifts.yml

The menu is a layout mask plus named regions, so no slot number ever appears in your config.

```yaml
title: "&8Daily Gifts (&e{time-played}&8)"
rows: 3

# Ticks between repaints of the gift cells and the info cell. 0 turns the
# repaint off entirely (states then update only when you reopen the menu).
update-interval: 20

layout:
  - "fffgggggg"
  - "fggg    f"
  - "ffffiffff"

regions:
  gifts: g
  info: i
```

`f` is the filler, `g` is a gift cell, `i` is the info item, and a space is an empty slot. The tier
with the lowest `time-needed-minutes` takes the first `g` in reading order, then the next, and so
on. More tiers than cells shows the first ones. More cells than tiers leaves the rest empty.

{% hint style="info" %}
The title is resolved when the menu opens and is not repainted. A title cannot be written into an
open window, so a live one would blink the whole inventory every minute. The info item carries the
value that ticks.
{% endhint %}

### Templates

One template per state. The plugin picks which one each cell gets, per player.

```yaml
templates:
  locked:
    material: CHEST
    display-name: "&cGift {gift_index}"
    lore:
      - "&7Status: {status}"
      - "&7Missing: &f{time-left}"
    click-actions:
      - "[gift-claim] {gift_id}"

  unlocked:
    material: ENDER_CHEST
    display-name: "&aGift {gift_index}"
    glow: true
    flags:
      - HIDE_ENCHANTS
    lore:
      - "&7Status: {status}"
      - "&eClick to claim!"
    click-actions:
      - "[gift-claim] {gift_id}"

  claimed:
    material: MINECART
    display-name: "&7Claimed Gift"
    lore:
      - "&7Status: {status}"
      - "&7Already claimed."
    click-actions:
      - "[gift-claim] {gift_id}"

  info:
    material: CLOCK
    display-name: "&eYour Time"
    lore:
      - "&7You played: &f{time-played}"
      - "&7Next reset: &f{time-until-reset}"
```

| Placeholder | Where | Meaning |
|---|---|---|
| `{gift_id}` | gift templates | The tier's id from `config.yml` |
| `{gift_index}` | gift templates | The tier's position in threshold order, starting at 1 |
| `{status}` | gift templates | The state word from the `status:` section of the language file |
| `{time-left}` | gift templates | Playtime still missing for this tier |
| `{time-total}` | gift templates | The tier's full requirement |
| `{time-played}` | gift templates, info, title | Playtime accrued today |
| `{time-until-reset}` | gift templates, info | Time left until the daily reset |

{% hint style="danger" %}
Keep `[gift-claim] {gift_id}` on all three gift templates. Every gift is clickable in every state on
purpose: the plugin answers a click on a locked or already claimed gift with the reason and the
`sounds.locked` feedback. A template without it makes that cell silently do nothing when clicked.
{% endhint %}

`material` also accepts `texture-<base64>`, `texture:<base64>` and `basehead-<base64>` for a custom
player head.

### Static items

```yaml
items:
  filler:
    key: f
    material: GRAY_STAINED_GLASS_PANE
    display-name: " "
```

Plain decoration. Static items are not bound by the plugin and resolve none of the `{token}`
placeholders above, only `%papi%` ones.

## lang/messages_en.yml

Every player facing string lives here. Colors accept `&a` legacy codes, `&#RRGGBB` and `<#RRGGBB>`
hex, and `[rgb]` gradients.

The file has five parts:

| Section | What it holds |
|---|---|
| `prefix` | Prepended automatically to every single line message |
| `snlib` | SnLib's shared command contract: no permission, usage, help header and so on |
| `messages` | The plugin's own single line messages |
| `lists` | Multi line messages, sent without the prefix |
| `status` | The short state words |
| `time-format` | How durations are rendered |

### State words

```yaml
status:
  locked: "&cLocked"
  unlocked: "&aUnlocked"
  claimed: "&7Claimed"
  enabled: "&aEnabled"
  disabled: "&cDisabled"
  unknown: "Unknown"
```

These are substituted into the `{status}` slot of the chat messages and of the menu templates
alike, so restyling a word here restyles it everywhere at once.

### Durations

```yaml
time-format:
  zero: "0m"
  minutes: "{minutes}m"
  hours: "{hours}h"
  hours-minutes: "{hours}h {minutes}m"
  seconds: "{seconds}s"
  minutes-seconds: "{minutes}m {seconds}s"
  hours-minutes-seconds: "{hours}h {minutes}m {seconds}s"
```

Two formatters read these. The minute based one renders playtime and time remaining, and never
emits seconds or days. The second based one renders the time until reset, and never uses the `zero`
key.

{% hint style="info" %}
Never write `{prefix}` in a value. The prefix is prepended automatically, and a literal one renders
verbatim. Lines under `lists` are sent bare, which is why a single line list value carries
`[noprefix]` to pin it.
{% endhint %}
