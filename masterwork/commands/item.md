---
title: Item
mod: masterwork
---

<div class="eyebrow">Masterwork &rsaquo; Commands &rsaquo; Item</div>

# Item
{: .no_toc }

Test and correction hooks against the item stack in the caller's active hotbar slot. Both `item set crafter` leaves write an audit line at warning severity, naming the caller, the change, and the item id, since granting or changing a crafter stamp is not something a `debug` switch should be able to silence.

1. TOC
{:toc}

### `/mw item resync`

<p class="cmd-access">masterwork.admin</p>

Run the item resync sweep over the caller's own six containers right now (hotbar, storage, tool belt, utility belt, backpack, worn armor), instead of waiting for the sweep's own triggers.

### `/mw item set crafter <player>`

<p class="cmd-access">masterwork.item.crafter</p>

Stamp the held item with an online player's name and UUID, if it is not already signed. Re-signing an already signed item is refused, since a signature is written once.

### `/mw item set crafterByUuid <uuid>`

<p class="cmd-access">masterwork.item.crafter</p>

The same, for a player who is currently offline. Their name and signature color are read from their player file; a UUID with no player file is refused rather than stamped.

### `/mw item set grade <grade>`

<p class="cmd-access">masterwork.item.grade</p>

Swap the held item to the chosen tier's variant. Setting it back to the base tier reverts it.
