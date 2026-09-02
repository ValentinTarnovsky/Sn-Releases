# FAQ

### How do I update SnCaptcha?
Download the newer `sncaptcha-v*` release and replace the jar. Configs auto-merge on restart.

### Does it support Folia?
No, SnCaptcha is not Folia-compatible. It targets Paper 1.20.x and 1.21.x.

### No captcha ever fires. What should I check first?
The console at startup. It reports which worlds it is tracking, and warns when the list is empty or when a configured world does not exist. World names are case sensitive, so `Gens` will not match a world named `gens`. If the list is right, remember the trigger is accumulated farming time in those worlds: a player has to break blocks through EdTools for a while before anything happens.

### How long until a player gets one?
Between `captcha.threshold-min-seconds` and `captcha.threshold-max-seconds` of accumulated farming time, drawn per player. With the shipped values that is roughly 25 to 40 minutes of actual farming, not of being online. Keep the maximum above the minimum: if they are inverted, every player draws the same fixed threshold and the interval becomes something a macro user can measure once and then avoid.

### Can a player dodge it by logging out?
No. Quitting mid-captcha is neither a pass nor a fail: the challenge is dropped without a sanction, but the accumulated time is kept, so the captcha comes back once they break another block.

### Are Bedrock players tested?
No. Bedrock clients are exempt, because the board relies on custom head skins that a Bedrock client may not render distinguishably. Their farming time still accumulates.

### Why is there no chat answer?
Because a mute applied at the proxy stops a chat message from ever reaching your server, which would make the captcha unanswerable for a muted player and get them punished for it. Inventory clicks and commands are the only channels.

### A player says the board was blank or the heads all looked the same.
Check the console for a warning about `heads.yml`. A texture value that was truncated when pasted, or one pointing somewhere other than `textures.minecraft.net`, renders as a blank head. The plugin validates every value at startup and refuses to send any captcha at all while fewer than four digits have a usable texture, so this should be reported rather than silent.

### My staff stopped seeing the alerts after updating.
That is the change, not a fault. The alerts are now opt in and off by default for everybody: `sncaptcha.notify` entitles a staff member to them, and each of them runs `/captcha alerts` once to switch them on. The choice is remembered per player.

### Staff chat is too noisy.
First, each staff member can simply run `/captcha alerts` to switch their own off. Server-wide, turn off `alerts.events.on-emit` and `alerts.events.on-resolved`: those two fire once per captcha each and are the high-volume pair, while `on-mid-timer` and `on-timeout-fail` are the ones that report a player who is not answering.

### Does `alerts.log` get cleaned up?
No. The file is append-only and the plugin never truncates or rotates it, so its size is yours to manage.

### A sanction tier did not run.
Its command probably does not exist on your server. The shipped tiers use `broadcast`, `kick`, `tempban` and `ban`, which come from a punishment or permissions plugin rather than from vanilla. The console shows an unknown-command line when that happens. Note that the staff alert says the tier was **dispatched**, which is what the plugin can actually guarantee: it hands the commands to the console and cannot know whether they worked.

### I removed the first tier and now a first failure bans people.
That is `sanctions.fallback-to-highest`, which runs the highest tier when a failure count falls below the lowest tier you defined. Add a tier `1`, or set the option to `false` so a count below the lowest tier does nothing. The plugin prints a warning at startup when your ladder is in that shape.

### Can I change the menu layout?
Yes: move the letters around in `layout:`, and change the filler and indicator materials and the title. What you must not do is give the `head` template a display name or lore, because the whole point is that nothing on the item says which digit it is. The file's own comments mark which parts are safe.

### Where is the data stored?
SQLite by default, in the plugin's folder. Point the flat `database:` block at MySQL if you prefer.
