# Configuration

`plugins/SnDrop/config.yml`. The file is managed: new keys from a future version are merged in on
boot and your values and comments are preserved. Never add a `config-version` key - SnDrop does
not use one.

## Drop window

```yaml
drop:
  # Length in seconds of the drop window /drop opens. Values below 1 are raised to 1.
  duration-seconds: 30

  # If true, /drop during an active window restarts the timer from now instead of
  # refusing. It never adds time to the running window.
  refresh-window-if-active: false
```

**`duration-seconds`** is how long a player can drop after running the command. A value below 1
is raised to 1 rather than rejected, so a typo cannot produce a window that is already over when
it opens. There is no upper limit.

**`refresh-window-if-active`** decides what a second `/drop` does while a window is running:

- `false` (default) - the command is refused and the player is told how many seconds remain. The
  running window keeps its original end.
- `true` - the timer restarts from now for the full duration. Note it does not stack: a player
  spamming the command holds a rolling window of `duration-seconds`, never a growing one.

## Messages

```yaml
messages:
  # Minimum gap in milliseconds between two "you cannot drop" warnings sent to the
  # same player. 0 warns on every blocked attempt.
  blocked-cooldown-millis: 1000
```

The rate limit is per player and applies only to the warning, never to the block itself: a drop
made while the warning is cooling down is still refused, silently. The timer starts when a warning
is actually sent, so a player holding the drop key gets one message per interval rather than one
message and then nothing.

`0` means every blocked attempt prints. Negative values are treated as `0`.

## Command aliases

```yaml
command:
  # Aliases of /drop. Re-read on /drop reload.
  aliases: []
```

Empty by default. Whatever you put here is authoritative and is re-read on `/drop reload`, so
aliases can be added or removed without a restart.

## Language

`plugins/SnDrop/lang/messages_en.yml` holds every line players see. Set the active file with the
`lang:` key at the top of `config.yml`.

| Key | When it is sent |
|---|---|
| `messages.blocked` | A drop was refused (rate-limited) |
| `messages.window-opened` | A window opened; `{seconds}` is the configured duration |
| `messages.already-active` | `/drop` refused because a window is running; `{seconds}` is the time LEFT |
| `messages.only-players` | Console tried to run `/drop` |

`{seconds}` means different things in the two window messages: the full duration when one opens,
the remaining time when one is refused. Remaining time always rounds up, so a window with half a
second left reports 1, never 0.

Blanking a value silences that line completely, which is the supported way to mute a message
without disabling anything. Colour codes are `&x` and `&#RRGGBB`.
