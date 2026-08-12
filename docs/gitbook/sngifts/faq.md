# FAQ

### How do I update SnGifts?

Download the newer `sngifts-v*` release and replace the jar. Your `config.yml`, language file and
menu auto-merge on restart, and `rewards.yml` is left exactly as you wrote it.

### Does it support Folia?

No, SnGifts is not Folia compatible. It runs on Paper 1.20 and newer.

### Why does the plugin not appear in `/plugins`?

Three usual causes, all reported in the server log at startup. Either `SnLib.jar` is missing from
`plugins/`, or the installed `SnLib.jar` is older than 1.27.0, or the bundle key in
`plugins/.Sn-License/license.yml` is still the placeholder.

### Why does the console say the SnLib version is too old?

The API level SnGifts needs is compiled into the jar. It refuses to enable against an older
`SnLib.jar` rather than failing halfway through startup on a method that is not there. Update
`SnLib.jar` and restart the server fully.

### How do I add a fourth gift?

Two edits. Add a fourth entry under `gifts.tiers` in `config.yml` with its own
`time-needed-minutes`, then add a fourth `g` to the layout in `guis/gifts.yml`. Run `/gifts reload`.

### Why is my new tier missing from the menu?

The menu shows as many tiers as it has `g` cells, in threshold order. With four tiers and three `g`
cells, the fourth one has nowhere to go. Add a letter to the layout.

### Why can nobody unlock my new tier?

Most likely `time-needed-minutes` is missing or misspelled on that entry. A tier without a
threshold gets an unreachable one, so it stays locked forever instead of quietly becoming free.
Check the key name and run `/gifts reload`.

### I reordered the entries in `config.yml` and nothing changed. Why?

Tier order is by `time-needed-minutes` ascending, ties broken by id. It is never the order you write
them in. That is deliberate: inserting a cheap tier never renumbers the gifts your players already
know.

### I renamed a tier and everyone can claim it again. Is that a bug?

No. The entry id is what is stored against a player's claim, so a renamed entry is a new gift as far
as the database is concerned. Change the display name in `guis/gifts.yml` instead when you only
want it to look different.

### Does playtime count time spent AFK?

Yes. Every online player accrues, with no idle detection. If you want AFK players excluded, keep the
thresholds low enough that it does not matter, or handle AFK kicking with another plugin.

### Is this the same playtime the server tracks?

No. SnGifts counts its own playtime for the current gift day only, and zeroes it at the daily reset.
It never reads the vanilla statistic.

### Two players in my house cannot both claim. Why?

`claim.ip-limit-per-gift` ships at `1`, and it counts the address a player connects from. Everyone
on one connection shares one allowance per gift per day. Raise the value, set `0` to disable the
limit, or grant `sngifts.claim.bypass-ip` to the accounts that should not be counted.

### Nobody on my network can claim after the first player. Why?

Your backend server is almost certainly behind a proxy without IP forwarding, so every player looks
like they connect from the proxy itself. The first claim of a gift then spends the allowance for the
whole network. Enable IP forwarding, or set `claim.ip-limit-per-gift: 0`.

### Do two players get the same reward from the same gift?

Yes, on the same day. The draw is per tier per day, not per player, and it is stored, so everyone
claiming that tier that day gets the same commands. `/gifts resetgifts` rerolls it for everybody.

### A `vault:500` reward did nothing. Why?

There is no economy on the server, or the transaction was refused. The player is told, that one
reward is skipped, and the rest of the gift still runs. Install Vault and an economy plugin.

### A player claimed and the gift was already spent. Can I give it back?

`/gifts reset <player>` clears their claims for today and refunds the per IP slots those claims
spent. Their playtime is untouched, so they can claim again immediately. It works on offline players
too.

### What is the difference between `/gifts resetall` and the daily reset?

`resetall` wipes claims, playtime, bypasses and the per IP counters. The daily reset does all of
that and also draws a fresh reward set. Use `/gifts resetgifts` when you only want new rewards.

### The menu does not update while it is open.

It repaints every `update-interval` ticks, 20 by default. Setting it to `0` turns repainting off, so
states then update only when a player reopens the menu. The window title is the one thing that never
repaints, because a title cannot be written into an open window.

### Clicking a locked gift does nothing.

The `[gift-claim] {gift_id}` click action was removed from that template. All three gift templates
need it: the plugin answers a click on a locked or claimed gift with the reason and a sound, and a
template without the action makes the cell inert.

### Can I change `/gifts` to something else?

The root command name is fixed, but its aliases are yours. Edit `command.aliases` in `config.yml`
and run `/gifts reload`. The default alias is `regalos`.

### My staff group has `sngifts.admin` but the subcommands do not work.

Grant `sngifts.use` as well. It sits on the root of the command tree and gates everything under it,
and it is not a child of `sngifts.admin`. Operators never hit this, because op satisfies both.

### Where is player data stored?

In the database configured under `database` in `config.yml`. SQLite by default at
`plugins/SnGifts/database.db`, with no setup, or MySQL. It holds playtime, claims, the day's drawn
commands, the last reset date and the per IP counters.

### Can I run the reset at a specific local time?

Yes. Set `reset.timezone` to your IANA zone and `reset.time` to the wall clock time, for example
`Europe/Madrid` and `"05:00"`. An invalid zone falls back to UTC and an invalid time to `00:00`.

### Do I need PlaceholderAPI?

No. SnGifts registers no placeholders of its own. With PlaceholderAPI installed you can use
`%placeholder%` tokens in the menu's static items.

### Why does my hex color show up as text?

Use `&#RRGGBB` or `<#RRGGBB>`. A bare `#RRGGBB` is not recognized as a color and renders literally.
