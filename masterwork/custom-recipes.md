---
title: Using graded gear in custom recipes
mod: masterwork
---

<div class="eyebrow">Masterwork &rsaquo; Custom Recipes</div>

# Using graded gear in custom recipes
{: .no_toc }

A recipe in your own pack, or in another mod, can ask for a piece of gear a player crafted: an iron dagger of a given Grade, any Exquisite weapon, a Blacksmith Hammer. This page is for whoever writes that recipe: which input to use, how to spell it, and what the crafting bench shows the player.

1. TOC
{:toc}

## Ask by resource type, not by item id

Every piece Masterwork grades carries the crafter's name, and a recipe input written as `ItemId` only takes a copy with no data on it at all. A signed piece matches at the bench and is then never taken. So a recipe that wants crafted gear names a **resource type** instead, with `ResourceTypeId`:

```json
"Input": [
  { "ItemId": "Potion_Empty", "Quantity": 1 },
  { "ResourceTypeId": "Masterwork_Weapon_Daggers_Iron_Only_Masterwork", "Quantity": 1 }
]
```

Every resource type on this page belongs to Masterwork and is created when the server starts. Your pack does not declare any of them, and it does not matter whether it loads before or after Masterwork: a recipe naming one loads either way, and Masterwork reads every recipe at startup to create the ones they name.

A recipe that still names a piece of gear by `ItemId` is not ignored either: see [Recipes that name gear by item id](#recipes-that-name-gear-by-item-id).

## One item, a range of Grades

To ask for one specific item, write its base id (the vanilla id, never a graded variant's id) followed by the range you want:

| Resource type | Takes |
|---|---|
| `Masterwork_<item>_AtMost_<Grade>` | the plain item and every copy up to that Grade |
| `Masterwork_<item>_AtLeast_<Grade>` | that Grade and every one above it |
| `Masterwork_<item>_Only_<Grade>` | that Grade alone |
| `Masterwork_<item>_Between_<Low>_<High>` | both Grades and every one between them, **lower Grade first** |

The Grade compared is the one the piece shows in game, its border color.

The bench lists the ingredient with the item's own icon and a label in the player's language:

| Resource type | Label at the bench |
|---|---|
| `Masterwork_Weapon_Daggers_Iron_AtMost_WellCrafted` | Iron Daggers (Well-Crafted or lower) |
| `Masterwork_Weapon_Daggers_Iron_AtLeast_Superior` | Iron Daggers (Superior or higher) |
| `Masterwork_Weapon_Daggers_Iron_Only_Masterwork` | Iron Daggers (Masterwork only) |
| `Masterwork_Weapon_Daggers_Iron_Between_Adequate_Superior` | Iron Daggers (Adequate to Superior) |

`AtLeast_Masterwork` and `Only_Masterwork` take the same pieces, since nothing ranks above Masterwork. Use whichever reads better to your players; each keeps its own label.

## Spelling a Grade

Each Grade is written as one word, in this exact case, with no underscore:

| Grade | In a resource type |
|---|---|
| Masterwork | `Masterwork` |
| Exquisite | `Exquisite` |
| Superior | `Superior` |
| Well-Crafted | `WellCrafted` |
| Adequate | `Adequate` |
| Common | `Common` |
| Poor | `Poor` |

So `Between_WellCrafted_Exquisite` works, while `Between_Well_Crafted_Exquisite`, `Between_WELL_CRAFTED_Exquisite` and `Between_Exquisite_WellCrafted` (higher Grade first) do not.

A resource type Masterwork cannot read is never created. The server log names it at startup:

```
recipe inputs -- no gradeable item behind [Masterwork_Ingredient_Life_Essence_AtLeast_Superior], those recipes cannot be crafted
```

That line also appears when the item is not gear Masterwork grades, or when the server owner has blacklisted it. Such a recipe still loads, but the bench draws no ingredient for it and it can never be crafted.

## Any piece of a Grade

To ask for any piece of a Grade rather than one item, name the Grade alone, or the Grade within one kind of gear:

| Resource type | Takes |
|---|---|
| `Masterwork_Graded_<Grade>` | any graded piece of that Grade |
| `Masterwork_Weapon_Graded_<Grade>` | any weapon of that Grade |
| `Masterwork_Armor_Graded_<Grade>` | any armor piece of that Grade |
| `Masterwork_Tool_Graded_<Grade>` | any tool of that Grade |
| `Masterwork_Shield_Graded_<Grade>` | any shield of that Grade |

```json
{ "ResourceTypeId": "Masterwork_Weapon_Graded_Exquisite", "Quantity": 2 }
```

Two Exquisite weapons of any kind, whoever crafted them.

## Blacksmith Hammers

Each hammer tier has its own resource type, `Masterwork_Weapon_Blacksmith_Hammer_T1` through `_T4`, the top tier included. A hammer is signed like any crafted piece, so it is consumed this way and never by `ItemId`.

## Recipes that name gear by item id

A recipe written against a vanilla id, such as a mod's bow that consumes `Weapon_Shortbow_Cobalt`, would otherwise never take a crafted bow. Masterwork rewrites such an input at startup to `Masterwork_<item>_AtMost_<cap>`, where the cap is the server's `recipeInputMaxGrade` (Well-Crafted by default), set in Settings &rsaquo; General or in the [configuration file](configuration.html). A better copy is refused and stays in the inventory, so a fine piece is not spent as a plain ingredient. Changing the cap takes a server restart.

A recipe that already names one of the ranges above is never rewritten: it keeps its own range whatever the cap. Two kinds of recipe are left exactly as written: a recipe whose output is the same item it consumes (a repair, a refill), and a recipe naming an item the server owner has blacklisted.

## Examples

A potion that costs a dagger of middling quality, so nobody burns a good one by accident:

```json
"Recipe": {
  "Input": [
    { "ItemId": "Potion_Empty", "Quantity": 1 },
    { "ResourceTypeId": "Masterwork_Weapon_Daggers_Iron_Between_Adequate_WellCrafted", "Quantity": 1 }
  ],
  "BenchRequirement": [
    { "Id": "Alchemybench", "Type": "Crafting", "Categories": ["Alchemy_Potions_Misc"] }
  ],
  "TimeSeconds": 1
}
```

An upgrade that only a Masterwork bow can go into:

```json
"Input": [
  { "ResourceTypeId": "Masterwork_Weapon_Shortbow_Cobalt_Only_Masterwork", "Quantity": 1 },
  { "ItemId": "Ingredient_Bar_Cobalt", "Quantity": 4 }
]
```

A trophy that takes one weapon of each of the top three Grades, of any kind:

```json
"Input": [
  { "ResourceTypeId": "Masterwork_Weapon_Graded_Superior", "Quantity": 1 },
  { "ResourceTypeId": "Masterwork_Weapon_Graded_Exquisite", "Quantity": 1 },
  { "ResourceTypeId": "Masterwork_Weapon_Graded_Masterwork", "Quantity": 1 }
]
```

The any-piece types name one Grade each; there is no "any weapon, Superior or higher". For a range, name one item: `Masterwork_<item>_AtLeast_Superior` with `"Quantity": 3` takes three copies of that item, each Superior or better.

## Things to know

- **Which copy is spent.** When a player carries several pieces that fit, the bench takes the first one it finds: backpack first, then storage, then the hotbar, then nearby chests. A player who wants to keep one of two fitting pieces puts the one to spend earlier in that order.
- **A pack added while the server runs.** Ranges are created at startup. A pack registered later can use any range some recipe already asked for at startup; a range only that pack names is created on the next restart.
- **An item's own resource types.** Everything on this page goes in a recipe's `Input`. Never list a `Masterwork_*` resource type in an item's own `ResourceTypes`: those do not exist yet when your pack loads, and the pack fails to load.
