# Installation

1. Download the latest `snrotatinheads-v*` release from [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases/releases?q=snrotatinheads-).
2. Place the `.jar` file into your server's `plugins/` folder.
3. Start the server once. SnRotatinHeads creates a shared license file at `plugins/.Sn-License/license.yml`.
4. Paste your Sn license key into that file, replacing the placeholder line.
5. Restart the server. SnRotatinHeads validates the key at startup and then enables.

{% hint style="warning" %}
Requires **SnLib** installed (`depend: [SnLib]`), version 1.28.0 or later. SnRotatinHeads refuses to enable against an older engine, so update `SnLib.jar` at the same time.
{% endhint %}

{% hint style="info" %}
SnRotatinHeads is licensed. The key in `plugins/.Sn-License/license.yml` is shared by every bundled Sn plugin on the server, so you paste it once. Without a valid key the plugin refuses to enable.
{% endhint %}

## Dependencies

| Plugin | Required |
|--------|----------|
| SnLib | Yes |
| DecentHolograms | No (optional, enables the text labels above heads) |
| PlaceholderAPI | No (optional, enables placeholders in messages, labels and click actions) |

{% hint style="info" %}
When an optional plugin is absent, the matching feature degrades gracefully. Without DecentHolograms heads spin, bounce and click normally and simply show no label; without PlaceholderAPI placeholders stay as typed.
{% endhint %}

**Requirements:** Java 21+, Paper 1.20.x or 1.21.x

## First boot

SnRotatinHeads generates `config.yml` and `lang/messages_en.yml` on first boot. New keys are auto-merged on later boots, and your values and comments are preserved. `heads.yml` appears after your first `/rh create`: it is data, written by the plugin after every change and never seeded, merged or regenerated, so you can edit it by hand and run `/rh reload`.

{% hint style="info" %}
Updating `SnLib.jar` always needs a full restart, never a `/reload`.
{% endhint %}
