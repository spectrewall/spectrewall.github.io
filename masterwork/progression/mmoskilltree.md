---
title: MMOSkillTree
mod: masterwork
---

<div class="eyebrow">Masterwork &rsaquo; Progression &rsaquo; MMOSkillTree</div>

# MMOSkillTree
{: .no_toc }

An optional integration with `Ziggfreed:MMOSkillTree`, an mcMMO / RuneScape style skill mod. Instead of Masterwork's own seven level ladder, the Grade roll is biased by a crafter's level in one of MMOSkillTree's own skills &mdash; by default, Crafting.

1. TOC
{:toc}

> **Hard dependency.** MMOSkillTree itself requires `Ziggfreed:ZiggfreedCommon`. Installing it without that dependency does not simply disable the integration &mdash; it breaks MMOSkillTree's own asset loading and the **whole server** fails to boot with any assets at all. Install both jars together.

## Turning it on

With both mods installed, switch the active system:

```
/masterwork progression system set mmoskilltree --confirmation=true
```

`/masterwork progression system` lists every id available, including `mmoskilltree` once MMOSkillTree is detected. If the mod is later removed, Masterwork falls back to the Built-In ladder automatically and logs a warning; the server's tuning for MMOSkillTree (its thresholds, track names, luck switch) is kept in the config file either way, so reinstalling the mod picks up exactly where it left off.

## Which skill each track reads

By default, all three gear tracks (weapon, armor, tool) read the crafter's level in MMOSkillTree's **Crafting** skill. A server that treats forging as smith's work can instead point weapons and armor at **Smithing**, without touching a config file by hand:

```
"progressionSources": {
  "mmoskilltree": {
    "customTrackKeys": true,
    "trackKeys": {
      "WEAPON": "Smithing",
      "ARMOR": "Smithing",
      "TOOL": "Crafting"
    }
  }
}
```

`customTrackKeys` must be `true` for `trackKeys` to be read at all; while it is `false`, all three tracks silently use MMOSkillTree's own default skill regardless of what is stored. The same three rows are editable from the Settings &rsaquo; Progression section of the `/masterwork` page, as a picker filled from MMOSkillTree's own registered skills (including any custom skills that server has defined), so a name never has to be typed by hand.

## Mapping 200 levels onto seven Grade tiers

MMOSkillTree's skills go up to level 200. Masterwork's odds table has exactly seven rows, one per Grade tier, so seven **anchor points** say where on that 200 level ladder each row lands:

```json
"levelThresholds": [1, 10, 25, 40, 60, 80, 99]
```

A level between two anchors blends toward the next row by default (`interpolateLevels: true`), so odds creep upward gradually rather than jumping. Turning `interpolateLevels` off makes the table a staircase instead: a level rolls its lower anchor's row unchanged until the next anchor is reached exactly, which reads better when the anchors are announced milestones ("at level 40 you can start forging Well-Crafted gear") rather than a smooth curve. Both ends clamp &mdash; level 1 never rolls below anchor 1's row, and level 200 never rolls above anchor 7's.

Edit the seven thresholds from `/masterwork progression set`, or from the ladder mapping block on Settings &rsaquo; Progression, which only appears while a third party system with more than seven levels is active.

## Luck

MMOSkillTree's Crafting skill tree includes a **Luck** stat, earned by claiming reward nodes and mastery packs, that has no effect in MMOSkillTree itself. Masterwork spends it: the crafter's total Crafting luck (skill tree nodes plus mastery bonuses, the same total MMOSkillTree's own Stat Sources screen shows) becomes three percentage point boosts &mdash; the full figure, half of it, and a quarter of it &mdash; landing on the three best Grade tiers that crafter can currently roll. That is the same shape the top crafting hammer's own bonus takes, so a crafter with, say, 25% Crafting luck is getting a quarter of that hammer's boost for free.

There is no ceiling on this bonus; every point spent on Crafting luck keeps buying better odds. Governed by `progressionSources.mmoskilltree.useLuckAttribute`, on by default. A crafter's luck figure shows on the Statistics tab (opposite the track's level bar) and as a note under their row in `/masterwork odds`, in the same percentage MMOSkillTree's own screen reports, so the two can be checked against each other.

## Hammer unlocks

The four crafting hammers' hidden recipes unlock at anchors `3 / 5 / 6 / 7` while MMOSkillTree is active &mdash; spread wider than Built-In's `2 / 3 / 5 / 7`, to give the longer ladder more room. Each hammer is also renamed after the stage it unlocks at (`renameHammersByAnchor`, on by default), so its name on screen tracks the active system's own milestones rather than Built-In's.

## Version and detection

Verified against MMOSkillTree `1.5.2`. Masterwork detects it by probing for its API class at boot; a server that disables the mod through its own config is read the same as one where it is not installed at all, and the `mmoskilltree` option simply does not appear. A version other than the one this integration was tested against is logged and used anyway, rather than refused.
