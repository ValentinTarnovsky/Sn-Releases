# Configuration

SnCaptcha ships with the four files below. New keys are auto-merged on boot; your edits and comments are preserved.

Two of them carry a warning worth reading before you edit: `heads.yml` holds the skin data that makes the digits readable, and `guis/captcha.yml` holds the anti-script contract. Both files explain in their own comments which parts are safe to change.

## config.yml

```yaml
# ============================================================
#  SnCaptcha - configuration
#  Managed by SnLib: new keys are auto-merged on boot; your values and
#  comments are preserved. Do NOT add a config-version key (retired).
#  Set update-configs: false to freeze this file (SnLib only warns about
#  missing keys instead of inserting them).
#  Sections marked "# sn:extensible" are yours: entries you delete there
#  stay deleted.
# ============================================================

# Active language code; loads lang/messages_<code>.yml (falls back to en).
lang: en

# Master switch of the SnLib auto-updater for this plugin's managed files.
update-configs: true

# Runtime debug output (also toggleable live via /captcha debug).
debug:
  # Master toggle of the debug output.
  enabled: false
  # Verbosity threshold: OFF, INFO, DEBUG or TRACE.
  level: DEBUG
  # Category filter; an empty list lets every category through.
  categories: []

# ------------------------------------------------------------
#  Main command.
# ------------------------------------------------------------
command:
  # Extra aliases of /captcha, re-read on /captcha reload: one you ADD here
  # starts working without a restart, and removing it again unregisters it.
  # The two below are also declared in the plugin's own plugin.yml, which the
  # server registers itself - so deleting them HERE does not switch them off.
  aliases: [snc, captchafarming]

# ------------------------------------------------------------
#  Database. type=sqlite needs nothing else; type=mysql reads host/port/etc.
# ------------------------------------------------------------
database:
  # sqlite or mysql
  type: sqlite
  # MySQL connection (ignored when type is sqlite).
  host: localhost
  port: 3306
  database: sncaptcha
  username: root
  password: ""

# ------------------------------------------------------------
#  Captcha. When it fires, how long the player has, and how it looks.
# ------------------------------------------------------------
captcha:
  # Seconds the player has to open /captcha and click the four number heads in
  # order before the captcha times out and counts as a failure. Clamped to
  # 5..3600: below the floor the captcha expires before anyone can answer it,
  # which fails every player on the server.
  duration-seconds: 30

  # Lower bound of the per-player farming-time threshold, in seconds. Each
  # player draws their own value once from this range and keeps it stored; a
  # new one is drawn after every captcha, solved or failed. Minimum 1.
  threshold-min-seconds: 1500
  # Upper bound of that same range, in seconds. Both bounds can be drawn.
  # Keep it ABOVE the minimum. If it is below, the range collapses and every
  # player draws the same fixed threshold, which someone can measure once and
  # then dodge on every account. The plugin warns once when that happens.
  threshold-max-seconds: 2400

  # How often (seconds) the plugin compares each player's accumulated farming
  # time against their threshold. Minimum 1.
  check-interval-seconds: 5

  # Fraction of the solve window after which staff get the "still not
  # answering" alert. 0.5 means halfway through. Clamped to 0.0..1.0, so a
  # value outside that range only moves the alert to one end of the window.
  mid-warn-fraction: 0.5

  # Apply Blindness while the captcha is active, cleared when it resolves or
  # fails. It does not affect the menu, so the player can still read the head
  # skins; it only stops them from farming until they solve it.
  blindness: true

  # The captcha menu. Its layout lives in guis/captcha.yml.
  gui:
    # A wrong (out-of-order) click restarts the sequence and spends one
    # attempt. This number is the wrong click that FAILS the captcha, so 3
    # means the third one runs the sanction ladder and the player is warned
    # twice before it. 0 and 1 both fail on the very first wrong click. The
    # budget persists across reopening the menu.
    max-wrong-clicks: 3

# ------------------------------------------------------------
#  Farming time. What counts as "currently farming", which is what accrues.
# ------------------------------------------------------------
farming:
  # A player counts as actively farming if they broke a block in a tracked
  # world within this many seconds. Minimum 1.
  active-window-seconds: 10

  # After the last break, farming time keeps accumulating for this long, as
  # long as the player stays in a tracked world. Leaving pauses the countdown
  # and returning resumes it where it left off. Minimum 0, which means the
  # timer stops the moment the active window above lapses.
  stop-farming-grace-seconds: 180

# ------------------------------------------------------------
#  Where it applies.
# ------------------------------------------------------------
# Only blocks broken in these worlds accrue farming time, and only here is a
# captcha ever emitted. Every other world is ignored entirely, and an empty
# list switches the plugin off.
worlds:
  - gens
  - gens2
  - work

# Region exemptions, read through WorldGuard when it is installed.
regions:
  # While a player stands inside any of these region ids the farming timer
  # pauses and no captcha fires. Matched case-insensitively, shared across
  # worlds. Without WorldGuard the list is simply ignored.
  exempt: []

# ------------------------------------------------------------
#  Sanctions. Console commands run when a player fails a captcha, by timeout
#  or by exhausting the wrong-click budget. Placeholder: %player%
#
#  IMPORTANT: the commands below are examples, and broadcast / kick / tempban /
#  ban are NOT all vanilla - they come from a permissions or punishment plugin
#  such as EssentialsX or LiteBans. A tier whose command your server does not
#  have runs nothing and prints "Unknown command" in the console, while staff
#  are still told the tier was dispatched. Replace them with the commands your
#  server actually has.
# ------------------------------------------------------------
sanctions:
  # When the failure count is below the lowest tier defined here, run the
  # HIGHEST tier instead of running nothing.
  #
  # WARNING: this only matters if you delete or renumber tier 1 below. With the
  # tiers as shipped, a first failure always lands on tier 1 and this setting
  # never comes into play. Remove the "1:" entry while this is true, and a
  # player's FIRST failed captcha runs the LAST tier - the permanent ban. The
  # plugin prints a warning at boot when your ladder is in that shape. Set this
  # to false if you would rather a failure below the lowest tier do nothing.
  fallback-to-highest: true

  # Restart the ladder once the highest tier has fired, instead of leaving the
  # player pinned to the top tier forever. With the tiers below, the sixth
  # failure starts again at tier 1.
  reset-after-highest-tier: true

  # Forgive a player's failure counter if this many hours pass without a new
  # failure. 0 = never forgive, and a negative value behaves the same way.
  # Lifetime statistics are never touched.
  reset-failures-after-hours: 24

  # One entry per tier: the failure count that reaches it, and the console
  # commands run there. A failure runs the HIGHEST tier whose number is at or
  # below the player's failure count, so with the tiers below a fourth failure
  # runs tier 3 again.
  # sn:extensible
  failures:
    1:
      - "broadcast &c[Captcha] &f%player% &cfailed the captcha (attempt 1)"
    2:
      - "kick %player% &cSecond captcha failed"
    3:
      - "tempban %player% 1h Suspected macro (3 captchas failed)"
    5:
      - "ban %player% Confirmed macro use (5+ captchas failed)"

# ------------------------------------------------------------
#  Alerts. Staff notifications, sent to holders of sncaptcha.notify.
# ------------------------------------------------------------
alerts:
  # Send the alerts in game.
  in-game: true
  # Append the alerts to alerts.log in the plugin folder. The file is
  # append-only and the plugin never truncates or rotates it, so its size is
  # yours to manage.
  log-file: true
  # Which events raise an alert. Turning all of them off, or turning both
  # sinks above off, skips the whole alert path.
  #
  # on-emit and on-resolved fire once per captcha each, so on a busy farming
  # server they are the two that fill staff chat. Turn them off first if the
  # volume is too high; on-mid-timer and on-timeout-fail are the ones that
  # report a player who is not answering.
  events:
    # A captcha was sent to a player.
    on-emit: true
    # The player has not answered by the mid-warn point.
    on-mid-timer: true
    # The captcha failed and a sanction was applied.
    on-timeout-fail: true
    # The captcha was solved.
    on-resolved: true

# ------------------------------------------------------------
#  Top farmers.
# ------------------------------------------------------------
top-farmers:
  # Rows shown by /captcha status. Clamped to 1..100: each printed row may cost
  # a name lookup for an offline player, so the ceiling bounds that work.
  limit: 10
```

## guis/captcha.yml

```yaml
# ============================================================
#  SnCaptcha - the captcha menu
#  Managed by SnLib: new keys merge in on boot, your values are kept.
# ============================================================
#
#  ANTI-SCRIPT CONTRACT - READ BEFORE EDITING
#
#  The four heads show the digits 1 to 4 and the player must click them in
#  ascending order. The ONLY thing that says which head is which digit is the
#  rendered skin. The heads therefore carry no display name, no lore and no
#  hidden data, and both the digit-to-cell assignment and the skin variant are
#  re-randomized every time the menu opens.
#
#  Do NOT add display-name or lore to the "head" template. Anything a client
#  can read, a bot can read, and a head that announces its digit turns the
#  captcha into a no-op for exactly the players it is meant to catch.
#
#  The menu is also deliberately silent: no open-sound and no close-sound.
#
#  You CAN safely: move the letters around in the layout, change the filler
#  and indicator materials, and change the title.
# ============================================================

title: "&#8354f2&l/CAPTCHA &8| &7Click 1 2 3 4"

# Discard anything that is not a plain mouse click (number keys, drops,
# off-hand swaps). The click handler additionally ignores shift-clicks and
# double-clicks, so a fast or laggy click can never spend an attempt.
strict-clicks: true

# h = a number head, p = the indicator under it, f = inert filler.
# The plugin checks at load that "h" and "p" resolve to FOUR cells each, and
# falls back to this shipped layout with a warning when they do not: a head
# with no indicator, or two heads on one cell, leaves a blinded player with a
# captcha they cannot finish and then sanctions them for it.
layout:
  - "fffffffff"
  - "fhfhfhfhf"
  - "fpfpfpfpf"
  - "fffffffff"

regions:
  # The four head cells, left to right.
  heads:
    key: 'h'
  # The four indicator cells, in the SAME left-to-right order: indicator i
  # belongs to head i, so keep the two rows aligned when you move them.
  progress:
    key: 'p'

items:
  filler:
    material: GRAY_STAINED_GLASS_PANE
    display-name: " "
    key: 'f'

templates:
  # The head. Its appearance is supplied by the plugin at bind time (a textured
  # skull built from heads.yml), and this template deliberately declares
  # NEITHER display-name NOR lore: that makes SnLib pass the supplied stack
  # through untouched, which is what keeps the head unreadable. See the
  # contract at the top of this file.
  head:
    material: PLAYER_HEAD
    # The plugin's own action. It routes the click by the CELL it landed on,
    # never by anything read off the item, which is what keeps the head
    # unreadable. Do not replace it and do not add a second action here.
    click-actions:
      - "[captcha-click]"

  # Indicator under a head that has not been clicked in order yet.
  indicator-pending:
    material: RED_STAINED_GLASS_PANE
    display-name: " "

  # Indicator under a head already clicked in the correct order.
  indicator-solved:
    material: LIME_STAINED_GLASS_PANE
    display-name: " "
```

## heads.yml

The base64 texture values are several hundred characters each, so only the first of each digit's list is shown below. The shipped file contains six variants per digit.

```yaml
# ============================================================
#  SnCaptcha - number head textures
#  Managed by SnLib: new keys merge in on boot, your values are kept.
# ============================================================
#
#  Base64 profile textures for the four captcha heads. One entry per digit,
#  each listing several visual variants of the SAME number. One variant per
#  digit is drawn at random every time the menu opens, which raises the cost
#  of a texture-to-digit lookup table without preventing one: the texture has
#  to reach the client for the skin to render at all, so a purpose-written
#  client mod can always map it back. What the variants buy is that the table
#  is bigger and has to be rebuilt whenever this list changes. Adding your own
#  private variants is what makes it expensive; the shipped ones travel with
#  the jar.
#
#  The heads render with no name, no lore and no hidden data: only the visible
#  skin says which digit a head is, so an unattended macro - the thing this
#  plugin exists to stop - has nothing to read. See guis/captcha.yml for the
#  rest of that contract.
#
#  ADD or REMOVE variants freely - more variants is strictly better. Do NOT
#  delete one of the four digit keys: the captcha always asks for 1 to 4, and
#  a digit with no variants renders as a blank head that nobody can order.
# ============================================================

heads:
  '1':
    - "eyJ0ZXh0dXJlcyI6eyJTS0lOIjp7InVybCI6Imh0...(truncated)"
    # ...the remaining texture values are elided here for readability.
  '2':
    - "eyJ0ZXh0dXJlcyI6eyJTS0lOIjp7InVybCI6Imh0...(truncated)"
    # ...the remaining texture values are elided here for readability.
  '3':
    - "eyJ0ZXh0dXJlcyI6eyJTS0lOIjp7InVybCI6Imh0...(truncated)"
    # ...the remaining texture values are elided here for readability.
  '4':
    - "eyJ0ZXh0dXJlcyI6eyJTS0lOIjp7InVybCI6Imh0...(truncated)"
    # ...the remaining texture values are elided here for readability.
```

## lang/messages_en.yml

```yaml
# ============================================================
#  SnCaptcha - language file (English)
#  Managed by SnLib: new keys merge in on boot, your values are kept.
#  Restyle any line freely; your edits survive updates.
# ============================================================

# Prefix prepended to every single-line message sent via sn.lang().send(...)
prefix: "&#8354f2&lSnCaptcha &8| &7"

# snlib.* is SnLib's shared command contract. Every Sn plugin ships
# this block UNCHANGED: SnLib bundles NEUTRAL defaults for any key you omit and
# merges them in on boot, so an incomplete block leaks unbranded lines. Shipping
# the FULL block is what keeps the whole fleet visually identical. Placeholders
# {plugin} {usage} {description} {page} {total} {min} {max} {value} {command}
# are filled by SnLib. NEVER put {prefix} here (it is auto-prepended).
snlib:
  no-permission: "&cYou do not have permission to use this command."
  usage: "&cUsage: &e{usage}"
  invalid-number: "&cInvalid number: &e{value}"
  invalid-value: "&cInvalid value: &e{value}"
  out-of-range: "&cValue must be between &e{min} &cand &e{max}&c: &e{value}"
  number-too-small: "&cValue must be at least &e{min}&c: &e{value}"
  player-not-found: "&cPlayer not found: &e{value}"
  unknown-subcommand: "&cUnknown subcommand: &e{value}"
  reload-done: "&aConfiguration reloaded."
  help:
    header: "&8&m----------&r &#8354f2&l{plugin} &8&m----------"
    entry: "&#8354f2{usage} &7{description}"
    footer: "&7Page &f{page}&7/&f{total} &8- &#8354f2/{command} help <page>"

# Your own plugin messages. Do NOT write {prefix} in any value: SnLib
# auto-prepends the prefix above to every single-line message sent via
# sn.lang().send(...). A literal {prefix} renders verbatim, and SnLib
# logs a one-time WARN when the active lang file contains one.
messages:

  # ----------------------------------------------------------
  #  The captcha itself, as the challenged player sees it.
  # ----------------------------------------------------------
  captcha:
    # On-screen title while a captcha is active, re-sent once per second.
    # This ONE line is a screen title, not a chat message: it is sent with
    # sn.lang().title(...) and its parts are separated by semicolons, in this
    # order - title;subtitle;fade-in;stay;fade-out. The three times are in
    # ticks (20 per second). Placeholder: {seconds} (time left to solve).
    title: "&c&l/CAPTCHA;&7Run &e/captcha &7to solve it &8| &c{seconds}s;0;24;6"

    # An out-of-order click: the sequence restarts and one attempt is spent.
    # Placeholder: {remaining} (attempts left before the captcha fails).
    wrong-click: "&cWrong order. Start again. &7({remaining} attempts left)"
    # The wrong-click budget ran out, so the captcha counts as failed.
    wrong-fail: "&cToo many wrong clicks. Captcha failed."
    # The solve window ran out, so the captcha counts as failed.
    expired: "&cTime is up. Captcha failed."
    # The four heads were clicked in the right order.
    resolved: "&aCaptcha solved. Keep playing."
    # /captcha typed by a player who has no captcha waiting for them.
    no-active: "&7You have no active captcha right now."

  # ----------------------------------------------------------
  #  Staff alerts, sent to holders of sncaptcha.notify and written to
  #  alerts.log. Each one is gated by its alerts.events toggle in config.yml.
  # ----------------------------------------------------------
  alerts:
    # NOTE on PlaceholderAPI in this block: these lines are sent to every staff
    # member and also appended to alerts.log. A %papi% token therefore resolves
    # against the staff member READING the alert, never against the player it is
    # about, and in alerts.log it resolves against nobody and is written out raw.
    # Use the {tokens} documented below for anything about the offender.
    #
    # A captcha was sent to a player. {reason} comes from the reasons block.
    emit: "&eCaptcha sent to &f{player} &7(reason &f{reason}&7, world &f{world}&7)"
    # The player has still not answered when the mid-warn point is reached.
    # Placeholder: {seconds} (seconds since the captcha was sent).
    mid-timer: "&6{player} &7has been sitting on the captcha for &e{seconds}s&7."
    # The captcha failed and the sanction ladder ran. {tier} is the tier that
    # fired, {failures} is the player's failure count. They are two different
    # numbers: with non-contiguous tiers a count of 4 fires tier 3.
    # Says DISPATCHED, not "applied", and the distinction is deliberate: the plugin hands the
    # tier's lines to the console and cannot know whether they worked. A command for a plugin you
    # do not have installed, a command that errors, or a tier you deliberately left blank all end
    # here - so a line claiming the sanction landed would be a lie the console contradicts.
    timeout-fail: "&c{player} &7failed the captcha. Tier &f{tier} &7dispatched (&f{failures} &7failures)."
    # The player solved their captcha.
    resolved: "&a{player} &7solved the captcha."

  # ----------------------------------------------------------
  #  Why a captcha was sent. These are FRAGMENTS spliced into the {reason}
  #  placeholder of alerts.emit, never sent on their own.
  # ----------------------------------------------------------
  reasons:
    # The player reached their accumulated farming-time threshold.
    time-gate: "accumulated farming time"
    # A staff member ran /captcha force.
    forced: "forced by staff"

  # ----------------------------------------------------------
  #  Command feedback. The permission, usage, unknown-subcommand and
  #  player-not-found lines come from the snlib block above.
  # ----------------------------------------------------------
  commands:
    # /captcha force succeeded: the target now has a live captcha.
    force-success: "&aCaptcha forced on &f{player}&a."
    # /captcha force on a Bedrock player: they are exempt, nothing was sent.
    force-bedrock: "&e{player} &7is a Bedrock player and is exempt from captchas."
    # /captcha force on a player who is already solving one: nothing was sent, and the
    # captcha they already have is left running untouched.
    force-already-active: "&e{player} &7already has a captcha waiting for them."
    # /captcha force while no board can be drawn (a broken guis/captcha.yml or heads.yml).
    # No captcha is emitted to anybody in this state: one would blind the player and start a
    # deadline they cannot beat. The console names the exact file to fix.
    # /captcha force while the plugin is refusing to emit at all. Deliberately does NOT name the
    # board: the refusal has more than one cause (a broken menu, unreadable head textures, or a
    # missing captcha title), and naming the wrong one sends staff to the wrong file. The console
    # line carries the actual reason.
    force-no-board: "&cNo captcha can be sent right now - the plugin is refusing to emit. The reason is in the console."
    # /captcha force where the captcha started and then something threw while it was being
    # presented, so it was withdrawn rather than left half-applied. The server is fine; this
    # target can be tried again. The console carries the error.
    force-failed: "&cCould not send a captcha to &f{player}&c. See the console."
    # /captcha reset wiped the target's stored captcha data.
    reset-success: "&aCaptcha data reset for &f{player}&a."
    # /captcha reset could not write to the database.
    reset-failed: "&cCould not reset &f{player}&c. See the console for details."
    # /captcha info on a player who has no stored captcha data yet.
    no-data: "&7No captcha data stored for &f{player} &7yet."
    # The database read behind /captcha info or /captcha status failed.
    load-failed: "&cCould not read the captcha database. See the console for details."

# ============================================================
#  Multi-line chat views. Rendered with sn.lang().getList / get, so they are
#  sent WITHOUT the prefix, one chat line per entry.
# ============================================================
lists:

  # /captcha info <player>. Placeholders: {player} {farmed} {threshold}
  # {remaining} {state} {failures} {passed} {failed} {last-captcha} {world}.
  # {farmed} already includes the seconds accrued since the last save, so it
  # keeps climbing while the player farms. {last-captcha} is how long AGO the
  # player's last captcha ended, solved or failed.
  info:
    - "&8&m----------&r &#8354f2&lCaptcha Info &8&m----------"
    - " &#8354f2* &fPlayer        &8> &e{player}"
    - " &#8354f2* &fFarmed        &8> &b{farmed}"
    - " &#8354f2* &fThreshold     &8> &e{threshold}"
    - " &#8354f2* &fRemaining     &8> &c{remaining}"
    - " &#8354f2* &fState         &8> &f{state}"
    - " &#8354f2* &fFailures      &8> &c{failures} &7(sanction tier counter)"
    - " &#8354f2* &fLifetime      &8> &a{passed} solved &8/ &c{failed} failed"
    - " &#8354f2* &fLast captcha  &8> &f{last-captcha}"
    - " &#8354f2* &fWorld         &8> &a{world}"
    - "&8&m----------------------------------------"

  # Header of /captcha status, printed before the rows.
  top-header:
    - "&8&m----------&r &#8354f2&lTop Farmers &8&m----------"
  # One row of /captcha status. Placeholders: {medal} {rank} {player}
  # {farmed} {failures}.
  top-entry: " {medal} &f{player} &8> &e{farmed} &8| &c{failures} failed"
  # Printed instead of the rows when nobody has farmed yet.
  top-empty: " &7No data yet."
  # Footer of /captcha status.
  top-footer:
    - "&8&m----------------------------------------"

  # Rank badge spliced into the {medal} placeholder of top-entry.
  top-medal-1: "&#FFD700&l#1"
  top-medal-2: "&#C0C0C0&l#2"
  top-medal-3: "&#CD7F32&l#3"
  # Every rank past the third. Placeholder: {rank}.
  top-medal-other: "&8&l#{rank}"

# ============================================================
#  Short state words. Each one is a fragment substituted into a {state} or
#  {tier} slot of the messages and views above, so a restyle here applies
#  everywhere at once.
# ============================================================
status:
  # Not accruing farming time: never started, or the grace window ran out.
  idle: "&7Idle"
  # Broke a block in a tracked world recently; accruing.
  active: "&aActive"
  # No recent break but still inside the grace window; still accruing.
  grace: "&eGrace"
  # Left the tracked world or stands in an exempt region; frozen, not accruing.
  paused: "&6Paused"
  # Shown where the player has no value yet: no threshold rolled, no captcha
  # ever received, not online in any world. Uncolored so it inherits the color
  # of the line it lands in.
  none: "None"
  # Shown instead of the remaining time when the player has already reached
  # their threshold and is only waiting for the next check. Deliberately not
  # "None": accumulated time never decays, so a player sits here until they
  # farm again, and this is the state /captcha info is most often opened to
  # explain.
  due: "Due"
  # Shown instead of a name that no longer resolves. Color codes are stripped
  # on read: the value also feeds name matching, which does not render them.
  unknown: "Unknown"
  # Shown instead of the sanction tier in the failure alert when the player's
  # row could not be read, so neither the tier nor the failure count exists.
  unknown-tier: "?"
  # Shown instead of the sanction tier when the failure WAS recorded but no
  # configured tier matched it, so no commands ran. Reachable by emptying or
  # deleting sanctions.failures; the console says so at boot too.
  no-tier: "none"
```
