# Installation

1. Download the latest `snfourinline-v*` release from [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases/releases?q=snfourinline-).
2. Place the `.jar` file into your server's `plugins/` folder.
3. Start the server once. SnFourInLine creates a shared license file at `plugins/.Sn-License/license.yml`.
4. Paste your Sn license key into that file, replacing the placeholder line.
5. Restart the server. SnFourInLine validates the key at startup and then enables.

{% hint style="warning" %}
Requires **SnLib** installed (`depend: [SnLib]`), version 1.28.0 or later.
{% endhint %}

{% hint style="info" %}
SnFourInLine is licensed. The key in `plugins/.Sn-License/license.yml` is shared by every bundled Sn plugin on the server, so you paste it once. Without a valid key the plugin refuses to enable.
{% endhint %}

## Dependencies

| Plugin | Required |
|--------|----------|
| SnLib | Yes |
| PlaceholderAPI | No, but betting needs it: without it every balance reads as zero and every bet is refused |
| An economy plugin (EssentialsX, CMI, ...) | No: only needed when you configure currencies, whose money moves through that economy's own console commands |

**Requirements:** Java 21+, Paper 1.20.x or 1.21.x

## First boot

SnFourInLine generates its own files on first boot: `config.yml`, `lang/messages_en.yml`, and `guis/board.yml`, `guis/lobby.yml`, `guis/leaderboard.yml`. New keys are auto-merged on later boots; your values and comments are preserved. Set `update-configs: false` to freeze the files.

{% hint style="info" %}
The `currencies:` block is marked extensible. Entries you delete stay deleted and the updater never inserts new ones there, so read the currency documentation here rather than in your generated file.
{% endhint %}
