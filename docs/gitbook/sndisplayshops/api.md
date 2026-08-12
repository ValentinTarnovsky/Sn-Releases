# Developer API

SnDisplayShops exposes a small read-and-drain API for other plugins: enough to see what a shop is
and what it holds, and to take stock out of it. It deliberately cannot create a shop, change its
price, mode, currency or item, add to its storage, or move money.

## Getting the API

```java
SnDisplayShopsAPI api = SnDisplayShopsPlugin.getInstance().getAPI();
```

Add `SnDisplayShops` to your `softdepend` (or `depend`) in `plugin.yml`, and null-check the result:
it is null until the plugin has finished enabling, and stays null if the licence gate refused.

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

The four methods above and the `Shop` and `StorageEntry` types they return are a frozen surface:
they will not be renamed, removed or have their signatures changed. Anything added later is added
alongside them.

{% hint style="warning" %}
`Shop` and `StorageEntry` carry public setters. They are for the plugin's own use and calling them
from outside changes the plugin's memory without writing to the database and without telling the
hologram or any open menu, so the change silently reverts on the next restart. Read them; do not
write them. `removeFromStorage` is the only supported way to change what a shop holds.
{% endhint %}
