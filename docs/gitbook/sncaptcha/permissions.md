# Permissions

Every node defaults to `op`. There is deliberately no basic-access node and no wildcard: `/captcha` carries no permission at all, because solving a captcha has to be reachable by every player.

| Permission | Default | Description |
|-----------|---------|-------------|
| `sncaptcha.admin` | op | Parent node. Grants every admin action below, plus the staff notifications. |
| `sncaptcha.admin.status` | op | Use `/captcha status`. |
| `sncaptcha.admin.info` | op | Use `/captcha info <player>`. |
| `sncaptcha.admin.force` | op | Use `/captcha force <player>`. |
| `sncaptcha.admin.reset` | op | Use `/captcha reset <player>`. |
| `sncaptcha.admin.reload` | op | Use `/captcha reload`. |
| `sncaptcha.admin.debug` | op | Use `/captcha debug`. |
| `sncaptcha.admin.update` | op | Receive the update notice in chat when a newer version is published. |
| `sncaptcha.notify` | op | Receive the staff alerts: captcha sent, unanswered, failed, solved. Grantable on its own to a moderator rank. |
| `sncaptcha.bypass` | op | Total exemption. No farming time is tracked and no captcha is ever sent. |

{% hint style="info" %}
`sncaptcha.bypass` is cached rather than checked on every block break, because the break listener runs off the main thread. The cache is filled when a player joins and refreshed for everyone online on `/captcha reload`, so granting or revoking it for a player who is already connected takes effect on the next reload or their next login, not instantly.
{% endhint %}
