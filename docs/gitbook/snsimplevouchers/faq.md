# FAQ

## The plugin does not enable and the console says `[Sn] License: FAIL`

Three causes produce that same line, on purpose:

1. The key in `plugins/.Sn-License/license.yml` is missing or still the placeholder.
2. The server cannot reach the licence backend. Check outbound HTTPS.
3. The jar was modified, or is not the one that was released.

Check the key first, then the network. That file is shared by every bundle plugin, so if another Sn plugin already enables, the key is fine and the answer is not there.

## The console says `Unknown dependency SnLib`

Put `SnLib.jar` in `plugins/` and restart. It is a hard dependency.

## A voucher does nothing when right-clicked

- Confirm it loaded: `/voucher reload` and read the console. A bad file skips itself with a warning naming the voucher.
- Confirm you are clicking with the **main hand**. Off-hand right-clicks claim nothing.
- If its file was deleted or disabled, the item stays in the inventory as an ordinary item and behaves normally when clicked. That is deliberate.
- If every reward carries a `condition:` you do not meet, nothing is dispatched, the voucher is **not** consumed, and you get `messages.claim.no-eligible-reward`.

## My voucher file will not load

Read the console. Every load failure names the voucher id. Common causes: no `item:` section, no `item.material`, an empty `commands:` list, or a material that cannot exist as an item such as `AIR`.

## Clicking one voucher consumed others I did not click

That is `bulk.aggregate-by-category`, which ships **on**. A bulk claim also consumes vouchers of other ids in the same category folder when they declare the exact same reward list, and pays the summed total in one command. No value is lost.

Set `bulk.aggregate-by-category: false` in `config.yml` to consume only the clicked voucher's own id.

## A stack of 64 vouchers only ran the command once

That is the point. A voucher with an `amount:` consolidates the stack into **one** execution carrying the summed `{amount}`, instead of 64 separate commands. Without an `amount:`, each right-click consumes exactly one.

## Can a player claim from the offhand, or from a shulker?

No. The claim is main-hand only, and the bulk scan covers the main inventory only, not the offhand or the cursor.

## My condition never passes

- Is PlaceholderAPI installed, with the expansion the placeholder belongs to? With `conditions.fail-closed: true` (the default), an unresolvable condition makes the reward ineligible.
- Turn on `/voucher debug` to see parse failures.
- Remember that `!` binds **looser** than the comparators: `!a == b` means `!(a == b)`.
- A boolean-style placeholder may render as `yes`/`no` rather than `true`/`false`, depending on your PlaceholderAPI settings. Compare explicitly if you are unsure: `"%some_placeholder% == yes"`.

## Does it register PlaceholderAPI placeholders?

No. This plugin **consumes** PlaceholderAPI inside `condition:` expressions and publishes none of its own.

## Can staff give vouchers without being able to reload?

Yes. Grant `snsimplevouchers.admin` and negate what you want withheld. Note the root node is required: a child node alone is not enough to reach the command.

## Someone with only `/voucher open` handed out vouchers

That was possible before v2.0.0, when the menu performed a give with no permission check. It is now gated on `snsimplevouchers.admin.give`, the same node the command requires.

## A message shows a literal `{amount}` or a token I did not expect

Every message gets the same five placeholders: `{voucher}` `{category}` `{amount}` `{count}` `{player}`. If one renders literally, the key is misspelled in your language file. To disable a message entirely, blank its value - a missing key renders a visible marker instead.

## Will updating reset my config?

No. New keys merge in while your values and comments are preserved.
