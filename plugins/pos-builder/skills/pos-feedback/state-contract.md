# POS Feedback — State Contract

Save `learner-state.json` at phase transitions only. During a phase, keep working data in memory.

This skill writes only `arch_blocks.feedback`. Raw learner free-form dump is never persisted; only sanitized draft artifacts are stored.

```json
{
  "arch_blocks": {
    "feedback": {
      "schema_version": 1,
      "status": "in_progress | done",
      "current_phase": 1,
      "started_at": "<ISO8601>",
      "completed_at": "<ISO8601 | null>",
      "target_repo": "<owner/repo>",
      "draft": {
        "title": "<issue title>",
        "body_markdown": "<sanitized markdown body>",
        "contains_optional_excerpt": true
      },
      "last_issue_url": "https://github.com/<owner>/<repo>/issues/<N> | null"
    }
  }
}
```

Notes:

- `schema_version` is written on every transition write.
- `status: "done"` requires `last_issue_url` and `completed_at`.
- `draft.body_markdown` contains only sanitized text suitable for posting.
- Optional excerpt may appear only as `## Optional excerpt (approved and sanitized)` in `draft.body_markdown`.
- No raw transcript fragments, screenshots, tokens, local paths, private URLs, or direct personal identifiers may be persisted unless already sanitized in the final draft text.
