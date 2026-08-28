---
title: Progression Sources
mod: masterwork
---

<div class="eyebrow">Masterwork &rsaquo; Public API &rsaquo; Progression Sources</div>

# Progression Sources
{: .no_toc }

`ProgressionSource` is how a mod offers its own leveling system as one a server can point Masterwork's grade roll at, instead of Masterwork's built in seven level ladder. Registered through `MasterworkApi.registerProgressionSource(source)`.

> Looking to *use* progression on a server rather than implement a new source? See [Progression]({{ 'masterwork/progression/overview.html' | relative_url }}) for the Built-In ladder and the shipped MMOSkillTree integration.

1. TOC
{:toc}

## The shape

The built in source derives a level from Masterwork's own per craft XP. A third party source instead reads an external mod's ladder, MMOSkillTree being the shipped reference implementation for that shape.

```java
public final class YourLevelSource implements ProgressionSource {

    @Override
    public String id() {
        return "your-mod";
    }

    @Override
    public int level(Store<EntityStore> store, Ref<EntityStore> player, GearCategory category) {
        // read the crafter's level in your own mod, for this gear category
    }
}
```

| Method | Contract |
|---|---|
| `id()` | A stable id used to select this source in config. It keys this source's own entry in the `progressionSources` config block, so it must not change once a server has tuned against it. |
| `available()` | Whether this source is usable right now. Always `true` for the built in source; a third party source returns `false` once its own mod stops answering, so Masterwork falls back to the built in source. |
| `bind()` | Called once, at plugin setup: probe for the mod this source reads, and report whether registering is worth it at all. `false` means do not register (the mod is absent), which is the normal case for an uninstalled mod, never an error. |
| `level(store, player, category)` | The crafter's raw progression level for this craft, on this source's own ladder, `1` through `maxLevel()`. Raw, not normalized: a 200 level mod returns 143 directly, and the server (not the source) decides where the seven odds table anchors sit on that scale. |
| `hasLuckAttribute()` | Whether this source has a luck stat at all. `false` by default. |
| `luck(store, player, category)` | The crafter's luck as a fraction (`0.11` = 11%). This one is on the craft path: it must not throw and must not be expensive. |
| `progress(store, player, category)` | Display only: how far into the current level the crafter is, as a `LadderProgress`, or `null` when this source cannot say. Never called on the craft path. |
| `ladderName(category)` | What this source calls the ladder `category` reads from, or `null` when the ladder simply is the track (the built in source's case). |
| `ladderLabelKey(ladder)` | The message id that names `ladder` in the reader's own language, or `null` to print the name as it stands. |
| `ladderChoices()` | Every ladder name a server may point a track at, so the settings page can offer a picker instead of a free text field. |
| `maxLevel()` / `defaultThresholds()` / `defaultTrackKeys()` | How this source's ladder is shaped, and what its `progressionSources` config entry is seeded with the first time it is registered. |

## A source must never throw on the craft path

`level()` and `luck()` run inline in the roll. An uncaught exception there would propagate straight through the odds pipeline into the craft tick, and the player would lose the item they just made. A third party source talking to another mod must catch its own failures, report `available() == false` from then on, and let the resolver fall back to the built in source: a broken integration should cost a warning, never a craft.

Masterwork wraps every registered source at the resolver as an additional safety net, so a source that still throws despite this contract falls back to the built in source's level for that crafter (never to `1`, which would read as a tuning problem rather than a broken integration) and disables itself for the rest of the session after one throw.

## `LadderProgress`

The optional, display only reading `progress()` returns.

| Field | Meaning |
|---|---|
| `level` | The crafter's level on this ladder, the same number `level()` reports. |
| `fraction` | How far into that level, `0` to `1`, clamped on construction. |
| `xp` | The crafter's total XP on this ladder, on the source's own scale (never normalized to Masterwork's). |
| `nextXp` | The XP at which the next level is reached, or `0` at the top of the ladder. |

`capped()` reports whether `nextXp <= 0`, meaning there is no next level to progress toward.

## Registering, and when it closes

```java
MasterworkApi.get().registerProgressionSource(new YourLevelSource());
```

Register during the depending plugin's own `setup()`. Like gear registration, this closes at the same boot injection point: a source registered after that would work, but would appear in the settings page and in `progression system` half a boot after the server already decided what it was running, which is worse than never appearing at all. `registerProgressionSource` returns `false` when either registration has closed, or the source's own `bind()` declined, which is the normal answer for an uninstalled mod and not an error. A duplicate id is refused as well, and both refusals are logged with the offending mod's name.

A source is *offered* by registering, never *selected* by it: the server owner picks the active system explicitly, through `/masterwork progression system set <id>` or the settings page. Registering only puts a source on that list. Its config entry, thresholds, track names, luck switch, is seeded on the way in and kept even on a boot where the mod is absent, since that tuning belongs to the server rather than to whether the jar happened to load today.
