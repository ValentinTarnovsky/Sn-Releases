# FAQ

### How do I update SnDiscordLink?
Download the newer `sndiscordlink-v*` release and replace the jar. Configs auto-merge on restart, so your edits and comments stay.

### Does it support Folia?
No, SnDiscordLink is not Folia-compatible. It targets Paper, and it supports both 1.20.x and 1.21.x on Java 21.

### Do I need MySQL?
Only for a network. SQLite is the default and works for a single server. Several servers must share one MySQL database, because every signal between them is a database row.

### Which server runs the Discord bot?
Exactly one. With `discord.mode: auto` the servers elect a host through a database lease and renew it while they hold it. If the host dies, another server takes over in about 30 seconds.

### Why does the bot not start?
Check `discord.token` and `discord.guild-id`, and confirm the bot is a member of that guild. The **Server Members Intent** must be enabled, or role, boost and leave events never arrive.

### Do I need LuckPerms?
No. Without it, linking, claims, the linked role and `role-rewards` all keep working. Only `rank-roles`, the permanent group to Discord role sync, needs LuckPerms.

### Which Discord roles does the plugin touch?
Only the ones you map in `rank-roles` and `linked-role`. Roles managed by other bots, and roles above the bot's own role, are skipped and logged once.

### A player linked, so why did nothing happen in game?
Rewards run on the server where the player is online. If `rewards.claim-on` lists server names, the row waits until the player joins one of them.

### How do I add a language?
Copy `lang/messages_en.yml` to `lang/messages_<code>.yml`, translate it, and set `lang` in `config.yml` to that code. Missing keys fall back to English.

### Where do the "Yes" and "No" words come from?
From the `status:` block in `lang/messages_en.yml`. That single block feeds `/discord status`, `/discord info`, the Discord replies and the placeholders.

### Can one Discord user link several Minecraft accounts?
Yes, raise `linking.max-accounts-per-discord`. Every linked account then contributes its ranks, and the member keeps a role while any one of the accounts justifies it.
