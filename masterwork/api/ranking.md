---
title: Ranking
mod: masterwork
---

<div class="eyebrow">Masterwork &rsaquo; Public API &rsaquo; Ranking</div>

# Ranking
{: .no_toc }

`MasterworkRanking` is the ranking board, read only plus its one recovery operation. Obtained from `MasterworkApi.ranking()`.

1. TOC
{:toc}

## What the board ranks

The board ranks masterworks, not crafting volume: a craft scores only when it rolls the top tier, and then by how hard the piece was (`itemLevel * craftTimeSeconds`). It holds a row for every player the server has ever seen, written on every join, so the board is populated before anyone has scored; the tail of a young board is simply the server's oldest members sitting on zero, in the order they first arrived.

## Reading

| Method | Returns |
|---|---|
| `top(limit, track)` | The top `limit` rows on `track`, best first, or on the overall board for a `null` track. Rows with no points at all are included. |
| `of(uuid)` | One player's row, including an offline one, or `null` if this server has never seen them. |
| `rankOf(uuid, track)` | A player's position on `track`, `1` being the top, `0` if they are not on the board at all. |
| `size()` | How many players the board holds. |
| `rows()` | Every row, unordered, for a caller doing its own sorting or export. |
| `visibility()` | Who the server owner made the board visible to. Reported, not enforced: it governs what Masterwork's own page shows, but this class answers every query regardless, and whether a third party feature honors it is that feature's own call. |

Every method here is safe to call from any thread: the index lives in memory and is written by the craft path under its own synchronization.

## `RankingRow`

| Field | Meaning |
|---|---|
| `uuid` / `name` | The player, name as of their last join (corrected on every join, so a rename shows up without a rebuild). |
| `weaponPoints` / `armorPoints` / `toolPoints` | Points scored on each track. Shields fold into `weaponPoints`. |
| `scoredAt` | Epoch millis of when this row reached the totals it holds, or the player's first join while they have none. The board's tie break: on equal points, the earlier stamp ranks higher, so players on zero sit in arrival order. |

`totalPoints()` sums all three; `points(track)` reads one, or the total for `null`. `hasScore()` reports whether the player has scored at all. Absolute point values mean nothing on their own, since the scoring formula is a raw product with no normalizer: they exist only to be compared against each other.

## Recovery

`rebuild()` re-derives the whole board from the player files on disk, off the world thread, and completes a `CompletableFuture<RankingRebuildResult>` with what it found. This is a recovery operation, not a refresh: the board is already updated on every scoring craft and on every join, so a rebuild exists for a lost or stale index file, most often after the board was switched on months into a server's life (there is no backfill; points accumulate in every visibility mode, but nothing is indexed while the board itself is off).

```java
RankingRebuildResult result = api.ranking().rebuild().join();
```

| `RankingRebuildResult` field | Meaning |
|---|---|
| `scanned` | How many player files were read. |
| `scored` | How many of them held any ranking points. Can be far smaller than `scanned` on a server whose board was switched on late. |
| `rows` | How many rows the board holds afterward, not the same as `scored`, since the index also keeps a row for every player seen, scored or not. |

It loads players in batches, merges each track by maximum (points only ever climb, so a craft landing mid scan cannot be lost), and applies the result in one go, so a scan that fails part way changes nothing. `rebuildRunning()` reports whether one is in progress; only one runs at a time, and a call made while one is already running returns the same failed future rather than starting a second scan.

## `RankingVisibility`

| Value | Meaning |
|---|---|
| `PUBLIC` | Everyone sees the board. |
| `ADMIN_ONLY` | Only holders of `masterwork.admin` see it. |
| `NONE` | Nobody sees it, and the in memory index is not even built. Points keep accumulating on each crafter's own data regardless. |

`isEnabled()` reports whether the board exists at all in a given mode. `isVisibleTo(admin)` answers whether a viewer holding admin should see it under that mode.
