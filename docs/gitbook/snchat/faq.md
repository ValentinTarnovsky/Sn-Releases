# FAQ

### How do I update SnChat?

Download the newer `snchat-v*` release and replace the jar. Configs auto-merge on restart.

### Does it support Folia?

No, SnChat is not Folia-compatible. It targets Paper 1.20.x and 1.21.x.

### My players suddenly cannot run any commands. What happened?

The command whitelist is on out of the box. `blockcommands.yml` ships with a populated `default` group, and every group not listed there falls back to it. Operators hold `snchat.bypass.blockcommands`, so staff notice nothing while players are locked out.

Either add your groups to `blockcommands.yml`, or set `command-blocker.enabled: false` in `config.yml`.

### My staff stopped receiving violation alerts after updating to 2.1.0.

That is the change, not a bug. Alerts used to be on for every `snchat.notify` holder and the command existed to silence them; they are now opt-in. The permission decides who is **eligible**, and each staff member turns their own on with `/snchat alerts`.

It only has to be done once - the choice is saved in `data.yml` and survives relog and restart, which the old session-scoped toggle did not.

### A player typed `<red>hello` and it appeared as literal text. Why?

Styling applies only to a message that carries a `&` somewhere. A message with no `&` is published exactly as typed. That guard stops SnChat overwriting a line another plugin already coloured.

Type `&r<red>hello` instead. The player also needs `snchat.color`.

### I granted `snchat.color` but `[center]` still does nothing.

The four line tags are a separate grant, `snchat.prefixtags`. They affect the whole line rather than a span of it, so selling coloured chat does not also sell line centring. They only work at the very start of a message.

### My chat format has no `{message}` and the messages vanished.

That is deliberate. A format without `{message}` renders the format alone and drops the message. Add `{message}` back.

### Everything in the message body is grey. Where does that come from?

The colour written in front of `{message}` carries onto the body. The shipped format ends with `&7{message}`, so the body renders grey. Codes the sender typed themselves override it.

### Someone shared a barrier block and their whole message disappeared.

Sharing a blacklisted item cancels the message. The blacklist under `item-display.blacklist` ships with the command blocks, barriers, bedrock and the debug stick. It is an anti-abuse rule with no bypass permission, so it applies to operators too. Edit the list to change what is allowed.

### Can a viewer take items out of an `[inv]` or `[ec]` window?

No. The replica is frozen at the moment the message was sent, and every click and drag inside it is cancelled. Nothing in it is a real item.

### Why did an `[inv]` tag stop working after a few minutes?

Each snapshot expires after `snapshot-ttl-seconds`, which defaults to 600. The click stops working at exactly the same moment the snapshot does.

A player is also capped at `snapshots.max-per-player` live snapshots at once. Past that their oldest is dropped and its tag answers as expired.

### My announcement does not appear in the rotation.

Only keys listed under `rotation:` are cycled. A key configured but left out of the rotation is still sendable with `/snchat announce <name>`, which is how you keep a manual-only announcement.

### Does SnChat provide PlaceholderAPI placeholders?

No. SnChat resolves `%papi%` placeholders from other plugins inside its own formats, hovers, click actions and announcements. It does not register any of its own.

### The console says `[Sn] License: FAIL` and my key is correct.

The license check needs outbound HTTPS. A firewalled or offline server shows the same line as a wrong key. Check that the server can reach the internet.
