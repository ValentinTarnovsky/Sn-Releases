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
| `/captcha reload` | `sncaptcha.admin.reload` | Re-reads every configuration file. |
| `/captcha debug` | `sncaptcha.admin.debug` | Toggles the debug channel, which traces every accrual change, emit decision and click. |
| `/captcha help` | none | Lists the commands you have access to. |

## Notes on the admin commands

`/captcha status` and `/captcha info` read the persisted values **plus** the seconds accrued since the last save, so what you see always matches what the trigger is comparing. A player who is visibly farming never shows a stale number.

`/captcha force` reports what actually happened rather than assuming success. It tells you when the target is exempt as a Bedrock player, when they already have a captcha waiting, and when no captcha can be sent at all because the board cannot be drawn.

{% hint style="danger" %}
`/captcha reset <player>` is destructive and cannot be undone. It clears the player's lifetime passed and failed counts along with their current state.
{% endhint %}
