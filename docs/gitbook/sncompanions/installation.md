# Installation

SnCompanions is a licensed Sn plugin. Install it, boot once to generate the license file, paste
your key, then restart.

1. Download the latest `sncompanions-v*` release from [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases/releases?q=sncompanions-).
2. Place the `.jar` file into your server's `plugins/` folder.
3. Start the server once. The plugin creates `plugins/.Sn-License/license.yml`.
4. Open that file and paste your Sn license key in place of the placeholder line.
5. Restart the server. The plugin validates the key at startup.

{% hint style="warning" %}
SnLib is required and must be installed as its own plugin in `plugins/`. This build targets
`com.sn:snlib` 1.31.0, so run that version or newer. SnLib is never bundled inside the jar.

1.31.0 is what the drop key (Q) in the storage menu needs, and it is a requirement the startup
API check cannot enforce: SnLib 1.31.0 grew no new Java surface, so its API level is the same as
1.28.0's. On an SnLib older than 1.31.0 SnCompanions still enables normally and everything else works -
only Q over a stored companion does nothing.
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
| EdTools | No |

SnLib is the engine behind the config, language, database, item, menu and command layers.
packetevents draws every companion and sends the formation updates, so the plugin cannot run
without it. PlaceholderAPI adds the `%sncompanions_...%` expansion. BetterModel lets a companion type
render as an animated model instead of a head. EdTools lets an equipped companion grant currency and
global-enchant boosters, and unlocks the `EDTOOLS_BLOCK_BREAK` experience source so a companion can level
from the blocks an omnitool consumes. It is reached through only two isolated classes, and a server
without it never loads an EdTools type: the break listener is not even registered.

{% hint style="warning" %}
Folia is not supported. The plugin does not declare `folia-supported`, so run it on Paper.
{% endhint %}

The plugin targets Paper 1.20.4 and newer, and both the 1.20 and 1.21 lines are supported.
Java 21 or newer is required, since the jar is compiled at release 21.

## What the first boot creates

On a clean install the plugin writes its own files into `plugins/SnCompanions/`:

| File | Purpose |
|------|---------|
| `config.yml` | Every tunable: formation, animation, buffs, fusion, boxes and more |
| `lang/messages_en.yml` | Every user facing message, in English |
| `lang/messages_es.yml` | The same messages, in Spanish |
| `guis/*.yml` | The seven menu layouts |
| `traits.yml` | Trait definitions and their roll weights |
| `boost-grades.yml` | The boost grade ladder |
| `companions/*.yml` | One file per companion type, three examples seeded |
| `boxes.yml` | Every companion box, one top-level key each, two examples seeded |

The `companions/` folder and the `traits.yml`, `boost-grades.yml` and `boxes.yml` files are seeded
exactly once, on a true fresh install. After that they belong to you. A companion file you delete
stays deleted and is never restored on the next boot, and the three keyed files carry a
`# sn:extensible-root` header so a key you delete stays deleted too. `config.yml`, the language
files and the menu layouts are managed instead: new keys are merged in on boot while your
values and comments are preserved.

Upgrading from a version older than 1.4.0? Your `boxes/` folder is folded into `boxes.yml`
automatically on the first boot and kept as `boxes-migrated/`. Nothing is deleted and the box
items already handed out keep working.

{% hint style="success" %}
Once the server starts cleanly, run `/companions admin give <player> ember_fox` to hand out the
first companion, then open `/companions` to see it in storage.
{% endhint %}
