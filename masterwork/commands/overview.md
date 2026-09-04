---
title: Overview
mod: masterwork
---

<div class="eyebrow">Masterwork &rsaquo; Commands &rsaquo; Overview</div>

# Overview
{: .no_toc }

Masterwork's admin command tree lives under `/masterwork`, with `/mw` as a shorter alias. See [Permissions]({{ 'masterwork/permissions.html' | relative_url }}) for the four nodes that gate this tree and the rules deciding which node a given subcommand needs.

Edits made through the tree are written immediately. A blacklist or grade change replies "applies on restart," since those decide which item variants exist and are only read again at boot. A progression edit applies live, on the next craft.

1. TOC
{:toc}

### `/mw`

<p class="cmd-access">any player</p>

Opens the Masterwork UI page for a player. The console instead gets the usage listing, since it has no page to open.

### `/mw commands`

<p class="cmd-access">any player</p>

Prints the same usage listing the bare root prints for the console, for a player who wants the listing without opening the page.

## Command groups

| Page | Covers |
|---|---|
| [Blacklist](blacklist.html) | `blacklist add`, `blacklist remove`, `blacklist list`, and the `blacklist mod` trio: keeping specific items, or whole mods, out of grading entirely. |
| [Export](export.html) | `export <item> [--full=true]`: dumping a graded variant's resolved JSON for a resource pack override. |
| [Grade](grade.html) | `grade list`, `grade set`, `grade weight`, `grade enable`, `grade disable`: tuning the seven tier ladder. |
| [Hammer](hammer.html) | `hammer set`, `hammer bind`, `hammer recover`, `hammer stash`: crafting hammer tuning and recovery. |
| [Item](item.html) | `item set grade`, `item set crafter`, `item set crafterByUuid`, `item resync`, `item break`: test and correction hooks against the item in the caller's hand. |
| [Notifications](notifications.html) | `notifications xp`, `notifications level`, `notifications announce`, `notifications welcome`: server wide notification switches. |
| [Odds](odds.html) | `odds [level]`: the grade roll odds table. |
| [Progression](progression.html) | `progression current`, `progression notify`, `progression list`, `progression set`, `progression system`, `progression system set`: progression system selection and per player toast preferences. |
| [Ranking](ranking.html) | `ranking visibility`, `ranking rebuild`: ranking board visibility and recovery. |
| [Signature](signature.html) | `signature set`, `signature color`: crafter signature colors, server defaults and each player's own pick. |
