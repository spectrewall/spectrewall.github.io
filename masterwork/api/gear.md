---
title: Gear Registration
mod: masterwork
---

<div class="eyebrow">Masterwork &rsaquo; Public API &rsaquo; Gear Registration</div>

# Gear Registration
{: .no_toc }

`GearRegistry` tells Masterwork what to do with a mod's own items; `VariantRegistry` (with `VariantTransformer` and `SalvageTransformer`) edits every graded item and salvage recipe Masterwork mints, in bulk. Reached through `MasterworkApi.gear()` and `MasterworkApi.variants()`.

1. TOC
{:toc}

## Why this exists

Masterwork decides what is gradeable by reading an item's own stat blocks: a weapon has a `Weapon` block, armor an `Armor` block, and so on. That inference is right for everything the base game ships, but it is still an inference, and a modded item built in an unusual shape can be missed, mis-typed, or graded when its author never wanted it to be. `GearRegistry` is the recourse.

## `GearRegistry`

```java
Masterwork.getInstance().getGearRegistry()
    .register("YourMod", "Weapon_Spear_Bronze", GearCategory.WEAPON);
```

| Method | Effect |
|---|---|
| `register(source, itemId, category)` | Have `itemId` graded as `category`, overriding whatever Masterwork would have inferred. Opts the item in even if Masterwork would not otherwise have recognized it as gear. `GearCategory.UNKNOWN` is refused here (with a warning): it is the "not gear" answer, not a category to register. |
| `register(source, Collection<String>, category)` | The same, for a batch of ids. |
| `exclude(source, itemId)` | Keep `itemId` out of Masterwork entirely: no variants minted, no grade rolled at craft, no XP, native rarity border kept. Wins over a registration: an item both registered and excluded is excluded. |
| `exclude(source, Collection<String>)` | The same, for a batch. |
| `isExcluded(itemId)` / `registeredCategory(itemId)` / `sourceOf(itemId)` / `sources()` | Read back what has been registered or excluded, and by whom. |

`source` is the registering mod's own name, used in the boot log and to answer "who registered this item?" later, on a server that might be running dozens of mods.

One hard limit: a **stackable** item (`MaxStack > 1`) can never be graded, registered or not. A grade and a crafter stamp live in per-instance metadata, and stacking would merge that metadata away.

The server owner's own blacklist in `masterwork-config.json` is evaluated separately from registration and always wins: blacklisting a registered item degrades it to the plain base exactly as it would any vanilla item.

Register from the depending plugin's own `setup()`, or from a `LoadAssetEvent` listener at a priority below `PRIORITY_LOAD_LATE` (`64`). The registry freezes the moment the boot injection mints the graded variants; a call made after that point is refused with a logged warning, since the variants for that item would already not exist.

## `VariantRegistry`, `VariantTransformer`, `SalvageTransformer`

`GearRegistry` says what an item *is*. `VariantRegistry` is the bulk counterpart: a hook that sees, and can rewrite, every graded item Masterwork mints, and every salvage recipe it mints alongside them, the way a mod changes gear in bulk instead of shipping a pack override per item.

```java
Masterwork.getInstance().getVariantRegistry()
    .register("YourMod", (context, variant) -> {
        Grade tier = GradeIds.gradeOf(context.variantId());

        if (tier.qualityValue() >= Grade.EXQUISITE.qualityValue()) {
            variant.put("YourMod.SalvageBonus", new BsonString("Ingredient_Forge_Essence"));
        }

        return variant;
    });
```

Masterwork builds a variant by encoding the resolved base item to a `BsonDocument`, patching it (quality, durability, the stat families), and decoding it back under the new id. A `VariantTransformer` is invoked at the last moment of that round trip: after Masterwork's own patches, before the decode. So it sees the finished document, scaled numbers included, and can change anything the item's JSON can express. `SalvageTransformer` is the same shape over the cloned salvage recipe Masterwork mints for each tier (input re-pointed at the variant id, outputs still the base's), called only for the tiers Masterwork mints a recipe for at all.

`VariantContext`, passed to both, describes what is being minted:

| Field | Meaning |
|---|---|
| `base` | The fully resolved source item the variant is built from. |
| `variantId` | The id this variant is registered under (equals `base.getId()` for the base tier itself). |
| `grade` | The tier the variant will render as, which is usually but not always the tier its id encodes: a tier disabled in config still mints under its own id while rendering as its nearest enabled neighbor. |
| `category` | The gear category Masterwork resolved for the base item. |
| `neutral` | Whether this variant is a blacklisted item's placeholder, minted only so its id keeps resolving for copies already in the world. A transformer should usually leave these alone. |
| `baseOverride` | Whether this is the in-place rewrite of the base item itself (the base tier is the base item), reaching every copy of the plain item in the world, crafted or not. |

### Contract both hooks share

- **Return the document to continue with.** Editing it in place and returning it is the normal shape; returning a different instance is equally fine.
- **Transformers chain in registration order**, each seeing the previous one's output, so two mods editing the same key resolve last writer wins, in plugin load order.
- **A throw is contained**, logged against the offending mod's name, and the variant continues from the document as it stood before that call. One bad transformer must not cost the server its other roughly 2,600 variants.
- **A `null` return is treated as no change**, and warned about once per source rather than per item.
- **Called a lot**: roughly gradeable items times seven (one call per tier). Filter on `VariantContext` first and return immediately when a variant is not yours to touch.
- **Whatever is written must survive a decode.** The document is handed to the engine's own item or recipe codec next; a malformed value fails that build, and the variant (or recipe) is skipped and counted in the boot log.

Called only for the graded variants, the base tier rewrite and the neutral placeholders included; never for the crafting hammers or the injected qualities, which are Masterwork's own assets rather than transformations of the game's.

Register during `setup()`, like `GearRegistry`: this registry freezes at the same boot injection point, for the same reason.
