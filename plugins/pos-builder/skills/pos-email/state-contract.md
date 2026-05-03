# POS Email — State Contract

This skill writes in two zones:

- top-level `mental_models_taught`, `security_primer_taught`, `pending_resume`
- `arch_blocks.email`, including `pending_email_accounts`

Semantics:

- `scopes_granted` is the course permission boundary the learner accepted, not raw provider scopes.
- `security_primer_taught` belongs to the future `/pos-security` block; until that exists, this skill uses the stop-gap warning in Phase 8.
- `pending_email_accounts` stores named mailboxes or addresses that were deferred in this run.
- `gaps` stores conscious deferrals such as declining backup before the write path.

```json
{
  "mental_models_taught": {
    "adapter": { "at": "<ISO8601>", "by_skill": "<skill-name>" },
    "agent-as-assistant": { "...": "..." },
    "personal-vs-corporate": { "...": "..." },
    "scopes-risk": { "...": "..." },
    "inbox-as-flow": { "...": "..." },
    "sender-trust": { "...": "..." }
  },
  "security_primer_taught": false,
  "pending_resume": "pos-email-phase-8 | null",
  "arch_blocks": {
    "email": {
      "status": "partial | needs_verification | done",
      "completed_at": "<ISO8601>",
      "provider": "gmail | outlook | icloud | yandex | mailru | proton | imap-other | other",
      "provider_label": "<raw label for provider == 'other'>",
      "provider_setup_complete": "true | false",
      "connection_method": "mcp | gogcli | imap-cli | other-cli",
      "mcp_server_name": "<string or null>",
      "cli_tool": "<string or null>",
      "mailboxes_connected": [
        {"address": "...", "mailbox_class": "personal | work | shared", "role": "primary | secondary"}
      ],
      "pending_email_accounts": ["<address or label named by learner but not connected in this run; empty array if single-account>"],
      "first_read_done": false,
      "scopes_granted": ["read", "reply", "archive", "label", "delete"] | [],
      "credentials_location": "keyring | env-file | tool-native",
      "credentials_path": "<path or keyring service name>",
      "is_work_email": false,
      "infosec_warning_shown": false,
      "backup": {
        "enabled": false,
        "format": "json | mbox",
        "location": "<path>",
        "run_host": "vps | laptop",
        "cadence": "daily",
        "last_run": "<ISO8601>"
      },
      "rules_appended_to": {
        "claude_md_path": "<path-or-null>",
        "agents_md_path": "<path-or-null>",
        "section_name": "## Email | ## Почта | ### Email rules (updated YYYY-MM-DD) | null"
      },
      "wow_moment_question": "<string or null>",
      "first_reply_drafted": "<provider-message-id or null>",
      "send_handoff_confirmed": "true | false | null",
      "rules_of_use_skipped": false,
      "minimal_fallback_used": false,
      "my_architecture_skipped": false,
      "write_rolled_back": false,
      "gaps": []
    }
  }
}
```

Optional archived branch:

When the learner restarts from scratch (Step 0.2, option 2), the current `arch_blocks.email` is renamed to `arch_blocks.email_archived_<ISO8601>`. This key is not schema-enforced and is never read back by the skill — it exists only as a recoverable snapshot.

Notes:

- `first_read_done` stays `false` until Phase 8 succeeds. In rollback, `scopes_granted` may become `[]`.
- `rules_appended_to.claude_md_path` and `rules_appended_to.agents_md_path` are populated only for the files actually written or explicitly accepted as valid. The primary target depends on `learner_profile.primary_agent`; the sibling path is populated only when sync is enabled.
- `connection_method` values: `"mcp"`, `"gogcli"`, `"imap-cli"`, `"other-cli"`.
- `rules_of_use_skipped` and `my_architecture_skipped` become `true` only when the learner declines the corresponding step. `minimal_fallback_used: true` means a short safe block was written instead of the full rules block.
- `write_rolled_back` becomes `true` only if the learner declines both the full rules and the minimal fallback after write scope was opened.
