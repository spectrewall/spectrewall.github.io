---
title: Craft Events
mod: masterwork
---

<div class="eyebrow">Masterwork &rsaquo; Public API &rsaquo; Craft Events</div>

# Craft Events
{: .no_toc }

`CraftProcessedEvent` is Masterwork's main integration surface: it fires once per freshly crafted piece of gear, after Masterwork has resolved it, rolled the Grade, applied the variant swap, stamped the crafter and awarded whatever the craft was worth.

1. TOC
{:toc}

## Listening for it

A depending mod reacts by registering its own `EntityEventSystem<EntityStore, CraftProcessedEvent>`:

```java
public final class MyCraftListener extends EntityEventSystem<EntityStore, CraftProcessedEvent> {

    public MyCraftListener() {
        super(CraftProcessedEvent.class);
    }

    @Override
    public void handle(int index, ArchetypeChunk<EntityStore> chunk, Store<EntityStore> store,
            CommandBuffer<EntityStore> buffer, CraftProcessedEvent event) {
        if (event.outcome() == CraftProcessedEvent.Outcome.GRADED && event.grade() == Grade.MASTERWORK) {
            // reward the crafter
        }
    }
}
```

It is dispatched entity targeted at the crafter and delivered synchronously, so a listener already holds the crafter as its queried entity. `crafterUuid()` is on the payload anyway, since almost every listener wants it and resolving it from the entity reference is boilerplate.

## When it fires

For every craft that reaches Masterwork's pipeline, including the ones Masterwork deliberately leaves ungraded, read `outcome()` rather than assuming a grade was rolled. A craft that never reaches Masterwork at all, a non gear recipe such as a block, an ingredient or a potion, fires nothing.

A bulk craft that overflows the crafter's inventory is reported per unit: each unit that spills to the ground is rolled independently and gets its own event, with `destination()` set to `DROPPED`.

## `Outcome`

| Value | Meaning |
|---|---|
| `GRADED` | A grade was rolled: `grade()` is the tier, the crafter was tallied, and XP and ranking points were awarded by the rules currently in force. |
| `HAND_CRAFTED` | The piece was made in hand, from the inventory screen with no bench, while the field craft rule is on. Held at the base grade (no roll at all, deliberately not the same as a bad one), earning no XP, no tally and no ranking points. `grade()` is the base tier and `handCrafted()` is `true`. |
| `NOT_GRADED` | Masterwork left the craft alone: the item is blacklisted, excluded by a sibling mod, a crafting hammer, or an item that transforms into another id. The crafter stamp is still written, but `grade()` is `null` and nothing was awarded. |

## Payload

| Method | Returns |
|---|---|
| `crafterUuid()` / `crafterName()` | The crafter. |
| `baseItemId()` | The item id the recipe produced, before any grade swap. |
| `itemId()` | The id the crafter actually received: the variant id when a grade above the base was rolled, `baseItemId()` otherwise. |
| `craftedStack()` | The final, graded, stamped stack, read only. Mutating this reference does not change what the player received; `ItemStack` is copy on write, and the value was already written elsewhere. |
| `grade()` | The Grade this craft came out at, or `null` for `NOT_GRADED`. May be the base tier either as a genuine roll or because the piece was hand crafted; `handCrafted()` tells the two apart. |
| `category()` | The item's own gear category. Not always the track its XP went to: a shield reports `SHIELD` here while its XP levels `WEAPON`. Call `GearCategory.progressionCategory()`, or read `NativeXpResult.track()`, for the track. |
| `outcome()` | What Masterwork did with this craft. |
| `handCrafted()` | Shorthand for `outcome() == HAND_CRAFTED`. |
| `xp()` | The progression XP this craft awarded, `0` when it earned none. |
| `rankingPoints()` | The ranking points this craft scored, non-zero only on a `MASTERWORK` roll. |
| `itemLevel()` | The crafted item's `ItemLevel`, one of the two scoring inputs. |
| `craftTimeSeconds()` | How long the recipe took, the other scoring input. An instant recipe reports whatever `instantRecipeSeconds` gave it. |
| `destination()` | `INVENTORY` or `DROPPED` (an inventory that was full at the moment of crafting). |
| `nativeXp()` | See below. |

Both enums, `Outcome` and `Destination`, are public API and may gain constants in the future: never write an exhaustive `switch` over either without a default branch.

## `NativeXpResult`

What this one craft did to the crafter's standing on Masterwork's own progression ladder. Present only while the built in source is the one driving the roll; with progression off, or with another mod's system active, Masterwork awards no XP of its own to report, and this is `null` rather than a row of zeroes.

| Field | Meaning |
|---|---|
| `track` | The progression track the XP landed on. |
| `xpGained` | The XP this craft awarded. |
| `xpBefore` / `xpAfter` | The crafter's total on `track`, before and after. |
| `levelBefore` / `levelAfter` | The level each total buys. Equal unless this craft crossed a rung. |

`leveledUp()` reports whether `levelAfter > levelBefore`. The figures are exact for the craft they describe, including the second and third craft of the same server tick, since they come from the same per-tick ledger the XP toast is built from.
