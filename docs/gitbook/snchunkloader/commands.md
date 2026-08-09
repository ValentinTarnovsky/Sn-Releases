# Commands

The root command is `/chunkloader`, with the aliases `/scl` and `/snchunkloader`. The alias list lives in `config.yml` under `command.aliases` and is re-read on `/chunkloader reload`.

Every subcommand is administrative. Players never type a command: they interact with loaders by placing them, breaking them and right-clicking them, which is gated by `snchunkloader.use`.

| Command | Permission | Description |
|---------|-----------|-------------|
| `/chunkloader help [page]` | `snchunkloader.admin` | Lists the available subcommands. |
| `/chunkloader reload` | `snchunkloader.admin.reload` | Reloads the configuration, the language file and the menu. Task intervals restart with the new values. |
| `/chunkloader debug` | `snchunkloader.admin.debug` | Toggles the runtime debug output. |
| `/chunkloader give <player> <size> <duration>` | `snchunkloader.admin.give` | Gives one chunk loader item to an online player. |
| `/chunkloader list [player]` | `snchunkloader.admin.list` | Lists the chunk loaders a player has placed. Without an argument it lists your own. |
| `/chunkloader chunks` | `snchunkloader.admin.chunks` | Shows how many chunks are kept loaded, by how many loaders, and how many of those are on. |

## The give arguments

**`size`** is the side of the square of chunks the loader keeps loaded, centered on its own chunk. It must be an odd number inside the range set by `chunk-loader.size.min` and `chunk-loader.size.max`. Size 1 keeps 1 chunk, 3 keeps 9, 5 keeps 25, 7 keeps 49.

**`duration`** is the lifetime stored on the item. Write it as a number plus a unit, and combine units freely: `30m`, `12h`, `1d`, `1d12h`. Use `-1` for a loader that never expires.

{% hint style="info" %}
The size range only governs what `give` may mint. It never confiscates: loaders already placed keep running, and loader items already in players' hands stay placeable even if their size now sits outside the range. Narrowing the range stops you selling a size, it never destroys one somebody already paid for.
{% endhint %}

{% hint style="warning" %}
`-1` mints a permanent loader. It is deliberate and there is no expiry to fall back on, so treat it as the product it is.
{% endhint %}

{% hint style="info" %}
`/chunkloader reload` does not move existing floating displays onto a new backend or height. A display keeps the `hologram.provider` and `hologram.offset-y` it was created with, so those two keys need a server restart to apply everywhere.
{% endhint %}

## Tab completion

`size` offers every odd size the configuration currently allows, read live, so an edited range applies after a reload rather than after a restart. `duration` offers `-1`, `1d`, `12h`, `6h` and `30m`, and still accepts anything you type. `list` offers the online players but also accepts an offline name.
