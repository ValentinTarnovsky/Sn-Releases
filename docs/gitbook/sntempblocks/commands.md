# Commands

Every command lives under `/tempblocks`, with the aliases `/tb` and `/stb`. You can change the
aliases in `config.yml` under `command.aliases`, and they are re-read on reload. All commands are
administrative: SnTempBlocks has no player-facing command.

| Command | Permission | Description |
|---------|-----------|-------------|
| `/tempblocks list` | `sntempblocks.admin.list` | List every zone with its engine, tracked block count and next wipe |
| `/tempblocks info <zone>` | `sntempblocks.admin.info` | Show everything known about one zone, including blocks waiting on unloaded chunks |
| `/tempblocks here` | `sntempblocks.admin.here` | Show which zone you are standing in, and its countdown |
| `/tempblocks wipe <zone\|all>` | `sntempblocks.admin.wipe` | Remove every tracked block of a zone right now, or of all zones |
| `/tempblocks purge <zone>` | `sntempblocks.admin.purge` | Stop tracking a zone's blocks and leave them in the world permanently |
| `/tempblocks reload` | `sntempblocks.admin.reload` | Re-read `config.yml` and `zones.yml` |
| `/tempblocks debug` | `sntempblocks.admin.debug` | Toggle debug output |
| `/tempblocks help` | `sntempblocks.admin` | Show the help menu |

The zone argument tab-completes from your loaded zones, so you never have to remember an id.

## What wipe and purge actually do

They are opposites, and the difference matters:

- **Wipe** removes the blocks from the world and forgets them. On an `INTERVAL` zone it also
  restarts the countdown, because you just did by hand what the timer was going to do.
- **Purge** leaves every block standing and only stops tracking them. The blocks become permanent
  parts of your world.

Both obey the same per-tick removal budget as a scheduled wipe, so neither can stall the server
no matter how full the zone is.

{% hint style="danger" %}
These commands cannot be undone: `/tempblocks wipe` deletes the tracked blocks of a zone, and
`/tempblocks wipe all` does it for every zone at once. `/tempblocks purge` permanently forgets a
zone's blocks, so nothing will ever clean them up afterwards.
{% endhint %}
