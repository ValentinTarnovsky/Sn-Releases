# Installation

1. Download the latest `snrtp-v*` release from [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases/releases?q=snrtp-).
2. Place the `.jar` file into your server's `plugins/` folder.
3. Restart the server.

{% hint style="warning" %}
Requires **SnLib** installed (`depend: [SnLib]`), version 1.12.0 or later.
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
