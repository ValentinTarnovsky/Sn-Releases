# Installation

1. Download the latest `snclans-v*` release from [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases/releases?q=snclans-).
2. Place the `.jar` file into your server's `plugins/` folder.
3. Restart the server.

{% hint style="warning" %}
Requires **SnLib** installed (`depend: [SnLib]`), version 1.4.0 or later.
{% endhint %}

## Dependencies

| Plugin | Required |
|--------|----------|
| SnLib | Yes |
| PlaceholderAPI | No (optional, enables placeholders) |
| WorldGuard | No (optional, enables region gating) |

{% hint style="info" %}
When PlaceholderAPI or WorldGuard is absent, the matching feature degrades gracefully. Placeholders stay raw and region rules are skipped.
{% endhint %}
