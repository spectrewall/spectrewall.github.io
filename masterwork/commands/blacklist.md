---
title: Blacklist
mod: masterwork
---

<div class="eyebrow">Masterwork &rsaquo; Commands &rsaquo; Blacklist</div>

# Blacklist
{: .no_toc }

Blacklist management. A blacklisted item mints no graded variants, rolls no grade at craft, and keeps its native rarity border. There are two lists: one of item ids, and one of whole mods.

1. TOC
{:toc}

## Items

### `/mw blacklist add <itemId>`

<p class="cmd-access">masterwork.admin</p>

Add an item id to the grade blacklist. Suggests every item the server knows.

If the item's mod is already on the mod blacklist, the reply says so. The entry is still recorded: it keeps the item excluded on its own once that mod comes off the other list.

### `/mw blacklist list`

<p class="cmd-access">masterwork.admin / masterwork.config.view</p>

List every blacklisted item id. When the mod blacklist is not empty, a closing line says how many mods are also excluded whole — none of their items appear in this list.

### `/mw blacklist remove <itemId>`

<p class="cmd-access">masterwork.admin</p>

Remove an item id from the blacklist. Suggests the blacklisted ids themselves, so an id whose mod has since been uninstalled can still be lifted.

## Mods

Excluding a mod excludes every item it ships, without listing them one by one. Items of a blacklisted mod behave exactly like individually blacklisted ones.

A mod is named by its `Group:Name` identity — the same pair its `manifest.json` declares, for example `KiriOfTheWoods:Homesteadin`.

### `/mw blacklist mod add <Group:Name>`

<p class="cmd-access">masterwork.admin</p>

Exclude an installed mod's items from grading. Suggests every mod that could ship an item, including one that is installed but currently disabled. The base game and Masterwork itself are not offered.

### `/mw blacklist mod list`

<p class="cmd-access">masterwork.admin / masterwork.config.view</p>

List every blacklisted mod.

### `/mw blacklist mod remove <Group:Name>`

<p class="cmd-access">masterwork.admin</p>

Re-include a previously excluded mod. Suggests the blacklisted mods as well as the installed ones, so an entry left behind by a mod you have since uninstalled can still be removed.

## Notes

Both lists apply **on restart**: the item variants are minted when the server boots, so a change takes effect the next time it does. Every reply says so.

Both are also editable in-game on the `/masterwork` page, under **Blacklist** and **Mod Blacklist** in Settings.

An entry for a mod that is not installed is kept, not discarded — your decision about a mod should not disappear because you took that mod out for a week.
