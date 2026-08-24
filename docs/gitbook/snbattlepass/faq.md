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
Its source plugin is probably not installed, or the tool or crate id in the pool entry does not match. Challenge types map to SnCrates, SnGens, SnEnvoys and EdTools, and an id is a plain string that cannot be validated at load. Set `enabled: false` on pool entries whose source plugin is missing.

### What happens if I lower the challenge slot count?
Challenges already running in the removed slots freeze: they stop earning and stop paying. Their deadline keeps counting in real time, so raising the count back either resumes them or expires them on the next sweep.

### What happens if I lower `xp.max-tier`?
Stored levels above the new ceiling are clamped to it, and that clamp is written back. Raising the ceiling again does not restore them, so lower this value deliberately.

### Which settings need a restart?
The `database` block, `integrations.rival-pets.*` and `integrations.sn-pets.*`. Everything else applies on `/battlepass reload`.
