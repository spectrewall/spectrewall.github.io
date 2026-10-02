---
title: Forge Souls
permalink: /masterwork/addons/forge-souls/
mod: masterwork
version: 1.0.0
---

<div class="eyebrow">Masterwork &rsaquo; Addons &rsaquo; Forge Souls</div>

# Forge Souls <span class="version-tag">v{{ page.version }}</span>
{: .no_toc }

An optional addon for Masterwork, shipped as its own jar, `Masterwork-ForgeSouls-<version>.jar`, attached to every release beside Masterwork's. It adds the **Soul Anvil**, where a graded piece of gear is offered up for its soul, a hundred essences merge into one soul, and a soul restores a worn piece to its full durability.

1. TOC
{:toc}

> **Hard dependency.** Forge Souls requires Masterwork `1.3.0` or newer. Never install it alone, and update the two together: the jars attached to one release were built from the same commit and tested as a pair.

## Installing

Drop the jar into the server's `mods/` folder beside Masterwork's, and restart. The addon is on as soon as it is installed, so leave the jar out if you only want Masterwork: dropping both into `mods/` by reflex adds a new bench and two new items to the game.

Forge Souls has no commands and no permission nodes of its own. Its settings are edited from Masterwork's **Settings** tab, so `masterwork.admin` covers them (see [Permissions]({{ '/masterwork/permissions.html' | relative_url }})).

## The Soul Anvil

The anvil is crafted at a **tier 3 Workbench** from 32 Adamantite Bars, 8 Tree Sap, 100 Fire Essence and 1 Voidheart. Place it, then use it with the piece you want to work on in your hand. The anvil reads what you are holding and opens on the job that piece allows; a piece both jobs accept asks which one you mean.

Opened with nothing it can take in your hand, the anvil shows a short tutorial instead. Each player can turn that tutorial off, and the locked third person camera during an offering, from the **Preferences** tab of `/masterwork`.

A job takes time, and the anvil holds the piece and the payment until it finishes. Walk away, or let the server stop mid job, and both go back to the player with nothing made.

## Forge Essence and Forge Soul

Two items, one worth a hundred of the other: **one Forge Soul is 100 Forge Essence**.

Both carry a **level**, the item level of the piece they came from, shown in their name (`Forge Essence (Level 40)`). Two different levels never stack together. The level is what a restore checks: the payment has to be at the piece's own level or higher.

## The three jobs

| Job | Takes | Gives |
|---|---|---|
| **Offering** | A graded piece of gear. | Forge Essence or a Forge Soul at the piece's level, in the amount its grade is worth (see [`offerings`](#offerings)). The piece is gone. |
| **Merging** | 100 Forge Essence of one level. | One Forge Soul of that level. |
| **Restoring** | A worn piece, plus its restore cost (see [`restoreCosts`](#restorecosts)). | The same piece at its full original durability. |

An offering takes longer the more it pays, and a restore takes longer the higher the piece's item level. To pay for a restore, the anvil takes the weakest level the player holds enough of, as long as it meets the piece's level.

Each finished job pays crafting XP on whichever progression system the server has active (see [Progression]({{ '/masterwork/progression/overview.html' | relative_url }})), as a share of what crafting the same piece pays (see [`xpMultipliers`](#xpmultipliers)). A merge is priced as a 10 second craft at the essence's level, and its XP is split evenly across weapons, armor and tools.

## Configuration

Forge Souls' settings live in `forgesouls-config.json`, under `mods/ForgeSouls/` in the server's own working directory, beside `mods/Masterwork/` and likewise outside the jar, so deleting or updating the jar never touches it.

Every key below is also editable from the **Settings** tab of `/masterwork`, under **Addons &rsaquo; Forge Souls &rsaquo; Configure**, where a save takes effect at once. The file itself is read only when the server starts, so edit it by hand with the server stopped; a save from the Settings tab rewrites it.

Every grade keyed table below is keyed by `MASTERWORK`, `EXQUISITE`, `SUPERIOR`, `WELL_CRAFTED`, `ADEQUATE`, `COMMON` and `POOR`. A grade left out keeps its default.

### `offerings`

What an offering of each grade pays, shown in the Settings tab as **Offering**. Each row is a `kind`, `FORGE_SOUL` or `FORGE_ESSENCE`, and an `amount` from `0` to `100`. **An amount of `0` makes the anvil refuse pieces of that grade.**

| Grade | Default |
|---|---|
| `MASTERWORK` | `1` `FORGE_SOUL` |
| `EXQUISITE` | `10` `FORGE_ESSENCE` |
| `SUPERIOR` | `1` `FORGE_ESSENCE` |
| `WELL_CRAFTED`, `ADEQUATE`, `COMMON`, `POOR` | `0`, refused |

```json
"offerings": {
  "MASTERWORK": { "kind": "FORGE_SOUL", "amount": 1 },
  "EXQUISITE": { "kind": "FORGE_ESSENCE", "amount": 10 }
}
```

### `restoreCosts`

What a restore of each grade costs, shown as **Restoring**. Same rows and the same rule, `0` refusing that grade, plus an eighth row, `UNGRADED`, for a piece with no grade. A crafting hammer is priced at the grade of its tier. Every row defaults to `1` `FORGE_SOUL`.

```json
"restoreCosts": {
  "MASTERWORK": { "kind": "FORGE_SOUL", "amount": 1 },
  "UNGRADED": { "kind": "FORGE_SOUL", "amount": 1 }
}
```

### `blacklist`

Items the anvil refuses, shown as **Blacklist**, each id carrying two flags. Empty by default.

| Key | Description |
|---|---|
| `offering` | `true` keeps the item from being offered. |
| `repairing` | `true` keeps the item from being restored. |

A graded piece counts as its base item, so listing a base item covers every grade of it. An id with both flags `false` is off the list, which is what unticking both boxes in the Settings tab does.

```json
"blacklist": {
  "Weapon_Sword_Iron": { "offering": true, "repairing": false }
}
```

### `xpMultipliers`

One multiplier per job, shown as **Progression**: the share of the piece's crafting XP that job pays, from `0` to `100`. At `0` a job pays no XP.

| Key | Default | Description |
|---|---|---|
| `OFFERING` | `0.1` | An offering pays a tenth of what crafting the piece pays. |
| `CONVERSION` | `0.1` | A merge, priced as described in [The three jobs](#the-three-jobs). |
| `RESTORE` | `0.1` | A restore pays a tenth of what crafting the piece pays. |

## Public API

Forge Souls has a public API of its own, under `com.spectrewall.forgesouls.api`, following the same stability rule as [Masterwork's]({{ '/masterwork/api/overview.html' | relative_url }}#versioning). Depend on it softly, under `OptionalDependencies`, and reach it through:

```java
ForgeSoulsApi api = ForgeSoulsApi.get();

if (api != null) {
    api.offeringExclusions().exclude("YourMod", "YourMod_Relic");
}
```

`ForgeSoulsApi.get()` returns `null` until Forge Souls' own setup has run, and forever on a server where it is not installed: always null check.

| Method | Effect |
|---|---|
| `config()` | The settings above, live. Read through it; a setter called on it directly applies at once but is lost on the next restart. |
| `editConfig(source, edit)` | Change the settings in one write: the edit runs against the live config, is saved, and is logged under `source`, your mod's name. |
| `offeringExclusions()` / `restoreExclusions()` | Keep your own items off the anvil for one job, beside the server owner's blacklist. |
| `isOfferingAllowed(itemId)` / `isRestoreAllowed(itemId)` | Whether the anvil would take an item id for that job. |
| `canOffer(stack)` / `canRestore(stack)` | The same, for the stack in hand. |
| `essence(kind, level, quantity)` / `essenceLevel(stack)` | Make a Forge Essence or Forge Soul stack at a level, or read the level off one. |
| `offeringCamera(ref)` / `stationTutorial(ref)` and their setters | A player's two Preferences. |
| `jobCount(ref, mode)` | How many jobs of one kind a player has finished. |
| `apiVersion()` / `pluginVersion()` | The running addon's API version and plugin version. |

### `SoulAnvilJobEvent`

Fired at the player when a job finishes, synchronously, on the world thread. Listen with an `EntityEventSystem<EntityStore, SoulAnvilJobEvent>`. A job abandoned midway fires nothing. Every stack it carries is a copy: changing one changes nothing the player holds.

| Method | Effect |
|---|---|
| `mode()` | `OFFERING`, `CONVERSION` or `RESTORE`. May gain constants, so never switch over it without a default branch. |
| `playerUuid()` | The player who ran the job. |
| `source()` | What was put on the anvil: the piece offered or restored, or the hundred essence merged. |
| `payment()` | What a restore spent besides the piece, all of one level; `null` for an offering or a merge. |
| `reward()` | What the job made: the essence or soul paid, or the restored piece. |
| `itemLevel()` | The level the job ran at: the piece's item level, or the essence's level for a merge. |
| `anvil()` | The anvil's block position, or `null` when the engine did not name the block. |
| `craftTimeSeconds()` | How long the job took. |
