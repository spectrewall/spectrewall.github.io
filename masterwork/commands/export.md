---
title: Export
mod: masterwork
---

<div class="eyebrow">Masterwork &rsaquo; Commands &rsaquo; Export</div>

# Export
{: .no_toc }

A content authoring tool for a server owner building a resource pack around a graded item: dump a Grade variant's live, resolved JSON to a file already trimmed down to just what that tier changes, ready to hand edit and drop into a pack.

1. TOC
{:toc}

### `/mw export <item> [--full=true]`

<p class="cmd-access">masterwork.admin</p>

Writes `mods/Masterwork/Exports/<itemId>.json`. `<item>` has to be a Grade variant id, one Masterwork itself minted (for example `Weapon_Sword_Iron_Graded_Superior`) &mdash; a base item's own id is refused, since there is nothing graded to export from it.

The file is a **diff against the base item**, not the whole resolved item: it sets `"Parent": "<baseId>"`, the same inheritance mechanism a resource pack override already uses, and keeps only the top level keys whose value actually differs from the base's own. Everything a variant never changes &mdash; model, icon, name, tags, and so on &mdash; is left for `Parent` to supply, so the file only carries what is genuinely specific to that tier.

Without `--full`, the export also drops the stat blocks Masterwork computes on its own from the base item and the tier's multipliers (max durability, weapon damage, tool efficiency, armor bonuses), leaving only what a pack author would actually want to hand edit, such as the description. `--full` keeps those blocks too, with their computed values already filled in, useful as a reference for what the numbers came out to.

Three keys are stripped unconditionally, `--full` included: the tier's custom quality, its drop particle, and its salvage resource type. Those name assets Masterwork itself only mints at boot, after every static pack file has already been validated, so a pack declaring one of them fails to load outright. Masterwork's own boot injection is the only thing that ever sets them; leaving them out of an override lets it fill them back in for you.

The written file is staging output, not something the asset loader scans on its own &mdash; copy it into the pack under `Server/Item/**` for it to take effect.
