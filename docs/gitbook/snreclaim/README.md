# SnReclaim

One-time, rank-based reclaims driven by your permission groups.

Every entry under `ranks:` names a Vault permission group. A player who belongs to that
group can redeem that rank's reward **exactly once, ever** - the claim is recorded per
player and per rank and never expires. A player who belongs to several ranks redeems them
all with a single `/reclaim`.

## What it does

- **One reclaim per rank, per player, claimable once ever.** Persisted in SQLite (or MySQL).
- **`/reclaim` redeems everything available at once.** A player in `vip` and `mvp` gets both.
- **Per-rank rewards**: a list of commands, run from the console or as the player.
- **Per-rank private message**: your own multi-line text, with colours, HEX and
  PlaceholderAPI, laid out exactly as you write it.
- **A broadcast per rank redeemed**, so the server sees each one.
- **Join notification** telling players how many reclaims are waiting.
- **Season lock**: `/reclaim disable` closes redemption for everyone, and the state
  survives restarts.
- **`/reclaim reset <player>`** gives a player another go.

## Typical use

Season start. You disable reclaims, wipe the map, let players re-rank, then enable them
again - and everyone who bought a rank gets their reward once, without staff handing
things out by hand.

## Requirements

Java 21+, Paper 1.20.4+, **SnLib**, **Vault** and a groups-based permissions plugin such
as LuckPerms. PlaceholderAPI is optional.
