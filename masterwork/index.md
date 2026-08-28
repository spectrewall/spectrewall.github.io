---
title: Masterwork
mod: masterwork
permalink: /masterwork/
description: A Hytale server plugin that introduces a craftsmanship system to forging.
---

<div class="eyebrow">Masterwork</div>

# Masterwork

A Hytale server plugin that introduces a craftsmanship system to forging. Each piece of gear is signed with its creator's name and receives a unique quality roll, reflecting how well it was crafted.

Concretely, when a player crafts a weapon, an armor piece or a tool, the plugin does two things.

1. **Stamps the crafter's name.** The name is persisted on that single item instance and shown in the item's in game description as "Crafted by \<name\>".
2. **Rolls a Grade.** A craftsmanship tier is rolled at craft time. A higher Grade swaps the crafted item to a per tier item variant with natively scaled stats (damage, armor, durability) and a native rarity border in the Grade's color. The crafter stamp rides along on the swapped instance.

Both features are implemented and verified in game. The crafter stamp works across instant, timed and bulk crafts, and it survives a relog. The Grade system rolls a seven tier ladder, supports custom quality injection, mints per tier variants with native stats and a border, swaps the item on craft, and honors a blacklist through neutral variants.

## The Grade ladder

Grade is a seven tier craftsmanship ladder, independent from the game's own material rarity system:

<ul class="grade-list">
  <li><span class="grade-swatch" style="background:var(--grade-masterwork)"></span><span class="grade-name">Masterwork</span><span class="grade-note">top tier, red</span></li>
  <li><span class="grade-swatch" style="background:var(--grade-exquisite)"></span><span class="grade-name">Exquisite</span><span class="grade-note">gold</span></li>
  <li><span class="grade-swatch" style="background:var(--grade-superior)"></span><span class="grade-name">Superior</span><span class="grade-note">purple</span></li>
  <li><span class="grade-swatch" style="background:var(--grade-well)"></span><span class="grade-name">Well-Crafted</span><span class="grade-note">blue</span></li>
  <li><span class="grade-swatch" style="background:var(--grade-adequate)"></span><span class="grade-name">Adequate</span><span class="grade-note">green</span></li>
  <li><span class="grade-swatch" style="background:var(--grade-common)"></span><span class="grade-name">Common</span><span class="grade-note">base tier, gray</span></li>
  <li><span class="grade-swatch" style="background:var(--grade-poor)"></span><span class="grade-name">Poor</span><span class="grade-note">bottom tier, dark gray</span></li>
</ul>

Common is the base tier: a common roll yields the item's shipped id and stats, only with its rarity border normalized to Masterwork's own Common quality. Every tier above and below it mints a distinct item variant, so a Masterwork sword is a different item instance from a Common one, with its own stats and its own quality border.

> **Grade vs. Quality.** Quality is the base game's `ItemQuality` asset system, the colored slot and border a player already knows from vanilla rarity. Masterwork injects its own custom quality assets, one per Grade, and stamps them on graded items so the border communicates the Grade rather than the item's native material rarity.

## Drops and chests

Grade is not only rolled at the crafting bench. Gear the world gives a player, a mob's drop or a chest's contents, is graded too, on its own odds table with its own tuning. By default the top tier is not available there at all: Masterwork is a title a crafter earns, not one a player finds lying on the ground. A drop table can also pin an exact tier on a hand authored reward, so a quest chest can hand out a guaranteed Superior sword.

A found piece rolls no XP, no ranking points and no crafter stamp: finding a piece of gear is not the same as forging one. See [Droplist]({{ 'masterwork/droplist.html' | relative_url }}) for how the roll works and how to pin a guaranteed tier on a hand authored reward.

## Where to go next

<div class="card-row">
  <a class="nav-card" href="{{ 'masterwork/progression/overview.html' | relative_url }}">
    <span class="card-title">Progression</span>
    <span class="card-desc">What biases the Grade roll as a crafter improves: the Built-In ladder, or a mod like MMOSkillTree.</span>
  </a>
  <a class="nav-card" href="{{ 'masterwork/droplist.html' | relative_url }}">
    <span class="card-title">Droplist</span>
    <span class="card-desc">How gear the world hands a player is graded, and how to pin a guaranteed tier on a hand authored reward.</span>
  </a>
  <a class="nav-card" href="{{ 'masterwork/permissions.html' | relative_url }}">
    <span class="card-title">Permissions</span>
    <span class="card-desc">The nodes that gate the admin command tree and the UI's Settings tab.</span>
  </a>
  <a class="nav-card" href="{{ 'masterwork/commands/overview.html' | relative_url }}">
    <span class="card-title">Commands</span>
    <span class="card-desc">The full /masterwork admin command tree.</span>
  </a>
  <a class="nav-card" href="{{ 'masterwork/configuration.html' | relative_url }}">
    <span class="card-title">Configuration</span>
    <span class="card-desc">Every setting in masterwork-config.json, its default and what it controls.</span>
  </a>
  <a class="nav-card" href="{{ 'masterwork/api/overview.html' | relative_url }}">
    <span class="card-title">Public API</span>
    <span class="card-desc">How another plugin reads a crafter's standing, reacts to a craft, registers its own items for grading, or edits graded gear in bulk.</span>
  </a>
</div>
