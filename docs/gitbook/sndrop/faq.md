# FAQ

### A player says they cannot drop anything. Is that a bug?

No, that is the plugin working. Everyone is blocked until they run `/drop`. If they ran it and are
still blocked, their window has expired - it lasts `drop.duration-seconds` and there is no
notification when it ends.

### Can I let staff bypass the restriction?

Not with a setting. There is no bypass permission, and operators and creative-mode players are
blocked like everybody else. This is deliberate. If you need an exempt rank, ask for it as a
change.

### Does a drop window survive a relog or a restart?

No, in both cases. Windows are kept in memory and are cleared when the player disconnects and when
the server stops. After either, the player runs `/drop` again. This stops somebody opening a
window and banking it for a later session.

### Why did items in my crafting grid drop when I died?

That is the anti-bypass rule and it is the point of the plugin. The 2x2 crafting grid and the
stack on your cursor are not part of the normal death drops - the server hands them back after the
respawn - which made them a pocket for carrying loot past the drop restriction. SnDrop moves all
five slots into the death drops instead. The crafting result slot is cleared but never dropped, so
nothing is duplicated.

With the `keepInventory` gamerule on nothing is confiscated: those five stacks go back into your
inventory with the rest of it, exactly as keepInventory implies.

### I logged out with my inventory screen open and lost what was in the crafting grid.

Fixed in 2.0.1. Both the grid and the cursor stack are now returned to your inventory when you
disconnect, before your player file is written, so they are there on the next login. Older builds
let the server empty those slots after the save, which destroyed them.

### `/drop reload` did not change how long my window lasts.

Correct. The duration is baked in when a window opens, so a changed `duration-seconds` applies to
the next window, not to one already running. Reloading never cancels a running window.

### I edited a message and it came back after an update.

That should not happen, and it is worth reporting. Files are merged rather than regenerated: a new
version's new keys are inserted and everything you wrote is kept. Note the merge is insert-only,
so if you want the NEW default wording of a line you already edited, you have to copy it in
yourself - your version is never overwritten.

### A player is being spammed with the blocked message.

Raise `messages.blocked-cooldown-millis`. It is the minimum gap per player between warnings; the
default of `1000` means at most one per second. Setting it to `0` warns on every attempt.

### The plugin did not enable and the log says `[Sn] License: FAIL`.

The bundle key in `plugins/.Sn-License/license.yml` is missing, wrong, or the backend was
unreachable at boot. The message is intentionally short. Check the key, then restart. Nothing is
half-enabled: the plugin either validates and runs or disables itself entirely, which means drops
are simply unrestricted until it is fixed.

### Will it run on 1.20?

No. It needs Paper 1.21 or newer. The death handler uses an inventory API whose type changed in
1.21, so a 1.20 server would break on every death.

### Does it need a database?

No. SnDrop stores nothing on disk beyond its two configuration files.

### Does it provide PlaceholderAPI placeholders?

No. It registers no expansion.
