---
title: Configuration
mod: masterwork
---

<div class="eyebrow">Masterwork &rsaquo; Configuration</div>

# Configuration
{: .no_toc }

Masterwork's settings live in `masterwork-config.json`, under `mods/Masterwork/` in the server's own working directory (not inside the plugin jar, so deleting or updating the jar never touches it). The file is written out in full on first boot, so it is self documenting: every key below is already there with its default value, ready to edit by hand or through `/masterwork` and the in game settings page.

1. TOC
{:toc}

## How settings apply

Most of the file is read live: a progression, notification, ranking, hammer or signature edit takes effect on the next craft, join or command. A handful of blocks decide which item variants and recipes exist at all, `blacklist`, `grades`, `lootGrades` and `instantRecipeSeconds`, so an edit there is read again only at the next server restart, exactly as the settings page and the commands that touch them say.

## Top level

| Key | Default | Description |
|---|---|---|
| `debug` | `false` | Verbose console diagnostics: per craft grade and XP lines, and the boot injection dump. Off by default; warnings and errors always print regardless of this switch. |
| `instantRecipeSeconds` | `1` | The craft time, in seconds, given at boot to a gradeable item whose shipped recipe is instant. A `0` recipe scores no ranking points and only the flat XP branch, so the crude tier (which ships instant) is given a real craft time to score against. `0` leaves every recipe as it ships. |
| `excludeWansWonderWeapons` | `true` | Keeps a specific sibling mod's weapons out of the grade system entirely, since two of that mod's items transform into another item id and cannot be graded at all. The row only shows on the settings page while that mod is installed, but the key is kept in the file either way. |

## `blacklist`

A flat array of item ids kept out of grading entirely: no variants are minted for them, crafting one rolls no grade and earns no XP, and the item keeps its native rarity border.

```json
"blacklist": [ "Weapon_Sword_Crude" ]
```

## `grades`

One entry per Grade tier, each field optional and falling back to the ladder's tuning table below when omitted. The base tier's `enabled` is always coerced to `true`, since it is the roll's own fallback.

| Field | What it scales |
|---|---|
| `enabled` | Whether this tier is minted at all. A disabled tier's items render as their nearest enabled neighbor instead of vanishing. |
| `damageMultiplier` | Weapon damage. |
| `defenseMultiplier` | Armor stat modifiers (bonus health, and a shield's guard stamina efficiency). |
| `efficiencyMultiplier` | Tool power. |
| `weaponDurabilityMultiplier` | Max durability of graded weapons (and shields, which have no durability of their own). |
| `armorDurabilityMultiplier` | Max durability of graded armor. |
| `toolDurabilityMultiplier` | Max durability of graded tools. |

Shipped defaults, one row per tier:

| Grade | damage / defense / efficiency | weapon / armor / tool durability |
|---|---|---|
| Masterwork | 1.50 | 5.00 |
| Exquisite | 1.30 | 2.00 |
| Superior | 1.20 | 1.30 |
| Well-Crafted | 1.10 | 1.20 |
| Adequate | 1.05 | 1.10 |
| Common | 1.00 | 1.00 |
| Poor | 0.90 | 0.80 |

Durability steps deliberately widen toward the top of the ladder rather than scaling linearly: the jump to 2.00 at Exquisite and 5.00 at Masterwork is what keeps the top tier feeling worth its rarity, since it is what a player feels over an item's whole life rather than per swing.

## `noProgressionOdds`

One relative weight per Grade, used as the roll distribution while the progression modifier is off (`progression.enabled: false`). Normalized at roll time, so the numbers do not have to sum to 100, though the defaults do:

| Grade | Weight |
|---|---|
| Masterwork | 1 |
| Exquisite | 3 |
| Superior | 8 |
| Well-Crafted | 15 |
| Adequate | 25 |
| Common | 30 |
| Poor | 18 |

## `lootGrades`

Grading for gear the world hands a player, separate from the craft roll: a mob's drop, or a chest's contents.

| Field | Default | Description |
|---|---|---|
| `enabled` | `true` | Whether a piece of found gear is rolled at all, the first time it is reached. |
| `alwaysDropGear` | `false` | Experimental. Forces every drop table capable of yielding gear to actually yield it, which is the only practical way to look at a graded drop without farming for an hour. It costs whatever else competed with the gear in the same table, since that entry's weight is otherwise unchanged. Applies at the next restart, since it is part of the boot rewrite of the drop tables. |
| `odds` | see below | One percentage per Grade, normalized like the other two tables. |

Shipped `odds` defaults:

| Grade | Percent |
|---|---|
| Masterwork | 0 |
| Exquisite | 1 |
| Superior | 4 |
| Well-Crafted | 25 |
| Adequate | 15 |
| Common | 40 |
| Poor | 15 |

Masterwork ships at `0` for loot on purpose: the top tier is a title a crafter earns, not one a player finds on the ground. The row is editable like any other, since that is a default and not a rule enforced in code.

A drop table entry can also carry an exact Grade directly, in its `"Metadata"` block, which pins that piece to the named tier and bypasses this table entirely. That is how a hand authored reward (a quest chest, a boss drop) hands out a guaranteed tier &mdash; see [Droplist](droplist.html) for how the roll works and a worked pinning example.

## `fieldCraft`

Governs a craft made in hand, from the inventory screen with no bench, for the handful of recipes that ship both a `Workbench` and a `Fieldcraft` requirement.

| Field | Default | Description |
|---|---|---|
| `enabled` | `true` | While on, a hand craft is held at the base Grade (no roll, no XP, no tally, no ranking points) instead of being treated as an ordinary craft. |
| `warningIntervalHours` | `4` | How often the crafter is reminded, in chat, that the piece they just made in hand was not graded. |

## `notifications`

Server wide switches. A player has their own opt out for the first two, layered on top of these.

| Field | Default | Description |
|---|---|---|
| `xpEnabled` | `true` | Whether the per craft XP toast can fire for anyone. |
| `levelEnabled` | `true` | Whether the level up toast can fire for anyone. |
| `announceMasterwork` | `true` | The global chat broadcast whenever anyone crafts a Masterwork tier item. |
| `welcomeEnabled` | `true` | The one time welcome chat message a player gets on their first ever join, naming the plugin and how to open its page. Turning it off only stops it reaching players who have not seen it yet. |

## `ranking`

| Field | Default | Description |
|---|---|---|
| `visibility` | `public` | Who can see the ranking board: `public` (everyone), `admin_only`, or `none`. Points accumulate in every mode; `none` additionally skips building the in memory index at all. Switching back on later and running `/masterwork ranking rebuild` recovers the full history from the player files on disk. |

## `signature`

The colors of the crafter signature line shown on a stamped item's tooltip. Read at stamp time and baked onto that item instance, so recoloring the server default reaches gear already in the world only through the item resync sweep.

| Field | Default | Description |
|---|---|---|
| `lineColor` | `#B0A48C` | The "Crafted by \<name\>" line itself. |
| `hammerNoteColor` | `#FFEA00` | A crafting hammer's ownership note. |
| `hammerLuckColor` | `#AAAAAA` | The luck bonus line on a crafting hammer's tooltip. |

## `hammer`

The crafting hammer feature.

| Field | Default | Description |
|---|---|---|
| `enabled` | `true` | Gates the roll bonus a held hammer grants and the durability it costs. Nothing else in the hammer feature depends on this switch, so turning it off does not make hammers uncraftable. |
| `bonusOwnerOnly` | `true` | Whether the bonus (and the wear it costs) apply only for the hammer's own creator. Ignored while the hammers carry the active ladder's own luck stat (see [MMOSkillTree](progression/mmoskilltree.html)): that stat is declared on the item, so it reaches whoever holds one and nothing can read the owner stamp. The Settings tab grays the row out and says so; use `bindToOwner` to keep a hammer in its owner's hands. |
| `craftDurabilityDrain` | `1.0` | Exactly how much durability one craft costs the hammer in the crafter's hand. `0` means no wear; a negative value repairs the hammer instead, capped at its maximum. |
| `nonOwnerDamageMultiplier` | `0.0` | Fraction of a hit a non owner deals while wielding someone else's hammer in combat. |
| `nonOwnerDamageReflectedMultiplier` | `0.5` | Fraction of that hit reflected back onto the non owner. |
| `bindToOwner` | `true` | Whether a hammer left in a non owner's inventory is confiscated back to its owner's recovery stash. |
| `customUnlocks` | `false` | Whether the four hidden hammer recipes are gated by the `unlocks` block below instead of the shipped defaults. |

### `hammer.unlocks.<TIER>`

Consulted only while `customUnlocks` is `true`. One block per hammer tier (four in total), each with:

| Field | Description |
|---|---|
| `anchor` | The odds table row (a rank on the active progression ladder, not a raw level) the crafter must have reached. |
| `allTracks` | Whether that rank must be reached on all three gear tracks, or on any one of them. |
| `grade` | A Grade the crafter must already have forged at least once, or `NONE`. |

The shipped anchors (used while `customUnlocks` is `false`) are `2 / 3 / 5 / 7` under Masterwork's own progression, and `3 / 5 / 6 / 7` under a third party ladder, spread wider to give a longer ladder more room.

## `progression`

The source agnostic switch.

| Field | Default | Description |
|---|---|---|
| `enabled` | `true` | Whether any progression system biases the roll at all. Off, every craft uses `noProgressionOdds`. |
| `source` | `self` | Which system is active: `self` (Masterwork's own ladder), `none`, or a registered third party mod's id. **Applies on the next server restart**: the crafting hammers are minted around the active system at boot, so the running server keeps the one it started with, and every surface that reports the setting says which system is actually driving the roll meanwhile. |

## `progressionSources`

One entry per registered progression source, keyed by its id, kept in the file even for a source not registered this boot (so uninstalling a mod for a while does not discard its tuning).

| Field | Description |
|---|---|
| `levelThresholds` | Seven levels on that source's own ladder, one per odds table anchor, saying where each row lands. Identity (`[1, 2, 3, 4, 5, 6, 7]`) for a source whose ladder already is seven ranks. |
| `trackKeys` | For a source with per skill ladders, what it calls each of the three gear tracks. Only read while `customTrackKeys` is `true`. |
| `customTrackKeys` | Default `false`. Off, all three tracks read the source's own default skill. |
| `renameHammersByAnchor` | Default `true`. Whether the four hammers are renamed after the progression stage they unlock at, under a third party ladder. Set from the **Hammer** section of the Settings tab (not Progression, despite being stored per source); inert there while Built-In or `none` is the selected system. |
| `interpolateLevels` | Default `true`. Off, a level between two anchors rolls the lower anchor's row unchanged instead of blending toward the next one. |

## `builtInProgression`

Everything specific to Masterwork's own seven level ladder. Despite living in this block, `levelOdds` is read for whichever progression source is active, since every source's raw level is mapped onto these same seven anchor rows.

| Field | Default | Description |
|---|---|---|
| `itemLevelMultiplier` | `2.0` | XP formula coefficient on the crafted item's `ItemLevel`. |
| `timeSecondsMultiplier` | `10.0` | XP formula coefficient on the recipe's craft time. |
| `itemLevelExponent` | `2.2` | Exponent applied to `ItemLevel`. |
| `timeSecondsExponent` | `2.3` | Exponent applied to craft time. |
| `normalizer` | `10.0` | Divides the raw product down to a workable XP figure. |
| `xpBase` | `20756.25` | The level curve's base: XP needed to reach level 2. |
| `xpGrowth` | `2.0` | The level curve's growth factor between levels. |
| `levelOdds` | see below | A percentage table, one row per level 1 through 7, each row holding one percentage per Grade. |

The XP awarded by one craft is `((itemLevel * itemLevelMultiplier) ^ itemLevelExponent + (craftTimeSeconds * timeSecondsMultiplier) ^ timeSecondsExponent) / normalizer`, and the level a total buys follows a geometric curve seeded by `xpBase` and `xpGrowth`. `levelOdds` ships with its own tuned curve across the seven levels; open the generated `masterwork-config.json` to see or edit the full table, or use `/masterwork odds [level]` to read it in game.
