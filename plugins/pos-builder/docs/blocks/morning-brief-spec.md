# pos-morning-brief — Locked Frame

> Frame locked 2026-04-19 during brainstorming session with Stas. References:
> - Skill contract: `docs/skill-contract.md` (normative — do not restate here)
> - Sibling (tone/rhythm only, anti-anchored): `skills/pos-tasks/SKILL.md` — most recent shipped

## Overview

`pos-morning-brief` is the first **composite** skill in the course. It consumes already-connected adapters (calendar + tasks mandatory; email + telegram preferred), reads life goals from the vault, optionally captures current-period priorities, and delivers a daily brief through a learner-chosen channel on a scheduled substrate. Two-way interaction is explicitly out of v0.1 proactive scope (natural in OpenClaw injection; opt-in only for TG two-way).

The skill is agentic on content composition (agent proposes a starter shape, learner redirects freely) and prescriptive on safety, consent, credential location, and observability.

## Prerequisites

**Hard-gate at Phase 0** — any missing → route to corresponding `/pos-<skill>` with one-line Russian reason, no silent skip:
- `arch_blocks.vault.status = "done"`
- `arch_blocks.goals.status = "done"` *(provided by `pos-goals`, epic #100, build #101 — not yet shipped; morning-brief authors the frame assuming it will be, routes via handoff when missing)*
- `arch_blocks.calendar.status = "done"`
- `arch_blocks.tasks.status = "done"`

Morning-brief reads goals artifacts at whatever location `pos-goals` documents; **no inline goals elicitation fallback lives in this skill** — missing goals always hard-route to `/pos-goals` with the standard resume pattern. This skill does not fork the goals schema.

**Soft-preferred** — missing → brief still runs, some composition items are unavailable, `pending_adapters` records intent:
- `arch_blocks.email.status = "done"`
- `arch_blocks.telegram.status = "done"`

**Conditional** — triggered only if learner picks TG delivery channel AND no bot exists yet:
- `arch_blocks.basic_vibecoding.status = "done"` → soft handoff to `/pos-basic-vibecoding` with return.

## End State (outcome-shaped, soft on content)

The learner walks away having **chosen and configured** (not received a prescribed config):

1. Delivery time (in learner's timezone) and channel(s) — any non-empty subset of `{telegram, email, openclaw_injection}`. Multiple allowed.
2. Brief composition — final shape is what the learner picked from agent's starter offer plus any additions/subtractions the learner directed. The starter offer always includes `events`, `tasks`, `focus`, `alignment`; the agent may propose tier-up items (`day_shape`, `dont_miss`, `meeting_prep`, `over_scheduling`, `email_digest`, `tg_highlights`); the learner may add anything else.
3. Scheduler running on the chosen substrate — `systemd_timer` / `cron` / `vps_cron` / `bot_internal`. Agent discovers the appropriate substrate at runtime based on OS + VPS presence + delivery channel. **Never OpenClaw cron** (F1).
4. Current-period priorities state — either `captured` (with weekly and/or monthly items + timestamps) OR `declined` (valid outcome).
5. Observability declared — log path, alert surface (distinct from primary OR explicitly prefixed when same), retry policy. Concrete path runtime-discovered; declaration is the gate (G6).
6. Rules-of-use — `## Morning brief` section appended to the learner's primary project agent-config file (`CLAUDE.md` for Claude Code, `AGENTS.md` for Codex), with optional mirroring to both when `learner_profile.keep_agent_configs_in_sync == true`; append-only, diff+confirm if section exists.
7. Wow moment — first brief delivered at a learner-chosen near-future time, observed live on the **primary** channel; non-primary channels receive the same fire but their delivery is verified *out-of-band* and surfaced via the alert surface if failed, not blocking the wow (G5).
8. State written under `arch_blocks.morning_brief` per the State Contract below, including `schema_version`.

## Mental Models taught (5)

1. **Брифом управляет ученик, не агент.** Агент предлагает стартовую форму; ученик формирует состав. Если ученик хочет чего-то, чего агент не предложил — агент идёт туда. *(Reusable across skills — reinforces permissive-framing principle.)*

2. **Чем больше агент знает, тем больше он может решать.** День / неделя / месяц приоритетов приходят от ученика с регулярным обновлением. Видимый цикл: агент спрашивает → ученик отвечает → бриф точнее. По мере накопления данных часть цикла уйдёт агенту. *(Tease toward future delegation skills.)*

3. **Если не записано — агент не видит.** Жизненные цели лежат в vault как реальное содержимое; приоритеты — в state. Без этих источников бриф может только описывать день, не оценивать его. *(Reuses the pos-vault MM — shared vocabulary.)*

4. **Бриф читает план, не строит его.** Скилл работает с тем, что ученик уже положил в календарь и трекер. Подсвечивает alignment и misalignment с целями/приоритетами — не подменяет решения ученика. *(Scope-boundary mental model — prevents agent overreach.)*

5. **Планировщик сработал ≠ ученик получил.** Scheduled delivery требует отдельной видимой поверхности на случай сбоя. Иначе тишина выглядит как «сегодня ничего важного», а на деле доставка упала. *(Grounds the observability gate G6 / forbidden F6 in a principle the learner can internalize.)*

## Required Gates

- **G1. Hard-gate prereqs verified at Phase 0.** Read `arch_blocks.{vault, goals, calendar, tasks}.status`. Any missing → route to `/pos-<skill>` with one-line Russian reason, then pause this skill with the standard resume pattern.
- **G2. Priority-layer consent.** Setup must explicitly promote weekly + monthly priority capture and record the learner's choice (`captured` / `declined`). No silent assumption.
- **G3. Channel consent.** Delivery channel(s) chosen by the learner. Agent proposes; no default assumed.
- **G4. Scheduler consent before activation.** Show the learner the planned installation — unit name, fire time, removal command, file location — before `Build:`-ing. No silent scheduling.
- **G5. First-fire wow-moment is verified on the primary channel.** The learner nominates a near-future time (not 24h away) so they observe delivery live on the **primary** channel; this verification is required before `status: "done"`. Non-primary channels fire at the same time; their outcomes are captured in state (`first_fire.secondary_status`) and surfaced via the alert surface if failed, but do not block G5.
- **G6. Observability surface declared.** By end of setup the learner knows: (a) where the log / status lives; (b) what channel receives failure alerts (distinct from primary OR explicitly prefixed on same); (c) retry policy. Concrete mechanism is runtime-discovered; **declaration is the gate**.
- **G7. Credentials follow course convention.** Tokens, SMTP creds, VPS keys live in tool-native stores — keyring / `.env` / bot platform — never in the learner's vault.
- **G8. Primary agent-config rules-of-use append-only.** `## Morning brief` section appended with English labels / Russian values to the learner's resolved target set; diff-and-confirm if section already exists.
- **G9. Pending-adapter surface.** Declined-now-but-intends-later soft-preferred adapters go into `pending_adapters`; final phase queues them for the next-block recommender.

## Forbidden

- **F1. No OpenClaw cron for scheduling.** Even when OpenClaw is the delivery channel, scheduling lives elsewhere.
- **F2. No credentials in the vault.**
- **F3. No full-schedule generation.** Skill reads the plan; does not create tasks or calendar events for the learner. Future skills may; this one does not.
- **F4. No proactive proposal of two-way TG.** Learner must explicitly ask before that path is even mentioned. Brief stays non-interactive by default.
- **F5. Staleness insertion is unconditional.** When life-goal or priority staleness crosses its threshold, the brief MUST include a dedicated staleness line — regardless of the learner's `composition.chosen` set. Staleness is NOT a composition item the learner may opt out of; it's a mandatory conditional insertion. Silent staleness breaks MM2's feedback loop.
- **F6. No silent delivery failure.** Failures reach the learner via the declared alert surface. Output suppression (`2>/dev/null` style) forbidden.
- **F7. No cross-account mixing.** Inherit adapter scope from pos-calendar / pos-email / pos-telegram — personal/work separation preserved.
- **F8. No mandatory priority capture.** Promote, don't force. Declining is a valid outcome.
- **F9. No silent adapter selection.** The set of adapters the brief reads is confirmed with the learner in setup.
- **F10. No overriding learner content choices.** If the learner redirects away from the agent's starter offer, the agent follows — even if dropping items the frame considered starter-default.
- **F11. No inline goals fallback.** The skill does not elicit, imply, or write goals content under any circumstance. If `arch_blocks.goals.status ≠ "done"`, the only valid path is the hard route to `/pos-goals` (G1). No «давай быстро тебя сейчас спросим про цели» shortcut exists.

## State Contract — `arch_blocks.morning_brief`

```json
{
  "arch_blocks": {
    "morning_brief": {
      "status": "in_progress | done",
      "schema_version": 1,
      "started_at": "<ISO-8601>",
      "completed_at": "<ISO-8601> | null",
      "current_phase": 0,

      "delivery": {
        "channels": ["telegram"],                // non-empty subset of {telegram, email, openclaw_injection}
        "primary": "telegram",                   // required when channels.length > 1; picks the wow-moment verification channel
        "alert_surface": "email",                // one of: any channel in `channels` except `primary`, OR "log_file", OR "same_channel_prefixed" (last-resort when only one channel exists)
        "log_path": "<agent-discovered>",
        "retry": { "attempts": 3, "interval_minutes": 5 }
      },

      "schedule": {
        "time_local": "HH:MM",
        "timezone": "<IANA-tz>",
        "substrate": "systemd_timer | cron | vps_cron | bot_internal",
        "scheduler_unit_ref": "<agent-discovered path or name>"
      },

      "composition": {
        "offered": ["events","tasks","focus","alignment","day_shape","dont_miss","meeting_prep","over_scheduling","email_digest","tg_highlights"],
        "chosen": ["events","tasks","focus","alignment"],
        "added_by_learner": []
      },

      "adapters_used": ["calendar","tasks"],
      "pending_adapters": [],

      "priorities": {
        "mode": "captured | declined",
        "weekly":  { "items": ["..."], "captured_at": "YYYY-MM-DD" },
        "monthly": { "items": ["..."], "captured_at": "YYYY-MM-DD" },
        "life_goals_age_days": 12,               // recomputed when goals artifact is read; drives F5 staleness enforcement
        "staleness_thresholds_days": { "weekly": 10, "monthly": 40, "life_goals": 60 }
      },

      "openclaw_detected": true,
      "two_way_requested": false,
      "first_fire": {
        "scheduled_at": "<ISO-8601>",            // learner-nominated near-future first-fire time; distinct from daily schedule.time_local; must resolve in a 1-30 min window; persists across session pauses so Phase 10 resume can still verify G5
        "primary_verified_at": "<ISO-8601>",     // G5 wow-moment; required before status: "done"
        "secondary_status": {                    // per non-primary channel: outcome of the simultaneous fire; agent populates ok/failed/pending; failures surface via alert_surface post-wow, not blocking G5
          "email": "ok | failed | pending"
        }
      },
      "basic_vibecoding_status_at_entry": "done | not_done"
    }
  }
}
```

Notes:
- `priorities.weekly` / `priorities.monthly` are `null` when `mode = "declined"` OR when only one horizon was captured.
- `pending_adapters` captures soft-preferred adapters the learner wants but hasn't connected this session — picked up by the next-block recommender in the final phase.
- `schema_version = 1` required per v0.1 DoD cross-cutting rule.

## Wow moment

First brief delivered at a learner-chosen near-future time (e.g., «через 3 минуты»), observed live in the chosen channel(s). Skill verifies delivery hit the channel before writing `status: "done"`. If the first fire fails, the error-visibility contract (skill contract principle 22) kicks in — agent offers `retry` / `switch channel` / `manual diagnose` paths and does not claim success.

## Body-authoring notes (for Phase 1 writer, not part of the frame contract itself)

- Setup phases should include, in any order the agent judges best for the learner: prereq verification, goals/priorities freshness check, channel + time choice, composition offer + learner-redirect loop, scheduler choice + consent, observability declaration, first-fire live wow, agent-config append, state writeback + handoff.
- The permissive-framing principle applies to phase ordering itself: the agent may reshape the phase order for a given learner if it serves clarity (e.g., a learner who already knows what they want can compress several phases into one).
- OpenClaw injection mechanism is unknown to this frame on purpose. If OpenClaw is chosen as a delivery channel, the Build phase researches the current OpenClaw injection surface at runtime, documents the chosen mechanism in the learner's resolved agent-config rules-of-use, and verifies wow-moment delivery before state-write. No specific mechanism is pre-named in the frame.

## Course-wide principles applied

Body inherits all 22 principles from `docs/skill-contract.md`. Especially load-bearing for this skill: **3** (bite-sized Say/Check rhythm), **11** (state writes at phase transitions only), **16** (skip pre-answered Checks), **17** (state writes are silent — no JSON keys in learner-visible text), **18** (pre-warn predictable anxiety), **19** (one mental model per Say), **21** (proof before choice — qualified by consequence), **22** (error-visibility contract for every external-dependency Build).

## References

- `docs/skill-contract.md` — normative 22 principles; do not restate in `SKILL.md`.
- `docs/block-runtime-pattern.md` — runtime flow reference.
- `skills/pos-tasks/SKILL.md` — latest shipped sibling; Phase 1 writer anchors tone/rhythm only, anti-anchoring guard active on structure/content.
