# FAQ

### How do I update SnEconomyRobots?

Download the newer `sneconomyrobots-v*` release and replace the jar. Configs auto-merge on restart,
so your edits and comments are preserved and only missing keys are added.

### Can I run it without EdTools?

Yes, since 1.1.0. EdTools is optional and the plugin runs on your Vault economy alone, logging one
line at startup to say EdTools is absent. What you lose is every EdTools currency as an economy id
and booster-multiplied claims: a robot production entry or upgrade cost priced in an EdTools
currency never settles, so that income stays in the claim bag rather than being lost.

Vault is the one you cannot run without - the reserved `economy.vault-economy-id` always routes
through it, and Vault itself needs an economy provider behind it (EssentialsX, CMI, ...).

### Does it support Folia?

No, SnEconomyRobots does not declare Folia support.

### Where does the income go?

Into a claim bag, not straight into the player's balance. Robots accrue in memory while their owner
is online, and the balance moves only when the player claims it. A claim makes one payment per
economy, no matter how many robots contributed.

### Do robots produce while the player is offline?

No. Robots accrue only while their owner is online.

### Why does a brand new robot not pay immediately?

A robot the production engine has never seen is scheduled on its first visit and paid from the next
one. This is deliberate, and it stops a player farming payouts by re-equipping.

### A player deleted the example robot and it came back. Why?

It should not. The example is seeded only when the `robots/` folder holds no robot file at all, so
deleting it while other robots exist is permanent.

### Can I add my own robots?

Yes. Drop a new `robots/<id>.yml` into the plugin folder and reload. The file name is the robot id,
lowercased. Use `robots/example-robot.yml` as the starting point.

### Why is a robot's economy showing as locked?

That economy is gated behind an upgrade level through its `unlocked-at` block. The robot's lore
shows which track and level unlock it.

### A player holding boxes cannot open chests. Is that expected?

No, and it is fixed. If you see it, check `robot-boxes.blocked-on` in `config.yml`. Every vanilla
interactable block is excluded already, so that list is only for blocks added by other plugins.
