# Installation

1. Download the latest `sncaptcha-v*` release from [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases/releases?q=sncaptcha-).
2. Place the `.jar` file into your server's `plugins/` folder.
3. Start the server once. The plugin creates `plugins/.Sn-License/license.yml`.
4. Paste your Sn license key into that file.
5. Restart the server. The plugin validates the key at startup.

{% hint style="info" %}
SnCaptcha uses the shared bundle license, so the key lives in `plugins/.Sn-License/license.yml` rather than in the plugin's own folder, and one key unlocks every plugin in your bundle. The plugin refuses to enable without a valid key.
{% endhint %}

{% hint style="warning" %}
Requires **SnLib** installed (`depend: [SnLib]`), version 1.28.0 or later, and **EdTools**. Both are hard dependencies: the server will not load SnCaptcha if either is missing.
{% endhint %}

On its first successful start the plugin generates `config.yml`, `heads.yml`, `lang/messages_en.yml` and `guis/captcha.yml` in its own folder. New keys are merged into those files on every later update, and your values and comments are preserved.

## Dependencies

| Plugin | Required |
|--------|----------|
| SnLib | Yes |
| EdTools | Yes |
| WorldGuard | No |
| Floodgate | No |

## First configuration

Two settings need your attention before the plugin does anything useful:

- **`worlds:`** must list your farming world names. Nothing is tracked anywhere else, and an empty list switches the plugin off. World names are case sensitive, and the console reports at startup which worlds it is tracking.
- **`sanctions.failures`** ships example commands that assume a punishment plugin such as EssentialsX or LiteBans. Replace them with commands your server actually has, or a tier will run nothing while staff are told it fired.

Run `/captcha reload` after editing, or restart.
