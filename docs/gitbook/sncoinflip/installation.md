# Installation

1. Download the latest `sncoinflip-v*` release from [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases/releases?q=sncoinflip-).
2. Place the `.jar` file into your server's `plugins/` folder.
3. Start the server once. SnCoinFlip creates a shared license file at `plugins/.Sn-License/license.yml`.
4. Paste your Sn license key into that file, replacing the placeholder line.
5. Restart the server. SnCoinFlip validates the key at startup and then enables.

{% hint style="warning" %}
Requires **SnLib** installed (`depend: [SnLib, Vault]`), version 1.24.1 or later. SnCoinFlip refuses to enable against an older engine, so update `SnLib.jar` at the same time.
{% endhint %}

{% hint style="danger" %}
**Vault has been a hard dependency since v2.1.0.** Without `Vault.jar` in `plugins/`, SnCoinFlip does not load at all - Bukkit refuses it before the plugin gets a chance to say anything. If you are upgrading from 2.0.x, install Vault before you drop the new jar in.
{% endhint %}

{% hint style="info" %}
SnCoinFlip is licensed. The key in `plugins/.Sn-License/license.yml` is shared by every bundled Sn plugin on the server, so you paste it once. Without a valid key the plugin refuses to enable.
{% endhint %}

{% hint style="warning" %}
The licence is checked on every startup, not only the first one. The check is an outbound HTTPS call to `sn-license-server.okimc-dev.workers.dev`, retried three times before the plugin gives up and disables itself with a single `[Sn] License: FAIL` line in the console. Allow that host through your firewall. Expect the same failure if the jar has been repacked or modified, because its hash is part of the check.
{% endhint %}

## Dependencies

| Plugin | Required |
|--------|----------|
| SnLib | Yes |
| Vault | Yes |
| An economy plugin (EssentialsX, CMI, XConomy, ...) | No to load, but the default `money` currency has nothing to read without one |
| PlaceholderAPI | No, but every command-backed currency needs it, and so do the placeholders |
| EdTools | No, but every `edtoolapi: true` currency needs it |

{% hint style="danger" %}
"Optional" here means the plugin starts without them, not that the shipped configuration works without them. Three currencies ship in `config.yml`: a Vault-backed `money`, a command-backed one and an EdTools one. On a server with no economy plugin, no PlaceholderAPI and no EdTools, every one of them is either skipped at startup or refuses every wager.
{% endhint %}

{% hint style="info" %}
Vault is the one dependency you can satisfy late. It is required to LOAD, but the economy provider behind it is not required to be registered yet: a `vault: true` currency starts working the moment an economy plugin registers one, with no restart and no reload.
{% endhint %}

**Requirements:** Java 21+, Paper 1.20.x or 1.21.x

## First boot

SnCoinFlip generates its own files on first boot: `config.yml`, `lang/messages_en.yml`, `lang/messages_es.yml` and `guis/lobby.yml`, `guis/stats.yml`, `guis/animation.yml`. New keys are auto-merged on later boots, and your values and comments are preserved. Set `update-configs: false` to freeze the files, after which SnLib only warns about missing keys instead of inserting them.

{% hint style="info" %}
The `currencies:` block is marked extensible. Entries you delete stay deleted: the updater never puts them back and never inserts new ones there on your behalf. That also means new comments written into that block by a later version never reach an existing install, so read the currency documentation here rather than in your own generated file.
{% endhint %}

## Before you let players use it

The shipped `currencies:` block wagers your Vault economy by default and then shows one example of each other backend, written against the OkiMC network's own currencies. Replace the examples with yours. Three things are worth getting right before the first wager:

- **The `currencies:` block is not inserted into a config that already exists.** It is extensible, so upgrading from 2.0.x does not add the new Vault-backed `money` currency to your file. Add it by hand (`display-name` plus `vault: true` is the whole entry) or delete the whole `currencies:` block and let the plugin write the new defaults back on the next boot.
- **`balance-placeholder` must render an exact number.** It is not a display string. It is the only evidence a command-backed currency ever produces that a wager really left the player. A placeholder that renders `1.5M` cannot show a 100 debit leaving, so wagers it could not display are refused before anything is charged. See [Configuration](configuration.md).
- **`verification.confirm-look-ticks` depends on your balance provider, not on this plugin.** If your take-command is forwarded to a proxy, the placeholder may still be reading a stale local cache when the plugin looks. Raise it if you see aborted matches in the console for wagers that were actually paid.
