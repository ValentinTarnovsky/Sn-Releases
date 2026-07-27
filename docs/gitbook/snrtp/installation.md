# Installation

1. Download the latest `snrtp-v*` release from [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases/releases?q=snrtp-).
2. Place the `.jar` file into your server's `plugins/` folder.
3. Restart the server. SnRTP creates `plugins/.Sn-License/license.yml` and stops, because the file
   still holds a placeholder.
4. Paste your Sn bundle key on the first non-comment line of that file and restart again.

{% hint style="warning" %}
Requires **SnLib** installed (`depend: [SnLib]`), version 1.20.0 or later. SnRTP 1.4.0 needs
API level 12 and disables itself with a console notice on an older SnLib, so update both jars
together.
{% endhint %}

## License

SnRTP is part of the Sn **bundle**: one subscription key unlocks every plugin in the pack. The key
goes in a single shared file, `plugins/.Sn-License/license.yml` - paste it once per server and
every bundle plugin reads it, not once per plugin.

```yaml
# Paste your Sn license key on the line below (replace the placeholder).
YOUR-BUNDLE-KEY-HERE
```

The check runs exactly once, at startup, against the Sn license backend. If it fails the console
shows `[Sn] License: FAIL` and SnRTP disables itself. There are no runtime checks afterwards: once
the plugin is up it stays up for the whole session, so a network hiccup later changes nothing. The
server does need outbound HTTPS while booting.

{% hint style="warning" %}
Use the jar exactly as downloaded from the release. The license is bound to the released build's
checksum, so a repackaged or modified jar fails validation.
{% endhint %}

## Dependencies

| Plugin | Required |
|--------|----------|
| SnLib | Yes |
| WorldGuard | Optional (protected-region avoidance) |
| WorldEdit | Optional (used by WorldGuard) |
| ProtectionStones | Optional (claims avoided through WorldGuard) |

{% hint style="info" %}
Install WorldGuard to make random teleports avoid protected land. ProtectionStones claims are WorldGuard regions, so WorldGuard alone covers them.
{% endhint %}
