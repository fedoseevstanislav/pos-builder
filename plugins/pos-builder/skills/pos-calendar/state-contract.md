# POS Calendar — State Contract

This skill writes the `arch_blocks.calendar` branch in `learner-state.json` and reuses top-level mental-model slugs.

Semantics:

- `scopes_granted` is the course permission boundary the learner accepted, not the raw OAuth scope returned by the provider.
- Concrete scope strings come from the chosen tool's documentation or live output.

```json
{
  "mental_models_taught": {
    "adapter": { "at": "<ISO8601>", "by_skill": "<skill-name>" },
    "agent-as-assistant": { "...": "..." },
    "personal-vs-corporate": { "...": "..." },
    "scopes-risk": { "...": "..." },
    "time-awareness": { "...": "..." },
    "voice-agent-over-ui": { "...": "..." }
  },
  "arch_blocks": {
    "calendar": {
      "status": "partial | needs_verification | done",
      "completed_at": "<ISO8601>",
      "provider": "google | outlook | icloud | yandex | other",
      "provider_label": "<сырая метка для provider == 'other', например 'Яндекс Календарь' или 'Outlook.com'; null для стандартных провайдеров>",
      "pending_providers": ["<provider ids the learner named but aren't being connected in this run; empty array if single-provider>"],
      "connection_method": "mcp | gogcli | other-cli",
      "mcp_server_name": "<имя MCP-сервера, подобранное в Phase 6; null если не MCP>",
      "cli_tool": "<имя CLI, подобранное в Phase 6; null если не CLI>",
      "calendars_connected": [
        {"id": "...", "name": "...", "role": "primary | secondary"}
      ],
      "scopes_granted": ["read", "write", "delete"],
      "credentials_location": "keyring | env-file | tool-native",
      "credentials_path": "<path-or-keyring-service-name>",
      "is_work_calendar": false,
      "infosec_warning_shown": false,
      "backup": {
        "enabled": false,
        "format": "json",
        "location": "<path>",
        "run_host": "vps | laptop",
        "cadence": "daily",
        "last_run": "<ISO8601>"
      },
      "rules_appended_to": {
        "claude_md_path": "<path-or-null>",
        "agents_md_path": "<path-or-null>",
        "section_name": "## Calendar"
      },
      "wow_moment_question": "<the question the learner asked>",
      "first_event_created": "<event-id-or-null>",
      "rules_of_use_skipped": false,
      "my_architecture_skipped": false,
      "write_rolled_back": false
    }
  }
}
```

Notes:

- `connection_method` values: `"mcp"` for an MCP server, `"gogcli"` for gogcli, and `"other-cli"` for any other CLI.
- `rules_appended_to.claude_md_path` and `rules_appended_to.agents_md_path` are populated only for the files actually written or explicitly accepted as valid. The primary target depends on `learner_profile.primary_agent`; the sibling path is populated only when sync is enabled.
- `rules_of_use_skipped` and `my_architecture_skipped` become `true` only if the learner declines that step. If either is `true`, Phase 14 closes the block as `"partial"`.
- `write_rolled_back` becomes `true` only if the learner declines both the rules write and the minimal fallback after the write path was opened.
