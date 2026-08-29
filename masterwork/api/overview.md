---
title: Overview
mod: masterwork
---

<div class="eyebrow">Masterwork &rsaquo; Public API &rsaquo; Overview</div>

# Overview
{: .no_toc }

Masterwork exposes a small, stable surface under the `com.spectrewall.masterwork.api` package for another plugin to build against: react to a craft, read a crafter's standing, register a mod's own items for grading, edit graded gear in bulk, or offer a progression ladder of its own.

1. TOC
{:toc}

## Depending on Masterwork

Masterwork is meant to be a soft dependency. Declare it under `OptionalDependencies` in the depending mod's manifest, so the loader puts Masterwork first when it is present, and reach it through:

```java
MasterworkApi api = MasterworkApi.get();

if (api != null) {
    api.gear().register("YourMod", "Weapon_Spear_Bronze", GearCategory.WEAPON);
}
```

`MasterworkApi.get()` returns `null` until Masterwork's own setup has run, and forever on a server where it is not installed, which is exactly how a soft dependency should read: always null check.

## Versioning

`MasterworkApi.API_VERSION` is an integer, independent from the plugin's own version, that moves only when this surface gains a member, never when the implementation behind it changes. A mod supporting more than one Masterwork release can branch on `API_VERSION >= n` rather than reflecting on a method to ask whether it exists. Do not derive it from the plugin's version: the plugin version moves on every release, including ones that change nothing here.

Once Masterwork reaches `1.0.0`, this surface only ever grows. Adding a method to a public interface always gives it a default implementation, adding a constant to a public enum is allowed (which is why the javadoc tells consumers never to write an exhaustive `switch` over `Grade` or `GearCategory` without a default branch), and changing an event's payload means a new accessor beside the old one rather than a changed signature.

That rule has been broken once, and this is the whole of it: `CraftProcessedEvent.xp()` was removed in `1.1.0`, three days after `1.0.0` was published and with no mod known to have compiled against the event yet. The reasoning, and what to read in its place, are on the [Craft Events](craft.html) page. `API_VERSION` did not move for it, since it marks what this surface gained and a removal is not a gain, and a consumer cannot branch defensively on a method that is already gone. Assume from here that the rule holds: a member that outlives its usefulness gets deprecated and kept, not deleted.

## Registration closes at boot

Three things close for writing the moment the boot asset injection mints the graded item variants: the gear registry ([Gear Registration](gear.html)), the variant registry (the same page) and third party progression source registration ([Progression Sources](progression.html)). Register from the depending plugin's own `setup()`, or from a `LoadAssetEvent` listener below that priority. A call made after the freeze is refused with a logged warning rather than silently doing nothing.

Everything else on this surface, the config, a player's data, the ranking board, is live for the server's whole life and carries no such restriction.

## Reading and writing configuration

`MasterworkApi.config()` returns the same `MasterworkConfig` object the built in commands and settings page read from. Read directly from it; write only through `editConfig`:

```java
api.editConfig("YourMod", config -> config.setAnnounceMasterwork(false));
```

`MasterworkConfig`'s own setters apply a value live but do not persist it, do not move the revision that tells already stamped gear it is stale, and do not start the sweep that re-syncs that gear. `editConfig` runs all three follow ups after the edit: it saves the file, re-pushes the hammers' names and tooltips (which are sent to each client as strings rather than shipped as assets), and re-syncs every online player's gear if the revision moved. `source` is the depending mod's name, and every edit is logged at a level the `debug` switch cannot silence, since these are the server owner's settings and an edit through this seam should always be traceable to whoever made it.

Not every setting takes effect immediately even through this seam: the ones that decide which item variants and recipes exist (the grade multipliers, the loot tables, the instant recipe time) are read at boot and apply on the next restart, exactly as they do when the server owner edits them by hand or through the settings page.

## What this surface does not cover

`MasterworkConfig` itself, and its nested settings value types, live outside `api`, even though every one of their getters is contract a mod may safely call. Moving roughly 2,800 lines of parsing, validation and defaults into `api` would have inverted the point of the boundary, turning "only what is public" into "everything." The rule that keeps this workable is that everything kept outside `api` is something a mod reads or names, never something it implements: no internal type outside `api` is an interface a sibling mod subclasses, so no internal refactor of Masterwork can break someone else's `@Override`.

## Where each part lives

| Page | Covers |
|---|---|
| [Vocabulary](vocabulary.html) | `Grade`, `GearCategory`, `MasterworkMetadata`: the shared types every other part of the surface speaks in. |
| [Gear Registration](gear.html) | `GearRegistry`, `VariantTransformer`, `SalvageTransformer`, `VariantRegistry`: telling Masterwork what a mod's own items are, and editing graded gear in bulk. |
| [Craft Events](craft.html) | `CraftProcessedEvent`, `NativeXpResult`: reacting to a craft after Masterwork has resolved it. |
| [Players](player.html) | `MasterworkPlayer`, `CraftTally`: reading and writing what Masterwork knows about one online player. |
| [Progression Sources](progression.html) | `ProgressionSource`, `LadderProgress`: offering a mod's own leveling system as one a server can select to bias the grade roll. |
| [Ranking](ranking.html) | `MasterworkRanking`, `RankingRow`, `RankingVisibility`: the ranking board, read only plus its one recovery operation. |
