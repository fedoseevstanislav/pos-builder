# POS Telegram — State Contract

This skill writes into three places:

- top-level `mental_models_taught`
- top-level `pending_resume`
- `learner-state.json -> learner_profile` and `learner-state.json -> arch_blocks.telegram`

Write semantics:

- `write_mode` is the course permission boundary, not a raw Telegram API scope string.
- `app_credentials_path` and `session_path` store locations only, never secret values.
- `security_stub_acknowledged` means the learner saw the final checklist about `2FA` and active sessions. It is not a security lesson.

```json
{
  "mental_models_taught": {
    "adapter": { "at": "<ISO8601>", "by_skill": "<skill-name>" },
    "agent-as-assistant": { "at": "<ISO8601>", "by_skill": "<skill-name>" },
    "personal-vs-corporate": { "at": "<ISO8601>", "by_skill": "<skill-name>" },
    "scopes-risk": { "at": "<ISO8601>", "by_skill": "<skill-name>" },
    "inbox-as-flow": { "at": "<ISO8601>", "by_skill": "<skill-name>" },
    "transport-vs-memory": { "at": "<ISO8601>", "by_skill": "<skill-name>" },
    "whitelist-as-boundary": { "at": "<ISO8601>", "by_skill": "<skill-name>" },
    "chat-as-ontology": { "at": "<ISO8601>", "by_skill": "<skill-name>" }
  },
  "pending_resume": "pos-telegram-phase-1 | pos-telegram-phase-10 | null",
  "learner_profile": {
    "vibe_coded_tools_repo_path": "<path-or-null>",
    "vibe_coded_tools_built": ["pos-telegram"]
  },
  "arch_blocks": {
    "basic_vibecoding": {
      "status": "done | skipped | not_yet"
    },
    "telegram": {
      "status": "partial | needs_verification | done",
      "current_phase": 0,
      "last_completed_step": "0 | 1 | 2 | 3 | 4 | 5 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 | 16",
      "completed_at": "<ISO8601-or-null>",
      "surface_scope": "saved-messages | selected-dms | selected-groups | selected-channels | mixed-selected",
      "surface_scope_label": "<string-or-null>",
      "account_class": "personal | work",
      "work_warning_shown": false,
      "session_mode": "user-account-mtproto",
      "library_choice": "telethon | existing-telethon-project",
      "project_path": "<path-or-null>",
      "app_credentials_location": "keyring | env-file | tool-native",
      "app_credentials_path": "<path-or-keyring-service-or-null>",
      "session_location": "tool-native | dedicated-dir | keyring",
      "session_path": "<path-or-keyring-service-or-null>",
      "whitelist": [
        {
          "chat_ref": "<username-or-id>",
          "chat_label": "<human-name>",
          "chat_kind": "saved | dm | group | channel",
          "read_enabled": true,
          "write_enabled": false
        }
      ],
      "first_read_done": false,
      "voice_mode": "off | on",
      "stt_provider": "telegram-native | openai | groq | deepgram | other | null",
      "stt_provider_label": "<string-or-null>",
      "logs": {
        "jsonl_path": "<path-or-null>",
        "last_run": "<ISO8601-or-null>"
      },
      "kill_switch": {
        "kind": "env-flag | file-flag | alias | null",
        "location": "<path-or-name-or-null>",
        "tested": false
      },
      "write_mode": "read-only | send-only | send-and-forward",
      "rules_skill": {
        "skill_name": "telegram-rules",
        "source_path": "<path-or-null>",
        "claude_installed": false,
        "claude_path": "<path-or-null>",
        "codex_installed": false,
        "codex_path": "<path-or-null>"
      },
      "wow_moment": {
        "mode": "digest | save-to-vault | send-summary | forward-selected | null",
        "artifact_path": "<path-or-null>",
        "preview": "<string-or-null>"
      },
      "security_stub_acknowledged": false,
      "gaps": []
    }
  }
}
```
