# SnChunkLoader

SnChunkLoader gives your players a placeable chunk loader that keeps a square of chunks loaded and ticking. Every loader carries its own size and its own stored lifetime, so you can sell them by both.

It runs on any Paper server. **SnSuperiorSkyblock is optional**: install it and every loader is bound to the island it stands on and obeys the island rules, leave it out and the same plugin is a plain chunk loader.

## Features

- A placeable loader item that keeps an odd `N x N` square of chunks loaded, centered on its own chunk.
- A stored lifetime per loader, which counts down only while that loader is on and actually holding chunks.
- Permanent loaders, minted with the infinite duration token, for servers that sell them.
- Stacking: right-click a placed loader with another loader item of the same size to add its time.
- A floating display above every placed loader showing owner, remaining time and status.
- A menu on right-click to pause and resume a loader without losing its stored time.
- A loader never hits the ground. Breaking one hands it back with its remaining time to the player who broke it, and on an island server any member of the island may be that player. Membership loss, a disband and the reconciliation sweep return it to its owner instead. When the receiving inventory is full or that player is offline, the loader is queued in the database and handed over on their next join.
- A placed loader can only be removed by a player breaking it. Explosions skip it, fire and block decay cannot take it, endermen and falling blocks cannot change it, and a piston that would push or pull it does not move at all.
- A per-player limit on every server, plus a per-island limit and an island membership gate on island servers.

## Optional integrations

- **SnSuperiorSkyblock**: turns the plugin into an island chunk loader. Loaders are placed on islands, gated on island membership, counted against the per-island limit, and returned to their owner when that owner is kicked, quits, is banned or the island is disbanded. Without it the plugin still runs: loaders go anywhere the server lets a player place a block, the per-player limit is the only placement rule, and the island rules under `chunk-loader.island` are not evaluated at all. Everything else - lifetime, stacking, displays, the menu, protection and returns of loaders whose block vanished - is identical either way.

- **DecentHolograms**: an alternative backend for the floating display, selected with `hologram.provider`. When it is absent, the display falls back to SnLib's built-in renderer.
- **PlaceholderAPI**: resolves PAPI tokens written into the language file, the menu file and the hologram lines. When it is absent, those tokens stay raw. SnChunkLoader provides no placeholders of its own.

## Links

- Source and releases: part of the shared [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases) repo
