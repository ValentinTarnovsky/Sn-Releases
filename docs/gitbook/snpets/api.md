# Developer API

SnPets exposes a public developer API for other plugins: custom Bukkit events and a read-only
query service. The API lives in the `com.sn.pets.api` package inside the plugin jar. There is no
separate artifact.

{% hint style="warning" %}
Only `com.sn.pets.api` is a stable, kept contract. Everything else in the jar is obfuscated and
internal. Do not call it.
{% endhint %}

## Getting the jar

1. Download the latest release jar.
2. Install it into your local Maven repository:

   ```
   mvn install:install-file -Dfile=SnPets-<version>.jar -DgroupId=com.sn \
     -DartifactId=SnPets -Dversion=<version> -Dpackaging=jar
   ```

3. Depend on it with `provided` scope. Never shade it.

## Quick start

Declare the dependency in your `plugin.yml`:

```yaml
depend: [SnPets]      # or softdepend if optional
```

Resolve the API when you need it:

```java
SnPetsAPI api = SnPetsProvider.get();
if (api != null) {
    api.getEquippedPets(player.getUniqueId())
       .forEach(pet -> getLogger().info(pet.petId() + " lvl " + pet.level()));
}
```

{% hint style="info" %}
Resolve the reference when you need it. Do not cache it across a SnPets reload.
{% endhint %}

## Master switch

API events can be disabled by the server owner with `api-events.enabled: false` in `config.yml`.
Cancellable hooks then report "not cancelled" so gameplay proceeds. The query service stays
available either way.

## Events

All events are in `com.sn.pets.api.event` and fire on the main thread.

Cancellable events fire before the action. Cancelling aborts it.

| Event | Fired when | Cancel effect |
|-------|-----------|---------------|
| `PetEquipEvent` | A player equips a stored pet from the pet menu | The pet stays in storage |
| `PetUnequipEvent` | A player sends an equipped pet back to storage | The pet keeps its slot and its buffs |
| `PetBoxOpenEvent` | A player opens one or more pet boxes, before the roll | Every box is handed back |
| `PetFuseEvent` | A player commits a fusion, before anything is charged | Nothing is charged and no pet is destroyed |
| `PetGroupDeleteEvent` | A player mass-deletes a group from the bulk delete menu | Every pet in the group stays |
| `PetRollEvent` | A player rolls a trait or a boost grade, before paying | No currency is spent and no modifier is written |

{% hint style="warning" %}
These fire only for player-driven actions. The `/pets admin` equivalents bypass them on purpose,
so an operator override is never vetoable.
{% endhint %}

Notification events fire after the fact. They cannot be cancelled.

| Event | Fired when | Payload |
|-------|-----------|---------|
| `PetBoxRewardEvent` | A box open resolved to a set of pets | Which pets were won, and how many boxes opened |
| `PetLevelUpEvent` | An equipped pet gained at least one level | The pet after the level up, and how many levels it gained |
| `PetFusedEvent` | A fusion consumed its parents | Whether it succeeded, and the pet it produced |
| `PetRolledEvent` | A trait or boost roll was decided and written | One entry per rewritten slot, with the old and new id |

{% hint style="info" %}
Since 1.4.0 a pet box carries a success chance and can fail to open, consuming the box and
granting nothing. A failed open fires **no** event: `PetBoxRewardEvent` only ever fires when
there was a reward, and `PetBoxOpenEvent` has already fired by then (it runs before the roll,
so it sees the attempt whether or not it succeeds). A dedicated fail event may be added in a
later version; adding one would be a MINOR `API_VERSION` bump, never a breaking change.
{% endhint %}

{% hint style="info" %}
Since 1.7.0 a stored pet can leave the storage as a physical item and be redeemed back into
somebody's storage - since 1.17.0 only by the player who took it out, unless the item predates
that version. Neither half fires an event in this version, and `API_VERSION` stays `1.0.0`
because nothing on the surface changed, the 1.17.0 owner lock included: it is a refusal inside
the redemption, not a new type or method. A pet that arrives by redemption is an ordinary new row,
so every query on the facade sees it immediately; what you cannot do yet is veto or observe the
extraction and the redemption themselves. `PetExtractEvent` and `PetRedeemEvent` are candidates
for a later version, and adding them would be a MINOR `API_VERSION` bump, never a breaking change.
{% endhint %}

Listen like any Bukkit event:

```java
@EventHandler
public void on(PetGroupDeleteEvent event) {
    if (isProtected(event.getPlayer())) {
        event.setCancelled(true);
    }
}
```

Event payloads are read-only. `setCancelled` is the only setter any event carries.

{% hint style="info" %}
A failed fusion is a normal outcome, not an error. It destroys both parents and still charges the
price, so `PetFusedEvent` reports `FAILED` with no produced pet.
{% endhint %}

## Query service

Every method is synchronous and reads in-memory state, so all of them are safe to call on the main
thread. None of them touches the database.

| Method | Returns | Notes |
|--------|---------|-------|
| `getStorage(UUID)` | `Optional<PetStorageView>` | Counts and capacities. Empty exactly when the player is not loaded |
| `getPet(UUID, int)` | `Optional<PetView>` | One owned pet by its instance id |
| `getEquippedPets(UUID)` | `List<PetView>` | The player's equipped pets, in slot order |
| `getStoredPets(UUID)` | `List<PetView>` | The player's stored pets |
| `getCurrencyBalance(UUID, String)` | `int` | Balance of `TRAIT_TICKET`, `DICE_NORMAL` or `DICE_SPECIAL` |
| `getBuffTotal(UUID, String)` | `double` | Total percentage for `DAMAGE`, `RESISTANCE` or `SPEED` |
| `getPetTypes()` | `List<PetTypeView>` | Every pet definition the server declares |
| `getPetType(String)` | `Optional<PetTypeView>` | One pet definition by id |
| `getTraits()` | `List<TraitView>` | Every trait in the global table |
| `getBoostGrades()` | `List<BoostGradeView>` | Every grade in the global ladder |
| `getApiVersion()` | `String` | The API contract version |

{% hint style="warning" %}
SnPets loads a player's collection on join and saves it on quit, so these queries only answer for a
player who is currently loaded. For anyone else they return an empty list or an empty `Optional`.
Use `getStorage` to tell "not loaded" apart from "owns nothing".
{% endhint %}

## Views

Views live in `com.sn.pets.api.model` and are immutable snapshots taken when you call the method.
They never change afterwards, so re-query when you need current values.

| View | Describes |
|------|-----------|
| `PetView` | One owned pet: level, experience, trait, the three boost slots, equip slot |
| `PetTypeView` | One pet definition: display name, level cap, group, buff, fusion target |
| `PetStorageView` | One player's counts and capacities |
| `TraitView` | One trait: weight, table share, and what it modifies |
| `BoostGradeView` | One boost grade: weight, percentage range, and the slots it fits |
| `RollChangeView` | One rewritten modifier slot: the old id and the new one |

Pets, traits and boost grades cross the API as their id `String`, and internal enums cross as their
name. Renaming or reordering a config entry can never break your plugin at runtime.

{% hint style="info" %}
An unequipped pet reports `equippedSlot()` as `0`. Slots are numbered from 1, so `equipped()` is the
clearest way to ask.
{% endhint %}

## Versioning

Call `getApiVersion()` for the API contract version. It is independent of the plugin version.
Additions bump the minor component. Existing members are never removed or changed; deprecated
members keep working.
