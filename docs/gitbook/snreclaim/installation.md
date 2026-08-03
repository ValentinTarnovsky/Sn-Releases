# Installation

## Requirements

| | |
|---|---|
| Java | 21 or newer |
| Server | Paper 1.20.4+ |
| Required | `SnLib.jar`, Vault, a groups-based permissions plugin (LuckPerms) |
| Optional | PlaceholderAPI |
| License | Yes - SnReclaim is part of the licensed bundle |

## Steps

1. Download `SnReclaim-<version>.jar` from the releases page (tags prefixed `snreclaim-`).
2. Put it in `plugins/`, together with `SnLib.jar`.
3. Make sure Vault and your permissions plugin are installed. SnReclaim resolves ranks
   **through Vault's permission-group service** - without a provider it refuses to enable
   and says so in the console.
4. Put your license key in `plugins/.Sn-License/license.yml`. That file is **shared by
   every bundle plugin on the server**, so if you already run another one, you are done.
5. Start the server, then edit `plugins/SnReclaim/config.yml` and
   `plugins/SnReclaim/lang/messages_en.yml`.
6. `/reclaim reload` applies config and language changes without a restart.

{% hint style="info" %}
Updating **SnLib** always needs a full restart, never a `/reload`.
{% endhint %}

## First boot

The plugin creates:

```
plugins/SnReclaim/
  config.yml            ranks, redemption and join-notification settings
  lang/messages_en.yml  every message players and admins see
  data.db               the claim records (SQLite)
```

The shipped `config.yml` contains two example ranks, `vip` and `mvp`. Replace them with
your own - the `ranks:` section is yours, and entries you delete are never re-added.

## Storage

SQLite by default, in `data.db`. Switch to MySQL in the `database:` block if you want the
claims shared across servers:

```yaml
database:
  type: mysql
  host: localhost
  port: 3306
  database: snreclaim
  username: root
  password: ""
```

The table is `snreclaim_claims(uuid, rank_key, claimed_at)`.
