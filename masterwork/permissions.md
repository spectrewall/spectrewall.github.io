---
title: Permissions
mod: masterwork
---

<div class="eyebrow">Masterwork &rsaquo; Permissions</div>

# Permissions
{: .no_toc }

Most of the `/masterwork` tree is gated by `masterwork.admin`, but four nodes exist so a server owner can hand out a narrower slice of the mod without making someone a full admin.

1. TOC
{:toc}

## Nodes

| Node | Grants |
|---|---|
| `masterwork.admin` | Every command in the tree, plus the Settings tab of the UI page with its controls live. Server operators hold this implicitly. |
| `masterwork.config.view` | Read only access: the settings page rendered as read only text, and the read only command listings (`grade list`, `blacklist list`, `progression list`, and the state readout printed by a bare group command). |
| `masterwork.item.grade` | `/masterwork item set grade`, swapping the item in the caller's active hotbar slot to a chosen tier. |
| `masterwork.item.crafter` | `/masterwork item set crafter` and `/masterwork item set crafterByUuid`, signing an unsigned held item with a crafter's name. |

`masterwork.admin` is a strict superset of the other three: whatever a narrower node opens, the admin node opens too. A server granting `masterwork.*` gets everything, present and future.

## Gating rules

Two gating rules worth knowing if a subcommand looks like it should be locked down and is not:

- **A command is admin gated only if it writes to server configuration.** A command that only reads it (a `list` leaf, or the state a bare group prints) also accepts `masterwork.config.view`.
- **A group command that holds an open leaf is never itself gated.** The client only offers tab completion for a subtree up to the first node a player lacks, so gating the group would hide its open leaves from autocomplete. `/masterwork`, `progression` and `hammer` are open groups for this reason; their bare reply differs by whether the caller can administer or merely view the configuration.

A command that only reports the caller's own state, or acts on their own behalf, is open to any player regardless of node: `/masterwork odds`, `progression current`, `progression notify`, and `hammer recover`.

## Granting a node in game

The `/masterwork` UI page's Settings tab has its own Permissions section for an operator (`*`), letting them search for an online or offline player and grant or revoke one of the four nodes above without leaving the page. See [Commands]({{ 'masterwork/commands/overview.html' | relative_url }}) for the full command tree these nodes gate.
