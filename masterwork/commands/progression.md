---
title: Progression
mod: masterwork
---

<div class="eyebrow">Masterwork &rsaquo; Commands &rsaquo; Progression</div>

# Progression
{: .no_toc }

Progression system selection and per player toast preferences.

1. TOC
{:toc}

### `/mw progression current`

<p class="cmd-access">any player</p>

Print the caller's own level (and luck, where the active source has one) on every ladder biasing their rolls.

### `/mw progression list`

<p class="cmd-access">masterwork.admin / masterwork.config.view</p>

List the registered progression sources and the currently active one.

### `/mw progression notify [xp|level] <on|off>`

<p class="cmd-access">any player</p>

The caller's own opt out of the per craft XP toast and the level up toast. Bare `progression notify` prints both states.

### `/mw progression set <param> <value>`

<p class="cmd-access">masterwork.admin</p>

Tune the active built in ladder's parameters (the XP formula coefficients, the level curve, level odds), or a registered third party source's anchor thresholds and track names.

### `/mw progression system`

<p class="cmd-access">masterwork.admin / masterwork.config.view</p>

Print the progression system the config names and the full list of ids `system set` accepts (`none`, `self`, plus every registered third party source). A shorter, more targeted readout than `progression list`, which dumps the built in ladder's whole formula alongside the active system.

### `/mw progression system set <none|self|<source id>> --confirmation=true`

<p class="cmd-access">masterwork.admin</p>

Switch which system biases the roll: off, Masterwork's own ladder, or a registered third party source's id. **The change applies at the next server restart**, since the crafting hammers are minted around the active system at boot; the reply says so, and `progression system` and `progression list` both name the system the roll is still using until then. The confirmation flag exists because this changes what every other progression control on the page means.
