# Installation

1. Download the latest `snkills-v*` release from [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases/releases?q=snkills-).
2. Place the `.jar` file into your server's `plugins/` folder.
3. Restart the server. SnKills creates `plugins/.Sn-License/license.yml` and stops, because the
   file still holds a placeholder.
4. Paste your Sn bundle key on the first non-comment line of that file and restart again.

{% hint style="warning" %}
Requires **SnLib** installed (`depend: [SnLib]`), version 1.20.3 or later. SnKills 2.0.0 needs
API level 12 and disables itself with a console notice on an older SnLib, so update both jars
together.
{% endhint %}

## License

SnKills is part of the Sn **bundle**: one subscription key unlocks every plugin in the pack.
The key goes in a single shared file, `plugins/.Sn-License/license.yml` - paste it once per
server and every bundle plugin reads it, not once per plugin.

```yaml
# Paste your Sn license key on the line below (replace the placeholder).
YOUR-BUNDLE-KEY-HERE
```

The check runs exactly once, at startup, against the Sn license backend. If it fails the
console shows `[Sn] License: FAIL` and SnKills disables itself. There are no runtime checks
afterwards: once the plugin is up it stays up for the whole session, so a network hiccup later
changes nothing. The server does need outbound HTTPS while booting.

## First run

Out of the box SnKills ships two populated templates:

- `PLAYER_KILL` - a PvP line with the killer's weapon and its tooltip.
- `FALLBACK` - a plain `%victim% died.` for everything else.

Every individual damage cause ships empty, which means it uses the fallback. Fill in the ones
you care about and leave the rest alone.
