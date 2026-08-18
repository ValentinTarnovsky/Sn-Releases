# Installation

1. Download the latest `snenvoys-v*` release from [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases/releases?q=snenvoys-).
2. Place the `.jar` file into your server's `plugins/` folder.
3. Start the server once. SnEnvoys writes its config files and creates
   `plugins/.Sn-License/license.yml` if it does not already exist, then disables itself because
   that file still holds a placeholder.
4. Paste your Sn bundle key on the first non-comment line of `plugins/.Sn-License/license.yml`.
5. Restart the server; SnEnvoys validates the key at startup.

{% hint style="info" %}
SnEnvoys is part of the licensed bundle. One shared bundle key unlocks every bundle plugin on
the server, so if you already run another Sn bundle plugin here, step 3-4 is already done.
{% endhint %}

{% hint style="warning" %}
Requires **SnLib** installed (`depend: [SnLib]`), version 1.28.0 or later.
{% endhint %}

## Dependencies

| Plugin | Required |
|--------|----------|
| SnLib  | Yes      |
| PlaceholderAPI | No (unlocks the countdown placeholders) |
| WorldGuard | No (unlocks region avoidance for Supply Drop) |

## First boot

```
plugins/
  .Sn-License/
    license.yml           your bundle key - shared by every Sn bundle plugin on this server
  SnEnvoys/
    config.yml             envoy timing, Supply Drop, rewards, claims, API
    lang/
      messages_en.yml       every line the plugin sends to players and admins
    locations.yml           envoy spawn points registered with /envoy editor
```

There is no `messages.yml` and no `config-version` key anywhere: SnLib merges new keys into
`config.yml` and `lang/messages_en.yml` on every boot and keeps your values and comments.
`locations.yml` is different: it is player data, never merged or re-seeded, so editor changes
always stick.

## Checking it worked

- The console shows no `Unknown dependency SnLib` and no `[Sn] License: FAIL`.
- With PlaceholderAPI installed you also get a line confirming the `snenvoys` expansion
  registered.
- `/envoy` as a player shows the countdown to the next event.
- `/envoy editor` as an admin hands you diamond blocks and enters editing mode.
