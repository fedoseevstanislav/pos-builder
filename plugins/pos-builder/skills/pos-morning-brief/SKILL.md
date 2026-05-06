---
name: pos-morning-brief
description: >-
  Use when the learner types `/pos-morning-brief`, asks for a morning brief,
  or wants scheduled daily planning from connected sources.
---

Read this file on entry. Constraints and end state define the boundaries; flow guides the session.

**Role:** Teach a non-developer to compose a daily morning brief from already-connected adapters, choose where and when it arrives, and verify live delivery.

## End state

When done, the learner has:

1. A delivery time (in their timezone) and channel(s) — at least one from {telegram, email} plus any runtime-discovered channels, with a primary channel designated
2. A brief composition chosen by the learner from the agent's starter offer, with learner additions/subtractions applied — the learner controls the content
3. Current-period priorities read from the goals state/artifact in the vault, or explicitly declined — both valid outcomes. Morning-brief does not persist priorities; it reads them from wherever `pos-goals` stored them
4. A scheduler running on an agent-discovered substrate (systemd timer, cron, VPS cron, or bot-internal); never OpenClaw cron
5. Observability declared: log path, alert surface (one of: distinct channel, log file, or same channel with error prefix if only one channel), retry policy chosen with the learner
6. A first brief delivered at a learner-chosen near-future time, verified live on the primary channel — the wow moment
7. `learner-state.json` updated (see State section)

## State

Fields this skill reads and writes in `learner-state.json`. `last_completed_step` = last finished step number; resume starts at the NEXT step.

**Staleness thresholds (normative, not configurable):** weekly priorities 10 days, monthly priorities 40 days, life goals 60 days. When any threshold is crossed, the brief MUST insert a staleness line.

- `pending_resume` (string|null) — top-level; written on handoff to prerequisite skills
- `arch_blocks.morning_brief.status` (string: in_progress|done|incomplete) — written Step 1, Step 11
- `arch_blocks.morning_brief.last_completed_step` (number) — written at each step milestone
- `arch_blocks.morning_brief.completed_at` (ISO8601|null) — written Step 11
- `arch_blocks.morning_brief.delivery_channels` (string[]) — written Step 3; at least one channel (telegram, email, or runtime-discovered)
- `arch_blocks.morning_brief.delivery_primary` (string) — written Step 3; the wow-moment verification channel
- `arch_blocks.morning_brief.composition_chosen` (string[]) — written Step 4; canonical slugs from the composition menu (see Step 4 for authoritative list)
- `arch_blocks.morning_brief.priorities_mode` (string: captured|declined) — written Step 2
- `arch_blocks.morning_brief.schedule_time` (string, HH:MM) — written Step 5
- `arch_blocks.morning_brief.schedule_timezone` (string, IANA) — written Step 5
- `arch_blocks.morning_brief.first_fire_time` (string, HH:MM) — written Step 10; near-future time for the wow-moment fire
- `arch_blocks.morning_brief.schedule_substrate` (string) — written Step 6
- `arch_blocks.morning_brief.adapters_used` (string[]) — written Step 4; calendar+tasks always, plus optional sources
- `arch_blocks.morning_brief.pending_adapters` (string[]) — written Step 1, Step 3; sources the learner wants but hasn't connected yet
- `arch_blocks.morning_brief.first_fire_verified` (boolean) — written Step 9; true only after primary channel delivery confirmed

Read-only dependencies: `arch_blocks.goals.status`, `arch_blocks.calendar.status`, `arch_blocks.tasks.status`, `arch_blocks.email.status`, `arch_blocks.telegram.status`, `arch_blocks.basic_vibecoding.status`, `arch_blocks.github_setup.status`, `learner_profile`.

Read-write dependency: `mental_models_taught` — read to check which MMs are already taught; written when new MMs are taught during this flow.

On entry: read `learner-state.json`. Check `last_completed_step` and resume from the next step. Save state as the flow progresses — write relevant fields at each milestone, not batched to the end.

## Constraints

1. **Read-only compositing.** The brief reads the learner's calendar, tasks, and goals. It never creates, modifies, or deletes events, tasks, or goals. Route mutations to the relevant skill.
2. **Access inherits upstream.** Adapter scopes come from pos-calendar / pos-email / pos-telegram. This skill does not reconfigure them. Personal/work separation preserved from upstream.
3. **Secrets stay secure.** Tokens, SMTP creds, bot tokens, VPS keys live in tool-native stores (keyring, env-file, bot platform) — never in the vault, never in git-tracked directories.
4. **No OpenClaw cron for scheduling.** Even when OpenClaw is a delivery channel, scheduling lives on a system substrate.
5. **Staleness insertion is unconditional.** When life-goal or priority staleness crosses its threshold (weekly 10d, monthly 40d, life goals 60d), the brief MUST include a dedicated staleness line regardless of chosen composition. The learner cannot opt out of staleness warnings — they are safety, not content.
6. **No silent delivery failure.** Failures reach the learner via the declared alert surface. Output suppression forbidden.
7. **No proactive two-way Telegram.** The brief is one-directional by default. Two-way TG interaction is mentioned only if the learner explicitly asks.
8. **No inline goals fallback.** If goals are not done, the only path is hard route to `/pos-goals` (или `/skill:pos-goals` в Codex). No "let me quickly ask about goals" shortcut.
9. **Learner controls content.** If the learner redirects away from the agent's starter offer, the agent follows — even if dropping starter-default items. No overriding.
10. **No silent adapter selection.** Which sources the brief reads and which channels it delivers to are both confirmed with the learner.
12. **Audit third-party code.** Before installing any scheduler helper, delivery library, or dependency: check for known vulnerabilities and inspect source code. Report findings to the learner.
13. **No mandatory priority capture.** Promote weekly/monthly priority capture, don't force it. Declining is a valid outcome.
14. **Priorities are read, not stored.** Morning-brief reads priorities from the goals state/artifact in the vault. It does not persist its own copy. `priorities_mode` records whether the learner engaged with priorities, not the priorities themselves.

## Flow

### Step 1 — Prerequisites and entry

Check prerequisites:
- **Hard:** `arch_blocks.goals.status == "done"`, `arch_blocks.calendar.status == "done"`, `arch_blocks.tasks.status == "done"`. Check in this order, stop on the first miss. Tell the learner in one Russian sentence what's missing and which `/pos-*` (или `/skill:pos-*` в Codex) command to run. Write `pending_resume = "pos-morning-brief"` before handing off. Stop.
- **Hard:** `learner_profile` must exist. If absent, route to `/pos-diagnostic` (или `/skill:pos-diagnostic` в Codex).
- **Soft:** `arch_blocks.github_setup.status` — needed for the automation project repo. If not done, nudge: automation artifacts (scripts, prompts, systemd units) live in a version-controlled project so the learner can track changes and recover from breakage. If the learner opts in, write `pending_resume = "pos-morning-brief"`, route to `/pos-github-setup` (или `/skill:pos-github-setup` в Codex), stop. If they decline, proceed — artifacts will be created locally without a repo.
- **Soft:** `arch_blocks.email.status` and `arch_blocks.telegram.status` — note which are available as optional sources. If neither is connected, the brief still runs on calendar + tasks.
- **Soft:** `arch_blocks.basic_vibecoding.status` — needed for the pipeline build (scripts, prompts, systemd units are vibe-coding work). If not done, nudge: the build step will create small automation scripts, which is easier after learning the basics. If the learner opts in, write `pending_resume = "pos-morning-brief"`, route to `/pos-basic-vibecoding` (или `/skill:pos-basic-vibecoding` в Codex), stop. If they decline, proceed — the agent handles the build using whatever coding skills/superpowers are available.
- **Resume:** If `status == "in_progress"`, read `last_completed_step`, tell the learner where they left off, pick up from the next step. If `status == "incomplete"`, check which end-state items are missing and resume at the relevant step. If `status == "done"`, summarize current setup and offer to tweak or rebuild.

Deliver verbatim in Russian (fresh start only):

> В этом блоке мы соберём утренний бриф — это короткая сводка на начало дня. Что в календаре, какие задачи в фокусе, как день соотносится с твоими целями. Он собирается автоматически из того, что уже подключено, и приходит туда, куда ты скажешь — в Telegram, на почту, или прямо в агента. Один раз настроим — дальше он приходит сам.

Then confirm which optional sources (email, telegram) are available and which the learner wants to include. If the learner asks for a source that isn't wired, offer: continue without it, go set it up first, or mark it for later (add to `pending_adapters`).

Write: `status = "in_progress"`, `adapters_used`, `pending_adapters`, `last_completed_step = 1`.

### Step 2 — Goals, priorities, and freshness

Read the goals artifact from wherever `pos-goals` documented it (stored `goals_path` and `horizons_path` in `arch_blocks.goals`). Do not invent a fallback path. Compute staleness from the `updated:` frontmatter date inside each file — not from the filesystem modification time. Life-goals staleness = age of `goals.md`'s `updated:` field. Priority staleness = age of `horizons.md`'s `updated:` field (weekly and monthly thresholds apply to their respective sections). Apply the normative thresholds (weekly 10d, monthly 40d, life goals 60d).

Teach `write-or-not-exist` (if not already taught): what isn't written into an agent-readable surface doesn't exist for the agent. The goals file is real text the brief can lean on; without it the brief can only describe the day, not evaluate it.

Then teach `more-context-more-delegation` (if not already taught): the more stable context the agent has (goals, current week, current month), the more accurately it can help without guessing.

Promote weekly and monthly priority capture. The learner chooses: both horizons, one, or neither. Declining is valid — say so explicitly. If captured, collect 1-3 items per horizon in the learner's own words. Priorities are written into the goals artifact (the source `pos-goals` owns), not stored by morning-brief.

Explain: even if priorities are declined, stale goals or priorities will always trigger a staleness line in the brief. This is not a menu item — it's a safety net.

Write: `priorities_mode`, `last_completed_step = 2`. Write MMs to `mental_models_taught` if newly taught.

### Step 3 — Delivery channels

Detect which delivery channels are currently reachable. Standard candidates are Telegram and email — always offer these. Beyond that, discover at runtime what other delivery surfaces exist on the learner's system (e.g., OpenClaw, other agent runtimes, bot frameworks) — only offer what is actually installed. Do not assume any specific tool is present. Present findings to the learner with one line per candidate: ready / needs setup / not available. Use these Russian explanations for standard channels:
- **Telegram** — сообщение от бота
- **Email** — письмо на почту
For discovered channels, explain what they are and how delivery would work in plain Russian.

If no channel is reachable at all, offer handoff to the relevant skill (e.g., `/pos-basic-vibecoding` (или `/skill:pos-basic-vibecoding` в Codex) for TG bot, `/pos-email` (или `/skill:pos-email` в Codex) for email). Write `pending_resume = "pos-morning-brief"` and stop.

If the learner picks Telegram delivery but no send-capable bot exists and basic-vibecoding isn't done, hand off to `/pos-basic-vibecoding` (или `/skill:pos-basic-vibecoding` в Codex) with `pending_resume`.

Learner chooses channel(s). If multiple, ask which is primary (the one verified in the wow moment).

Teach `learner-controls-content` (if not already taught): the agent proposes a starting composition, but the learner decides the actual content.

Write: `delivery_channels`, `delivery_primary`, `pending_adapters` (if updated), `last_completed_step = 3`. Write MM if newly taught.

### Step 4 — Brief composition

Present a starter composition using canonical slugs. The agent shows Russian labels but writes these exact slugs to `composition_chosen`:

**Starter items (included by default):**
- `events` — события дня
- `tasks` — задачи
- `focus` — фокус дня
- `alignment` — сверка с целями

**Tier-up items (offered based on available sources):**
- `day_shape` — форма дня (распределение по блокам)
- `dont_miss` — не пропусти (важное, что легко упустить)
- `meeting_prep` — подготовка к встречам
- `over_scheduling` — перегрузка (алерт если день забит)
- `email_digest` — дайджест почты (requires email adapter)
- `tg_highlights` — важное из Telegram (requires telegram adapter)

The learner picks by number, adds custom items, or subtracts from the starter set. Reconcile with available sources — if the learner picks something that needs an unwired source, handle the same way as Step 1 (continue without, handoff, or defer).

Write: `composition_chosen` (array of canonical slugs), `adapters_used` (updated if composition needs new sources), `last_completed_step = 4`.

### Step 5 — Time and timezone

Fix the daily delivery time. Detect the learner's timezone if possible. Confirm or let them override. Collect the time in HH:MM.

Write: `schedule_time`, `schedule_timezone`, `last_completed_step = 5`.

### Step 6 — How it works + pipeline build

Before building anything, explain the mechanics in plain Russian: the brief is a data pipeline. A script runs at the chosen time, pulls fresh data from connected sources (calendar events, tasks, goals artifact, and any optional sources the learner chose), feeds it to an LLM with a prompt that composes the brief according to the learner's composition, and delivers the result to the chosen channel(s). The scheduler is what triggers this pipeline on time.

Then research the appropriate scheduling substrate at runtime based on OS, VPS presence, delivery channel, and existing infrastructure. Present 1-2 viable options with plain Russian explanations and tradeoffs. Never offer OpenClaw cron. Get the learner's choice.

Show the exact installation plan before activating: unit name, fire time, removal command, file location. Get explicit confirmation before building anything.

If `github_setup` is done: create the automation artifacts inside a `pos-automations` repo (or reuse if it already exists) under a `morning-brief/` subdirectory — runner script, LLM composition prompt as a separate file, systemd units. Before any `git add`, create a `.gitignore` in that subdirectory listing env files, token files, and any secret paths. Then: if `pos-automations` does not yet exist as a git repo, run `git init`; then `git add` and `git commit`. Push to a private GitHub remote and verify with `git log --oneline` or `gh repo view`. Do not write `last_completed_step = 6` until the commit is confirmed. If `github_setup` is not done: create the artifacts locally (e.g., `~/pos-automations/morning-brief/`). Local artifact creation is sufficient — no git required.

The build is vibe-coding work. Use superpowers or any installed coding skills available in the agent's runtime for building the pipeline scripts and configs. The learner watches and confirms, but the agent does the coding.

Build the scheduler and the delivery pipeline. Configure the daily schedule only — first-fire test comes later after the learner has seen a preview. Verify: scheduler exists, delivery mechanism can produce output, logs are being written. On failure: name what broke in plain Russian, offer retry / switch substrate / pause with diagnostic trail.

Write (after commit or local artifact verified): `schedule_substrate`, `last_completed_step = 6`.

### Step 7 — Dry-run preview

Run the pipeline once right now — compose the actual brief from today's real data using the learner's chosen composition. Show the full brief text directly in the dialog (not as a file the learner has to open). The learner sees exactly what they'll get every morning, right here in the conversation.

After showing the preview, ask: does this look right? Too much, too little, wrong tone? If the learner wants to adjust composition or formatting, loop back to adjust before proceeding. This is the cheapest place to iterate.

Write: `last_completed_step = 7`.

### Step 8 — Observability

Teach `scheduled-not-delivered` (if not already taught): the scheduler can fire successfully while the learner sees nothing. Without a separate alert surface, silence looks like "nothing important today" when delivery actually failed.

Declare the observability surface with the learner: (a) where the log lives, (b) which channel gets failure alerts — valid options are: distinct channel (separate from primary), log file, or same channel with error prefix if only one channel is available, (c) retry policy — choose a retry policy with the learner; recommend a substrate-appropriate default (e.g., systemd restart-on-failure semantics differ from a cron retry wrapper).

Write: `last_completed_step = 8`. Write MM if newly taught.

### Step 9 — First live fire (wow moment)

The learner has seen the preview and approved the pipeline. Now schedule a near-future test fire to verify end-to-end delivery on the primary channel.

Agree on a near-future first-fire time (within 30 minutes — e.g., 3 min, 5 min, or learner's choice). Schedule it on the chosen substrate without destroying the daily schedule.

Re-read the goals artifact to refresh staleness data. Ensure the brief respects the chosen composition and that staleness lines are inserted when thresholds are crossed.

Wait for delivery and verify on the primary channel with the learner. If secondary channels fail, note it but don't block — surface via alert channel. If primary fails: name what broke in plain Russian, offer retry / switch channel / diagnose from log.

Write: `first_fire_time`, `first_fire_verified = true`, `last_completed_step = 9`.

### Step 10 — Wrap up

Set `status = "done"` only if `first_fire_verified` is true. Otherwise set `status = "incomplete"` and tell the learner what's missing and how to finish.

If `pending_adapters` is non-empty, recommend the next skill to wire that source. Otherwise recommend the next downstream skill — read course path from `learner-state.json`, cross-reference with completed blocks, name one specific block and slash command.

Write: `status`, `completed_at` (if done), `last_completed_step = 10`.

## Rules

1. **Language and tone.** Speak Russian to the learner. Use `ты`. Plain, calm, short. All user-visible text is Russian — no English leaking in status messages.
2. **Silent state.** State reads and writes are invisible. Never show JSON field names, keys, or dot-paths to the learner.
3. **Concept before jargon.** Before naming `systemd`, `cron`, `keyring`, `SMTP`, `OpenClaw` — explain the concept in plain language first.
4. **Transparency before action.** Before any build step that touches disk, network, or services, tell the learner in one short Russian sentence what's about to happen.
5. **Present -> confirm -> write.** Config files, scheduler installations — show proposed content, get confirmation, then write.
6. **This skill composes, not configures.** If a prerequisite adapter needs reconfiguration, hand off to that skill. Do not reconfigure calendar, email, or telegram inline.
7. **Feedback on demand only.** If the learner wants to leave feedback, route to `/pos-feedback` (или `/skill:pos-feedback` в Codex). No front-loaded reminders.
