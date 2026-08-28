---
title: Grade
mod: masterwork
---

<div class="eyebrow">Masterwork &rsaquo; Commands &rsaquo; Grade</div>

# Grade
{: .no_toc }

Grade tuning: enable or disable a tier, its stat multipliers, and its no progression roll weight.

1. TOC
{:toc}

### `/mw grade disable <grade>`

<p class="cmd-access">masterwork.admin</p>

Turn a tier off. A disabled tier's items render as their nearest enabled neighbor instead of being skipped. The base tier cannot be disabled.

### `/mw grade enable <grade>`

<p class="cmd-access">masterwork.admin</p>

Turn a tier back on, so it is minted again.

### `/mw grade list`

<p class="cmd-access">masterwork.admin / masterwork.config.view</p>

List every Grade tier with its enabled state and its six stat multipliers.

### `/mw grade set <grade> <param> <value>`

<p class="cmd-access">masterwork.admin</p>

Set one of a tier's six stat multipliers: `damageMultiplier`, `defenseMultiplier`, `efficiencyMultiplier`, `weaponDurabilityMultiplier`, `armorDurabilityMultiplier`, `toolDurabilityMultiplier`. `<value>` accepts a number, or the literal `default` to restore that one multiplier to the ladder's built in default. Replies "applies on restart."

### `/mw grade weight <grade> <weight>`

<p class="cmd-access">masterwork.admin</p>

Set the tier's relative roll weight in the no progression odds table.
