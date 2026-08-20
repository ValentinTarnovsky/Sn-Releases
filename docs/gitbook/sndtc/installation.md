# Installation

1. Download the latest `sndtc-v*` release from [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases/releases?q=sndtc-).
2. Place the `.jar` file into your server's `plugins/` folder.
3. Start the server once. SnDTC writes its config files and creates
   `plugins/.Sn-License/license.yml` if it does not already exist, then disables itself because
   that file still holds a placeholder.
4. Paste your Sn bundle key on the first non-comment line of `plugins/.Sn-License/license.yml`.
5. Restart the server; SnDTC validates the key at startup.

{% hint style="info" %}
SnDTC is part of the licensed bundle. One shared bundle key unlocks every bundle plugin on the
server, so if you already run another Sn bundle plugin here, steps 3-4 are already done.
{% endhint %}

{% hint style="warning" %}
Requires **SnLib** installed (`depend: [SnLib]`), version 1.29.0 or later. SnDTC refuses to
enable against an older engine, so update `SnLib.jar` alongside it.
{% endhint %}

## Dependencies

| Plugin | Required |
|--------|----------|
| SnLib  | Yes      |
| PlaceholderAPI | No (unlocks the `%sndtc_*%` placeholders) |

## First boot

```
plugins/
  .Sn-License/
    license.yml           your bundle key - shared by every Sn bundle plugin on this server
  SnDTC/
    config.yml             defaults for new cores, limits, display, holograms, rewards
    dtcs.yml               the cores themselves - yours to edit by hand
    lang/
      messages_en.yml       every line the plugin sends to players and admins
```

## Your first core

1. Place the block you want to use, and look at it from within 5 blocks.
2. `/sndtc create spawn`

That is the whole setup. The core is alive at the config default health, on the config default
schedule, wearing the config default materials. Adjust it with `/sndtc sethealth`,
`/sndtc setcron`, `/sndtc setblock` and `/sndtc setrange`, or edit its section in `dtcs.yml` and
run `/sndtc reload`.

{% hint style="warning" %}
**Work out what your rewards cost before you go live.** How often a payout fires is decided by
`defaults.max-health` and `defaults.cron` together, not by the `rewards` block. The shipped
values are deliberately quick so you can see the plugin work - they are not an economy. See
[Configuration](configuration.md).
{% endhint %}
