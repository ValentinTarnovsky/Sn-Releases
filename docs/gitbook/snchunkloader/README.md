# SnChunkLoader

SnChunkLoader gives your players a placeable chunk loader that keeps a square of chunks loaded and ticking. Every loader carries its own size and its own stored lifetime, so you can sell them by both.

## Features

- A placeable loader item that keeps an odd `N x N` square of chunks loaded, centered on its own chunk.
- A stored lifetime per loader, which counts down only while that loader is on and actually holding chunks.
- Permanent loaders, minted with the infinite duration token, for servers that sell them.
- Stacking: right-click a placed loader with another loader item of the same size to add its time.
- A floating display above every placed loader showing owner, remaining time and status.
- A menu on right-click to pause and resume a loader without losing its stored time.
- A loader never hits the ground. Breaking one hands it back with its remaining time to the player who broke it, and any member of the island may be that player. Membership loss, a disband and the reconciliation sweep return it to its owner instead. When the receiving inventory is full or that player is offline, the loader is queued in the database and handed over on their next join.
- A placed loader can only be removed by a player breaking it. Explosions skip it, fire and block decay cannot take it, endermen and falling blocks cannot change it, and a piston that would push or pull it does not move at all.
- Per-player and per-island limits, plus an island membership gate.

## Integrations

- **SnSuperiorSkyblock**: required. It owns the island model the whole plugin is built on. Loaders are placed on islands, counted per island, and returned to their owner when that owner stops being a member.

## Optional integrations

- **DecentHolograms**: an alternative backend for the floating display, selected with `hologram.provider`. When it is absent, the display falls back to SnLib's built-in renderer.
- **PlaceholderAPI**: resolves PAPI tokens written into the language file, the menu file and the hologram lines. When it is absent, those tokens stay raw. SnChunkLoader provides no placeholders of its own.

## Links

- Source and releases: part of the shared [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases) repo
