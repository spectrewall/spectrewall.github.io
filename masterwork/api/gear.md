---
title: Gear Registration
mod: masterwork
---

<div class="eyebrow">Masterwork &rsaquo; Public API &rsaquo; Gear Registration</div>

# Gear Registration
{: .no_toc }

`GearRegistry` tells Masterwork what to do with a mod's own items; `VariantRegistry` (with `VariantTransformer` and `SalvageTransformer`) edits every graded item and salvage recipe Masterwork mints, in bulk; `ItemResyncStep` keeps a mod's own item instances in line through Masterwork's resync sweep. Reached through `MasterworkApi.gear()`, `MasterworkApi.variants()` and `MasterworkApi.registerItemResync()`.

1. TOC
{:toc}

## Why this exists

Masterwork decides what is gradeable by reading an item's own stat blocks: a weapon has a `Weapon` block, armor an `Armor` block, and so on. That inference is right for everything the base game ships, but it is still an inference, and a modded item built in an unusual shape can be missed, mis-typed, or graded when its author never wanted it to be. `GearRegistry` is the recourse.

## `GearRegistry`

```java
MasterworkApi.get().gear()
    .register("YourMod", "Weapon_Spear_Bronze", GearCategory.WEAPON);
```

| Method | Effect |
|---|---|
| `register(source, itemId, category)` | Have `itemId` graded as `category`, overriding whatever Masterwork would have inferred. Opts the item in even if Masterwork would not otherwise have recognized it as gear. `GearCategory.UNKNOWN` is refused here (with a warning): it is the "not gear" answer, not a category to register. |
| `register(source, Collection<String>, category)` | The same, for a batch of ids. |
| `exclude(source, itemId)` | Keep `itemId` out of Masterwork entirely: no variants minted, no grade rolled at craft, no XP, native rarity border kept. Wins over a registration: an item both registered and excluded is excluded. |
| `exclude(source, Collection<String>)` | The same, for a batch. |
| `isExcluded(itemId)` / `registeredCategory(itemId)` / `sourceOf(itemId)` / `sources()` | Read back what has been registered or excluded, and by whom. |
| `isGear(itemId)` | Whether Masterwork treats `itemId` as gear at all: an item it would grade, a graded variant, or a crafting hammer, read from the item's own shape. No blacklist and no exclusion is consulted, so ask this when acting on gear for some other reason, such as repairing or salvaging it. For whether a fresh craft comes out graded, read `isBlacklisted` off `MasterworkApi.config()` and `isExcluded` instead. Since `1.3.0`. |
| `baseId(itemId)` | The item `itemId` is a graded variant of, or `itemId` itself when it is not one. A graded piece carries its variant's id, so a mod keeping its own list of item ids reads a stack's id through this before looking it up. Since `1.3.0`. |
| `gradeOf(itemId)` | The grade a variant id stands for, or `Grade.base()` for any other id. For a mod that kept only an item id and not the stack, so it has no grade metadata to read. It is the tier the id spells, which is the one the piece shows unless the server owner has since turned that tier off or blacklisted the item. A crafting hammer reads as base here; use `hammerGrade`. Since `1.3.0`. |
| `hammerGrade(itemId)` | The grade a crafting hammer's tier stands for, the one its border shows (the fourth tier at Masterwork, down to the first at Well-Crafted), or `null` when `itemId` is not a hammer. A hammer is never rolled and carries no grade metadata, so this is where a mod reads it. Since `1.3.0`. |
| `categoryOf(itemId)` | The `GearCategory` Masterwork files `itemId` under, a graded variant reading as its base: a mod's registration first, then the item's tags, then its stat blocks. `UNKNOWN` for an excluded or unknown id, and for anything that is not gear. Since `1.3.0`. |

`source` is the registering mod's own name, used in the boot log and to answer "who registered this item?" later, on a server that might be running dozens of mods.

One hard limit: a **stackable** item (`MaxStack > 1`) can never be graded, registered or not. A grade and a crafter stamp live in per-instance metadata, and stacking would merge that metadata away.

The server owner's own blacklist in `masterwork-config.json` is evaluated separately from registration and always wins: blacklisting a registered item degrades it to the plain base exactly as it would any vanilla item.

Register from the depending plugin's own `setup()`, or from a `LoadAssetEvent` listener at a priority below `PRIORITY_LOAD_LATE` (`64`). The registry freezes the moment the boot injection mints the graded variants; a call made after that point is refused with a logged warning, since the variants for that item would already not exist.

## `VariantRegistry`, `VariantTransformer`, `SalvageTransformer`

`GearRegistry` says what an item *is*. `VariantRegistry` is the bulk counterpart: a hook that sees, and can rewrite, every graded item Masterwork mints, and every salvage recipe it mints alongside them, the way a mod changes gear in bulk instead of shipping a pack override per item.

```java
MasterworkApi api = MasterworkApi.get();

api.variants()
    .register("YourMod", (context, variant) -> {
        Grade tier = api.gear().gradeOf(context.variantId());

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

Gear that toggles between states while held (a throw stance, an area-break mode) carries them under `State` in the variant's document, one entry per state holding only what that state restates. A `VariantTransformer` edit that must survive the toggle has to be applied to each of those entries as well as to the top level.

### Contract both hooks share

- **Return the document to continue with.** Editing it in place and returning it is the normal shape; returning a different instance is equally fine.
- **Transformers chain in registration order**, each seeing the previous one's output, so two mods editing the same key resolve last writer wins, in plugin load order.
- **A throw is contained**, logged against the offending mod's name, and the variant continues from the document as it stood before that call. One bad transformer must not cost the server its other roughly 2,600 variants.
- **A `null` return is treated as no change**, and warned about once per source rather than per item.
- **Called a lot**: roughly gradeable items times seven (one call per tier). Filter on `VariantContext` first and return immediately when a variant is not yours to touch.
- **Whatever is written must survive a decode.** The document is handed to the engine's own item or recipe codec next; a malformed value fails that build, and the variant (or recipe) is skipped and counted in the boot log.

Called only for the graded variants, the base tier rewrite and the neutral placeholders included; never for the crafting hammers or the injected qualities, which are Masterwork's own assets rather than transformations of the game's.

Register during `setup()`, like `GearRegistry`: this registry freezes at the same boot injection point, for the same reason.

## `ItemResyncStep`

Some of what an item instance carries is derived from something else on it and does not follow on its own: a display name baked from a metadata key, a description composed from a setting. A stack handed over by `/give --metadata=` carries the key and nothing derived from it; a stack minted before a key was renamed carries the old shape. Masterwork repairs its own crafter stamp and grade data with the item resync sweep, which runs when a player joins, when an inventory changes, when a container is opened, and on `/masterwork item resync`. `registerItemResync` lets a mod add one step of its own to that same sweep, for its own items, instead of building a second sweep with the same triggers:

```java
api.registerItemResync("YourMod", stack -> {
    if (!"YourMod_Charm".equals(stack.getItemId())) {
        return null;
    }

    Integer charge = stack.getFromMetadataOrNull("YourMod.Charge", Codec.INTEGER);

    if (charge == null) {
        return null;
    }

    Message name = Message.translation("server.items.YourMod_Charm.charged").param("n", charge);

    return stack.withMetadata(ItemDisplayMetadata.KEYED_CODEC, new ItemDisplayMetadata(name, null));
});
```

The step is offered every stack the sweep reaches, after Masterwork's own repairs on that stack, in registration order, and its contract is short:

- **Return `null` for a stack that needs nothing**, which is nearly every stack in the world. A returned stack is written back into its slot, and a write is itself an inventory change that runs the sweep again, so a step that always returns a stack never settles.
- **Be cheap on the way out.** The step sees every stick and stone a player carries or handles. Ask the item id first, a keyed metadata read second, and never clone the whole document before either has said yes.
- **Touch only your own items.** A metadata key changes what a stack stacks with, and a foreign item you stamped stops matching the template an interaction spends it against. Masterwork holds itself to the same rule.
- **Do not throw.** A step that throws once is switched off for the rest of the run with one logged warning naming its source, so a bug in one mod cannot take the sweep down for everyone.

Unlike the two registries above, this never closes: the sweep runs for the server's whole life and a step joins it the moment it is registered. Register from `setup()` all the same, so the first join is swept with it. Since `1.3.0`.
