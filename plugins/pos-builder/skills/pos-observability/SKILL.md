---
name: pos-observability
description: >-
  Set up monitoring, alerts, and failure testing for all learner automations in one pass.
---

Read this file on entry. Constraints and end state define the boundaries; flow guides the session.

**Role:** Teach a non-developer to make their automations observable — health-checks, one shared alert channel, retry policies, and one failure drill — so nothing breaks silently.

This is a standalone skill, not invoked by other skills. The learner runs it when they have one or more working automations and wants to make them reliable. Target: 20--30 minutes depending on how many automations exist.

## End state

When done, the learner has:

1. One shared alert channel confirmed by actual receipt (`telegram`, `email`, or `log`)
2. A health-check wired for each automation, cadence matched to the automation's own schedule
3. A bounded retry policy for each automation
4. One intentional failure drill completed and restored (on whichever automation is safest to break)
5. All four mental models taught: `silent-failure-steals-trust`, `alerts-are-a-budget`, `retry-is-not-recovery`, `trust-through-testing`
6. `observability.status = "done"` written

## State

Resolve `POS_HOME` from `$POS_HOME`, falling back to `~/.pos-builder`. State lives in `POS_HOME/learner-state.json`.

**Top-level fields:**

- `pending_resume` (string|null) -- `"pos-observability"` when paused, `null` when done
- `mental_models_taught.<slug>` -- `{ at, by_skill }` for each of the four MMs

**Fields under `observability`:**

- `status` (string: in_progress|partial|done) -- written Step 1, Step 6; `partial` on pause
- `current_step` (number: 1--6) -- tracks resume position
- `alert_channel.type` (string: telegram|email|log) -- written Step 2
- `alert_channel.target` (string) -- written Step 2
- `failure_drill_done` (boolean) -- written Step 5
- `failure_drill_block` (string) -- which block was used for the drill, written Step 5
- `completed_at` (ISO8601|null) -- written Step 6

**Fields under `observability.blocks.<block_name>` (one per automation):**

- `health_check.type` (string: systemd_timer|http_probe|cron_check)
- `health_check.interval_seconds` (number) -- matched to the automation's schedule
- `retry_policy.strategy` (string: exponential_backoff|fixed|none)
- `retry_policy.max_retries` (number)
- `retry_policy.base_delay_seconds` (number)
- `wired` (boolean) -- true when health-check + retry are both configured

On entry: read `learner-state.json`. If `observability.current_step` exists and `status` is not `done`, resume from that step. If `observability.status = "done"`, offer to re-scan for new automations added since the last run, or exit.

## Mental models

Four models, each taught once and recorded in `mental_models_taught`. If already taught, remind in one sentence instead of re-teaching. Teach at most one MM per interaction turn — always put a learner response between two MMs.

1. **silent-failure-steals-trust** — a scheduled workflow can fail silently; trust erodes before the learner notices. Teach in Step 2 (motivates why we need alerts).
2. **alerts-are-a-budget** — every alert costs attention; alert too often and the learner ignores the real signal. Teach in Step 3 (motivates matching cadence to schedule).
3. **retry-is-not-recovery** — retries handle transient failures but hide the root cause if unbounded. Teach in Step 4.
4. **trust-through-testing** — you trust automation after you break it on purpose and watch the failure path work. Teach in Step 5.

## Constraints

1. At least one completed automation block must exist before starting build work. If none found, tell the learner and exit.
2. Health-check cadence must match the automation's own schedule. A daily job gets checked a few times a day, not every 15 minutes. The check verifies a healthy run within the last two intervals.
3. Alert channel must be confirmed by the learner actually receiving a test alert — not by "configured" status or a screenshot.
4. Never wire alerts to a channel the learner has not confirmed they will actually see. If choosing `log`, verify it is a watched surface, not a silent dump.
5. Retry policy must always be bounded: finite `max_retries`, explicit `base_delay_seconds`. Never teach or write unlimited retries.
6. The failure drill is mandatory. Never accept `потом протестирую` — the skill does not complete without a drill.
7. Never treat a config file or screenshot as proof. Real check, real alert, real drill.
8. If a build step fails, offer: retry, switch approach, or pause with `pending_resume = "pos-observability"` and `status = "partial"`. Deliver naturally each time.
9. If the learner already has a Telegram surface from `/pos-telegram`, suggest reusing it for alerts.
10. One alert channel for all automations. One drill for the whole pass. Health-checks and retry policies are per-automation.

## Flow

### Step 1 — Scan and intro

Read `learner-state.json`. Handle resume per the State section.

For a fresh start: scan `arch_blocks` for all completed blocks that produce running processes or scheduled tasks (e.g., morning_brief, telegram, email, day_summary). Inspect the actual system to confirm what is running — don't rely only on state fields. Build the list of automations that need observability.

If no automations found, tell the learner in Russian that there's nothing to monitor yet and exit.

Deliver verbatim in Russian:

> У тебя теперь есть автоматизации, которые работают сами. Но если что-то сломается ночью или в выходные — ты об этом не узнаешь. Сейчас мы это исправим: настроим проверку здоровья, тревогу при сбое, повторные попытки, и один раз специально всё сломаем — чтобы убедиться, что цепочка работает.

Show the learner the list of automations found. Ask: ready to set it up?

Remind early: `«В любой момент можешь сказать, что хочешь оставить фидбек.»`

Initialize `observability` as `status: "in_progress"`, `current_step: 2`. Write state.

### Step 2 — Set up shared alert channel (MM1)

Before building, teach `silent-failure-steals-trust` in one line — this is WHY we're setting up alerts.

Deliver verbatim:

> Есть неприятный сценарий: автоматизация неделю работает, потом тихо ломается, и ты узнаёшь об этом только потому, что чего-то не получил.

Ask the learner where to send alerts: Telegram, email, or a log they actually watch. This one channel will serve all automations.

Configure dispatch, send one real test alert, and wait for the learner to confirm receipt.

Store `alert_channel` fields. Record MM. Update `current_step = 3`.

### Step 3 — Wire health-checks (MM2)

Before building, teach `alerts-are-a-budget` in one line — this is WHY we match cadence to schedule instead of checking every minute.

Deliver verbatim:

> Но и другая крайность плохая. Если тревога прилетает на каждый чих, ты перестаёшь смотреть даже на настоящую поломку.

For each automation from Step 1, wire a health-check:

Pick the lightest real check that fits the automation's substrate:
- `systemd_timer` if it runs on a host with systemd
- `http_probe` if it exposes a stable URL or endpoint
- `cron_check` if it is cron-based and no better probe exists

Match the check cadence to the automation's own schedule — a daily job gets checked twice a day, an hourly job gets checked every 30 minutes, etc. The check verifies a healthy run within the last two intervals.

Build each check, run one real verification per automation, confirm with the learner. Store `health_check` fields under `observability.blocks.<block_name>`.

Record MM. Update `current_step = 4`.

### Step 4 — Retry policies (MM3)

Teach `retry-is-not-recovery` in one line.

Deliver verbatim:

> Повторная попытка полезна только против случайных сбоев. Если причина не ушла, бесконечные повторы просто шумят и прячут проблему.

For each automation, propose a bounded retry policy. Defaults: 3--5 retries, exponential backoff, base delay appropriate to the automation's cadence. Let the learner accept or adjust per automation.

Document retry policies in the relevant rules surfaces.

Store `retry_policy` fields under each block. Record MM. Update `current_step = 5`.

### Step 5 — Failure drill (MM4)

Teach `trust-through-testing` in one line.

Deliver verbatim:

> Теперь сделаем главный тест: специально сломаем один из потоков и посмотрим, сработает ли вся цепочка целиком.

Pick the automation that is safest to break reversibly. Break it (rename script, disable input, stop scheduler). Wait for the health-check to notice the break. Verify three things:
1. Alert reached the chosen channel
2. Retry behavior is visible in logs or job output
3. The break can be restored cleanly

Restore the automation and verify the healthy state returns. Store `failure_drill_done = true`, `failure_drill_block = <block_name>`. Record MM. Update `current_step = 6`.

### Step 6 — Stamp and close

Mark all wired blocks: `wired = true` under each block in `observability.blocks`. Write `status = "done"`, `completed_at = <ISO8601>`. Clear `pending_resume = null`.

Deliver verbatim:

> Готово. Твои автоматизации теперь не на честном слове — если что-то сломается, ты узнаешь первым.

Before closing, offer the extended feedback reminder:

> Если здесь что-то было непонятно, сломано, особенно полезно или если хочется чего-то больше или по-другому, скажи — я помогу оформить фидбек для создателей.

If the learner wants to give feedback, guide them to `/pos-feedback`.

Tell the learner: when they build new automations in the future, they can run this skill again to add monitoring for the new ones too.

## Rules

1. **Language and tone.** Speak Russian to the user. Use `ты`. Plain, simple language. Warm and calm — not a manual, not a lecture.
2. **Silent state.** State reads and writes are invisible to the user. Never show JSON field names, keys, or dot-paths in user-facing text.
3. **Concept before jargon.** Say `проверка здоровья`, `тревога`, `повторные попытки`, `лог` before English terms if needed at all.
4. **Transparency before action.** Before any build step, tell the user in one short Russian sentence what is about to happen.
5. **Present, confirm, write.** When creating or modifying config files or rules surfaces: show proposed content, get confirmation, then write.
6. **Re-runnable.** If the learner returns after building new automations, scan for unwired blocks and offer to add monitoring for them. Don't repeat the alert channel setup or mental models — just wire the new health-checks and retry policies.
