# Installation

## Requirements

| | |
|---|---|
| Java | 21 or newer |
| Server | Paper 1.20 or newer |
| Required | `SnLib.jar` (SnLib 1.27.0 or newer) |
| Optional | Vault, PlaceholderAPI |
| Database | SQLite (default, no setup) or MySQL |
| License | Yes - SnGifts is part of the licensed bundle |

The jar is compiled for Java 21. A server still running Java 17 refuses to load it, so move the
server to Java 21 first.

{% hint style="warning" %}
The SnLib floor is not a soft warning. The required API level is compiled into this jar, so SnGifts
refuses to enable on any server whose `SnLib.jar` is older than 1.27.0, and reports it in the
console. Update `SnLib.jar` before you drop SnGifts in.
{% endhint %}

## Steps

1. Download `SnGifts-<version>.jar` from the releases page (tags prefixed `sngifts-`).
2. Put it in `plugins/`, together with `SnLib.jar`.
3. Start the server. SnGifts writes its files, creates `plugins/.Sn-License/license.yml` and then
   disables itself, because that file still holds a placeholder.
4. Paste your bundle key on the first non-comment line of `plugins/.Sn-License/license.yml`.
5. Restart the server. SnGifts validates the key at startup.
6. Edit `plugins/SnGifts/config.yml` and `plugins/SnGifts/rewards.yml`, then run `/gifts reload`.

{% hint style="info" %}
One key unlocks the whole bundle. `plugins/.Sn-License/license.yml` is shared by every Sn bundle
plugin on the server, so if you already run one of them the file exists and SnGifts reads the key
that is already there. Nothing to paste twice.
{% endhint %}

{% hint style="info" %}
The key is checked once, at startup. SnGifts refuses to enable without a valid one, so the server
log is the place to look if the plugin is missing from `/plugins`.
{% endhint %}

{% hint style="warning" %}
Updating **SnLib** always needs a full restart, never a `/reload`.
{% endhint %}

## First boot

```
plugins/
  .Sn-License/
    license.yml          your bundle key - shared by every Sn bundle plugin on this server
  SnGifts/
    config.yml           tiers, reset clock, claim limits, playtime, database, sounds
    rewards.yml          the reward pool every tier draws from
    guis/
      gifts.yml          the layout, the templates and the info item of the gifts menu
    lang/
      messages_en.yml    every line the plugin sends to players and admins
    database.db          SQLite database, when database.type is sqlite
```

Nothing has to be created by hand. Every file above is written on first boot.

{% hint style="info" %}
The shipped `rewards.yml` gives diamonds, emeralds and a `vault:500` payout as examples. Replace
them with commands your server actually has before you let players claim.
{% endhint %}

## Dependencies

| Plugin | Required |
|--------|----------|
| SnLib | Yes |
| Vault | No |
| PlaceholderAPI | No |

## Updating

Replace the jar and restart. New keys are merged into your `config.yml`, `lang/messages_en.yml` and
`guis/gifts.yml` on boot, and your values, comments and additions are preserved. Tiers you deleted
under `gifts.tiers` stay deleted.

{% hint style="info" %}
`rewards.yml` is yours. SnGifts writes it once on first boot and never touches it again, so nothing
is ever merged into your reward pool.
{% endhint %}
