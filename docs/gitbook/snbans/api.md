# Developer API

SnBans exposes a public developer API for other plugins: custom Bukkit events and a read-only
query service. The API lives in the `com.sn.bans.api` package inside the plugin jar. There is no
separate artifact.

{% hint style="warning" %}
Only `com.sn.bans.api` is a stable, kept contract. Everything else in the jar is obfuscated and
internal. Do not call it.
{% endhint %}

## Paper only

One SnBans jar runs on Paper and on Velocity, but the API is the Paper-side surface. A Velocity
proxy has no Bukkit `ServicesManager` and no Bukkit event bus. A proxy-only install therefore
exposes no API at all: the internal hook seam is bound to a no-op that vetoes nothing and observes
nothing.

{% hint style="danger" %}
On a network, your listener has to live on the Paper backends. A plugin loaded only on the proxy
never receives an SnBans event and never resolves the service.
{% endhint %}

A backend still sees the whole network. A punishment issued on the proxy, or on a peer backend,
reaches every backend through cross-server sync. It surfaces there as a notification event carrying
a `REMOTE` origin.

## Getting the jar

1. Download the jar from the `Sn-Releases` release tagged `snbans-v<version>`.
2. Install it into your local Maven repository:

   ```
   mvn install:install-file -Dfile=SnBans-<version>.jar -DgroupId=com.sn \
     -DartifactId=SnBans -Dversion=<version> -Dpackaging=jar
   ```

3. Depend on it with `provided` scope. Never shade it.

## Quick start

Declare the dependency in your `plugin.yml`:

```yaml
depend: [SnBans]      # or softdepend if optional
```

Resolve the API when you need it:

```java
SnBansAPI api = SnBansProvider.get();
if (api != null) {
    // use the api
}
```

`SnBansProvider.isAvailable()` answers the same question without handing back the service.

{% hint style="info" %}
Resolve the reference when you need it. Do not cache it across a SnBans reload: a reload replaces
the registered service instance.
{% endhint %}

With `softdepend` instead of `depend`, guard for `null` on every call. Nothing guarantees that
SnBans enabled before you.

## Master switch

API events can be disabled by the server owner with `api-events.enabled: false` in `config.yml`.
The default is `true`.

While the switch is off, no event is dispatched at all and nothing is even built. The cancellable
hooks report "not cancelled", so punishments proceed exactly as if no consumer existed. The query
service stays available either way.

{% hint style="warning" %}
The owner of the server, not you, decides whether your listeners run. Behaviour that must not be
switched off cannot depend on an SnBans event.
{% endhint %}

The setting is read per dispatch, so `/snbans reload` takes effect at once.

## Events

Cancellable events fire before the action. Cancelling aborts it, with no partial state left behind.

| Event | Fired when | Cancel effect |
|-------|-----------|---------------|
| `PunishmentIssueEvent` | A punishment initiated on this server is about to be written, fully resolved | No row is inserted, nothing is enforced, nothing is announced and no webhook is sent |
| `PunishmentRevokeEvent` | A single revert on this server is about to be recorded: `/unban`, `/unmute` or `/unblacklist` | The punishment stays exactly as it is, no mute is released and nothing is announced |
| `PunishmentRollbackEvent` | A confirmed rollback sweep on this server is about to DELETE one staff member's punishments in a window | Not one matched punishment is erased, no mute is released and nothing is announced |

Notification events fire after the fact. They cannot be cancelled.

| Event | Fired when | Thread |
|-------|-----------|--------|
| `PunishmentIssuedEvent` | A punishment has been stored, either here (`LOCAL`) or on a peer that cross-server sync delivered (`REMOTE`) | Main |
| `PunishmentRevokedEvent` | A punishment has been lifted, either here (`LOCAL`) or on a peer that cross-server sync delivered (`REMOTE`) | Main |
| `AltScanCompletedEvent` | The automatic join scan found accounts worth warning about on the address a player just connected from | Main |

Every event fires synchronously on the server thread, cancellable and notification alike. A remote
arrival resolves on a database worker, and SnBans hops onto the server thread before dispatching.
Your listener may touch the Bukkit API freely.

Listen like any Bukkit event:

```java
@EventHandler
public void on(PunishmentIssuedEvent event) {
    // read the payload; cancellable events support event.setCancelled(true)
}
```

Event payloads are read-only. Cancellation is the only mutation a listener gets: there is no setter
for the duration, the reason or the scope.

### Payloads

- `PunishmentIssueEvent.getRequest()` returns a `PunishmentRequestView`.
- `PunishmentRevokeEvent` carries `getPunishment()`, `getActor()`, `getActorName()` and
  `getReason()`. The actor is `null` for the console, and the reason is `null` today.
- `PunishmentRollbackEvent` carries `getStaff()`, `getStaffName()`, `getWindowMillis()`,
  `getMatched()`, `getActor()` and `getActorName()`.
- `PunishmentIssuedEvent` and `PunishmentRevokedEvent` carry `getPunishment()` and `getOrigin()`.
- `AltScanCompletedEvent.getScan()` returns an `AltScanView`.

`getMatched()` is how many still-active punishments the sweep would revert. It is a count taken
moments earlier, not a promise: a peer writing in the same window can still move the real total.

See the payload views under the query service for the records themselves.

### Local and remote

A cancellable event fires only for an action initiated on this server. A punishment or revert that
arrives from a peer is already committed elsewhere, so there is nothing left to veto.

Notifications fire for both origins. `getOrigin()` is exactly `LOCAL` or `REMOTE`. That makes one
backend's listener a network-wide feed, which is what a Discord relay or a statistics collector
wants.

{% hint style="warning" %}
On several backends the same punishment is reported once per backend: once as `LOCAL`, once as
`REMOTE` on each of the others. Deduplicate on `PunishmentView.id()`.
{% endhint %}

A rollback is one bulk action, so it is vetoed as one. `PunishmentRevokeEvent` does not fire per
swept punishment, and neither does `PunishmentRevokedEvent` on the sweeping server.

{% hint style="warning" %}
**A peer's rollback reaches you as nothing at all, since v1.2.0.** A confirmed sweep now DELETEs
the punishments it undoes rather than stamping them as removed, and the cross-server revert feed
reads rows whose removal was just recorded - so an erased row is in no feed. `PunishmentRollbackEvent`
and `PunishmentRevokedEvent` therefore fire only on the server that ran the sweep. On other backends
the rows simply stop existing: `getHistory` and `getActivePunishments` stop returning them, and a
consumer that mirrors punishments elsewhere should reconcile against the facade rather than rely on
a revoke event for this one case. Ordinary `/unban` and `/unmute` reverts still propagate row by row
with a `REMOTE` origin, unchanged.
{% endhint %}

`AltScanCompletedEvent` covers the automatic join warning only. It never fires for a clean join,
for a manual `/alts` lookup, or for `scanAlts` on the facade. It is off entirely while
`alts.auto-scan-on-join` is off or `alts.notify-states` is empty.

### Cancelling politely

A cancelled action is not silent to the staff member. SnBans answers them the branded
`messages.api-denied` line, which reads "Another plugin blocked that action; it should have told you
why".

{% hint style="warning" %}
SnBans cannot know why you objected. A plugin that cancels should send the staff member its own
message explaining the refusal.
{% endhint %}

A listener that throws never blocks staff work. The action goes through, and SnBans logs a warning
naming the failure. Keep a listener fast for the same reason: a cancellable dispatch that has not
answered within a few seconds is read as "not cancelled".

Here is a guard that refuses permanent bans to staff without a permission of its own:

```java
public final class PermanentBanGuard implements Listener {

    @EventHandler(ignoreCancelled = true)
    public void onIssue(PunishmentIssueEvent event) {
        PunishmentRequestView request = event.getRequest();
        if (!"BAN".equals(request.type()) || request.expiresAt() != null) {
            return;   // not a permanent ban
        }
        if (request.staff() == null) {
            return;   // the console may always do it
        }
        Player staff = Bukkit.getPlayer(request.staff());
        if (staff == null || staff.hasPermission("myplugin.permanent")) {
            return;
        }
        event.setCancelled(true);
        staff.sendMessage("Permanent bans need the myplugin.permanent permission.");
    }
}
```

## Query service

The facade is read-only by design. No method issues, edits or lifts a punishment, because that
would bypass the permission and hierarchy checks the commands apply. To influence a punishment, use
the cancellable events above.

Synchronous methods read in-memory state and answer on the calling thread.

| Method | Returns | Notes |
|--------|---------|-------|
| `isMuted(uuid)` | `boolean` | Whether a player on THIS server is muted; one map lookup, safe on a hot path |
| `activeMute(uuid)` | `Optional<PunishmentView>` | The mute silencing that player, with its reason, expiry and issuer |
| `getApiVersion()` | `String` | The API contract version |

Both mute methods read the in-memory cache the login check populated. A mute whose duration ran out
is treated as absent and dropped on the spot.

{% hint style="warning" %}
The mute cache is keyed by the connecting player. For an offline player, or one connected to another
backend, `isMuted` returns `false` whatever the database says. It is not a database question.
{% endhint %}

Use `hasActive(uuid, "MUTE")` for the authoritative answer out of the shared database, subject to the
account scope described below. Because the cache follows the session, an IP mute is answered
correctly for every account it reaches on this server.

Asynchronous methods return `CompletableFuture` and complete off the main thread.

| Method | Returns | Notes |
|--------|---------|-------|
| `activePunishments(uuid)` | `CompletableFuture<List<PunishmentView>>` | Punishments still in force against the account, newest first; an empty list means clean |
| `hasActive(uuid, type)` | `CompletableFuture<Boolean>` | Type name matched case-insensitively: `BAN`, `MUTE` or `BLACKLIST`; an unknown name answers `false` instead of throwing |
| `history(uuid, page, size)` | `CompletableFuture<List<PunishmentView>>` | Every punishment ever recorded against the account, lifted and expired included, newest first |
| `staffHistory(staffUuid, page, size)` | `CompletableFuture<List<PunishmentView>>` | Every punishment one staff member issued, newest first; console-issued rows carry no staff UUID and are unreachable |
| `scanAlts(uuid)` | `CompletableFuture<AltScanView>` | Accounts whose last known address is this account's last known address |

{% hint style="warning" %}
`CompletableFuture` results complete on a database worker thread. Hop back to the main thread before
touching the Bukkit API.
{% endhint %}

A future may also complete exceptionally when the database is unreachable, or when the pool was torn
down mid-reload. Attach a handler rather than assuming success.

```java
SnBansAPI api = SnBansProvider.get();
if (api == null) {
    return;
}
api.hasActive(target, "MUTE")
        .thenAccept(muted -> Bukkit.getScheduler().runTask(plugin, () -> {
            // back on the server thread: the Bukkit API is safe here
            sender.sendMessage(muted ? "That account is muted." : "That account may talk.");
        }))
        .exceptionally(failure -> {
            plugin.getLogger().warning("SnBans lookup failed: " + failure);
            return null;
        });
```

### Paging

`history` and `staffHistory` clamp their bounds rather than refusing them, so no argument makes them
throw. A `page` below 1 reads as page 1. A `size` outside 1 to 50 is pulled to the nearest end of
that range.

A page past the end of the history is an empty list, not the last real page. Ask for page 1 with a
large size, or walk the pages until one comes back short.

### Scope of the account queries

`activePunishments` and `hasActive` answer about the ACCOUNT. They see the punishments scoped to the
UUID, which is scope `ACCOUNT` and scope `BOTH`.

{% hint style="info" %}
A purely address-scoped punishment is not counted, even when it would still deny that player's
login. The address a player will next connect from is not a property of their account.
{% endhint %}

`scanAlts` is the heaviest query on the facade. Do not run it per tick or per chat message. An
account with no login row yields a scan with an empty account list, which is the correct answer for
an account SnBans has never seen.

### Payload views

Every event payload and every query result is one of four immutable records. Internal enums cross
the boundary as their names, so a new punishment type in a later release cannot break your build.

| View | Carries |
|------|---------|
| `PunishmentView` | `id`, `type`, `scope`, `target`, `targetName`, `staff`, `staffName`, `reason`, `template`, `server`, `createdAt`, `expiresAt`, `status`, `liftedByName`, `liftedAt`, `liftReason` |
| `PunishmentRequestView` | `type`, `scope`, `target`, `targetName`, `staff`, `staffName`, `reason`, `template`, `expiresAt`, `silent` |
| `AltScanView` | `target`, `targetName`, `accounts`, `truncated` |
| `AltAccountView` | `uuid`, `name`, `state`, `lastSeen` |

`status()` is exactly `ACTIVE`, `EXPIRED` or `LIFTED`, and `LIFTED` wins: a reverted punishment
reads `LIFTED` even when its duration would also have run out. `expiresAt()` is `null` for a
permanent punishment, and `staff()` is `null` when the console acted.

`PunishmentRequestView` is the pre-write twin of `PunishmentView`. It has no id, no status and no
lift fields, because the row does not exist yet. Every other value is already final: the template
matched, the escalation ladder was walked and the expiry was computed.

{% hint style="info" %}
No view carries an IP. SnBans stores and enforces against addresses, but an address is personal data
that a consumer has no contract reason to read. Whether a punishment covers the address is still
visible through `scope()`.
{% endhint %}

`AltAccountView.state()` is the single state the account is presented in, ranked internally. It is
exactly `BLACKLISTED`, `IPBANNED`, `BANNED`, `ONLINE` or `OFFLINE`. A muted account is never
presented as muted, because a mute does not stop it being online.

`AltScanView.accounts()` is an unmodifiable copy, normally starting with the scanned player. Two
cases hold no entry for the target: an empty scan, and a join scan, which keeps only the accounts
worth warning about. Match on `AltAccountView.uuid()` when you need the target's own entry.

{% hint style="warning" %}
Check `truncated()` before presenting a scan as complete. SnBans caps how many accounts one address
may contribute, so a saturated address yields its most recently seen accounts only.
{% endhint %}

## Versioning

Call `getApiVersion()` for the API contract version. It is independent of the plugin version.
Additions bump the minor component. Existing members are never removed or changed; deprecated
members keep working.
