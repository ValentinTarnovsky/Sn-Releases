# FAQ

### How do I update SnCredits?

Download the newer `sncredits-v*` release and replace the jar, on the proxy and on every backend,
then restart them. Always replace both halves: they speak one protocol and are meant to match.

Your configuration files are not merged. When a file's `config-version` falls behind, SnCredits
renames yours to `old-<name>-v<version>-<timestamp>.yml` and writes a fresh default in its place.
Open the renamed copy and move your values across by hand.

### Why did a reload disconnect players from a backend?

Before 1.14.2 the proxy sent each backend its whole shop in one plugin message. A backend refuses
a plugin message larger than 32767 bytes and disconnects the player it arrived through, logging
`Payload may not be larger than 32767 bytes`. Once a shop grew past about 32 KB, every reload
kicked a player from that backend, and so did joining it right after it restarted.

Since 1.14.2 the proxy sends a message that large in parts and the backend puts it back together,
so the size of a shop no longer matters. The fix lives in the proxy: update the proxy and every
backend, and restart the proxy as well, or large shops keep disconnecting players.

After a reload the proxy console reports how many backends received the new configuration. A
backend with nobody online cannot receive it and keeps its previous shop until it restarts.

### Does it support Folia?

No. SnCredits declares no Folia support. It runs on Velocity and on Paper, from the same jar.

### Do I install it on the proxy or on the backends?

Both, always. The proxy half owns the balances, the shop definitions and every transaction. The
backend half draws the shop, coinflip and leaderboard menus, answers placeholders and forwards
console commands.

A backend without the jar cannot open any of those menus. `/credits balance` and `/credits pay`
still work there, because the proxy answers them.

### Can I run it with SQLite?

Yes. Set `database.type: sqlite` in `config.yml` and everything lives in
`plugins/sncredits/sncredits.db`. That suits a single proxy and needs no setup.

Choose MySQL or MariaDB when the data has to be reachable from outside the plugin, or when your
database already runs on another host. Note that `database.type` falls back to MySQL for any value
that is not exactly `sqlite`.

### Why does the console say there are no players online?

Because a backend console cannot talk to the proxy by itself. It forwards your command through a
player connected to that backend, so an empty backend leaves it no route.

Run the command from the proxy console instead. There it executes directly and needs nobody online.

### How do I rename the currency and the command?

`currency.name` in `config.yml` is both the display name and the root command, so setting it to
`coins` gives you `/coins`. `currency.symbol` sets the character shown beside amounts, and
`currency.command-aliases` lists the extra names.

Restart the proxy afterwards. The command name is claimed while the proxy starts, so a reload alone
does not move it.

### What does bypass mode do?

`/credits bypass on` puts you into test-purchase mode. Your shop clicks still run the item's
commands, so you can check that a reward actually works.

Those clicks cost nothing, write no history row, fire no webhook and announce nothing. Turn it off
with `/credits bypass off`. It applies to you alone, not to the server.

### Where do I put my license key?

In `plugins/.Sn-License/license.yml` on each backend, which the plugin creates on its first start
there. It is one shared file for every Sn bundle plugin on that server, so a single key unlocks the
whole pack.

Paste your key over the placeholder line and restart. Repeat it on every backend you run.
