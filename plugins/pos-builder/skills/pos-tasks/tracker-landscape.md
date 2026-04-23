# POS Tasks — Tracker Landscape

> Agent-internal only. Never show tiers to the learner. Use this file as a
> routing and research hint map, not as truth. Before suggesting any concrete
> step, research the provider's current state at runtime and show receipts.

## Principles

- Tiers are hints for routing, not promises about what works today.
- Do not hardcode install commands, package names, CLI syntax, MCP server names, or auth steps into learner-facing text.
- Prefer the smallest trustworthy path:
  - ready bridge first,
  - terminal tool second,
  - vibe-coded adapter only when ready paths are missing or clearly worse.
- Re-check account class after provider confirmation. Jira, Monday, Linear, and Notion often live in work or shared contexts even when the diagnostic wording looked personal.
- If write scope is requested, confirm four things before recommending a path:
  - permission model,
  - export or snapshot path,
  - smallest marker-carrying first write,
  - rollback path if the learner pauses before first write.

## Tier Hints

### Tier 1 — likely ready or native

- `github-issues`
  - Preferred shape: `native-gh` when GitHub setup already exists.
  - Default agent-memory pitch.
  - Marker-friendly first write: comment, child issue, label.
- `linear`
  - Usually ready bridge or stable API surface.
  - Project opt-in required.
  - Often `work` or `shared`.
- `notion`
  - Usually ready bridge or API surface.
  - Workspace / database opt-in required.
  - Often `shared`.
- `jira`
  - Usually API or bridge path exists.
  - Project opt-in required.
  - Often `work`.
- `todoist`
  - Usually ready bridge or API path exists.
- `asana`
  - Usually ready bridge or API path exists.
- `clickup`
  - Usually ready bridge or API path exists.
- `trello`
  - Usually ready bridge or API path exists.
- `ticktick`
  - Often bridge or API path exists, but verify current stability.
- `monday`
  - Often ready API path exists.
  - Often `work`.
- `motion`
  - Often ready API path exists.
  - Often `work` or `shared`.

### Tier 2 — conditional or local

- `things`
  - Often local / OS-specific.
  - Export may be weak or manual, so `"none"` can be a legitimate G10 result with a reason.
- `obsidian-tasks`
  - File-based path inside the vault.
  - Credentials rule still applies to any helper tool; task files themselves are not credentials.
  - Marker may need to be a new comment line or new task line, not an edited existing line.
- `markdown-files`
  - File-based path.
  - Snapshot is often the files themselves; `"none"` may still be the right recorded gap if there is no separate export.

### Tier 3 — likely vibe-coded

- `other`
- Any provider where runtime research finds no trustworthy ready bridge or terminal path.
- Any niche local tracker where the safe path is a small dedicated adapter under the learner's repo.

## Routing Heuristics

- If agent-memory home is GitHub Issues and `arch_blocks.github_setup.status != "done"`:
  - no tier-3 need detected → hand off to `/pos-github-setup`
  - any tier-3 need detected → hand off to `/pos-basic-vibecoding`
- If any confirmed tracker resolves to tier 3 and `arch_blocks.basic_vibecoding.status != "done"`:
  - hand off to `/pos-basic-vibecoding` before wiring continues, even if agent-memory home is not GitHub.
- If provider confirmation inside the loop changes the provisional tier:
  - stop,
  - set resume state,
  - hand off,
  - resume the same tracker after return.

## Research Receipts Before Recommendation

- Confirm the provider's current docs or tool help.
- Confirm whether the learner must pick projects, spaces, databases, boards, or repos explicitly.
- Confirm which permissions are requested and how to show them before consent.
- Confirm where credentials will live and whether that location is outside the Obsidian vault.
- Confirm whether a provider-specific export or snapshot exists for G10.
- Confirm the smallest real first write that can visibly carry the chosen attribution marker.
