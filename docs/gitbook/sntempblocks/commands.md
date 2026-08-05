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
| `/tempblocks bypass` | `sntempblocks.admin.bypass` | Toggle whether the blocks **you** place are tracked |
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

## Building inside a zone

`/tempblocks bypass` is the staff build switch. While it is on, nothing you place inside a zone is
tracked, so your work does not expire under you. Buckets count too.

It is **off until you turn it on**, even if you are an operator. Holding
`sntempblocks.admin.bypass` only means you are allowed to flip the switch, never that your blocks
stop being tracked on their own.

The toggle lasts for your session and is dropped when you log out. That is deliberate: a bypass
left on by accident cannot keep filling an arena with permanent blocks. Revoking the permission
also ends an active bypass at the next block placed, without waiting for a logout.

{% hint style="info" %}
The bypass is evaluated at placement time, not at expiry time. Blocks you placed before switching
it on are already tracked and still expire; blocks you place while it is on are never tracked at
all, so they stay forever. Switching it back off does not retroactively track anything.
{% endhint %}
