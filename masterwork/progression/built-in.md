---
title: Built-In
mod: masterwork
mathjax: true
---

<div class="eyebrow">Masterwork &rsaquo; Progression &rsaquo; Built-In</div>

# Built-In
{: .no_toc }

Masterwork's own progression system, `self`, active by default. A crafter earns XP on each of the three gear tracks (weapon, armor, tool) by crafting graded gear, and XP buys levels on a seven level ladder that lines up one to one with the seven rows of the Grade odds table &mdash; no anchor mapping needed, since the ladder already has exactly seven rungs.

1. TOC
{:toc}

## Earning XP

Every graded craft awards XP on the track its gear category belongs to. The amount is a function of the item's own `ItemLevel` and the recipe's craft time:

$$
\text{xp} = \frac{(L \cdot m_L)^{e_L} + (T \cdot m_T)^{e_T}}{n}
$$

| Symbol | Config key | Meaning |
|---|---|---|
| $L$ | &mdash; | The crafted item's own `ItemLevel`. |
| $m_L$ | `itemLevelMultiplier` | Coefficient on `ItemLevel`. |
| $e_L$ | `itemLevelExponent` | Exponent on `ItemLevel`. |
| $T$ | &mdash; | The recipe's craft time, in seconds. |
| $m_T$ | `timeSecondsMultiplier` | Coefficient on craft time. |
| $e_T$ | `timeSecondsExponent` | Exponent on craft time. |
| $n$ | `normalizer` | Divides the sum down to a workable XP figure. |

All five coefficients (everything but $L$ and $T$ themselves) are tunable under `builtInProgression` &mdash; see [Configuration](../configuration.html) &mdash; through `/masterwork progression set` or the settings page. Crafting time weighs XP more heavily than the item's own power level does, so a harder, longer recipe is worth more than a quick one of the same item level.

A craft made in hand, with no bench, earns no XP at all &mdash; see [Configuration &rsaquo; fieldCraft](../configuration.html).

## The level curve

Total XP on a track buys a level, 1 through 7, on a geometric curve seeded by `xpBase` (the XP needed to reach level 2) and `xpGrowth` (the multiplier between levels). Read a crafter's own progress with `/masterwork progression current`, or on the `/masterwork` page's Statistics tab, which draws a bar toward the next level for each of the three tracks.

## Per level odds

`builtInProgression.levelOdds` holds one row of Grade percentages per level, 1 through 7 &mdash; the same shape the odds table always has, since Built-In's ladder already sits exactly on the seven anchors. `/masterwork odds [level]` prints any row, live; with no level given, it prints the caller's own row on all three tracks.

## Toasts

Two notifications narrate a crafter's Built-In progress: a small one on every craft that earns XP, and a larger one on a level up. Both are server switches (`notifications.xpEnabled` / `.levelEnabled`) layered under each player's own opt out (`/masterwork progression notify xp|level <on|off>`, also reachable from the Preferences tab). Both go quiet while another system is active, since neither toast narrates a ladder nobody is climbing.

## Hammer unlocks

The four crafting hammers' hidden recipes unlock at Built-In levels `2 / 3 / 5 / 7`, on one gear track or all three depending on the tier &mdash; see [Configuration &rsaquo; hammer.unlocks.\<TIER\>](../configuration.html). Under a third party system, the same four unlocks spread wider, across that ladder's own anchor points.

## Turning it off

Setting the active system to `none` (`/masterwork progression system set none --confirmation=true`) drops every craft back to the flat `noProgressionOdds` table, regardless of any Built-In level a crafter has already reached. No new XP is added while another system is active, and switching back to `self` resumes exactly where a crafter's Built-In levels left off &mdash; the craft tally and ranking points, which are not gated on which system is running, are untouched either way.
