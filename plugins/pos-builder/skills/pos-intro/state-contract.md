# POS Intro — State Contract

Save `learner-state.json` at phase transitions only. During a phase, keep working data in memory.

The skill writes top-level `mental_models_taught`, plus the `arch_blocks.intro` namespace:

```json
{
  "mental_models_taught": {
    "build-without-traditional-coding": {
      "at": "<ISO8601>",
      "by_skill": "pos-intro"
    },
    "conversation-is-build-surface": {
      "at": "<ISO8601>",
      "by_skill": "pos-intro"
    },
    "understanding-through-action": {
      "at": "<ISO8601>",
      "by_skill": "pos-intro"
    }
  },
  "arch_blocks": {
    "intro": {
      "schema_version": 1,
      "status": "in_progress | done",
      "current_phase": 1,
      "started_at": "<ISO8601>",
      "completed_at": "<ISO8601 | null>",
      "last_exit": "diagnostic | not_now | recap | null"
    }
  }
}
```

Notes:

- `mental_models_taught` is merge-only. If a slug is already present, do not overwrite it.
- First-teach happens in the body when a slug is absent. Persist that slug in the next phase-transition save after the teach. Reminder branches do not write state.
- `status: "in_progress"` is the fresh-entry and replay state.
- Resume is phase-level only. `current_phase` stores the numeric phase number, not a per-step checkpoint. Allowed values in this skill are `1 | 2 | 3 | 4`. If it is missing or invalid, restart from Phase 1.
- The final `status: "done"` write updates the existing `arch_blocks.intro` object in place and preserves its original `started_at`.
- `last_exit` records only the most recent exit shape from this skill. It is not a handoff token.
- For intro branching, top-level `complete == true` is the supported signal that `/pos-diagnostic` is already done.
- This skill does **not** use top-level `pending_resume`.
