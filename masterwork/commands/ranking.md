---
title: Ranking
mod: masterwork
---

<div class="eyebrow">Masterwork &rsaquo; Commands &rsaquo; Ranking</div>

# Ranking
{: .no_toc }

Ranking board visibility and recovery. The whole group is admin gated: reading the board is the UI page's Ranking tab, not a command.

1. TOC
{:toc}

### `/mw ranking rebuild`

<p class="cmd-access">masterwork.admin</p>

Re derive the whole board from every player file on disk, asynchronously. The command replies once when the scan starts and again with the final counts, since the scan can take a while on a large server.

### `/mw ranking visibility <public|admin_only|none>`

<p class="cmd-access">masterwork.admin</p>

Who can see the ranking board: everyone, admins only, or nobody (which also stops the in memory index from being built at all). Points keep accumulating in every mode; nothing is ever lost by switching visibility off.
