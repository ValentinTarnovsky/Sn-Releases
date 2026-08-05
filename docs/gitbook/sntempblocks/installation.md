# Installation

1. Download the latest `sntempblocks-v*` release from [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases/releases?q=sntempblocks-).
2. Place the `.jar` file into your server's `plugins/` folder.
3. Start the server once. The plugin creates `plugins/.Sn-License/license.yml`.
4. Paste your Sn license key into that file, replacing the placeholder line.
5. Restart the server. The plugin validates the key at startup.

{% hint style="warning" %}
Requires **SnLib** installed (`depend: [SnLib]`), version 1.24.0 or later.
{% endhint %}

{% hint style="warning" %}
Requires **WorldGuard**. Zones are WorldGuard regions, so there is nothing to resolve without it
and the plugin refuses to enable.
{% endhint %}

{% hint style="info" %}
SnTempBlocks is a bundle plugin. Its key lives in the shared `plugins/.Sn-License/license.yml`,
not in the plugin folder, and one bundle key unlocks every plugin of the bundle on that server.
The plugin refuses to enable without a valid key.
{% endhint %}

## Dependencies

| Plugin | Required |
|--------|----------|
| SnLib | Yes |
| WorldGuard | Yes |
| PlaceholderAPI | No |

## First run

The plugin seeds `plugins/SnTempBlocks/zones.yml` with two example zones that point at regions
your server probably does not have. Both are reported in the console and disabled until you edit
them. Open the file, point each zone at one of your own WorldGuard regions, and run
`/tempblocks reload`.

{% hint style="info" %}
`zones.yml` is yours. It is seeded once and never modified again, so updates never rename, add or
remove anything you wrote there.
{% endhint %}
