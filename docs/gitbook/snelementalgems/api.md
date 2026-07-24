# Developer API

SnElementalGems exposes a public developer API for other plugins: custom Bukkit events and a
read-only query service. The API lives in the `com.sn.elementalgems.api` package inside the
plugin jar. There is no separate artifact.

{% hint style="warning" %}
Only `com.sn.elementalgems.api` is a stable, kept contract. Everything else in the jar is
obfuscated and internal. Do not call it.
{% endhint %}

## Getting the jar

1. Download the latest release jar.
2. Install it into your local Maven repository:

   ```
   mvn install:install-file -Dfile=SnElementalGems-<version>.jar -DgroupId=com.sn \
     -DartifactId=SnElementalGems -Dversion=<version> -Dpackaging=jar
   ```

3. Depend on it with `provided` scope. Never shade it.

## Quick start

Declare the dependency in your `plugin.yml`:

```yaml
depend: [SnElementalGems]      # or softdepend if optional
```

Resolve the API when you need it:

```java
SnElementalGemsAPI api = SnElementalGemsProvider.get();
if (api != null) {
    // use the api
}
```

{% hint style="info" %}
Resolve the reference when you need it. Do not cache it across a SnElementalGems reload.
{% endhint %}

## Master switch

API events can be disabled by the server owner with `api-events.enabled: false` in
`config.yml`. The query service stays available either way.

## Events

Cancellable events fire before the action. Cancelling aborts it.

| Event | Payload | Fired when | Cancel effect |
|-------|---------|-----------|---------------|
| `PlayerGemDropEvent` | `player`, `cause`, `amount` | Before a gem drop is credited or spawned for a player | No balance is credited and no gem spawns |
| `PlayerGemPayEvent` | `sender`, `receiver`, `amount` | Before one player transfers gems to another through `/gems pay` | Neither balance changes |
| `PlayerGemPurchaseEvent` | `player`, `category`, `rewardId`, `price` | Before a player is charged for a shop reward | The player is not charged and no reward is delivered |
| `PlayerGemUpgradeEvent` | `player`, `upgradeType`, `newLevel`, `price` | Before a player is charged for an upgrade level | The player is not charged and the level is not applied |

Notification events fire after the fact. They cannot be cancelled.

| Event | Payload | Fired when | Thread |
|-------|---------|-----------|--------|
| `PlayerGemsChangeEvent` | `uuid`, `cause`, `oldBalance`, `newBalance` | After a player's gem balance changed | Main |

The `cause` on `PlayerGemsChangeEvent` is a short tag for the origin of the change. It is one of
`DROP`, `PAY`, `SHOP`, `UPGRADE`, `WITHDRAW`, `REDEEM` or `ADMIN`.

Listen like any Bukkit event:

```java
@EventHandler
public void on(PlayerGemDropEvent event) {
    // read the payload; cancellable events support event.setCancelled(true)
}
```

Event payloads are read-only. Cancellation is the only mutation a listener gets; look for
`setCancelled`, not value setters.

## Query service

Synchronous methods read in-memory state. Call them on the main thread. Balance and upgrade
queries serve online players; an offline player reads as a zero balance and an empty view.

| Method | Returns | Notes |
|--------|---------|-------|
| `getBalance(uuid)` | `double` | Cached balance, `0` when the player is offline |
| `has(uuid, amount)` | `boolean` | Whether the cached balance covers `amount` |
| `getUpgradeLevel(uuid, upgradeType)` | `int` | Level for a named upgrade, `0` when offline or the name is unknown |
| `getPlayer(uuid)` | `Optional<GemPlayerView>` | Balance and upgrade-level snapshot, empty when the player is offline |
| `getTop(limit)` | `List<GemTopEntry>` | Cached leaderboard rows, best first, up to `limit` |
| `getRank(uuid)` | `OptionalInt` | The 1-based leaderboard rank, empty when the player is unranked |
| `isGemItem(item)` | `boolean` | Whether a stack is a gem item, tested by its persistent data tag |
| `createGemItem(amount)` | `ItemStack` | A fresh, independent gem stack; `amount` is floored at 1 |
| `createValuedGemItem(value)` | `ItemStack` | A single value-carrying gem worth `value`, clamped to at least 0.01 (`@since` API 1.1.0) |
| `getApiVersion()` | `String` | The API contract version |

Views are immutable snapshots. `GemPlayerView` copies its upgrade-level map, keyed by lowercase
upgrade name. `GemTopEntry` is a plain record whose `name` may be `null` when never recorded.

## Versioning

Call `getApiVersion()` for the API contract version. It is independent of the plugin version.
Additions bump the minor component. Existing members are never removed or changed; deprecated
members keep working. As of plugin 1.2.0, `getApiVersion()` returns `1.1.0`.
