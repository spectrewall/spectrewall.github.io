---
title: Overview
mod: masterwork
---

<div class="eyebrow">Masterwork &rsaquo; Progression &rsaquo; Overview</div>

# Progression
{: .no_toc }

Progression is what biases the Grade roll toward the higher tiers as a crafter gets better at their craft. Without it, every craft rolls against a single flat table (`noProgressionOdds`); with it, the odds a crafter rolls against improve as they climb a ladder, up to seven anchor points that map onto the seven Grade tiers.

1. TOC
{:toc}

## Two systems, one server picks

Masterwork ships its own ladder, [Built-In](built-in.html), on by default. A server can instead point the roll at a supported third party mod's own leveling system, [MMOSkillTree](mmoskilltree.html) today, so a crafter's odds are driven by the same numbers their skill screen already shows.

Only one system is ever active. Switch with `/masterwork progression system set <none|self|<source id>> --confirmation=true`, or from the Settings tab of the `/masterwork` page:

- `none` &mdash; progression is off; every craft uses the flat `noProgressionOdds` table.
- `self` &mdash; Masterwork's own Built-In ladder.
- a registered third party source's id (`mmoskilltree`) &mdash; that mod's ladder.

Read the active system and every id available with `/masterwork progression system`, or a crafter's own standing on it with `/masterwork progression current`.

## What a level buys

Whichever system is active, its raw level lands on the same seven row odds table Grade always rolls against &mdash; one row per tier, from Poor through Masterwork. A system with more than seven levels (MMOSkillTree's 200) maps onto those seven rows through seven configurable anchor points, with the levels in between interpolated toward the next anchor. A system with exactly seven already, Built-In's own, needs no mapping at all.

`/masterwork odds [level]` prints the resulting table for whichever system is active, folding in a crafter's luck where the active source has one.

## Per track, not per player

A crafter does not have one progression level, they have three: weapon, armor and tool are separate standings, and Grade rolls each craft against whichever one matches the item being forged.

Built-In tracks this natively, as three independent XP totals. A source with per skill ladders, like MMOSkillTree, instead maps each track to one of its own skills &mdash; by default the same skill for all three, though a server can point them at different skills (weapon and armor at Smithing, tool at Crafting, say).

## Where it shows up

- The `/masterwork` page's **Statistics** tab draws the active system's own ladder, one row per track, with a level bar and (where the source has one) a luck figure.
- The four crafting hammers' hidden recipes unlock at anchor points on the active ladder, and are renamed after the stage they unlock at while a third party system is active.
- The per craft and level up toasts are Built-In's own and go quiet while another system drives the roll, since neither fires from a ladder that is not being climbed.

## Where to go next

<div class="card-row">
  <a class="nav-card" href="{{ 'masterwork/progression/built-in.html' | relative_url }}">
    <span class="card-title">Built-In</span>
    <span class="card-desc">Masterwork's own seven level ladder: the XP formula, the level curve, and per level odds.</span>
  </a>
  <a class="nav-card" href="{{ 'masterwork/progression/mmoskilltree.html' | relative_url }}">
    <span class="card-title">MMOSkillTree</span>
    <span class="card-desc">Point the roll at MMOSkillTree's own skills instead, luck stat included.</span>
  </a>
  <a class="nav-card" href="{{ 'masterwork/configuration.html' | relative_url }}">
    <span class="card-title">Configuration</span>
    <span class="card-desc">Every progression key in masterwork-config.json.</span>
  </a>
  <a class="nav-card" href="{{ 'masterwork/commands/progression.html' | relative_url }}">
    <span class="card-title">Commands</span>
    <span class="card-desc">The full /masterwork progression command tree.</span>
  </a>
</div>
