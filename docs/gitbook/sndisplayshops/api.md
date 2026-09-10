# Developer API

SnDisplayShops exposes a small read-and-drain API for other plugins: enough to see what a shop is
and what it holds, and to take stock out of it. It deliberately cannot create a shop, change its
price, mode, currency or item, add to its storage, or move money.

It has two halves. The **facade** below answers questions and drains stock. The
[event](#events) tells you when a player clicks a shop, and lets you take that click for yourself.

## API version

The surface has its own semantic version, separate from the plugin's:

```java
String version = api.getApiVersion();   // "1.1.0" as of SnDisplayShops 2.9.0
```

Only the MINOR goes up, and only when something is added. Read it through `getApiVersion()` rather
than through the `SnDisplayShopsAPI.API_VERSION` constant: a constant is inlined into your own jar
at compile time, so it reports the version you built against, not the one that is installed.

| API version | Added |
|---|---|
| 1.0.0 | the four facade methods, which is everything the API was before 2.9.0 |
| 1.1.0 | `ShopMenuOpenEvent` |

`getApiVersion()` itself arrived with 1.1.0, so on any older build the call does not exist. If you
support those too, treat a `NoSuchMethodError` as "1.0.0".

## Getting the API

```java
SnDisplayShopsAPI api = SnDisplayShopsPlugin.getInstance().getAPI();
```

Add `SnDisplayShops` to your `softdepend` (or `depend`) in `plugin.yml`, and null-check the result:
it is null until the plugin has finished enabling, and stays null if the licence gate refused.

Events need none of that - they are ordinary Bukkit events, registered the ordinary way.

## Methods

```java
Shop getShopAt(Location loc);
List<StorageEntry> getStorageSnapshot(UUID shopId);
long getStoredAmount(UUID shopId, ItemStack template);
long removeFromStorage(UUID shopId, ItemStack template, long qty);
```

### `getShopAt(Location)`

The shop whose block stands at that location, or null. One shop per location.

### `getStorageSnapshot(UUID)`

Everything the shop holds, ordered by slot. The list and the rows in it are copies, so nothing you
do to what you get back reaches the shop, and the amounts are the amounts at the instant you asked.

### `getStoredAmount(UUID, ItemStack)`

How much of one item the shop holds, summed across every variant similar to your template. A read,
so it is already stale when it returns: it says what may be worth attempting.

### `removeFromStorage(UUID, ItemStack, long)`

Takes up to `qty` out of the shop and returns **how much was actually removed**, which may be less
than you asked for and may be 0. Serialised against the shop's other trades and written through to
the database.

{% hint style="danger" %}
Act on the return value, never on `qty`. A caller that pays out, credits or hands over the amount it
REQUESTED hands out items the shop did not have - a buyer trading at the same shop can drain a
variant between your snapshot and your removal, and this call is the only step that knows what was
really there.
{% endhint %}

{% hint style="info" %}
**Since 2.8.0 a successful removal is recorded**, as one `EXTERNAL` line in the shop's
[trade log](logs.md) with no actor on it - the shop's owner can otherwise only see that stock went
missing. Nothing about the call changed: same signature, same return meaning, same threading. The
recording is the plugin's own business and cannot fail your removal.
{% endhint %}

## Events

One event, in `com.sn.displayshops.api.event`.

| Event | Cancellable | Fired when |
|---|---|---|
| `ShopMenuOpenEvent` | yes | a player right-clicks a shop and a menu is about to open |

### `ShopMenuOpenEvent`

```java
Player       getPlayer();     // who clicked
Shop         getShop();       // the shop they clicked
ShopMenuType getMenuType();   // OWNER or BUYER
```

`getMenuType()` is decided from the clicker, not from the shop: `OWNER` for the shop's owner and for
staff with `sndisplayshops.admin.bypass` toggled on, `BUYER` for everybody else.

**Cancelling opens no menu.** It does nothing else. The right-click stays claimed by
SnDisplayShops - a shop built out of an enchanting table does not fall through to the vanilla
enchanting screen the moment you take the menu away - and the player is told nothing, because the
plugin has no way to know what you want them to see instead. Cancel it and the interaction is
yours.

```java
@EventHandler
public void onShopMenu(ShopMenuOpenEvent event) {
    // A sellwand: sell into the shop instead of showing its buyer menu.
    if (event.getMenuType() != ShopMenuType.BUYER) {
        return;   // never get between an owner and their own shop
    }
    ItemStack held = event.getPlayer().getInventory().getItemInMainHand();
    if (!isSellwand(held)) {
        return;
    }
    event.setCancelled(true);
    drainAndPay(event.getShop(), event.getPlayer());
}
```

{% hint style="info" %}
Always fired on the main thread, between the click and the menu, so you can touch the Bukkit API
directly - and should keep the handler short, because a player is waiting inside it.
{% endhint %}

{% hint style="warning" %}
A fire is not a promise a menu appeared. The event goes out as soon as the click resolves to a
shop, BEFORE the menu's own refusals: the buyer menu still turns away players without
`sndisplayshops.use`, and still refuses a paused shop or one whose currency or item is gone. The
other direction IS a promise - cancel, and no menu opens.

It fires only for a CLICK. Re-opening the owner menu after its owner types a price in chat is the
plugin talking to itself, and dispatches nothing.
{% endhint %}

The `Shop` you get is the plugin's live instance, with the same rule as everywhere else on this
page: read it, do not write it. And remember the operator's switch - `api-events.enabled: false` in
`config.yml` means this event never fires. The facade is not affected by it.

## Example

A sellwand that drains a shop and pays its owner:

```java
SnDisplayShopsAPI api = SnDisplayShopsPlugin.getInstance().getAPI();
if (api == null) {
    return;
}
Shop shop = api.getShopAt(block.getLocation());
if (shop == null) {
    return;
}
for (StorageEntry entry : api.getStorageSnapshot(shop.getId())) {
    ItemStack template = entry.getItem();
    if (template == null) {
        continue;
    }
    // Ask for what the snapshot showed...
    long removed = api.removeFromStorage(shop.getId(), template, entry.getAmount());
    // ...and pay for what actually came out.
    if (removed > 0L) {
        payOwner(shop.getOwnerUuid(), removed, template);
    }
}
```

## Stability

Everything on this page - the facade methods, the event, its payload, and the `Shop` and
`StorageEntry` types they hand back - is a frozen surface: nothing is renamed, removed or
re-signatured. Anything added later is added alongside it, and the
[API version](#api-version) is how you tell what an installed copy has.

{% hint style="warning" %}
`Shop` and `StorageEntry` carry public setters. They are for the plugin's own use and calling them
from outside changes the plugin's memory without writing to the database and without telling the
hologram or any open menu, so the change silently reverts on the next restart. Read them; do not
write them. `removeFromStorage` is the only supported way to change what a shop holds.
{% endhint %}
