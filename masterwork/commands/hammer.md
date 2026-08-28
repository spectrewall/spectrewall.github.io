---
title: Hammer
mod: masterwork
---

<div class="eyebrow">Masterwork &rsaquo; Commands &rsaquo; Hammer</div>

# Hammer
{: .no_toc }

Crafting hammer tuning and recovery. Bare `/mw hammer` prints the enabled state, the owner only bonus switch, all three scalars, and the bind state.

1. TOC
{:toc}

### `/mw hammer bind <on|off>`

<p class="cmd-access">masterwork.admin</p>

Toggle whether a hammer left in a non owner's inventory is confiscated back to its owner's recovery stash.

### `/mw hammer recover`

<p class="cmd-access">any player</p>

Hand the caller back every one of their own hammers waiting in the confiscation stash.

### `/mw hammer set <param> <value>`

<p class="cmd-access">masterwork.admin</p>

Set one of the hammer's scalars: `craftDurabilityDrain` (how much durability one craft costs; negative repairs instead), `nonOwnerDamageMultiplier`, `nonOwnerDamageReflectedMultiplier`. Applied live.

### `/mw hammer stash`

<p class="cmd-access">masterwork.admin</p>

Deposit the hammer in the caller's active hotbar slot into its owner's stash. A solo test tool for the stash and recover round trip.
