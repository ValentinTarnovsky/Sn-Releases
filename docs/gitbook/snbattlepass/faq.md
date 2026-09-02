# FAQ

### How do I update SnBattlePass?
Download the newer `snbattlepass-v*` release and replace the jar. Configs auto-merge on restart, so your values and comments are kept and new keys appear with their defaults.

### Does it support Folia?
No, SnBattlePass is not Folia-compatible.

### Does the pass reset each season?
No. The pass is permanent: tiers, XP and claims are kept until an admin resets them with `/battlepass reset` or `/battlepass resetall confirm`.

### A player bought a pass after reaching tier 30. Do they lose those premium rewards?
No. Buying a pass does not write claim history, it changes which lane the player may open. Every premium reward on a tier they already passed becomes claimable immediately.

### How do I sell passes on my web store instead of in game?
Set `passes.allow-purchase: false`. The menu then shows the pass as sold elsewhere, and the direct purchase route refuses as well, so the store is the only way in. Grant the pass from your store with `/battlepass setpass <player> <pass>`.

### My currency is not Vault. Can players still buy a pass?
Yes. The plugin never talks to an economy API. It reads a balance by expanding a placeholder you configure and charges by running a console command you configure, so any currency with a placeholder and a take command works.

### Purchases are refused and the console mentions the balance placeholder. What is wrong?
The configured placeholder did not expand to a number. Check that PlaceholderAPI and the expansion providing it are installed, and avoid placeholders that abbreviate or group digits, such as `1.2K` or `1,200`. Setting `economy.deny-on-unreadable-balance: false` lets the purchase through and leaves the charge command to refuse a player who cannot pay.

### Can I remove a button from a menu?
Yes. Delete its letter from the menu's `layout:` grid. A key the layout does not use is hidden, and because the layout is a list value rather than a key, the auto-merge never restores it.

### One of my challenge types never progresses. Why?
Its source plugin is probably not installed, or the tool or crate id in the pool entry does not match. Challenge types map to SnCrates, SnGens, SnEnvoys, SnPets and EdTools, and an id is a plain string that cannot be validated at load. Set `enabled: false` on pool entries whose source plugin is missing.

### Which pet box openings count?
SnPets boxes are counted through the SnPets API, and only when a box really opens: an attempt SnPets refuses - a cooldown still running, storage full, a failed success roll - hands the boxes back and adds no progress, and a shift-click stack credits the boxes that fitted. RivalPets boxes are still counted from the right-click on the box item. Both plugins can be installed at once; they never count the same opening.

### What happens if I lower the challenge slot count?
Challenges already running in the removed slots freeze: they stop earning and stop paying. Their deadline keeps counting in real time, so raising the count back either resumes them or expires them on the next sweep.

### What happens if I lower `xp.max-tier`?
Stored levels above the new ceiling are clamped to it, and that clamp is written back. Raising the ceiling again does not restore them, so lower this value deliberately.

### Which settings need a restart?
The `database` block, `integrations.rival-pets.*` and `integrations.sn-pets.*`. Everything else applies on `/battlepass reload`.

### How do I know a pet's farming boost is actually working?
Since 2.4.0, look at `%battlepass_boost%` (or the `{boost}` line of the `xp-info` tile, if your menu prints it): it is the boost in percentage points the farming drain is paying with right now, `0.0` when nothing matching is equipped. Before 2.4.0 the only visible number was the per-tool XP rate, which a level-1 pet moves from `0.5` to `0.502` - not something anyone notices.

Two things to check when it reads `0.0` with a pet equipped. The pet's id has to start with `integrations.sn-pets.pet-id-prefix` exactly, capital letters included. And the boot log has to say `SnPets detected: farming-XP boost active`: since 2.4.0 that line is printed only when the boost really is wired, and an enabled SnPets whose API service is not registered is refused with a warning instead - that means SnPets' own enable did not finish, so read its log and restart. A boost lookup that throws at runtime is reported once with the exception, and later failures go to `/battlepass debug`.

Remember that a pet's boost is `level x percent-per-level` up to its rarity ceiling: a freshly obtained level-1 pet is worth `0.4`, and only levelling it - which happens while it is equipped and its owner farms - makes the number grow.
