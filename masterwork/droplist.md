---
title: Droplist
mod: masterwork
---

<div class="eyebrow">Masterwork &rsaquo; Droplist</div>

# Droplist
{: .no_toc }

Grade is not only rolled at the crafting bench. Gear the world hands a player &mdash; a mob's drop, a chest's contents &mdash; is graded too, on its own odds table, tuned separately from crafting. This page is for a server owner shaping what that gear looks like: how the roll works, and how to hand author a guaranteed drop.

1. TOC
{:toc}

## How a found piece gets its grade

**A table's odds are rewritten at boot.** Every drop table shipped with the game, or with any installed pack, is scanned once at startup. Each entry that names a piece of gear is turned into a weighted choice across that item's Grade variants, using the `lootGrades` odds table (see [Configuration &rsaquo; lootGrades](configuration.html)). This is the only layer that can put an already graded item on the ground: a graded item's drop beam and model belong to its variant, so the tier has to be decided at the moment the entity spawns, not later when a player picks it up. A high tier lying in the grass glows and looks the part from the instant it appears.

**Anything the boot pass could not reach is caught when it is reached.** A table another mod injects after Masterwork's own boot hook, an item handed straight to an inventory, a `/give`, loot from before this feature shipped &mdash; none of that runs through the rewrite above. Masterwork's item resync sweep covers it instead: the same pass that keeps a stamped item's tooltip current also grades an unrolled piece of gear the first time it is reached, on join, on an inventory change, or when a container holding it is opened. A piece this layer catches is neutral until then; it will not glow on the ground, but it is graded the moment somebody touches it.

The two layers never double roll. Once a piece carries a breadcrumb saying it has been graded &mdash; whatever tier it came out at, the base tier included &mdash; neither layer touches it again.

**A found piece is not a forging.** It earns no XP, no ranking points, and no crafter stamp, and it does not go into a crafter's tally. The Masterwork tier itself ships at 0% in the loot odds table by default: it is a title a crafter earns, not one a player finds lying around. Both are ordinary defaults, editable like any other row.

## Pinning a guaranteed Grade

For a hand authored reward &mdash; a quest chest, a boss's one drop &mdash; the odds table is the wrong tool. A drop table entry can carry an exact Grade directly, bypassing the roll entirely. Two engine facts make this possible with nothing but JSON:

- **An item id in a drop entry is never checked against the item catalog at load.** A table can name an id that does not exist yet when the pack loads, including a Grade variant id Masterwork itself mints at boot. The check happens at drop time instead; an id that fails to resolve simply yields nothing, and the rest of the table still drops normally.
- **A drop entry can carry per instance metadata**, handed to the item verbatim the moment it is created.

Putting those together, a drop entry gets one of two treatments:

**Pin the breadcrumb on the base item** (recommended): add `"Masterwork.Grade"` to the entry's `Metadata`, naming the tier, on the item's ordinary base id. Masterwork's own boot rewrite performs the same swap on this entry that it performs everywhere else, so the pinned tier glows on the ground exactly like a rolled one. If loot grading is off, or the pin is added to a table registered after boot, the item resync sweep still honors it the first time the piece is reached. This form degrades gracefully: if the pinned tier is later disabled, or the item blacklisted, the entry simply drops the plain base item instead of nothing.

**Or name the variant id directly**, skipping `Metadata` entirely. The outcome is identical while the tier stays minted, but it does not degrade the same way: if that tier is disabled or the item is blacklisted, the id resolves to nothing and the entry drops nothing at all.

A pin naming a bad tier is safe either way &mdash; a name no Grade recognizes is read as no pin at all, and the piece is simply rolled normally. The tier's name is matched case-insensitively, so `masterwork`, `Masterwork` and `MASTERWORK` all pin the same tier.

### The two forms side by side

The clearest way to see that the two treatments are equivalent is on a single item, with no table involved. Both of these hand the caller the same Masterwork tier Mithril sword:

```
/give Weapon_Sword_Mithril_Graded_Masterwork
/give Weapon_Sword_Mithril --metadata="{'Masterwork.Grade':'MASTERWORK'}"
```

The first names the variant id directly; the second hands the base id and lets the metadata pin it. They produce the same stack today, but only the second survives the tier being disabled or the item being blacklisted later &mdash; it falls back to a plain `Weapon_Sword_Mithril` instead of resolving to nothing.

The quoting on the second line matters: the whole JSON document has to sit inside one pair of double quotes so the chat command reads it as a single argument, which means the document's own strings need a different quote character. Single quotes work for that and are what the example above uses; a `"Masterwork.Grade":"MASTERWORK"` written with double quotes throughout breaks on the outer pair before it ever reaches the parser.

### In a drop table

The same metadata pin, used in an actual drop table rather than a single `/give`. A `Choice` between two guaranteed rewards, one Masterwork sword and one Exquisite sword, each pinned by metadata on its base id:

```json
{
  "Container": {
    "Type": "Choice",
    "Containers": [
      {
        "Type": "Single",
        "Weight": 50,
        "Item": {
          "ItemId": "Weapon_Sword_Mithril",
          "Metadata": { "Masterwork.Grade": "MASTERWORK" },
          "QuantityMin": 1,
          "QuantityMax": 1
        }
      },
      {
        "Type": "Single",
        "Weight": 50,
        "Item": {
          "ItemId": "Weapon_Sword_Iron",
          "Metadata": { "Masterwork.Grade": "EXQUISITE" },
          "QuantityMin": 1,
          "QuantityMax": 1
        }
      }
    ]
  }
}
```

Masterwork ships exactly this table for its own testing, under the id `Masterwork_Test_Grade_Metadata` &mdash; try `/droplist Masterwork_Test_Grade_Metadata --count=10` on a running server to see both pins resolve to their tiers.

Pinning the **base** tier is the one case that would otherwise be impossible: an ordinary base item with no breadcrumb at all is exactly what the resync sweep treats as unrolled and grades on its own. Naming the base tier explicitly, `"Masterwork.Grade": "COMMON"`, is what guarantees a plain, ungraded drop instead of leaving it to the table's normal odds.

## Testing without killing anything

Two commands the game itself ships cover testing end to end, neither needing a mob, a chest, or anything to actually spawn:

- **`/droplist <id> [--count=N]`** rolls a table the given number of times (default 1) and prints what it would have handed out, merged into one list. Nothing is spawned and no inventory is touched, which makes it the right tool for checking that a table loaded and that a pin resolves to the tier intended. The `<id>` is the table's bare file name, not its path: `Server/Drops/Masterwork/Masterwork_Test_Grade_Metadata.json` answers to `Masterwork_Test_Grade_Metadata`, not the folder in front of it. `--count` is a flag, not a bare number.
- **`/give <item> [quantity] [durability] [metadata]`** hands a player a stack directly, with `metadata` (flag form, `--metadata=...`) accepting the same JSON a drop entry's `Metadata` field would carry &mdash; the fastest way to test a pin, with no table involved at all. See [The two forms side by side](#the-two-forms-side-by-side) above.

A table still has to actually be reachable in the world to matter to a player: it needs a mob's role, a `Droplist` reference from another table, or a chest fill naming it. The cheapest way to test a real path once a table is right is to override an existing table by its own id, rather than write a new role from scratch.
