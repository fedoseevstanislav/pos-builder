# POS Vault — State Contract

Save `learner-state.json` at phase transitions only. During a phase, keep working data in memory.

The skill writes into top-level `mental_models_taught` and `pending_resume`, plus the `arch_blocks.obsidian_vault` namespace:

```json
{
  "mental_models_taught": {
    "data-ownership": { "at": "<ISO8601>", "by_skill": "pos-vault" },
    "md-is-world": { "at": "<ISO8601>", "by_skill": "pos-vault" },
    "write-or-not-exist": { "at": "<ISO8601>", "by_skill": "pos-vault" },
    "naming-first": { "at": "<ISO8601>", "by_skill": "pos-vault" },
    "ontology-3plus1": { "at": "<ISO8601>", "by_skill": "pos-vault" }
  },
  "arch_blocks": {
    "obsidian_vault": {
      "schema_version": 0.2,
      "status": "in_progress | partial | done | needs_verification",
      "current_phase": 1,
      "completed_at": "<ISO8601 when status=done; cleared to null when status flips back to in_progress on reopen>",
      "path": "/home/<user>/vault",
      "sync_method": "obsidian-sync | git | git_local_only | none",
      "sync_repo": "<github-url-if-git>",
      "naming_doc": "<vault-relative path to _meta/naming.md>",
      "ontology": "3+1-default | learner-variant | 3+1-migrated | preserved-learner-variant",
      "about_me_path": "<vault-relative path>",
      "imports": [
        {"source": "tg-saved", "scope": "...", "imported_at": "<ISO8601>"}
      ],
      "imports_pending_path": "<vault-relative path if deferrals exist>",
      "wow_moment_question": "<the question generated, for reproducibility>",
      "wow_moment_sample": ["<vault-relative path>", "..."],
      "pending_sync": "obsidian-sync | git | none | null",
      "gaps": ["<short note for any required gate that was skipped>"]
    }
  },
  "pending_resume": null
}
```

Schema note: v0.2 drops the old `goals_status` field.

Notes:

- Top-level `pending_resume` is set when another skill must take over and then resume back here.
- `pending_sync` is the in-flight sync choice carried between Phases 4 and 7. It is set at the end of Phase 4 and cleared once `sync_method` is finalized in Phase 7.
- `sync_method` values:
  - `"obsidian-sync"` — Obsidian's paid Sync service.
  - `"git"` — Git-backed sync, repo created and first push succeeded.
  - `"git_local_only"` — local Git history works, but the first commit or push failed.
  - `"none"` — local-only by explicit learner choice.
