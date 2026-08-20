# FAQ

Short answers to the questions server owners ask most about SnPacketBosses.

### How do I update SnPacketBosses?

Download the newer `snpacketbosses-v*` release and replace the jar. Configs auto-merge on restart: new keys are inserted, and your values and comments are preserved. Your `bosses/` folder is never rewritten, so custom boss files and deleted files stay exactly as you left them. Set `update-configs: false` in `config.yml` if you want SnLib to warn about missing keys instead of adding them.

### Does it support Folia?

No. `plugin.yml` declares no `folia-supported` flag, so the plugin runs on Paper only. The real platform floor is Paper 1.21.2, because the position sync packet the boss uses does not exist below it.

### Can other players see or interfere with a boss?

No. The boss is never a real entity: it only exists as packets sent to its owner. Other players see nothing, walk through nothing, and cannot hit it. An admin with `snpacketbosses.admin.view` can subscribe with `/packetbosses view <player>`, and the watched player is never told. A spectator's hits are no-ops, because every attack is authorized by entity ownership rather than by entity id.

### What actually damages the boss?

EdTools block breaks by the owner, and almost nothing else. Each valid `EdToolsBreakBlockEvent` removes `damage-per-break` health, reduced by `damage-multiplier` while the boss is armored. Melee hits on the boss deal no health damage at all: they only count toward `required-hits` to break the armored phase early. A deflected fireball is the one exception and damages the boss for 25% of `heal-on-hit`.

{% hint style="warning" %}
EdTools is a hard dependency. Without EdTools and packetevents installed, the server refuses to enable SnPacketBosses.
{% endhint %}

### How does a player get an egg, and what is an attempt?

An admin runs `/packetbosses give <player> <boss> [amount]`. The egg is an ordinary item that players can stack, store, sell and trade freely. Its identity lives in the item's persistent data, so renaming it in an anvil does not break it, and a hand made copy summons nothing. Each egg carries an attempt count, capped by `max-attempts` in the boss file. Right-clicking spends the egg from the stack and starts the fight.

### What happens when the timer runs out?

The boss escapes, no reward command runs, and one attempt is spent. The egg is minted back into the player's inventory reading one attempt fewer. Once the count would reach zero, the egg is gone for good. In creative mode the egg is never taken in the first place, so nothing is spent and nothing returns.

### What happens if a player logs out mid-fight?

Logging out pauses the fight instead of ending it. The clock stops, abilities stop firing, no attempt is spent, and the fight is saved to the database. About one second after they rejoin, the boss is redrawn with its old health and its old timer. A restart does the same thing: every paused fight comes back on its owner's next join.

### How do I add a second boss?

Drop another `.yml` file into `plugins/SnPacketBosses/bosses/`. The file name, lowercased and without the extension, becomes the boss id. Files whose name starts with `old-` are ignored, which makes them a handy shelf for drafts. Run `/packetbosses reload` and the new boss is live. Copy the shipped `guardian.yml` as your starting point: every key is documented inline.

{% hint style="info" %}
Set `movement.enabled: true` and the boss follows its owner everywhere, across worlds included. Set it to `false` and it is pinned to `location`, so it is only drawn while the owner stands in that world.
{% endhint %}

### What happens to a running fight if I delete or disable a boss file?

The next reload ends every live fight whose boss is now missing or disabled. The stored row is deleted, the owner is told, and the console logs how many fights were ended. This is deliberate: a fight pointing at a boss that no longer exists could never be won and would block its owner forever. If the last boss file load was incomplete, for example because one file failed to parse, paused fights are kept and a warning is logged instead.

### How do I free a player who is stuck in a fight?

Run `/packetbosses kill <player>`, which ends the live fight and deletes its stored row. It works on offline players and on paused fights too, so nothing comes back on their next login. Use `/packetbosses kill all` to wipe every fight at once. Grant `snpacketbosses.admin.lock-bypass` to exempt someone from the command, teleport and flight locks, or turn the whole lockdown off with `locks.enabled: false`.

{% hint style="success" %}
The `packetbosses` root is always runnable during a locked fight, whatever `locks.allowed-commands` says. Both `/packetbosses` and `/snpacketbosses:packetbosses` normalise to the same root, so the recovery path can never be closed by accident.
{% endhint %}
