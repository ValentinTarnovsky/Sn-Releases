# Placeholders

Requires PlaceholderAPI. Without it these simply do not resolve; everything else works normally.

Every placeholder names a core. For a core called `alpha`:

| Placeholder | Value |
|-------------|-------|
| `%sndtc_alpha_health%` | Health right now |
| `%sndtc_alpha_max_health%` | Size of the health pool |
| `%sndtc_alpha_percent%` | Health as a whole percentage, truncated |
| `%sndtc_alpha_status%` | The core's state word |
| `%sndtc_alpha_next%` | Countdown to the next regeneration |
| `%sndtc_alpha_damage%` | Hits the **requesting player** has landed this session |

## Details

**Case does not matter**, on either half. `%sndtc_MyCore_health%`, `%sndtc_mycore_health%` and
`%sndtc_mycore_HEALTH%` are all the same request.

**A typo stays visible.** A placeholder naming a core that does not exist is left in the text as
written rather than rendering blank, so `%sndtc_alpah_health%` is obvious on the scoreboard
instead of silently disappearing.

**`_status`** returns `status.active`, `status.inactive` or `status.no-schedule` from
`lang/messages_en.yml`, so restyling the word there restyles it here too.

**`_next`** shows the countdown to the next regeneration. For a core that is alive it shows the
`status.active` word, and for one with no schedule at all it shows `status.no-schedule` - there
is no time to count down to in either case.

**`_damage`** is per viewer: it answers for whoever the placeholder is being resolved *for*, not
for the core's top damager. It counts hits since the core last started, because starting a core
is what wipes the board.

## Where they do and do not work

These are **per-viewer** placeholders, so they need a player to resolve against.

They work in a scoreboard, a tab list, a chat format, an action bar, or anywhere else a plugin
resolves against a specific player.

They do **not** work in `hologram.lines`. One hologram is shown to everyone, so SnDTC's own
tokens have no viewer to answer for. Use the local `{core}` `{health}` `{max_health}` `{percent}`
`{status}` `{time}` tokens there instead - they do the same job and are documented in
[Configuration](configuration.md).

They also do not resolve in `display.bossbar.title`, for the same reason: a boss bar carries one
title for every viewer. The action bar **does** render per viewer, so they work there.
