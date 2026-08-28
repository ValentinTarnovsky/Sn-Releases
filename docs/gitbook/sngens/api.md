# Developer API

SnGens exposes a public developer API for other plugins: custom Bukkit events, a read-only
query service, and a sell multiplier extension point. The API lives in the `com.sn.sngens.api`
package inside the plugin jar. There is no separate artifact.

{% hint style="warning" %}
Only `com.sn.sngens.api` is a stable, kept contract. Everything else in the jar is obfuscated
and internal. Do not call it.
{% endhint %}

## Getting the jar

1. Download the latest release jar.
2. Install it into your local Maven repository:

   ```
   mvn install:install-file -Dfile=SnGens-<version>.jar -DgroupId=com.sn \
     -DartifactId=SnGens -Dversion=<version> -Dpackaging=jar
   ```

3. Depend on it with `provided` scope. Never shade it.

```xml
<dependency>
  <groupId>com.sn</groupId>
  <artifactId>SnGens</artifactId>
  <version><!-- the version you installed --></version>
  <scope>provided</scope>
</dependency>
```

Gradle users can drop the jar in `libs/` and use `compileOnly files("libs/SnGens-<version>.jar")`
instead.

## Quick start

Declare the dependency in your `plugin.yml`:

```yaml
depend: [SnGens]        # or softdepend if the integration is optional
```

Resolve the API when you need it:

```java
import com.sn.sngens.api.SnGensAPI;
import com.sn.sngens.api.SnGensProvider;

SnGensAPI api = SnGensProvider.get();
if (api != null) {
    int placed = api.getPlacedCount(player.getUniqueId());
}
```

`SnGensProvider.isAvailable()` is a boolean shortcut for the same check. The facade is
registered with the Bukkit services manager at the very end of SnGens' startup, so with
`depend` it is ready in your own `onEnable`.

{% hint style="info" %}
Resolve the reference when you need it. Do not cache it across a SnGens reload.
{% endhint %}

## Master switch

API events can be disabled by the server owner with `api-events.enabled: false` in
`config.yml`. No event is then dispatched at all, and cancellable events behave as if nothing
cancelled them, so gameplay is unaffected. The query service and the extension point stay
available either way.

## Events

All events live in `com.sn.sngens.api.event`, extend `org.bukkit.event.Event` and carry the
usual static `getHandlerList()`.

Cancellable events fire before the action. Cancelling aborts it cleanly, with no money taken
and no items lost.

| Event | Fired when | Cancel effect |
|-------|-----------|---------------|
| `GeneratorPlaceEvent` | A player places a generator | The generator is not placed |
| `HopperPlaceEvent` | A player places an infinite hopper | The hopper is not placed |
| `GeneratorUpgradeEvent` | A player upgrades one generator | No upgrade, no charge |
| `GeneratorBulkUpgradeEvent` | A bulk upgrade runs from the menu | The whole batch is aborted |
| `GeneratorBreakEvent` | A generator is broken or removed | The generator stays |
| `GeneratorRepairEvent` | A corrupted generator is repaired | It stays corrupted |
| `GeneratorSellEvent` | Any sale, before the deposit | No payout, no items consumed |
| `CollectorSellEvent` | A collector's contents are sold | The contents stay stored |

Notification events fire after the fact. They cannot be cancelled.

| Event | Fired when | Thread |
|-------|-----------|--------|
| `SellwandUseEvent` | A sellwand swing completed | Main or region thread |
| `GeneratorCorruptEvent` | A generator became corrupted | Main or region thread |
| `ServerEventStartEvent` | A server wide event started | Delivered on the main thread |
| `ServerEventEndEvent` | A server wide event ended | Delivered on the main thread |
| `TopUpdateEvent` | The leaderboard finished recomputing | Delivered on the main thread |
| `RefundIssuedEvent` | Generators were added to a player's vault | Delivered on the main thread |

The last four originate off the main thread inside SnGens, but the plugin hops through its
Folia aware scheduler before dispatching, so your listener always runs on the main thread and
can touch the Bukkit API freely.

Listen like any Bukkit event:

```java
@EventHandler
public void onUpgrade(GeneratorUpgradeEvent event) {
    Player player = event.getPlayer();
    getLogger().info(player.getName()
            + " upgrades " + event.getGenerator().generatorId()
            + " into " + event.getToGeneratorId()
            + " for " + event.getCost());
}
```

```java
@EventHandler
public void onHopperPlace(HopperPlaceEvent event) {
    if (event.getLocation().getWorld().getName().equals("pvp_arena")) {
        event.setCancelled(true);
        event.getPlayer().sendMessage("You cannot place hoppers here.");
    }
}
```

`GeneratorSellEvent#getContext()` returns a `SellContext` with the item count, the base value,
the effective multiplier, the final amount and the source of the sale
(`PLAYER_INVENTORY`, `SELLWAND`, `COLLECTOR`, `HOPPER`, `OTHER`).

```java
@EventHandler
public void onSell(GeneratorSellEvent event) {
    SellContext c = event.getContext();
    if (c.source() == SellSource.SELLWAND && c.finalAmount() > 1_000_000) {
        event.setCancelled(true);
    }
}
```

Event payloads are read-only. To influence the money a sale pays, use the extension point
below instead of looking for setters.

## Query service

Synchronous methods read in-memory state. Call them on the main thread.

| Method | Returns | Notes |
|--------|---------|-------|
| `getPlayerProfile(UUID)` | `PlayerGensProfile` | Placed count, max slots, stored multiplier, hide-stats flag, online flag |
| `getPlacedCount(UUID)` | `int` | Generators currently placed |
| `getMaxSlots(UUID)` | `int` | Slot allowance. Permission bonuses only count while the player is online |
| `getEffectiveSellMultiplier(Player)` | `double` | The fully resolved multiplier for a sale |
| `getGeneratorAt(Location)` | `Optional<GeneratorView>` | Empty when the block is not a generator |
| `isGenerator(Location)` | `boolean` | Cheap existence check |
| `getPlacedHopperCount(UUID)` | `int` | Infinite hoppers placed by that owner |
| `getTotalPlacedHoppers()` | `int` | Infinite hoppers placed server wide |
| `getOwnerHoppers(UUID)` | `List<HopperView>` | Every hopper that owner placed, with its contents |
| `getIslandHoppers(UUID)` | `List<HopperView>` | The same for every member of that player's island |
| `getPlacedCollectorCount(UUID)` | `int` | Collectors placed by that owner |
| `getTotalPlacedCollectors()` | `int` | Collectors placed server wide |
| `getOwnerCollectors(UUID)` | `List<CollectorView>` | Every collector that owner placed, with its contents |
| `getIslandCollectors(UUID)` | `List<CollectorView>` | The same for every member of that player's island |
| `getActiveServerEvent()` | `Optional<ServerEventView>` | Empty between events |
| `getTopSnapshot()` | `List<TopEntryView>` | The last computed leaderboard |
| `getRank(UUID)` | `OptionalInt` | Rank of a player or an island |
| `getValue(UUID)` | `OptionalDouble` | Value of a player or an island |
| `isIslandModeAvailable()` | `boolean` | Whether the skyblock integration is active |
| `getApiVersion()` | `String` | The API contract version |

```java
api.getGeneratorAt(block.getLocation())
   .ifPresent(view -> player.sendMessage("Generator: " + view.displayName()));

for (HopperView hopper : api.getOwnerHoppers(player.getUniqueId())) {
    for (StoredItemView stored : hopper.contents()) {
        // stored.item() is a decoded ItemStack, ready for a menu icon
        menu.addIcon(stored.item(), stored.amount(), stored.totalValue());
    }
}
```

{% hint style="info" %}
Hoppers and collectors are held in memory in full, so these listings need no database
read and no chunk scanning. You can call them straight from a menu open.
{% endhint %}

Asynchronous methods hit the database, so they also see generators in unloaded chunks.

| Method | Returns | Notes |
|--------|---------|-------|
| `getIslandGeneratorStats(UUID)` | `CompletableFuture<IslandGeneratorStats>` | Folds in every island member, or falls back to the single owner |
| `getOwnerGeneratorCount(UUID)` | `CompletableFuture<Long>` | Total generators of one owner |
| `getOwnerGeneratorValue(UUID)` | `CompletableFuture<Double>` | Total value of one owner |
| `getOwnerGenerators(UUID)` | `CompletableFuture<List<GeneratorView>>` | Every generator that owner placed, including unloaded chunks |
| `getIslandGenerators(UUID)` | `CompletableFuture<List<GeneratorView>>` | The same for every member of that player's island |

{% hint style="warning" %}
`CompletableFuture` results complete on an async thread. Hop back to the main thread before
touching the Bukkit API.
{% endhint %}

```java
api.getIslandGeneratorStats(player.getUniqueId()).thenAccept(stats ->
    Bukkit.getScheduler().runTask(this, () ->
        player.sendMessage("Island: " + stats.totalCount()
                + " generators worth " + stats.totalValue())));
```

## Views

Every returned object is an immutable snapshot. `Location` and `ItemStack` values are cloned,
so mutating them never affects SnGens.

| View | Fields |
|------|--------|
| `GeneratorView` | `owner`, `location`, `generatorId`, `displayName`, `corrupted`, `nextGeneratorId`, `upgradeCost`, `drops` |
| `DropView` | `id`, `chance`, `item`, `sellValue`, `commands` |
| `PlayerGensProfile` | `uuid`, `placedCount`, `maxSlots`, `sellMultiplier`, `hideStats`, `online` |
| `IslandGeneratorStats` | `islandId`, `members`, `totalCount`, `countByType`, `totalValue` |
| `ServerEventView` | `id`, `type`, `displayName`, `remainingSeconds` |
| `TopEntryView` | `rank`, `scopeId`, `headOwner`, `displayName`, `value` |
| `SellContext` | `itemCount`, `baseValue`, `effectiveMultiplier`, `finalAmount`, `source` |
| `HopperView` | `id`, `owner`, `location`, `maxTypes`, `slotsUsed`, `totalItems`, `contents`, `estimatedValue`, `createdAt`, `updatedAt` |
| `CollectorView` | `id`, `owner`, `location`, `totalItems`, `slotsUsed`, `contents`, `estimatedValue`, `createdAt`, `updatedAt` |
| `StoredItemView` | `key`, `item`, `amount`, `unitValue`, `totalValue` |

A `HopperView` or `CollectorView` is a snapshot taken when you asked for it. Both keep
absorbing items afterwards, so query again on each menu refresh instead of caching.

## Extension points

| Interface | What it influences | Neutral return |
|-----------|--------------------|----------------|
| `GensSellMultiplier` | Adds to the additive part of a player's sell multiplier | `0` |
| `MultiplicativeGensSellMultiplier` | Multiplies on top of the additive total, like equipment | `0` |

Register through the facade, from your own `onEnable`:

```java
// Additive: VIPs get +0.5x
api.registerSellMultiplier(this, ctx ->
        ctx.player().hasPermission("myrank.vip") ? 0.5 : 0.0);

// Multiplicative: +10% on top of whatever the player already has
api.registerSellMultiplier(this,
        (MultiplicativeGensSellMultiplier) ctx -> 0.10);
```

The context passed to a provider is a `GensSellContext`, with `player()` and
`sellwandMultiplier()`. The second is only present for sellwand sales.

Providers run inside the payout path of every sale, so keep them fast. The server's
`player-multiplier-limit` clamp still applies on top of whatever your provider contributes.
Registrations last until SnGens reloads or disables.

{% hint style="info" %}
The per drop generation path deliberately has no event. It is the hottest loop in the plugin,
and firing an event per drop would cost more than the feature is worth. Use
`getPlacedCount` or `getIslandGeneratorStats` for production totals instead.
{% endhint %}

## Versioning

Call `getApiVersion()` for the API contract version. It is independent of the plugin version.
Additions bump the minor component. Existing members are never removed or changed; deprecated
members keep working.
