# POS Tasks — State Contract

The locked frame defines only the `arch_blocks.tasks` branch below. Resume handoffs in this skill additionally use the top-level `pending_resume` string plus `pending_resume_tracker_slug` for both the per-tracker loop pause and the write-path pause.

```json
{
  "arch_blocks": {
    "tasks": {
      "status": "deferred_no_tracker | partial | done",
      "completed_at": "<ISO8601>",
      "no_tracker_reason": "<string>",
      "declared_trackers": [
        {
          "slug": "<stable ordered slug>",
          "label": "<learner-facing label>",
          "resolution_status": "pending | wired | deferred",
          "is_agent_memory_home": false
        }
      ],
      "connected": [
        {
          "tracker_slug": "<declared_trackers[].slug>",
          "provider": "linear | notion | github-issues | jira | todoist | asana | clickup | trello | ticktick | things | monday | motion | obsidian-tasks | markdown-files | other",
          "provider_label": "<raw label for provider == 'other'>",
          "connection_method": "mcp | cli | vibe-coded-adapter | native-gh",
          "mcp_server_name": "<string or null>",
          "cli_tool": "<string or null>",
          "adapter_skill_name": "<slug if vibe-coded, else null>",
          "tracker_class": "personal | work | shared",
          "projects_connected": [
            {"id": "...", "label": "...", "role": "primary | secondary"}
          ],
          "scopes_granted": ["read", "create", "edit", "comment", "link", "label", "close", "archive"],
          "hard_delete_enabled": false,
          "credentials_location": "keyring | env-file | tool-native",
          "credentials_path": "<path or keyring service name>",
          "infosec_consent_granted": false,
          "attribution_prefix": "<string, e.g. '[agent]'>",
          "attribution_style": "issue-title-prefix | comment-sig | label-suffix | other",
          "backup": {
            "enabled": false,
            "status": "enabled | none_accepted | null",
            "reason": "<string-or-null>",
            "format": "json | markdown",
            "location": "<path>",
            "cadence": "daily | on-demand",
            "last_run": "<ISO8601>"
          },
          "is_agent_memory_home": false
        }
      ],
      "rules_status": "appended | deferred_read_only",
      "rules_appended_to": {
        "claude_md_path": "<path-or-null>",
        "agents_md_path": "<path-or-null>",
        "section_name": "## Tasks"
      },
      "wow_moment_today_view_shown": false,
      "rules_of_use_skipped": false,
      "my_architecture_skipped": false,
      "gaps": [
        "<string gap note>",
        {"slug": "<tracker_slug>", "reason": "<learner raw phrase>", "deferred_at_phase": "<N>"}
      ]
    }
  }
}
```

Notes:

- Exactly one entry in `declared_trackers` and (once wired) its corresponding `connected` entry has `is_agent_memory_home: true`.
- `declared_trackers` keeps the learner-confirmed order for loop resume. `resolution_status` stays `pending` until the tracker is either wired or explicitly deferred.
- Every `connected[]` entry is keyed by `tracker_slug`, which must equal the owning `declared_trackers[].slug`.
- `backup.status` stays unset until Phase 3 (Step 3.2). After that it is `enabled` or `none_accepted` for every write-enabled tracker; `backup.enabled = false` alone does not satisfy G10.
- `rules_status` is unset until Phase 3 (Step 3.4). After that it is either `appended` or `deferred_read_only`.
- `rules_appended_to.claude_md_path` and `rules_appended_to.agents_md_path` are populated only for the files actually written or explicitly accepted as valid. The primary target depends on `learner_profile.primary_agent`; the sibling path is populated only when sync is enabled.
- `scopes_granted` is the current course permission boundary for that tracker.
- `hard_delete_enabled` flips only via explicit rules-of-use opt-in.
- `infosec_consent_granted: true` is required before any connection attempt on a `work` tracker; stays `false` on personal and shared trackers.
- `attribution_prefix` and `attribution_style` are populated at G15 before the tracker's first agent-driven write.
- `pending_resume_tracker_slug`, when present, always stores a `declared_trackers[].slug`; Resume Logic must use it to look up the same tracker's `connected[]` record directly.
