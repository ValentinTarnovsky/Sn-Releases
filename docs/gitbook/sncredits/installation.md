# Installation

SnCredits ships as one jar that is both the Velocity plugin and the Paper plugin. Install the same
file on your proxy **and** on every backend, never one or the other.

The proxy half holds all the logic and all the configuration. The backend half draws the menus,
forwards console commands and answers placeholders.

## Requirements

| Item | Requirement |
|------|-------------|
| Velocity | 3.4.0 or newer, for the proxy |
| Paper | 1.20.4 through 1.21.x, for every backend |
| Java | 21 or newer |
| Database | MySQL or MariaDB for a network, SQLite for a single proxy |
| PlaceholderAPI | Optional, on the backends |
| LuckPerms | Optional, on the proxy |

SnCredits declares no Folia support, so it is not Folia compatible.

## Install on the Velocity proxy

1. Download the latest `sncredits-v*` release from [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases/releases?q=sncredits-).
2. Place the `.jar` file into the proxy's `plugins/` folder.
3. Start the proxy once. SnCredits creates `plugins/sncredits/` with its configuration files.
4. Open `plugins/sncredits/config.yml`. Fill in the `database` block, and set `currency.name` if
   you want a name other than credits.
5. Restart the proxy. SnCredits creates its tables the first time it connects.

The proxy also writes one shop file per registered backend, at `plugins/sncredits/shops/<backend>.yml`.
Each one starts as a copy of the shipped template, and you edit them separately.

{% hint style="info" %}
Set `database.type` to `sqlite` for a single proxy that needs no database setup. Use `mysql` for a
network, or when the data has to be reachable from outside the plugin. Any value other than
`sqlite` is read as MySQL.
{% endhint %}

## Install on every Paper backend

1. Place the **same** jar into the backend's `plugins/` folder.
2. Start the server once. The plugin creates `plugins/.Sn-License/license.yml`.
3. Paste your Sn bundle key into that file, over the placeholder line.
4. Restart the server.

There is nothing else to do. The backend half creates no plugin folder and reads no settings of
its own, because the proxy sends it everything it needs.

{% hint style="info" %}
One `.Sn-License/license.yml` is shared by every Sn bundle plugin on that server, so you paste your
key once. The key is validated at startup, and the backend half refuses to enable without a valid
one.
{% endhint %}

{% hint style="warning" %}
A bundle key binds to a limited number of distinct public IP addresses, three by default, fixed
when the key is issued. Ask for enough slots if your backends leave through different public IPs.
{% endhint %}

{% hint style="warning" %}
A backend without the jar cannot open the shop, the coinflip list or the leaderboard. Players there
can still check and send credits, because those run on the proxy.
{% endhint %}

## Upgrading from the two-jar layout

Earlier releases shipped `SnCredits-Velocity-*.jar` and `SnCredits-Paper-*.jar` as two separate
files. Delete both before you drop the single jar in, on the proxy and on every backend.

## Verify the install

The proxy prints one line once its database is up:

```
SnCredits enabled. Currency: credits, Command: /credits, DB: MYSQL
```

Read it back to confirm the three things you just configured. `DB` says `SQLITE` when you chose
SQLite. If you see `Failed to initialize database!` instead, the `database` block is wrong and
nothing else will work.

Each backend prints:

```
PlaceholderAPI expansion registered.
SnCredits Bridge v<version> enabled.
```

The first line appears only when PlaceholderAPI is installed there. The bridge is built against the
Paper API, so the server does not print a `[PluginRemapper] Remapping plugin` line for this jar.
