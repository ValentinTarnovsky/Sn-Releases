# Developer API

SnEnvoys exposes a public developer API for other plugins: custom Bukkit events and a
read-only query service. The API lives in the `com.sn.envoys.api` package inside the plugin
jar. There is no separate artifact.

{% hint style="warning" %}
Only `com.sn.envoys.api` is a stable, kept contract. Everything else in the jar is obfuscated
and internal. Do not call it.
{% endhint %}

## Getting the jar

1. Download the latest release jar.
2. Install it into your local Maven repository:

   ```
   mvn install:install-file -Dfile=SnEnvoys-<version>.jar -DgroupId=com.sn \
     -DartifactId=SnEnvoys -Dversion=<version> -Dpackaging=jar
   ```

3. Depend on it with `provided` scope. Never shade it.

## Quick start

Declare the dependency in your `plugin.yml`:

```yaml
depend: [SnEnvoys]      # or softdepend if optional
```

Resolve the API when you need it:

```java
SnEnvoysAPI api = SnEnvoysProvider.get();
if (api != null) {
    // use the api
}
```

{% hint style="info" %}
Resolve the reference when you need it. Do not cache it across a SnEnvoys reload.
{% endhint %}

## Master switch

API events can be disabled by the server owner with `api-events.enabled: false` in
`config.yml`. The query service stays available either way.

## Events

Package: `com.sn.envoys.api.event`

Cancellable events fire before the action. Cancelling aborts it.

| Event | Fired when | Cancel effect |
|-------|-----------|---------------|
| `EnvoyStartEvent` | Before an envoy event spawns its blocks | The event does not start; the countdown is rescheduled |
| `EnvoyClaimEvent` | When a player claims an envoy, before the reward runs | The envoy is left in place, the interaction is still cancelled, and no reward is granted |

Notification events fire after the fact. They cannot be cancelled.

| Event | Fired when | Thread |
|-------|-----------|--------|
| `EnvoyEndEvent` | When an envoy event ends (reason: `ALL_CLAIMED`, `TIMEOUT`, `FORCED` or `SHUTDOWN`), after remaining blocks are restored | Main |

Listen like any Bukkit event:

```java
@EventHandler
public void onClaim(EnvoyClaimEvent event) {
    // event.getReward() may be null when the reward pool is empty
    if (event.getReward() != null) {
        plugin.getLogger().info(event.getPlayer().getName() + " claimed an envoy");
    }
}
```

Event payloads are read-only.

## Query service

Synchronous methods read in-memory state. Call them on the main thread.

| Method | Returns | Notes |
|--------|---------|-------|
| `getSecondsToNextEvent()` | `int` | Seconds until the next event while idle, `0` while one is running |
| `isEventActive()` | `boolean` | Whether an envoy event is currently running |
| `getActiveEnvoyCount()` | `int` | Unclaimed envoys in the running event, `0` while idle |
| `getRegisteredLocations()` | `List<EnvoyLocation>` | An immutable snapshot of every registered spawn location |
| `getApiVersion()` | `String` | The API contract version |

## Versioning

Call `getApiVersion()` for the API contract version. It is independent of the plugin version.
Additions bump the minor component. Existing members are never removed or changed; deprecated
members keep working.
