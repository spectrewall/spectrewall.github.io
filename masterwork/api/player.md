---
title: Players
mod: masterwork
---

<div class="eyebrow">Masterwork &rsaquo; Public API &rsaquo; Players</div>

# Players
{: .no_toc }

`MasterworkPlayer` is everything Masterwork knows about one online player, in one handle: where they stand on whichever progression ladder is biasing their rolls, what they have forged, what they have scored, and the handful of preferences that are theirs to set. Obtained from `MasterworkApi.player(playerRef)`.

1. TOC
{:toc}

```java
MasterworkPlayer crafter = MasterworkApi.get().player(playerRef);

int level = crafter.level(GearCategory.WEAPON);
int masterworks = crafter.tally().count(Grade.MASTERWORK);
```

## Threading

Every method reads or writes the player's live entity data, so call them on that player's own world thread. Reads are safe at any point on it, including from inside a system's tick; a player with nothing stored yet simply reads back as zeroes.

Writes are structural: creating the storage block the first time attaches a component, which the engine's store forbids while it is mid-processing a system. Call a setter from outside a tick, from a command, from a page, or from inside a `CommandBuffer.run(...)` deferral, which is what Masterwork's own craft path does internally. A setter called mid-tick on a player who has nothing stored yet throws `IllegalStateException: Store is currently processing`.

## Offline players

This handle needs a live `PlayerRef`, so it covers online players only. The one thing readable about an offline player is their ranking board standing, which `MasterworkRanking` keeps for everyone the server has ever seen (see [Ranking](ranking.html)).

## Progression

| Method | Returns |
|---|---|
| `progressionSourceId()` | The id of the source biasing this player's rolls: `self`, `none`, or a third party mod's id. |
| `level(category)` | The crafter's raw level on the active source's own ladder for `category`, `1` through that source's `maxLevel()`. This is the number that actually decides grade odds, whichever mod it comes from. |
| `progress(category)` | How far into that level the crafter is, as a `LadderProgress`, or `null` when the active source cannot say. Display grade only; never gate on it. |
| `ladderName(category)` | What the active source calls the ladder `category` reads from, or `null` when the ladder simply is the track. |
| `luck(category)` | The crafter's own luck stat on that ladder, as a fraction, `0` when the source has none or it is switched off. Does not include a held crafting hammer. |
| `effectiveLuck(category)` | `luck(category)` plus the luck of the crafting hammer currently in the crafter's hand, while the active ladder spends a hammer's bonus as luck. This is the whole figure biasing the crafter's next roll. |

## Masterwork's own ladder

| Method | Returns |
|---|---|
| `nativeXp(category)` | The crafter's accumulated XP on Masterwork's own ladder for `category`'s track. Frozen while another mod's system drives progression. |
| `nativeLevel(category)` | The level that XP buys on Masterwork's seven rung curve. Matches `level(category)` while the built in source is active, and is a stale reading otherwise. |

## Craft tally

| Method | Returns |
|---|---|
| `tally()` | A `CraftTally` snapshot: every piece this crafter has forged, by track and tier. |
| `craftCount(track, grade)` | How many `grade` pieces they forged on `track`. |
| `craftCount(track)` | How many pieces they forged on `track`, at any tier. |
| `totalCrafts()` | Every piece they have forged. |

The tally counts graded crafts only: a piece Masterwork left alone (blacklisted, excluded, a hammer) and a piece made in hand are tallied nowhere. It is never reset and never gated by which progression system is active, since it is a record of forging rather than of crafting. Keyed by progression track, not by an item's own category, so a shield's crafts show up under `WEAPON`, exactly as every other reader sees them.

## Ranking

| Method | Returns |
|---|---|
| `points(track)` | Ranking points on `track`, or across all three for a `null` track. Scored only by a `MASTERWORK` craft. |
| `scoredAt()` | Epoch millis of when the crafter reached the totals they currently hold; the board's tie break, and their first join while they have none. |

## Preferences

| Method | Effect |
|---|---|
| `notifyXp()` / `setNotifyXp(bool)` | Whether the crafter wants the per craft XP toast. Theirs to choose; the setting survives the server switching that toast off entirely, so it comes back exactly as they left it if the server turns it back on. |
| `notifyLevel()` / `setNotifyLevel(bool)` | The same, for the level up toast. |
| `signatureColorId()` / `setSignatureColorId(id)` | The palette id of the color their crafter signature is stamped in, `null` for the server default. Baked onto an item at stamp time, so a change reaches gear they have already forged only when the item resync sweep next reaches it. |
| `welcomed()` | Whether they have already seen the one time welcome message. |

## Hammers

| Method | Returns |
|---|---|
| `hasBrokenHammer(hammerItemId)` | Whether they have worn that hammer down to nothing at least once. |
| `stashedHammers()` | An immutable snapshot of the hammers waiting in their recovery stash, confiscated copies of hammers they forged, held for them to reclaim. |

## Attributed writes

Two writes exist for a mod that wants to reward something Masterwork itself did not craft, a quest, a lesson, a discovery. Both are attributed and logged, at a level the `debug` switch cannot silence, since the server owner is owed the name of whoever moved a crafter's standing or the public ranking board.

| Method | Effect |
|---|---|
| `grantNativeXp(source, track, xp)` | Award XP on Masterwork's own ladder. A no-op, logged as a warning, while the built in source is not the active one: nobody is earning that XP right now, and topping up a frozen total would surprise the server owner the day they switch back. It deliberately tallies nothing, since a grant did not forge a piece. |
| `awardRankingPoints(source, track, points)` | Award points on the ranking board, on `track`. The live board index is updated alongside the stored total, so the row on screen and the player's own file cannot drift. |

There is no way to write the craft tally at all: it is a factual record of pieces forged, and a mod that could fabricate rows would make the crafting statistics a claim nobody could trust.

## `CraftTally`

An immutable snapshot handed out by `tally()`.

| Method | Returns |
|---|---|
| `count(track, grade)` | How many `grade` pieces the crafter forged on `track`. |
| `count(track)` | How many pieces on `track`, any tier; `null` for the grand total across every track. |
| `count(grade)` | How many `grade` pieces across every track. |
| `total()` | Every piece the crafter has forged. |
| `row(track)` | The whole tier to count map for one track, immutable, holding only the tiers with a non-zero count. |
