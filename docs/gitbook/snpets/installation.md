# Installation

SnPets is a licensed Sn plugin. Install it, boot once to generate the license file, paste
your key, then restart.

1. Download the latest `snpets-v*` release from [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases/releases?q=snpets-).
2. Place the `.jar` file into your server's `plugins/` folder.
3. Start the server once. The plugin creates `plugins/.Sn-License/license.yml`.
4. Open that file and paste your Sn license key in place of the placeholder line.
5. Restart the server. The plugin validates the key at startup.

{% hint style="warning" %}
SnLib is required and must be installed as its own plugin in `plugins/`. This build targets
`com.sn:snlib` 1.30.0, so run that version or newer. SnLib is never bundled inside the jar,
so an older SnLib will fail the startup API check.
{% endhint %}

{% hint style="info" %}
The key file at `plugins/.Sn-License/license.yml` is shared by every Sn bundle plugin on the
server. One key unlocks all of them, so you only paste it once, no matter how many bundle
plugins you run. Without a valid key the plugin refuses to enable and disables itself during
startup.
{% endhint %}

## Dependencies

| Plugin | Required |
|--------|----------|
| SnLib | Yes |
| packetevents | Yes |
| PlaceholderAPI | No |
| BetterModel | No |

SnLib is the engine behind the config, language, database, item, menu and command layers.
packetevents draws every pet and sends the formation updates, so the plugin cannot run
without it. PlaceholderAPI adds the `%snpets_...%` expansion. BetterModel lets a pet type
render as an animated model instead of a head.

{% hint style="warning" %}
Folia is not supported. The plugin does not declare `folia-supported`, so run it on Paper.
{% endhint %}

The plugin targets Paper 1.20.4 and newer, and both the 1.20 and 1.21 lines are supported.
Java 21 or newer is required, since the jar is compiled at release 21.

## What the first boot creates

On a clean install the plugin writes its own files into `plugins/SnPets/`:

| File | Purpose |
|------|---------|
| `config.yml` | Every tunable: formation, animation, buffs, fusion, boxes and more |
| `lang/messages_en.yml` | Every user facing message, in English |
| `lang/messages_es.yml` | The same messages, in Spanish |
| `guis/*.yml` | The seven menu layouts |
| `traits.yml` | Trait definitions and their roll weights |
| `boost-grades.yml` | The boost grade ladder |
| `pets/*.yml` | One file per pet type, three examples seeded |
| `boxes/*.yml` | One file per pet box, two examples seeded |

The `pets/`, `boxes/`, `traits.yml` and `boost-grades.yml` files are seeded exactly once, on
a true fresh install. After that they belong to you. A pet or box file you delete stays
deleted and is never restored on the next boot. `config.yml`, the language files and the menu
layouts are managed instead: new keys are merged in on boot while your values and comments
are preserved.

{% hint style="success" %}
Once the server starts cleanly, run `/pets admin give <player> ember_fox` to hand out the
first pet, then open `/pets` to see it in storage.
{% endhint %}
