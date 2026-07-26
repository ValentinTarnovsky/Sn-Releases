# Configuration

SnKillEffects ships four YAML files. New keys are auto-merged on boot; your edits and comments are preserved. There is no `config-version` to bump.

| File | What it holds |
|------|---------------|
| `config.yml` | General settings, database, limits and the locked-entry actions |
| `effects.yml` | The effect catalogue: one block per effect |
| `lang/messages_en.yml` | Every message the plugin sends |
| `guis/effects_menu.yml` | The catalogue menu layout and tiles |

## config.yml

```yaml
# ============================================================
#  SnKillEffects - configuration
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

# Runtime debug output (also toggleable live via /killeffects debug).
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
  # Aliases of /killeffects. Re-read on /killeffects reload.
  aliases: [ke, killeffect]

# ------------------------------------------------------------
#  Database. type=sqlite needs nothing else; type=mysql reads host/port/etc.
# ------------------------------------------------------------
database:
  # sqlite or mysql
  type: sqlite
  # MySQL connection (ignored when type is sqlite).
  host: localhost
  port: 3306
  database: snkilleffects
  username: root
  password: ""

# ------------------------------------------------------------
#  Random mode. The reserved "random" selection rolls a different
#  owned effect on every kill instead of a fixed one.
# ------------------------------------------------------------
random:
  # Enabled effects a player must own before random mode can be selected.
  min-owned: 2

# ------------------------------------------------------------
#  Preview. Plays an effect at the player's own location, granting nothing.
# ------------------------------------------------------------
preview:
  # Master toggle of /killeffects preview and the GUI right-click preview.
  enabled: true
  # Whether an effect the player does not own can be previewed.
  allow-locked: true
  # Seconds a player waits between two previews.
  cooldown: 3

# ------------------------------------------------------------
#  Expiry. Timed grants are pruned by a background sweeper, and every
#  read already treats an elapsed grant as not owned.
# ------------------------------------------------------------
expiry:
  # Seconds between two runs of the expiry sweeper.
  sweeper-interval: 300
  # Whether a player is told when one of their effects expires.
  notify-on-expiry: true

# ------------------------------------------------------------
#  Restrictions. Global gates applied before any effect renders.
#  Limitation: the lightning flash and every sound are broadcast by the
#  server, so /killeffects toggle cannot suppress them.
# ------------------------------------------------------------
restrictions:
  # Blocks around the death location within which an effect renders.
  view-radius: 48
  # Effects rendered per second server-wide; kills past it render nothing.
  max-effects-per-second: 20
  # Seconds between two plays of one effect whose block sets no cooldown.
  default-cooldown: 0
  # Percentage chance to play for an effect whose block sets no chance.
  default-chance: 100.0
  # Worlds where effects never play, matched case-insensitively by name.
  world-blacklist: []

# ------------------------------------------------------------
#  Feedback. Actions run when a player clicks an effect they do not own.
#  {locked-hint} resolves to messages.locked-hint of the active language file.
# ------------------------------------------------------------
locked-click-actions:
  - "[message] {locked-hint}"
  - "[sound] ENTITY_VILLAGER_NO 1.0 1.0"
```

## effects.yml

Every effect is one top-level block. The ids are plugin schema: each one is bound to a Java renderer, so an id you invent renders nothing, and a block you delete comes back on the next boot so that plugin updates can ship new effects. To turn an effect off, set its `enabled: false`.

```yaml
# ============================================================
#  SnKillEffects - effect catalogue
#  Managed by SnLib: new keys and newly shipped effects are auto-merged on
#  boot; your values and comments are preserved.
#
#  Every block below is plugin schema: its id is bound to a Java renderer.
#  A block you delete is restored on the next boot, and an id you invent
#  renders nothing. To turn an effect off, set its enabled: false.
#
#  Shared keys of every block:
#    enabled   - false hides the effect from the menu, the commands and play
#    display   - how the effect looks in the catalogue menu; lore is a LIST
#    cooldown  - seconds between two plays of this effect, 0 = none
#    chance    - 0-100, rolled on every kill
#    duration  - animation length in ticks, 1 = instant
#    sounds    - "SOUND_ID" or "SOUND_ID volume pitch"; empty plays nothing
#  The remaining keys are that effect's own tunables.
#
#  Particle names use the 1.21 spelling; the 1.20.4 name is resolved
#  automatically, so both server lines render the same.
# ============================================================
```

Each block shares the same core keys. Here is one complete block; the other fourteen follow the same shape, differing only in their own tunables:

```yaml
lightning:
  enabled: true
  display:
    material: LIGHTNING_ROD
    name: "&#8354f2&lLightning"
    lore:
      - "&7A bolt cracks down on your victim."
      - "&7No damage, no fire, pure show."
  cooldown: 0
  chance: 100.0
  duration: 1
  sounds:
    main: "ENTITY_LIGHTNING_BOLT_THUNDER 1.0 1.0"
    secondary: "ENTITY_LIGHTNING_BOLT_IMPACT 0.8 1.2"
  # Bolts struck at the death location.
  bolts: 1
```

### Per-effect tunables

Beyond the shared keys, each effect exposes the values that make sense for it:

| Effect | Its own keys |
|--------|--------------|
| `lightning` | `bolts` |
| `totem` | `particles.count`, `particles.spread`, `particles.speed` |
| `warden` | `steps`, `step-distance` |
| `tnt` | `particles.count`, `ring.count`, `ring.radius` |
| `fire_ring` | `radius`, `points`, `particles.speed`, `y-offset` |
| `anvil` | `height`, `scale`, `particles.count`, `particles.spread`, `particles.speed` |
| `rocket` | `effect-type`, `colors`, `fade-colors`, `flicker`, `trail` |
| `blood_splash` | `particles.count`, `particles.spread`, `particles.speed`, `dust.color`, `dust.size` |
| `soul_escape` | `radius`, `height`, `points-per-tick` |
| `ender_void` | `radius`, `particles.count`, `points-per-tick` |
| `skull_trophy` | `height`, `scale`, `spin-degrees-per-tick` |
| `firework_show` | `count`, `interval`, `radius`, `colors` |
| `shockwave` | `radius`, `points`, `y-offset` |
| `ice_shatter` | `particles.count`, `particles.spread`, `shards` |
| `dragon_roar` | `radius`, `points-per-tick`, `particles.speed` |

{% hint style="info" %}
Every one of these values is clamped into a safe range when it is read. A mistyped number caps the effect instead of freezing your server, and the shipped values all sit well inside their ranges, so clamping is only ever visible to somebody who typed a number they did not mean.
{% endhint %}

## lang/messages_en.yml

Every player-facing line. Do not write `{prefix}` into a value: the prefix at the top of the file is prepended automatically. Three list rows carry a leading `[noprefix]` tag that keeps them prefix-free while still resolving placeholders for the viewer; keep that tag if you restyle them.

To translate, copy the file to `lang/messages_<code>.yml` and set `lang:` in `config.yml`.

## guis/effects_menu.yml

The catalogue menu: its layout mask, its tiles and the three templates the plugin fills. The `items:` section is marked `# sn:extensible`, so a tile you delete stays deleted rather than coming back on the next boot. The `templates:` section is not marked, because the plugin looks those up by name and a missing one would break the menu.

Each template documents the placeholders the plugin supplies to it in a comment right above it.
