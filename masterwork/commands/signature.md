---
title: Signature
mod: masterwork
---

<div class="eyebrow">Masterwork &rsaquo; Commands &rsaquo; Signature</div>

# Signature
{: .no_toc }

Crafter signature colors: the server defaults, and each player's own pick.

1. TOC
{:toc}

### `/mw signature set <lineColor|hammerNoteColor> <#RRGGBB|default>`

<p class="cmd-access">masterwork.admin</p>

Recolor part of the crafter signature line shown on a stamped item's tooltip. The color is read at stamp time and baked onto the item instance, so recoloring the server default does not repaint gear a player is already carrying; the item resync sweep catches up to it the next time that gear is reached (on join, on an inventory change, or when a container holding it is opened).

### `/mw signature color [entry]`

<p class="cmd-access">any player</p>

The caller's own pick for the color of their crafter signature line, overriding the server default `lineColor` on gear they forge from now on. With no entry, clears the pick and falls back to the server default. Like the admin `set` above, the color is read at stamp time, so it applies to items stamped from now on, not to gear already carrying a signature. Also reachable from the Preferences tab of the `/mw` page.
