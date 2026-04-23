# POS Observability — State Contract

This mini-skill writes in two zones:

- top-level `mental_models_taught` and `pending_resume`
- `arch_blocks.<invoking_block>.observability`

The `observability` branch has two layers:

- **Gate-required fields** — the fields that must satisfy the bundled `catalog/gates/observability.json`
- **Progress fields** — local resume helpers used by this mini-skill (`status`, `current_phase`)

```json
{
  "mental_models_taught": {
    "silent-failure-steals-trust": { "at": "<ISO8601>", "by_skill": "pos-observability" },
    "alerts-are-a-budget": { "at": "<ISO8601>", "by_skill": "pos-observability" },
    "retry-is-not-recovery": { "at": "<ISO8601>", "by_skill": "pos-observability" },
    "trust-through-testing": { "at": "<ISO8601>", "by_skill": "pos-observability" }
  },
  "pending_resume": "pos-observability-phase-<N> | null",
  "arch_blocks": {
    "<invoking_block>": {
      "observability": {
        "status": "in_progress | partial | done",
        "current_phase": 1,
        "gate_passed": false,
        "gated_at": null,
        "health_check": {
          "type": "systemd_timer | http_probe | cron_check",
          "interval_seconds": 900,
          "validation_rule": "exit 0 within last 2 intervals",
          "target": "<optional receipt: unit name | URL | cron reference>"
        },
        "alert_channel": {
          "type": "telegram | email | log",
          "target": "<chat_id | address | path>",
          "on_conditions": [
            "failure_streak >= 3",
            "timer_missed >= 1"
          ]
        },
        "retry_policy": {
          "strategy": "exponential_backoff | fixed | none",
          "max_retries": 5,
          "base_delay_seconds": 60
        },
        "sla_uptime_target": 0.95,
        "log_retention_days": 30,
        "first_failure_drill_done": false
      }
    }
  }
}
```

Notes:

- `health_check.target` is an implementation receipt for resume/debug convenience. The hard gate still hinges on the canonical gate fields: `type`, `interval_seconds`, and `validation_rule`.
- `status` and `current_phase` are mini-skill progress helpers. Downstream scheduled skills should not use them for their own `status: done` enforcement.
- `pending_resume` stores only the next unfinished observability phase. The invoking block itself must be recovered from handoff args or from the block that already carries the partial `observability` branch.
- `log_retention_days` uses the bundled gate default. A caller may store a higher retention period, but never less than `30`.
