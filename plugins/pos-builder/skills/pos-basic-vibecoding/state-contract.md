# POS Basic Vibecoding — State Contract

```json
{
  "arch_blocks": {
    "basic_vibecoding": {
      "status": "done | skipped | not_yet",
      "completed_at": "<ISO8601 when status == done>",
      "skipped_at": "<ISO8601 when status == skipped>",
      "skipped_reason": "<string>",

      "practice_issue_url": "<URL of the first real issue opened, or null>",
      "artifact_repo": {
        "name": "<artifact repo name or null>",
        "local_path": "<absolute local path of the artifact repo or null>",
        "github_url": "<GitHub URL of the artifact repo or null>",
        "initialized_at": "<ISO8601 when the scaffold repo was initialized, or null>"
      },

      "superpowers_pack_status": "installed | opted_out | null",
      "skill_creator_installed": false,

      "learner_skill_name": "<slug of the GitHub-issues skill the learner vibe-coded>",
      "learner_skill_path": "<~/.claude/skills/<slug>/ or ~/.agents/skills/<slug>/>",

      "agent_config_rules_appended": false
    }
  }
}
```

Notes:

- `practice_issue_url` always points at the build issue with the spec body.
- `artifact_repo` is separate from the learner's existing POS repo from `arch_blocks.github_setup.repo_url`.
- If `artifact_repo` is absent on a pre-retrofit state, treat it as `{ name: null, local_path: null, github_url: null, initialized_at: null }`.
- `superpowers_pack_status: opted_out` is meaningful to downstream skills.
- `learner_skill_name` can exist before `learner_skill_path`; write the path only after the built skill is verified on disk in the active agent registry.
