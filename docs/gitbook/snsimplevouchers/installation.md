# Installation

## Requirements

| | |
|---|---|
| Java | 21 or newer |
| Server | Paper 1.20.x or 1.21.x |
| Required | `SnLib.jar` |
| Optional | PlaceholderAPI (only for `condition:` expressions) |
| License | Yes - SnSimpleVouchers is part of the licensed bundle |
| Network | Outbound HTTPS, for the one-time startup licence check |

## Steps

1. Download `SnSimpleVouchers-<version>.jar` from the releases page (tags prefixed `snsimplevouchers-`).
2. Put it in `plugins/`, together with `SnLib.jar`. The plugin declares `depend: [SnLib]`, so without it the server refuses to load the plugin and logs `Unknown dependency SnLib`.
3. Put your license key in `plugins/.Sn-License/license.yml`. That file is **shared by every bundle plugin on the server**, so if you already run another one, you are done.
4. Start the server, then edit `plugins/SnSimpleVouchers/config.yml` and `plugins/SnSimpleVouchers/lang/messages_en.yml`.
5. `/voucher reload` applies config, language, menu and voucher-file changes without a restart.

{% hint style="info" %}
Updating **SnLib** always needs a full restart, never a `/reload`.
{% endhint %}

## First boot

The plugin creates:

```
plugins/SnSimpleVouchers/
  config.yml               bulk claim, conditions, sounds, aliases, debug
  lang/messages_en.yml     every message players and admins see
  guis/categories.yml      the root admin menu
  guis/vouchers.yml        the per-category menu
  vouchers/example.yml     a fully commented example voucher
```

`vouchers/example.yml` documents every available field. It is **never loaded**, even if you set `enabled: true` on it, so you can keep it as a reference. Copy it to a new filename to make a real voucher.

## The licence check

The check runs **once, at startup**. The plugin does not enable until it succeeds, and there are no runtime re-checks afterwards.

{% hint style="warning" %}
A missing key and a blocked network both surface as the same single `[Sn] License: FAIL` line. If you are sure the key is right, check that the server can make outbound HTTPS requests before assuming otherwise.
{% endhint %}

Also make sure your `SnLib.jar` is current. An older one, already installed for a different Sn plugin, can sit below the API level this release needs; that failure does name the level in the console.

## Your first voucher

Create `plugins/SnSimpleVouchers/vouchers/starter.yml`:

```yaml
item:
  material: PAPER
  display-name: "&#FFAA00&lStarter Kit"
  lore:
    - "&7Right-click to claim"
  glow: true

commands:
  - "give {player} diamond 5"
  - "broadcast &e{player} &7claimed a Starter Kit!"
```

Then `/voucher reload` and `/voucher give <you> starter`.
