# Installation

SnBans ships as one jar that is both a Paper plugin and a Velocity plugin. The same file installs on a backend or on a proxy, and you pick which one.

## Requirements

| Item | Requirement |
|------|-------------|
| Paper | 1.20.5 or newer, 1.21.x included (backend deployment) |
| Velocity | 3.4.0 or newer (proxy deployment) |
| Java | 21 or newer |
| SnLib | 1.18.0 or newer, in the same `plugins/` folder |
| MySQL or MariaDB | Only for the multi-backend deployment; SQLite needs no setup |
| LuckPerms | Optional on both platforms |

{% hint style="warning" %}
Requires **SnLib** installed (`depend: [SnLib]`), version 1.18.0 or later. Update `SnLib.jar` first: on an older engine the API level handshake fails and SnBans refuses to enable on Paper. On Velocity the proxy descriptor declares `snlib` as a required dependency, so the proxy does not load SnBans without it.
{% endhint %}

SnBans declares no `folia-supported` key, so it is not Folia compatible.

## Choose one deployment

| Deployment | Where the jar goes | Database | Notes |
|------------|--------------------|----------|-------|
| Paper backends | `plugins/` of every backend | One shared MySQL makes punishments network-wide | Recommended, and the only deployment that enforces mutes exactly |
| Velocity proxy | `plugins/` of the proxy | SQLite works standalone | Logins, chat and commands are enforced at the proxy |

{% hint style="danger" %}
Install SnBans on your backends or on your proxy, never on both. With both halves installed every punishment is announced twice, once by the backend and once by the proxy.
{% endhint %}

{% hint style="warning" %}
Backend installs stay the recommended path for **mutes**. Clients from 1.19.1 onward with enforced secure chat may not accept a chat message the proxy denies. Blocked commands, bans, IP bans and blacklists are enforced exactly on either deployment.
{% endhint %}

## Install on your Paper backends

1. Download the latest `snbans-v*` release from [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases/releases?q=snbans-).
2. Place the `.jar` file, together with `SnLib.jar`, into the server's `plugins/` folder.
3. Start the server once. SnBans creates a shared license file at `plugins/.Sn-License/license.yml`.
4. Paste your Sn license key into that file, replacing the placeholder line.
5. Restart the server. SnBans validates the key at startup, then enables and creates its punishment tables.
6. Open `plugins/SnBans/config.yml`. Set `database.type: mysql` with your host, port, schema, user and password, then give this server its own `server-name`.
7. Repeat every step on each backend, with the same `database` block and a different `server-name` on each one.
8. Restart the backends and read the boot log of each, as described below.

{% hint style="info" %}
Do not skip the `server-name` on step 6. Punishments still sync correctly without it - the poller matches its own rows by row id since 1.8.2, and it warns when it sees two servers sharing a name - but network presence (`/alts` knowing who is online where), `/helpop`, `/report` and the `{server}` placeholder are all keyed by that name and break between any two backends that share one. SnBans warns at boot while the value is still the shipped `"Server"` on a shared database.
{% endhint %}

{% hint style="warning" %}
Create the MySQL schema yourself: SnBans creates only its tables inside it. The configured user needs CREATE, SELECT, INSERT, UPDATE and DELETE on that schema.
{% endhint %}

## Install on your Velocity proxy

1. Download the same `snbans-v*` release from [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases/releases?q=snbans-).
2. Place the `.jar` file, together with `SnLib.jar`, into the proxy's `plugins/` folder.
3. Start the proxy once. SnBans seeds `config.yml`, `lang/messages_en.yml`, `templates.yml` and `webhooks.yml` into `plugins/snbans/`.
4. Edit `plugins/snbans/config.yml`. SQLite is the default and needs no further keys; set `server-name` to a name for this proxy.
5. Restart the proxy. SnBans creates its tables, and on SQLite its `database.db` file next to those four files.

{% hint style="info" %}
On the proxy, `/snbans reload` re-reads `config.yml`, the lang file, `templates.yml` and `webhooks.yml`, and re-sources `command.aliases`. Two things behave differently there: the proxy does not resend its command tree, so a new alias only tab-completes for players who are already connected after they reconnect, and `/snbans debug` does not exist on Velocity, so `debug.enabled` in the file is the only way to switch debug output on.
{% endhint %}

## License key

{% hint style="info" %}
SnBans is part of the Sn bundle: one shared key in `plugins/.Sn-License/license.yml` unlocks every bundle plugin on that server, so you paste it once. Without a valid key SnBans refuses to enable and logs a single `[Sn] License: FAIL` line.
{% endhint %}

The startup gate is part of the Paper entry point of the jar, so steps 3 to 5 above are the Paper flow. A standalone Velocity install creates and reads no license file, and has no key step.

{% hint style="warning" %}
The check runs once while the plugin enables and needs outbound HTTPS from the server. It also hashes the running jar, so use the released file as downloaded: a repacked or edited jar fails validation.
{% endhint %}

## Dependencies

| Plugin | Required |
|--------|----------|
| SnLib | Yes |
| LuckPerms | No (optional on both platforms, enables the staff hierarchy check) |

{% hint style="info" %}
Without LuckPerms the hierarchy check stays off, whatever `hierarchy.enabled` says, and every rank can punish every rank. The absence is reported once in the console and is never an error path.
{% endhint %}

## Verify the install

Both platforms print the same three lines on boot, so a network running the jar on a proxy and on backends pastes one readable log:

```
SnBans <version> enabled on <server or proxy>
Database: MySQL, cross-server sync every 5s
Hierarchy: LuckPerms group weights
```

Read the second line to confirm the deployment you meant to build. A standalone install prints `Database: SQLite, standalone install (no cross-server sync)` instead, and the sync clause only appears on MySQL. The third line reads `Hierarchy: off (LuckPerms not installed)` or `Hierarchy: off (disabled in config)` when the check is inactive.

{% hint style="info" %}
The punishment tables are created on a database worker while the server finishes starting, so there is a brief window where the login check has nothing to read yet. If somebody actually connects during it, the log says so once per start, naming the player: the login can be allowed without being checked, and a ban that misses it is enforced on the player's next connect. No connection in that window means no warning - up to and including v1.8.0 Paper printed the notice on every single boot whether or not anyone was affected.
{% endhint %}
