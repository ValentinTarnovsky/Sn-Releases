# FAQ

### How do I update SnFourInLine?

Download the newer `snfourinline-v*` release and replace the jar. Configs auto-merge on restart.

### Does it support Folia?

No, SnFourInLine is not Folia-compatible. It targets Paper 1.20.x and 1.21.x.

### Another menu replaced my board mid-game. Did I just lose?

No. Closing that menu brings the board back, and so does the bare `/fourinline`. Dying closes the board too; it returns when you respawn. Your turn timer keeps running throughout, so come back before it runs out.

### `/fourinline` prints help instead of opening the lobby

`presentation.main` is set to `chat`. Set it to `gui`, or use `/fourinline lobby`, which opens the lobby either way.

### When is a bet actually charged?

`bet` charges nothing when the invite is sent: both players are charged the moment it is accepted, once both balances are verified. An invite that expires or is rejected has cost nobody anything. `betcreate` charges the creator immediately, because a listed game has to be funded before anyone can join; the joiner is charged when they join. Cancelling the listing, quitting the server or a shutdown all return that stake.

### A player is told they have no balance when they clearly have one

The balance comes from the currency's `balance-placeholder`. Check it resolves to a plain number with `/papi parse <player> %your_placeholder%`. If PlaceholderAPI is missing, or the expansion behind the placeholder is, every balance reads as zero and every bet is refused.

### A stake is refused as too fine

The stake has more decimal places than the currency's `decimals` allows. A fractional charge cannot be verified against a whole-number balance, so it is refused before anything is charged. Use a coarser amount, or raise `decimals` if the placeholder really renders that finely.

### What happens to the money on a surrender, a timeout or a disconnect?

The opponent is paid exactly like a normal win, and the game is recorded as a loss for the player who gave up. A draw refunds both stakes.

### The server stopped while games were running

Every running game ends with both stakes returned and no result recorded. One caveat: plugins disable in reverse load order, so if your economy plugin goes down before SnFourInLine, those refunds cannot reach it. The console then holds one WARN line per refund, naming the player and the amount owed. After a hard crash (a kill, a watchdog halt) there are no refunds at all, because escrow lives in memory: check the log for games and listings that were running.

### A transfer failed. Who ate the money?

Nobody, silently. Every transfer is a console command with the balance read back around it: a take counts only when the balance dropped, a payout to an online player only when it rose. When a transfer does not move the money, the plugin refuses to start (or list) that bet, tells the player, and logs one WARN naming the exact `currencies.<id>.<give|take>-command` key, the player and the amount.

### Can I redesign the menus?

Yes. The three files under `guis/` own the layout mask, materials, names, lore and click actions, and the code reads the cells from the mask. Moving the sidebar or repositioning the 42 board cells works; the grid itself is always 7 columns by 6 rows.

### The plugin will not enable and the console says `License: FAIL`

Either the key in `plugins/.Sn-License/license.yml` is missing, wrong or expired, or the server cannot reach the license host, or the jar has been modified. The check runs on every startup, retries three times, then disables the plugin. Allow the host through your firewall and use the jar exactly as downloaded.
