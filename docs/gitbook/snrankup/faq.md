# FAQ

### How do I update SnRankUp?

Download the newer `snrankup-v*` release and replace the jar. Your `config.yml`, `rankup.yml`,
language file and menus auto-merge on restart, and player ranks are untouched.

### Does it support Folia?

SnRankUp targets Paper, on both 1.20.x and 1.21.x. Folia is not a supported platform.

### Why does the plugin not appear in `/plugins`?

Two usual causes. Either `SnLib.jar` is missing from `plugins/`, because SnRankUp declares
`depend: [SnLib]` and will not enable without it, or the bundle key in
`plugins/.Sn-License/license.yml` is still the placeholder. Both are reported in the server log at
startup.

### How do I add a rank?

Add an entry under `rankups:` in `rankup.yml` with an `order` and a `display`, then run
`/rankup reload`. Give it an order between the two ranks it belongs between. Nothing else has to
change: both menus read the ladder as it is.

### Why does my new rank not appear in the menu?

The single menu only ever shows the next rank, so a rank further ahead is not meant to be there.
The paginated menu shows every rank, so if one is missing it failed to load. Check the server log
after `/rankup reload`: a rank with no `order` or no `display` is reported by key.

### How do I make every claimed rank look the same in the ladder?

Write `material` (and `glow`, if you want it) on that state's template in `guis/rankup-list.yml` -
`rank-claimed`, `rank-ready`, `rank-next` or `rank-locked` - and every tile of that state uses it.
A rank that must stay different declares its own under `menu-item.states.<state>` in `rankup.yml`,
which beats the template. Before 2.1.0 those two fields were read and then ignored on a template,
so if the file looks configured and nothing changes, check the plugin version first.

### How do I make a rank cost money?

Keep the `vault` currency in `config.yml`, install Vault and an economy plugin, then price the rank
with `vault: 5000` under its `requirements`. See [Configuration](configuration.md).

### How do I make a rank cost tokens, gems or points?

Declare a `placeholder` currency in `config.yml`. It reads the balance from any PlaceholderAPI
placeholder and charges by running a console command, so anything that exposes a balance and a take
command works. Add `deposit` so the plugin can refund it.

### A rank became free, and I did not change its price. Why?

The plugin the currency comes from is not loaded. A currency whose backing plugin is missing is
skipped at load with a warning, and every requirement naming it is dropped, which makes the rank
cheaper rather than unreachable. The warning at boot is the signal.

### Why does a tile show `{req_vault}` as literal text?

That token only exists on a rank whose `requirements` name `vault`. If the currency was dropped at
load, or the line lives on a template shared by every rank, there is nothing behind the token and it
renders as written. Put currency lines on that rank's own `menu-item` lore.

### Why is `Next:` empty for my highest ranked players?

At the top of the ladder there is no next rank, so `%snrankup_next_prefix%` renders empty. Use
`%snrankup_next_num%`, which renders the max rank word instead, or remove the line from the menu.

### The leaderboard positions past a certain number are blank.

`top.size` bounds how many positions exist. The shipped single menu lists 1 to 10, so a smaller
`size` leaves the rest blank. Raise `size` or delete the extra lines.

### How do I rename a rank?

Change its key in `rankup.yml` and reload. Players stored on the old key keep their position: the
stored value is left alone when the key it names is gone, so nobody is knocked back to the start.

### Can a player rank up more than once at a time?

No. Each rankup advances exactly one step, to the first rank with a strictly greater `order`.

### How do I give someone a rank they earned elsewhere?

`/rankup force <player>` advances them one rank and fires that rank's rewards, without charging.
`/rankup set <player> <rank>` writes a position with no rewards at all.

### Can I run this across several servers?

Yes. Set `database.type: mysql` and point every server at the same database, and a player's rank
follows them. Each server still reads its own `rankup.yml`, so keep the ladders identical.
