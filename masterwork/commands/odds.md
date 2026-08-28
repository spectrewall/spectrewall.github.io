---
title: Odds
mod: masterwork
---

<div class="eyebrow">Masterwork &rsaquo; Commands &rsaquo; Odds</div>

# Odds
{: .no_toc }

### `/mw odds [level]`

<p class="cmd-access">any player</p>

Prints the grade roll odds table, source agnostic: it applies to whichever progression source (Masterwork's own ladder, or a third party mod's) is currently biasing rolls.

- With no argument, the table is grouped by ladder rather than by gear category. Masterwork's own three tracks (weapon, armor, tool) print as three separate rows, since each carries its own XP total; a third party source whose three tracks read one skill prints as a single row, named by that mod's own word for the skill where it ships a translation for it.
- With `<level>`, the level is read on the **active source's own ladder** (so under a 200 level mod, `200` is valid) and the row printed is the interpolated odds for that level.
- When the crafter has a luck stat (from progression, or from a held crafting hammer while hammers spend their bonus as luck), a note under each row's ladder states the bonus being applied, folding both sources together into one figure rather than printing two competing lines.
