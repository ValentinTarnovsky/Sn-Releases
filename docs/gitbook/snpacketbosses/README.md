# SnPacketBosses

SnPacketBosses gives each player a private boss fight that nobody else can see. The boss is never spawned on the server: it exists only as packets sent to its owner. Players summon it from a tradeable boss egg, then damage it by mining with EdTools. A fight ends in a kill, in a timeout that hands the egg back with one attempt fewer, or in a pause when the owner disconnects. Every boss is one YAML file, so adding a boss means adding a file.

{% hint style="info" %}
SnPacketBosses needs Paper 1.21.2 or newer, plus SnLib, EdTools and packetevents installed. It is not Folia compatible. The plugin is licensed in bundle mode: it reads a shared key from `plugins/.Sn-License/license.yml`, created on first boot. Without a valid key it refuses to enable.
{% endhint %}

## Features

- **Owner-only packet rendering.** Nothing is spawned, tracked or ticked by the server. The boss, its hologram and its fireballs are packets addressed to one client. Two players standing on the same block see different things. Admins subscribe with `/packetbosses view` and watch the same fight, and the owner is never told.

- **Mining is the damage model.** Every EdTools block break by the owner removes health from the boss. Damage applies per event, so bulk breaking pays off. The hurt flash, the boss bar and the hologram are coalesced into one flush per movement tick, so the client never strobes.

- **Boss eggs that carry their own attempts.** Each egg stores its boss id and its remaining tries in persistent data, so an anvil rename cannot break it. It is an ordinary item that players buy, sell, store and hand to each other. A timeout returns it with one attempt fewer, and a refused click spends nothing at all.

- **Three telegraphed abilities, each with a real answer.** Potion effects, a fireball and an armored phase announce themselves five seconds before they land. Walk out of range to dodge one. Hit the fireball back to deflect it and hurt the boss. Break the armor with physical hits to end the damage reduction early.

- **Bosses that follow, or bosses that stay put.** A boss with movement enabled orbits its owner and follows across worlds without ever being respawned. Large corrections use a position sync packet, so the client never snaps back. Leave movement off and the boss is pinned to a fixed location instead.

- **Fights survive quits and restarts.** Leaving pauses the clock and spends no attempt, and the session is saved asynchronously. You come back to the fight you left, redrawn shortly after you join. Fights are also restored after a server reload, when no join event ever fires.

- **A fight lockdown you control.** While a boss is active you can block commands, teleports and flight, per world. The command allowlist is yours to edit, and holders of the bypass permission are exempt from all of it. `/packetbosses` always stays runnable, so an admin can free a stuck player.

## Links

- Source and releases: part of the shared [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases) repo
