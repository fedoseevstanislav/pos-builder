# POS Basic Vibecoding — State Contract

```json
{
  "arch_blocks": {
    "basic_vibecoding": {
      "status": "done | skipped | not_yet",
      "completed_at": "<ISO8601 when status == done>",
      "skipped_at": "<ISO8601 when status == skipped>",
      "skipped_reason": "<string>",
      "last_completed_step": "<number, 1-6>",
      "experience_path": "novice | some | experienced",

      "superpowers_pack_status": "installed | opted_out | null",
      "build_artifact_description": "<free-text summary of what was built, or null>",
      "agent_config_rules_appended": false
    }
  }
}
```

Notes:

- `superpowers_pack_status: opted_out` is meaningful to downstream skills.
- `build_artifact_description` is a free-text summary written after the supervised build (Step 4). Content varies — could describe a skill, a script, a config, or any other artifact.
- Two valid skip reasons: `"experienced -- block skipped"` (Step 2) and `"opted out of Obra Superpowers"` (Step 3).
