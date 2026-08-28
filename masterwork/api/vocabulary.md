---
title: Vocabulary
mod: masterwork
---

<div class="eyebrow">Masterwork &rsaquo; Public API &rsaquo; Vocabulary</div>

# Vocabulary
{: .no_toc }

`Grade`, `GearCategory` and `MasterworkMetadata` are the shared word list every other part of the API speaks in: what tier a piece came out at, what kind of gear it is, and what an item's own instance metadata keys mean. All three live at the root of the `api` package, since every module uses them and none of them belongs to any one module.

1. TOC
{:toc}

## `Grade`

The seven tier craftsmanship ladder, from `POOR` up through `COMMON` (the base tier), `ADEQUATE`, `WELL_CRAFTED`, `SUPERIOR`, `EXQUISITE`, to `MASTERWORK`.

| Method | Returns |
|---|---|
| `qualityId()` | The custom `ItemQuality` id this tier stamps on its items (`MW_Masterwork`, and so on). |
| `token()` | The PascalCase token used in variant item ids, e.g. `WellCrafted`. |
| `templateQualityId()` | The vanilla quality whose art this tier's custom quality clones. |
| `qualityValue()` | Sort value for the quality; higher means a better tier. Useful for a "this tier or above" comparison. |
| `textColor()` | The tier's hex text color. |
| `dropParticleSystemId()` | The particle system drawn as a beam over a dropped item of this tier. |
| `labelKey()` | The translation key for this tier's display name. |
| `rollWeight()` | This tier's default relative weight in the no progression odds table. |
| `isBase()` | Whether this is the base tier (`COMMON`): no variant minted, only the quality normalized. |

`Grade.base()` returns the base tier directly, reading it from `isBase()` rather than hardcoding `COMMON`, so re-basing the ladder in a future version moves every caller with it. `Grade` values are stored in per-instance metadata as the enum's name (`Grade.name()`), never its display token, so they survive a tier being relabeled.

`Grade` is public API: new constants may be added in a future version, so never write an exhaustive `switch` over `Grade.values()` without a default branch.

## `GearCategory`

The broad kind of a piece of gear: `WEAPON`, `ARMOR`, `TOOL`, `SHIELD`, or `UNKNOWN` for anything not recognized as gradeable gear.

This enum is vocabulary, not classification. It says what a category is, what it is called, and which progression track it feeds; it does not itself decide what category an item belongs to. That inference lives in Masterwork's own `GearClassifier` internally, reached indirectly through registration (see [Gear Registration](gear.html)).

`SHIELD` is a stat and classification split only: it gets its own stat scaler and its own label, but a shield's crafts feed the `WEAPON` progression track. `GearCategory.progressionTracks()` returns the categories that carry their own XP total and level (`WEAPON`, `ARMOR`, `TOOL`), which is what to iterate when listing a crafter's progression, so `SHIELD` folding into `WEAPON` does not show up as a duplicate row. `progressionCategory()` on an instance returns the track that category's crafts count toward, itself for everything except `SHIELD`.

`labelKey()` gives the translation key for the category's crafter facing discipline name ("Weaponsmithing," and so on). Like `Grade`, this enum is public API and may gain constants in the future: never write an exhaustive `switch` over `GearCategory.values()` without a default.

## `MasterworkMetadata`

The per instance metadata keys Masterwork writes onto an item, and what a third party mod may do with each. Every mod that decorates an item instance writes into the same metadata document, keyed by string, and a collision is silent data loss, so these keys are gathered in one place rather than left to be discovered by reading Masterwork's source. All of them are namespaced `Masterwork.*`.

| Constant | Key | Access |
|---|---|---|
| `GRADE` | `Masterwork.Grade` | Readable and writable. The tier a piece was rolled or assigned, as the enum name. Writing it on an item a mod is creating (a drop table entry, a `/give`, a reward) is read as a statement of intent and honored: a drop table entry carrying it pins that exact tier. |
| `CRAFTER` | `Masterwork.Crafter` | Read only. The crafter's name, as shown in the item's description. |
| `OWNER` | `Masterwork.Owner` | Read only. The crafter's UUID, half of the identity pair the hammer's ownership rules read. |
| `BAKED` | `Masterwork.Baked` | Internal. A fingerprint of the description Masterwork last composed, used by the resync sweep to tell "still ours" from "a sibling mod has written here since." |
| `SIGNATURE_COLOR` | `Masterwork.SignatureColor` | Internal. The palette id of the color the signature was composed in. |
| `CONFIG_REVISION` | `Masterwork.ConfigRevision` | Internal. The settings revision the description was composed at. |

Only `GRADE` is meant to be written by a third party, and only on an item that mod is creating itself. Everything else is bookkeeping about a write only Masterwork can make correctly; setting one of those keys by hand produces an item that reads as freshly baked when it is not, which the resync sweep then declines to repair.

`MasterworkMetadata.isOurs(key)` reports whether a metadata key belongs to Masterwork, for a mod sweeping or copying an item's metadata that wants to make sure Masterwork's keys travel with the instance rather than being dropped.
