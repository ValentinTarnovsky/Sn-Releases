# Developer API

SnCompanions exposes a public developer API for other plugins: custom Bukkit events and a read-only
query service. The API lives in the `com.sn.companions.api` package inside the plugin jar. There is no
separate artifact.

{% hint style="warning" %}
Only `com.sn.companions.api` is a stable, kept contract. Everything else in the jar is obfuscated and
internal. Do not call it.
{% endhint %}

## Getting the jar

1. Download the latest release jar.
2. Install it into your local Maven repository:

   ```
   mvn install:install-file -Dfile=SnCompanions-<version>.jar -DgroupId=com.sn \
     -DartifactId=SnCompanions -Dversion=<version> -Dpackaging=jar
   ```

3. Depend on it with `provided` scope. Never shade it.

## Quick start

Declare the dependency in your `plugin.yml`:

```yaml
depend: [SnCompanions]      # or softdepend if optional
```

Resolve the API when you need it:

```java
SnCompanionsAPI api = SnCompanionsProvider.get();
if (api != null) {
    api.getEquippedCompanions(player.getUniqueId())
       .forEach(companion -> getLogger().info(companion.companionId() + " lvl " + companion.level()));
}
```

{% hint style="info" %}
Resolve the reference when you need it. Do not cache it across a SnCompanions reload.
{% endhint %}

## Master switch

API events can be disabled by the server owner with `api-events.enabled: false` in `config.yml`.
Cancellable hooks then report "not cancelled" so gameplay proceeds. The query service stays
available either way.

## Events

All events are in `com.sn.companions.api.event` and fire on the main thread.

Cancellable events fire before the action. Cancelling aborts it.

| Event | Fired when | Cancel effect |
|-------|-----------|---------------|
| `CompanionEquipEvent` | A player equips a stored companion from the companion menu | The companion stays in storage |
| `CompanionUnequipEvent` | A player sends an equipped companion back to storage | The companion keeps its slot and its buffs |
| `CompanionEggOpenEvent` | A player opens one or more companion eggs, after the gates and before anything is charged or granted | Nothing is charged and nothing is granted |
| `CompanionFuseEvent` | A player commits a fusion, before anything is charged | Nothing is charged and no companion is destroyed |
| `CompanionGroupDeleteEvent` | A player mass-deletes a group from the bulk delete menu | Every companion in the group stays |

{% hint style="warning" %}
These fire only for player-driven actions. The `/companions admin` equivalents bypass them on purpose,
so an operator override is never vetoable.
{% endhint %}

Notification events fire after the fact. They cannot be cancelled.

| Event | Fired when | Payload |
|-------|-----------|---------|
| `CompanionEggRewardEvent` | An egg open finished and its companions are persisted | `eggId`, the `companionsWon` map, `eggsOpened` and `charged` |
| `CompanionLevelUpEvent` | An equipped companion gained at least one level | The companion after the level up, and how many levels it gained |
| `CompanionFusedEvent` | A fusion consumed its parents | Whether it succeeded, and the companion it produced |

{% hint style="info" %}
`CompanionEggRewardEvent` fires only when the open actually produced something, and it fires
**after** the companion rows are persisted, so every companion it names is already queryable through
the facade when your listener runs. Its `companionsWon` map reports what was CREATED, not what the
table decided, so an open that only partly landed reports the smaller number. A refused open (storage
full, the master switch off, a cancelled `CompanionEggOpenEvent`) fires nothing at all.

`charged` tells a purchase from a gift, and since 1.5.0 it is really used: it is `true` when the
open came from a paid button and `false` for `/companions admin openegg`, which skips the creative
gate, the egg's cooldown and the price entirely. On `CompanionEggOpenEvent` it says what the open
IS ABOUT to be, because the event fires **before** the charge - cancelling it costs the player
nothing. On `CompanionEggRewardEvent` it says what the open WAS.

A paid open that could not write its companions is refunded in full and fires no reward event, so
a listener never sees a purchase that produced nothing.
{% endhint %}

{% hint style="info" %}
Since 1.7.0 a stored companion can leave the storage as a physical item and be redeemed back into
somebody's storage - since 1.17.0 only by the player who took it out, unless the item predates
that version. Neither half fires an event in this version, and `API_VERSION` stays `1.0.0`
because nothing on the surface changed, the 1.17.0 owner lock included: it is a refusal inside
the redemption, not a new type or method. A companion that arrives by redemption is an ordinary new row,
so every query on the facade sees it immediately; what you cannot do yet is veto or observe the
extraction and the redemption themselves. `CompanionExtractEvent` and `CompanionRedeemEvent` are candidates
for a later version, and adding them would be a MINOR `API_VERSION` bump, never a breaking change.
{% endhint %}

Listen like any Bukkit event:

```java
@EventHandler
public void on(CompanionGroupDeleteEvent event) {
    if (isProtected(event.getPlayer())) {
        event.setCancelled(true);
    }
}
```

Event payloads are read-only. `setCancelled` is the only setter any event carries.

{% hint style="info" %}
A failed fusion is a normal outcome, not an error. It destroys both parents and still charges the
price, so `CompanionFusedEvent` reports `FAILED` with no produced companion.
{% endhint %}

## Query service

Every method is synchronous and reads in-memory state, so all of them are safe to call on the main
thread. None of them touches the database.

| Method | Returns | Notes |
|--------|---------|-------|
| `getStorage(UUID)` | `Optional<CompanionStorageView>` | Counts and capacities. Empty exactly when the player is not loaded |
| `getCompanion(UUID, int)` | `Optional<CompanionView>` | One owned companion by its instance id |
| `getEquippedCompanions(UUID)` | `List<CompanionView>` | The player's equipped companions, in slot order |
| `getStoredCompanions(UUID)` | `List<CompanionView>` | The player's stored companions |
| `getBuffTotal(UUID, String)` | `double` | Total percentage for `DAMAGE`, `RESISTANCE` or `SPEED` |
| `getCompanionTypes()` | `List<CompanionTypeView>` | Every companion definition the server declares |
| `getCompanionType(String)` | `Optional<CompanionTypeView>` | One companion definition by id |
| `getApiVersion()` | `String` | The API contract version |

{% hint style="warning" %}
SnCompanions loads a player's collection on join and saves it on quit, so these queries only answer for a
player who is currently loaded. For anyone else they return an empty list or an empty `Optional`.
Use `getStorage` to tell "not loaded" apart from "owns nothing".
{% endhint %}

## Views

Views live in `com.sn.companions.api.model` and are immutable snapshots taken when you call the method.
They never change afterwards, so re-query when you need current values.

| View | Describes |
|------|-----------|
| `CompanionView` | One owned companion: level, experience, equip slot |
| `CompanionTypeView` | One companion definition: display name, level cap, group, buff, fusion target |
| `CompanionStorageView` | One player's counts and capacities |

Companions cross the API as their id `String`, and internal enums cross as their
name. Renaming or reordering a config entry can never break your plugin at runtime.

{% hint style="info" %}
An unequipped companion reports `equippedSlot()` as `0`. Slots are numbered from 1, so `equipped()` is the
clearest way to ask.
{% endhint %}

## Versioning

Call `getApiVersion()` for the API contract version. It is independent of the plugin version.

{% hint style="warning" %}
**The contract is still provisional and surface may be REMOVED.** SnCompanions is a new plugin whose
feature set is still being cut down, and removals are made outright rather than through a deprecation
cycle. `1.2.0` deleted `getTraits()`, the `TraitView` record, `CompanionView`'s trait component and
`CompanionRollEvent.TRAIT` / `TRAIT_TICKET` when the traits feature was removed. `1.3.0` deleted
`getBoostGrades()`, `getCurrencyBalance()`, the `BoostGradeView` and `RollChangeView` records, the
`CompanionRollEvent` and `CompanionRolledEvent` events, and `CompanionView`'s three boost id
components when the boosts, the roll infrastructure and the internal currencies were removed.
`1.4.0` deleted the `CompanionBoxOpenEvent` and `CompanionBoxRewardEvent` events when companion
boxes were replaced by companion eggs, and added `CompanionEggOpenEvent` and
`CompanionEggRewardEvent` in their place. `1.5.0` removed nothing and added nothing: eggs became
payable, so `isCharged()` can now be `true` on both of those events, but no type, method or
signature changed.

`API_VERSION` is deliberately held at `1.0.0` across those removals rather than claiming a stability
this contract does not have yet, so **the version does not tell you whether surface went away**.
Compile against the version of the jar you ship with, and re-check this page when you upgrade.
{% endhint %}

Once the feature set settles, the ordinary rule takes over: additions bump the minor component, and
nothing public is removed or changed after that; deprecated members keep working.
