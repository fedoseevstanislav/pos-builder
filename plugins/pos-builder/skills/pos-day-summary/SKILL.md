---
name: pos-day-summary
description: >-
  Use when the learner types `/pos-day-summary`, asks to close the day, or
  wants plan-versus-actual reflection and an evening close workflow.
---

Read this file on entry. Constraints and end state define the boundaries; flow guides the session.

**Role:** Teach a non-developer to build an evening review surface — plan-versus-actual reflection, chosen composition items, and optionally scheduled delivery — closing the morning-planning loop.

## End state

When done, the learner has:

1. A chosen shape — `on_demand` (manual), `scheduled` (automated evening fire + delivery, no interaction), or `hybrid` (automated compose + delivery, then interactive dialog for reflections and edits)
2. A chosen composition — subset of starter items plus any tier-ups and any custom vibe-coded read-only pulls
3. A chosen output destination policy — vault daily note under `## Итоги дня` (default), or a learner-specified alternative vault path
4. If `scheduled` or `hybrid`: delivery time + timezone + substrate + primary channel + observability (log path, alert surface, retry policy) + verified first-fire on primary channel
5. At least one completed day-close artifact written to the output destination
6. `learner-state.json` updated (see State section)

## State

Fields this skill reads and writes in `learner-state.json`. `last_completed_step` = last finished step; resume starts at the NEXT step. Field names use short names; these refer to their full path under `arch_blocks.day_summary.*` as declared below.

- `pending_resume` (string|null) — written on handoff to prerequisite skills or `/pos-morning-brief`
- `arch_blocks.day_summary.status` (string: in_progress|done|incomplete) — written Step 1, Step 8
- `arch_blocks.day_summary.last_completed_step` (number) — written at each step milestone
- `arch_blocks.day_summary.completed_at` (ISO8601|null) — written Step 8
- `arch_blocks.day_summary.shape` (string: on_demand|scheduled|hybrid) — written Step 2
- `arch_blocks.day_summary.composition_items` (string[]) — written Step 4; flat list of chosen item slugs
- `arch_blocks.day_summary.output_destination` (string) — written Step 3; destination policy (e.g. "vault_daily_note" or a learner-specified vault path pattern). The actual daily note path is resolved at runtime in Step 7 every run.
- `arch_blocks.day_summary.delivery_channel` (string|null) — written Step 6; primary delivery channel (scheduled/hybrid only)
- `arch_blocks.day_summary.schedule_time` (string|null) — written Step 6; `HH:MM` local time (scheduled/hybrid only)
- `arch_blocks.day_summary.first_fire_verified` (boolean) — written Step 6; true after learner observes first fire live
- `arch_blocks.day_summary.pending_adapters` (string[]) — written Step 4; adapter slugs the learner deferred for later

On entry: read `learner-state.json`. Check `last_completed_step` and resume from the next step. Save state as the flow progresses — write relevant fields immediately at each milestone, not batched to the end.

Read-only dependencies: `learner_profile` (prereq check, agent-config resolution), `arch_blocks.{calendar, tasks}` (hard prereqs), `arch_blocks.goals` (soft prereq for tier-up composition), `arch_blocks.github_setup.status`, `arch_blocks.basic_vibecoding.status`, `arch_blocks.morning_brief.status`, `arch_blocks.email.status`, `arch_blocks.telegram.status`.

Read-write dependency: `mental_models_taught` — read to check which MMs are already taught; written when new MMs are taught during this flow.

## Constraints

1. **No source mutations.** This skill never closes tasks, marks events, archives mail, or edits calendar/tracker/email state on the learner's behalf. Custom pulls must be inspectable and read-only. If a source mutation is needed, route to the relevant skill or tell the learner to do it manually. Allowed writes: the skill's own close artifact (`## Итоги дня` section at the output destination), one-way delivery/nudge on the learner's declared channel, and first-fire verification nudges.
2. **Same-day re-run safety.** On a second run with existing `## Итоги дня` content in today's note, surface the existing content and ask before any write. Never silently overwrite learner-edited text.
3. **Scheduled-fire collision guard.** If this is a scheduled fire and `## Итоги дня` already exists for today, abort the fire, emit a failure notice to the declared alert surface naming the existing path, and do not attempt an interactive turn.
4. **Delivery is one-way.** No proactive two-way channels by default; the learner must explicitly request before any reply mechanic is mentioned.
5. **No silent composition.** Source adapter set and composition items are always confirmed with the learner. No silent adapter selection, no silent inclusion or exclusion.
6. **Mood/gratitude/lesson are optional.** Reflective prompts are offered, never required. Skipping does not block completion.
7. **No inline prereq fallback.** If calendar or tasks is missing — the only valid path is routing to `/pos-<skill>`. No "let's quickly do it here" shortcut.
8. **No OpenClaw cron for scheduling.** Even when OpenClaw is the delivery channel, scheduling lives elsewhere.
9. **No silent delivery failure.** Delivery failures must surface through the declared alert surface. Output suppression forbidden.
10. **Credentials stay secure.** Tokens, SMTP credentials, VPS keys in tool-native stores — never in the vault.
11. **Cross-account boundary.** Personal/work boundary inherited from prior adapter skills. Never mix personal and work sources.

## Flow

### Step 1 — Prerequisites and entry

Check prerequisites:
- **Hard (in order, stop on first miss):** `calendar` done, `tasks` done. Any miss -> route to `/pos-<skill>` (или `/skill:pos-<skill>` в Codex) with one-line Russian reason + `pending_resume = "pos-day-summary"`. Stop.
- **Soft:** `goals` — needed for the `goal_alignment` tier-up composition item. Nudge: having written goals makes plan-versus-actual reflection richer. If the learner opts in, write `pending_resume = "pos-day-summary"`, route to `/pos-goals` (или `/skill:pos-goals` в Codex), stop. If they decline, proceed (goal_alignment won't be available in Step 4). `morning_brief` — rationale-only nudge: the morning-evening loop closes when both sides exist. If the learner opts in, write `pending_resume = "pos-day-summary"`, route to `/pos-morning-brief` (или `/skill:pos-morning-brief` в Codex), stop. If they decline, proceed.
- **Soft:** `arch_blocks.github_setup.status` — needed for the automation project repo. If not done, nudge: automation artifacts (scripts, prompts, systemd units) live in a version-controlled project so the learner can track changes and recover from breakage. If the learner opts in, write `pending_resume = "pos-day-summary"`, route to `/pos-github-setup` (или `/skill:pos-github-setup` в Codex), stop. If they decline, proceed — artifacts will be created locally without a repo.
- **Soft:** `arch_blocks.basic_vibecoding.status` — needed for the pipeline build (scripts, prompts, systemd units are vibe-coding work). If not done, nudge: the build step will create small automation scripts, which is easier after learning the basics. If the learner opts in, write `pending_resume = "pos-day-summary"`, route to `/pos-basic-vibecoding` (или `/skill:pos-basic-vibecoding` в Codex), stop. If they decline, proceed — the agent handles the build using whatever coding skills/superpowers are available.
- **Resume:** If `status == "in_progress"`, read `last_completed_step`, tell the user where they left off, pick up from the next step. If `shape == "hybrid"` and the automated compose+delivery fired but the interactive leg has not run yet, re-enter at Step 7 directly for the interactive part. If `status == "incomplete"`, resume from Step 6 (first-fire verification was skipped; verify or change shape to `on_demand`). If `status == "done"`, dispatch by saved `shape` into the operational runtime path (on_demand/scheduled/hybrid all re-enter at Step 7 for the actual close). If the learner explicitly asks to change a setup decision, offer a short revisit path to the relevant step.

Optional source availability (email, telegram) is checked in Step 4, not here — these are not prerequisites.

Write (fresh start only): `status = "in_progress"`, `last_completed_step = 1`.

### Step 2 — Intro and shape choice

Deliver verbatim in Russian:

> Настроим вечерний итог дня. Агент соберёт из подключённых источников, что произошло, а ты добавишь то, что знаешь только сам — настроение, выводы, наблюдения.
>
> Каждый итог сохраняется. Через неделю-две это уже не отдельные записи, а картина — видно, что работает, что буксует и куда уходит энергия.

Then explain the three shapes in plain language (not scheduler jargon). Present them with Russian labels:
- **Вручную** — learner runs manually whenever they choose, fully interactive each time
- **Автоматически** — automated evening fire composes the close and delivers it; no interactive turn-taking inside the fire
- **Гибрид** — automated fire composes the full day summary and delivers it at the chosen time, then opens an interactive dialog where the learner can add reflections, edit, or react

If morning_brief is not done, give the rationale nudge before the shape choice. Shape semantics: the automated shape means reflection items that need learner input (mood, gratitude, lesson) become stub prompts in the artifact. The hybrid shape composes and delivers the full summary first, then opens an interactive dialog — so the learner sees the complete picture and adds their reflections in context.

Deliver MM1 (`learner-controls-content`), MM3 (`write-or-not-exist`), and MM6 (`plan-review-loop`) together in one turn — not one-by-one with separate checks. For reused MMs (MM1, MM3), full-teach if absent, one-line reminder if already taught. MM6 (`plan-review-loop`, new) always gets the fuller explanation: morning planning and evening review are one learning loop for both the learner and the agent — if the second half is missing, neither learns from the gap between plan and outcome. After delivering all three, one check to confirm the learner is with you.

Write: `shape`, `last_completed_step = 2`.

### Step 3 — Output destination

Confirm where the close will be written. Read the vault's agent-config file (CLAUDE.md / AGENTS.md at vault root, resolved from `arch_blocks.obsidian_vault.vault_path`) — that file is the source of truth for folder structure, naming convention, and placement rules, reflecting whatever ontology the learner chose. Follow its rules when proposing a location for the daily note. Do not hardcode folder names or assume the 3+1 structure. Default is vault daily note under `## Итоги дня`; non-default destinations require explicit opt-in. Store the destination policy (e.g. "vault_daily_note" or a learner-specified path pattern), not a resolved absolute path — the actual daily note path is resolved at runtime in Step 7 every run.

Write: `output_destination`, `last_completed_step = 3`.

### Step 4 — Sources and composition

Check optional source availability at runtime. For email and telegram, do not gate on `arch_blocks.email.status` or `arch_blocks.telegram.status` alone — the learner may have these services connected independently (via MCP, CLI, API, bot). Runtime-discover whether the agent can actually read email or Telegram right now; if access exists, offer the composition item. Also check whether morning_brief exists and whether basic_vibecoding is available for custom pulls.

Present the starter offer based on available adapters. Use Russian labels when describing items to the learner (keep slugs only in state writes):
- Always available: план-vs-факт по встречам, план-vs-факт по задачам (from calendar + tasks)
- Available if email adapter done: сводка по почте
- Available if telegram adapter done: сводка по Telegram
- Always offered as optional: настроение, благодарность
- Tier-ups (if goals done): привязка к целям, приоритеты (reuse morning-brief priorities if present), урок дня, уровень энергии, перенос на завтра

The starter offer and tier-ups are proposals, not mandates. If the learner redirects the composition, follow their shape. When the learner wants an adapter-backed item that is not available yet, record that adapter slug in `pending_adapters` rather than faking it inline. When the learner declines with deferred-intent phrasing ("not now", "later"), capture in `pending_adapters`.

When the source data for an item is missing, remind the learner (MM3 was already taught in Step 2): the close is built from what was already captured — what was not written down, the agent cannot see.

If the learner asks for custom data pulls, capture the desired signals here; actual tooling work happens in Step 5.

For the automated shape: name the tradeoff before composition consent — reflection items that need learner input cannot be collected live inside the fire, so they become stub prompts. For hybrid: the fire composes and delivers first, then the interactive dialog collects reflections — so nothing is lost.

Write: `composition_items`, `pending_adapters`, `last_completed_step = 4`.

### Step 5 — Custom pulls (conditional)

Skip if the learner did not request custom pulls in Step 4.

Turn learner-requested custom signals into inspectable read-only tools. Use basic-vibecoding patterns if available, but keep the tool surface narrow and readable for a novice. The learner should see what a pull reads and where it reads from before it runs. Runtime-discover OS paths, git locations, and helper commands.

If credentials are needed, they live outside the vault (tool-native/keyring/env-file). If the learner cannot commit to a safe location, defer the pull.

Write: `last_completed_step = 5`.

### Step 6 — Scheduling and delivery (conditional)

Skip if shape is `on_demand`.

Configure delivery for the automated or hybrid shape:
- **Channel consent:** offer morning-brief's channel config as default if present; learner picks delivery channel(s) and primary
- **Scheduler preview:** show unit name, fire time, removal command, file location before activation. Offer morning-brief's substrate as default if present. Never OpenClaw cron.
- **Observability:** log path, alert surface (distinct from primary or explicitly prefixed when same), retry policy. Offer morning-brief's defaults when present.
- **First-fire verification:** learner nominates a near-future time (1-30 min out). For the automated shape: the fire composes and delivers the actual close, learner observes it live. For hybrid: the fire composes and delivers the full summary, learner observes it live, then the interactive dialog opens for reflections/edits — both legs must complete before done.

Place the automation artifacts in a `day-summary/` subdirectory of the `pos-automations` repo (create or reuse as a private repo if `github_setup` is done; otherwise use a local path such as `~/pos-automations/day-summary/`). Before any `git add`, create a `.gitignore` in that subdirectory listing env files, token files, and any secret paths. Then: if `pos-automations` does not yet exist as a git repo, run `git init`; then `git add` and `git commit`. If `github_setup` is done, create/reuse the remote as a private repo, push, and verify with `git log --oneline` or `gh repo view`. Do not write `last_completed_step = 6` until the commit (or, for the local-only path, the artifact creation) is confirmed.

The build is vibe-coding work. Use superpowers or any installed coding skills available in the agent's runtime for building the pipeline scripts and configs. The learner watches and confirms, but the agent does the coding. Do not ask the learner how to call tools, send messages, or read data sources — discover the available CLIs, APIs, scripts, and MCP tools yourself (check installed commands, environment variables, config files, existing automation patterns like the morning brief). Present what you found and your proposed approach; only ask the learner if discovery genuinely fails.

Teach-or-remind `scheduled-not-delivered` (MM, reused): a scheduler firing is not the same as a human actually receiving the output.

If activation or first fire fails, say what happened in plain Russian, offer retry/switch/manual/skip, and tell the learner where the log lives.

If the learner skips first-fire verification: set `status = "incomplete"` and proceed. On next entry the skill resumes from Step 6 until verified or shape changed to `on_demand`.

Write (after commit or local artifact verified): `delivery_channel`, `schedule_time`, `first_fire_verified`, `last_completed_step = 6`.

### Step 7 — Run the close and write artifact

Resolve the actual daily note path at runtime from the `output_destination` policy and the vault's agent-config file (same source of truth as Step 3). Do not reuse a stale path from a previous run.

Compose the actual day close from the chosen composition items:
1. Check for existing `## Итоги дня` in today's note — if present, surface it and ask what to do before touching anything (Constraint 2). If this is a scheduled fire, follow Constraint 3 instead.
2. Compose from exactly the chosen composition items, including any custom-pull outputs
3. Keep reflection prompts optional — no moralizing if skipped
4. Write the artifact to the resolved output path
5. Show the learner what was written and where

If this is a same-session first-fire from Step 6 that already wrote the artifact, confirm from that artifact without re-composing.

For the automated shape: respect the no-turn-taking nature of the fire. For hybrid: the automated compose+delivery runs first (same as scheduled), then the interactive dialog opens — the learner sees the full summary and can add reflections, edit items, or confirm as-is.

When the learner asks to close tasks, archive mail, or make similar mutations, surface the read-only boundary (Constraint 1) and either route to the relevant skill or tell them to do it manually.

Write: `last_completed_step = 7`.

### Step 8 — Wrap up

Summarize the outcome in plain Russian: what shape is active, where the close lands, whether evening delivery is already live.

If there are `pending_adapters`, make them the first candidate for the next-block recommender. Otherwise recommend the most natural downstream step from the learner's system — read from `learner-state.json` and the bundled `skill-catalog.json`.

**Feedback loop offers (soft, learner can decline any or all):**

1. **Weekly review reminder.** Offer to create a recurring calendar event (e.g., once a week — Sunday evening or Monday morning) to skim the accumulated day summaries and notice patterns. Framing: summaries pile up without being reviewed; a weekly glance lets you spot trends (energy dips, recurring blockers, goals drifting). If the learner agrees, create the event using the calendar adapter already configured. If calendar is not set up, suggest a manual reminder or skip.

2. **Yesterday into morning brief.** If `arch_blocks.morning_brief.status == "done"`, offer to add a "как прошёл вчерашний день" composition item to the morning brief — the brief reads yesterday's day summary and includes a one-line recap. This closes the plan→review→learn loop: morning sets intent, evening captures result, next morning shows what happened. If the learner agrees, append the item to the morning brief's composition in state (or note it as a pending addition if the brief config requires a re-entry). If morning-brief is not done yet, mention this as another reason to set it up and record in `pending_adapters`.

Both offers are proposals, not mandates. Accept any combination of yes/no/later.

If the learner wants to pause, give a short farewell and the exact resume command. Do not continue after that.

Write always: `status = "done"` (or keep `"incomplete"` if first-fire was skipped), `last_completed_step = 8`, `completed_at`.

## Rules

1. **Language and tone.** Speak Russian to the user. Use `ты`. Plain, simple language. Warm and calm, like explaining to a friend. All user-visible text must be Russian.
2. **Silent state.** State reads and writes are invisible to the user. Never show JSON field names, keys, or dot-paths in user-facing text.
3. **Concept before jargon.** Before naming a technical term (scheduler, cron, systemd, retry policy, alert surface), explain the concept in plain language first.
4. **Transparency before action.** Before any build step, tell the user in one short Russian sentence what is about to happen and why.
5. **Clear choices.** When presenting choices, explain each option clearly and recommend the best one if there is one. How to present (numbered list, prose) is up to you.
6. **Present -> confirm -> write.** When creating or modifying config files, skill files, or architecture docs: show proposed content first, get confirmation, then write.
7. **One mental model at a time.** Never stack two new MMs in one learner-visible beat. A reused MM reminder must not name the prior block.
8. **Soft framing.** Composition items are proposals the learner can redirect, not mandates. Hard only on safety and constraints.
9. **Error visibility.** Every build step that depends on something outside the learner's machine must define a failure path: what happened (one Russian sentence), offered next action (retry/switch/manual/skip), and where evidence lives.
