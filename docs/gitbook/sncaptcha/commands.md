# Commands

Everything lives under `/captcha`, with the aliases `/snc` and `/captchafarming`. You can add more aliases under `command.aliases` in `config.yml`, and they register on `/captcha reload` without a restart.

`/captcha` itself carries no permission, on purpose: a challenged player has to be able to answer, whatever their rank.

| Command | Permission | Description |
|---------|-----------|-------------|
| `/captcha` | none | Opens your captcha board when you have one waiting. Tells you so when you do not. |
| `/captcha status` | `sncaptcha.admin.status` | Lists the top farmers by accumulated farming time. |
| `/captcha info <player>` | `sncaptcha.admin.info` | Shows one player's farming time, their threshold, how much is left, and their captcha history. |
| `/captcha force <player>` | `sncaptcha.admin.force` | Sends a captcha to an online player immediately, bypassing the timer. |
| `/captcha reset <player>` | `sncaptcha.admin.reset` | Clears a player's stored captcha data: farming time, threshold, failure count and lifetime statistics. |
| `/captcha alerts` | `sncaptcha.notify` | Switches your own staff alerts on, or off again. They start off for everybody. |
| `/captcha reload` | `sncaptcha.admin.reload` | Re-reads every configuration file. |
| `/captcha debug` | `sncaptcha.admin.debug` | Toggles the debug channel, which traces every accrual change, emit decision and click. |
| `/captcha help` | none | Lists the commands you have access to. |

## Notes on the admin commands

`/captcha status` and `/captcha info` read the persisted values **plus** the seconds accrued since the last save, so what you see always matches what the trigger is comparing. A player who is visibly farming never shows a stale number.

`/captcha force` reports what actually happened rather than assuming success. It tells you when the target is exempt as a Bedrock player, when they already have a captcha waiting, and when no captcha can be sent at all because the board cannot be drawn.

{% hint style="danger" %}
`/captcha reset <player>` is destructive and cannot be undone. It clears the player's lifetime passed and failed counts along with their current state. It does **not** touch anybody's alert preference, which is stored separately.
{% endhint %}

## /captcha alerts

The staff alerts are **opt in, and off by default for everybody**. `sncaptcha.notify` entitles a staff member to them; `/captcha alerts` is what asks for them. Run it once to switch them on, run it again to switch them off.

The reason is volume: `on-emit` and `on-resolved` fire once per captcha each, so on a busy farming server every holder of an admin rank used to have their chat filled by a plugin they were not working on. Now only the people who asked for the alerts get them.

Your answer is stored per player and survives a relog and a restart. It has no effect on `alerts.log`, which is a server-wide record, and it sits **under** `alerts.in-game` in `config.yml`: with that switched off, nobody receives anything in game whatever they toggled.

{% hint style="info" %}
Upgrading from an earlier version? Your staff will stop seeing the alerts until each of them runs `/captcha alerts` once. That is the intended behaviour, not a regression - tell them, or they will report the plugin as broken.
{% endhint %}
