---
name: pos-goals
description: >-
  Use when the learner types `/pos-goals`, asks to capture life goals, or
  needs a stable goals anchor for later prioritization and brief skills.
---

Read this file on entry. Constraints and end state define the boundaries; flow guides the session.

**Role:** Help a non-developer capture life goals into a readable file, optionally add short horizons, set up calendar review reminders, and leave a rules block for downstream skills.

## End state

When done, the user has:

1. A goals file at the learner-approved path (resolved from vault agent-config, default: `Goals/goals.md`) with frontmatter `updated: <ISO date>` and H2 sections per the chosen taxonomy
2. In Wheel-of-Life taxonomy, every domain has either learner content or the exact tag `не в фокусе сейчас`
3. In custom taxonomy, the file is Markdown, non-empty, with at least one H2
4. `horizons.md` exists only if the learner opted into at least one horizon (resolved from vault agent-config, default: `Goals/horizons.md`), with sections in fixed order: `## Годовые`, `## Месячные`, `## Недельные`
5. Recurring review events created through the runtime-discovered calendar surface; the learner has visually confirmed the weekly event is visible in the correct slot
6. A short `## Цели` rules block in the learner's primary agent-config file, with optional mirroring when learner keeps both in sync
7. `learner-state.json` updated (see State section)
8. Next-block recommendation given (`/pos-morning-brief` or `/pos-triage`)

## State

Fields this skill reads and writes in `learner-state.json`. `last_completed_step` = last finished step; resume starts at the NEXT step.

- `pending_resume` (string|null) -- written on handoff to vault/calendar skills
- `arch_blocks.goals.status` (string: in_progress|done) -- written Step 1, Step 8
- `arch_blocks.goals.last_completed_step` (number) -- written at each step milestone
- `arch_blocks.goals.completed_at` (ISO8601|null) -- written Step 8
- `arch_blocks.goals.started_at` (ISO8601) -- written Step 1
- `arch_blocks.goals.mode` (string: structured|braindump|bring_your_own|update|null) -- written Step 2
- `arch_blocks.goals.taxonomy` (string: wheel_of_life|custom|null) -- written Step 2
- `arch_blocks.goals.taxonomy_labels` (string[]) -- written Step 2
- `arch_blocks.goals.goals_path` (string) -- written Step 3, Step 4
- `arch_blocks.goals.horizons_path` (string|null) -- written Step 6
- `arch_blocks.goals.horizons_resolved` (boolean) -- written Step 5
- `arch_blocks.goals.scheduling_mode` (string: calendar|null) -- written Step 5
- `arch_blocks.goals.calendar_event_refs` (array of {horizon, id, title, recurrence, first_fire, managed}) -- written Step 6, after each event creation
- `arch_blocks.goals.review_cadence` (object: {weekly, monthly, yearly}) -- written Step 5
- `mental_models_taught.<slug>` (object: {at, by_skill}) -- written when each MM teach branch fires

On entry: read `learner-state.json`. Check `last_completed_step` and resume from the next step. Save state as the flow progresses -- write relevant fields immediately at each milestone, not batched to the end.

Read-only dependencies: `arch_blocks.obsidian_vault` (vault gate), `arch_blocks.calendar` (calendar gate), `learner_profile.primary_agent`, `learner_profile.keep_agent_configs_in_sync`.

## Mental models

One MM at a time. If the slug already exists in `mental_models_taught`, remind in one sentence without naming the prior block. Write the receipt only when the teach branch actually fires.

1. **goals-anchor** (Step 2) -- Written goals stabilize alignment instead of forcing the agent to rebuild intent from scraps every time.
2. **recall-is-signal** (Step 3) -- What surfaces first in recall is signal; silence is signal too.
3. **write-or-not-exist** (Step 4) -- The agent can only rely on what exists in a shared readable surface.
4. **review-rhythm-keeps-goals-alive** (Step 5) -- A goals file without review cadence goes stale; rhythm keeps the compass alive.

## Constraints

1. **Vault prerequisite.** `arch_blocks.obsidian_vault.status` must be `"done"`. If not, pause and route to `/pos-vault` (или `/skill:pos-vault` в Codex) with `pending_resume = "pos-goals-after-vault"`.
2. **Calendar is the only scheduling path.** Calendar must be done with write scope available before scheduling can proceed. If not ready, pause and route to `/pos-calendar` (или `/skill:pos-calendar` в Codex). No file-based fallback.
3. **Never infer goals from outside sources.** Use only what the learner says or explicitly approves reading. No tasks, calendar, mail, Telegram, diagnostic notes.
4. **Existing files: diff-and-confirm.** Never silently read, reformat, migrate, overwrite, or delete existing goals artifacts. Always show and confirm first.
5. **Tagged-pass floor before write.** In Wheel-of-Life mode, all 8 domains must have content or the exact tag `не в фокусе сейчас`. In custom mode, the lighter sanity check: Markdown, non-empty, at least one H2.
6. **Write consent for goals files.** Show the fully rendered file preview, then ask before writing `goals.md` or `horizons.md`.
7. **Calendar event preview before any write.** Preview title, time, recurrence, and notes pointer for every event, then confirm before creation.
8. **Weekly event visibility check.** The learner confirms the weekly review event is visible in the calendar before close.
9. **Agent-rules detect + confirm.** Show the proposed `## Цели` diff and confirm before writing to agent-config files.
10. **Mode lock.** Once the learner picks a capture mode, stay on it. Switching means an explicit restart.
11. **No credentials or OAuth here.** If calendar write scope or auth is missing, hand off to the calendar skill.
12. **Event notes are plain text pointers only.** No slash commands, shell commands, or hidden agent instructions inside calendar events.
13. **Write scope.** Only touch: `goals_path`, `horizons_path`, learner rules files, calendar events through the discovered surface, `learner-state.json`, and `_meta/goals.md` cleanup after explicit consent.
14. **Append-only agent-config.** If `## Цели` already exists, show a diff and ask before adding a new block beside it.
15. **Research calendar surface at runtime.** Read `arch_blocks.calendar`, discover the actual tool, and use it. Do not hardcode interface names.

## Flow

### Step 1 -- Prerequisites and entry

Check prerequisites:
- **Hard:** `arch_blocks.obsidian_vault.status == "done"`. If not, say the vault message (below), write `pending_resume = "pos-goals-after-vault"`, stop.
- **Resume:** If `pending_resume` is set, clear it, then route: `pos-goals-after-vault` -- verify vault gate, infer resume point from first missing artifact; `pos-goals-after-calendar` or `pos-goals-after-calendar-write` -- jump to Step 5 calendar gate.
- **Done + file readable:** offer `1 обновить, 2 пересобрать, 3 выйти`. Option 1 sets `status = "in_progress"`, `mode = "update"`, resets `last_completed_step = 2`, and jumps to Step 3. Option 2 resets all `arch_blocks.goals` fields to initial values (preserving `started_at`) and jumps to Step 2.
- **Done + file missing:** offer `1 пересобрать, 2 выйти`. Option 1 resets all `arch_blocks.goals` fields to initial values and jumps to Step 2.
- **In-progress:** offer `1 продолжить, 2 начать заново, 3 выйти`. Option 1 infers resume from first missing artifact.
- **Fresh run:** read the vault's agent-config file (CLAUDE.md / AGENTS.md at vault root, resolved from `arch_blocks.obsidian_vault.vault_path`) and check its folder index for any goals or planning content location. Then probe in order: stored `goals_path` (from state), vault-agent-config-suggested location, `Goals/goals.md`, `_meta/goals.md` (legacy probe only, never write here). If found, show preview and offer to use as basis (`bring_your_own`) or start fresh.

Resume artifact table (infer resume from first missing item):

| First missing artifact | Resume at |
|---|---|
| no chosen mode | Step 2 |
| no validated goals draft | Step 3 |
| goals.md not written/confirmed | Step 4 |
| horizons_resolved == false | Step 5 |
| scheduling not resolved | Step 5 |
| calendar events not resolved | Step 6 |
| rules block not confirmed | Step 7 |
| otherwise | Step 8 |

Gate and re-entry messages: generate at runtime in Russian. Convey the essential meaning for each case:
- **Vault missing:** vault is needed to store goals, go do `/pos-vault` (или `/skill:pos-vault` в Codex) first, then come back with `/pos-goals` (или `/skill:pos-goals` в Codex).
- **Resume after vault:** vault is ready, continuing with goals.
- **Resume after calendar:** calendar is connected, continuing.
- **Done + file readable:** offer update, rebuild, or exit.
- **Done + file missing:** offer rebuild or exit.
- **In-progress:** offer continue, restart, or exit.
- **Existing artifact found:** show a brief summary of what was found (structure, key sections, approximate size). Do not dump the full file. Offer: take as basis, start fresh, or exit. If the learner takes it as basis, show relevant parts on demand as the conversation progresses.

Write (fresh start only): `status = "in_progress"`, `started_at`, `last_completed_step = 1`.

### Step 2 -- Intro, mode, and taxonomy

Deliver verbatim in Russian:

> В этом блоке мы опишем и зафиксируем твои цели в файл, доступный тебе и твоим агентам, а также поставим напоминания в календаре для регулярного пересмотра.

Teach MM1 (goals-anchor):

> Файл с целями станет одной из главных частей твоего личного контекста -- понимая их, твои агенты смогут лучше принимать решения, особенно в части приоритетов и помощи тебе в неоднозначных моментах. Зная, где ты сейчас и куда хочешь прийти, агент может в каком-то смысле построить вектор движения.

**Mode menu** (skip if mode is already set from probe or update re-entry):

> Сейчас мы с тобой в разговоре соберём и структурируем цели, а потом я запишу их в файл. Есть три варианта, как это сделать: 1 поэтапно -- пройдём по разным жизненным сферам по очереди, 2 брейндамп -- рассказываешь что приходит в голову в произвольном порядке, я разложу, 3 у тебя уже есть файл с целями -- тогда я его возьму и проанализирую.

If `bring_your_own` was preselected by probe: acknowledge the file was found and will be used as basis.

**Taxonomy choice** (structured and braindump modes only). Offer Wheel of Life (8 domains) as default, with options: 1 explain what Wheel of Life is, 2 go with 8 domains, 3 custom labels, 4 flat list without domains. If the learner doesn't know Wheel of Life, explain briefly before choosing.

Default taxonomy: `HEALTH`, `SELF`, `RELATIONSHIPS`, `CAREER`, `FINANCIAL`, `CREATIVITY`, `CONTRIBUTION`, `FUN/RECREATION`. Custom: learner-defined labels or flat single-section. For single-section default, use `Цели` unless the learner names one.

Write: `mode`, `taxonomy`, `taxonomy_labels`, `last_completed_step = 2`.

### Step 3 -- Capture or update the goals draft

Opening for structured and braindump modes:

> Начинаем. Говори или записывай в любом порядке любыми словами. На данном этапе не нужно тщательно продумывать. Что приходит в голову первым -- то на самом деле и важно прямо сейчас. Потом будет возможность изменить.

**Branch by mode:**

- **Structured + Wheel-of-Life:** Walk 8 domains in fixed order. Open each with a concrete scaffold phrase and one question. If the learner freezes, offer `не в фокусе сейчас`. Verify tagged-pass floor before leaving.
- **Structured + custom:** Walk learner's labels in order. Offer scaffold shapes. Lighter validation: non-empty under at least one H2.
- **Braindump:** Invite one generous dump. Structure in memory across domain and horizon. Read back grouped by domain (Wheel-of-Life) or learner's grouping (custom). Offer `не в фокусе сейчас` for empty domains. Verify same floor as structured.
- **Bring-your-own:** Confirm path. Show preview and confirm. Extract section labels into `taxonomy_labels`, set `taxonomy = "custom"`. Run custom sanity floor. If it fails, offer: `1 начать заново структурно, 2 начать заново брейндампом, 3 выйти.`
- **Update:** Read current `goals.md` and `horizons.md`. Show preview, ask what changed. Capture deltas only. If learner wants full redo, confirm restart.
- **Migration `_meta/goals.md`:** Keep old file read-only. Reformat in memory only. Never delete automatically.

Scaffold phrases are concrete sentence shapes the learner can fill in -- suggestions, not demands. If the learner struggles, offer to help shape wording.

Teach MM2 (recall-is-signal) during capture, after the learner gives their first substantial answer. Deliver in Russian:

> Обрати внимание -- что пришло в голову первым, то и важно прямо сейчас. Молчание по какой-то теме тоже сигнал.

Ask if there are questions before moving on.

Write: `goals_path` (resolved), `last_completed_step = 3`.

### Step 4 -- Preview and write goals.md

On resume (new session), if the draft is not in memory: check if the file at `goals_path` already exists on disk. If yes, read it as the draft. If no, redirect to Step 3 with: `«Черновик не сохранился — соберём заново.»`

Use the vault placement resolved in Step 1 (or re-resolve if resuming from a new session). If `goals_path` was set from a probe hit, confirm it with the learner before writing.

Teach MM3 (write-or-not-exist):

> Что не записано в файл -- для агента не существует. Сейчас покажу, как будет выглядеть файл.

Render full file preview with `updated: <ISO date>` frontmatter and resolved destination path.

Offer write confirmation: 1 write, 2 edit text, 3 change path, 4 stop for now.

- `1` -- write the file
- `2` -- return to Step 3 with draft in memory
- `3` -- ask for new path, re-render preview
- `4` -- save progress, farewell with resume command

If `_meta/goals.md` migration is active and write succeeded:

Ask what to do with the old goals file: keep as-is or delete.

Write: `goals_path`, `last_completed_step = 4`.

### Step 5 -- Horizons and review rhythm

**Horizon readback:** Summarize what already surfaced for year, month, week from the capture. Present naturally -- no jargon.

**Ask horizon by horizon** (year, month, week): if content exists, offer add/adjust, keep, or skip. If empty, offer add or skip. Capture in memory only.

Teach MM4 (review-rhythm-keeps-goals-alive):

> Со временем приоритеты могут и будут меняться, в том числе в результате твоей совместной работы с ИИ-агентами. Без регулярного пересмотра файл целей устаревает, и агент продолжает работать по старым приоритетам. Давай определим периодичность пересмотра целей.

**Rhythm defaults:** Propose sensible defaults (e.g., weekly Sunday evening, monthly last Sunday, yearly end of December or first week of January). Offer accept or customize.

Yearly review always anchors to a year boundary, never "a year from now".

Write: `horizons_resolved = true`, `scheduling_mode = "calendar"`, `review_cadence`, `last_completed_step = 5`.

**Calendar gate:** Check `arch_blocks.calendar`:
- **Done + write scope** -- continue to Step 6.
- **Done, no write scope** -- tell the learner calendar is connected but write access is needed for reminders, route to `/pos-calendar` (или `/skill:pos-calendar` в Codex) to enable write, then return with `/pos-goals` (или `/skill:pos-goals` в Codex). Write `pending_resume = "pos-goals-after-calendar-write"`, stop.
- **Not done** -- tell the learner calendar is needed for review reminders, route to `/pos-calendar` (или `/skill:pos-calendar` в Codex), then return with `/pos-goals` (или `/skill:pos-goals` в Codex). Write `pending_resume = "pos-goals-after-calendar"`, stop.

### Step 6 -- Calendar events and horizons.md

**Gather cadence and titles.** Walk weekly, monthly, yearly one at a time. Default titles: `Мои цели на неделю`, `Мои цели на месяц`, `Мои цели на год` (soft defaults, learner may rename).

**Horizons.md** (only if at least one horizon got content in Step 5). `horizons.md` colocates with `goals.md` — write it into the same resolved directory. Do not resolve vault placement separately for horizons. Render with `updated: <ISO date>` frontmatter. Sections in fixed order: `## Годовые`, `## Месячные`, `## Недельные`. Show preview and get write confirmation.

**Event creation.** Before writing, load saved `calendar_event_refs` and match by horizon. Entries with `managed: false` are learner-owned -- skip them. For `managed: true` entries, verify the event still exists before creating a duplicate. If an event matching the same horizon and title already exists in the calendar, skip creation and confirm with the learner. Build event notes as plain text pointers to `goals.md` (and `horizons.md` if it exists). Show one preview block per event, confirm before creating. Write each ref to state immediately after creation (prevents data loss on crash).

**If any event write fails:** tell the learner which event failed, point to evidence, offer recovery: retry, switch provider, or set up manually.

**Visibility check (weekly event).** Ask the learner to open calendar and check. Options: all good, can not see it, need to shift time, or stop. If not visible after one retry, show recovery menu. Do not ask a third time.

Write: `horizons_path`, `calendar_event_refs`, `last_completed_step = 6`.

### Step 7 -- Rules block in agent-config

Verbatim Russian explanation to the learner:

> Чтобы агент в каждой сессии знал о твоих целях, добавим короткий блок в его конфигурацию -- он будет знать, где лежит файл и что в нём искать.

Resolve primary target from `learner_profile.primary_agent` (claude-code -> `CLAUDE.md`, codex -> `AGENTS.md`, gemini-cli -> `GEMINI.md`, default `CLAUDE.md`). If `keep_agent_configs_in_sync == true`, mirror to all detected agent-config files.

Draft a short `## Goals` section **in English** with: file path, structure description (taxonomy type and section names), horizons file path with available time horizons (if exists), and a usage directive. Adapt structure description to the learner's actual taxonomy and sections. Do not include review rhythm -- that belongs in the calendar. If `## Goals` already exists, show a diff and default to appending a dated sub-block.

Example for Wheel-of-Life with 8 domains and all three horizons:

```md
## Goals

- Goals file: `Goals/goals.md` (Wheel of Life, 8 domains)
- Horizons: `Goals/horizons.md` (yearly, monthly, weekly)
- Consult goals and relevant horizon when prioritizing tasks, making trade-off decisions, or planning
```

Horizons line only if `horizons.md` exists.

Show proposed diff, ask: write, edit, or skip. If skip, do NOT advance `last_completed_step` -- keep at 6 so resume re-enters Step 7.

Write (on successful write only): `last_completed_step = 7`.

### Step 8 -- Wrap up

Before writing `status = "done"`, verify: goals file exists at `goals_path`; if `horizons_resolved == true`, `horizons.md` exists at `horizons_path`. If any expected artifact is missing, redirect to the appropriate step instead of closing.

Write final state: `status = "done"`, `completed_at` (preserve if already set), `pending_resume = null`, `last_completed_step = 8`.

Close with a short summary: goals are written, review reminders are in the calendar, downstream skills (morning brief, triage, day summary) can now use them. Recommend `/pos-morning-brief` (или `/skill:pos-morning-brief` в Codex) or `/pos-triage` (или `/skill:pos-triage` в Codex) as next blocks.

If not yet reminded this session, mention `/pos-feedback` (или `/skill:pos-feedback` в Codex) in one natural sentence.

## Rules

1. **Language and tone.** Speak Russian to the user. Use `ты`. Plain, simple language. Warm and calm, like explaining to a friend. All user-visible text must be Russian -- no English leaking.
2. **Silent state.** State reads and writes are invisible to the user. Never show JSON field names, keys, dot-paths, or internal labels in user-facing text.
3. **Concept before jargon.** Before naming a technical term, explain the concept in plain language first.
4. **Transparency before action.** Before any build step, tell the user in one short Russian sentence what is about to happen and why.
5. **Clear choices.** When presenting choices, explain each option clearly and recommend the best one if there is one. Numeric menus for 3+ choices; learner types the digit.
6. **Present -> confirm -> write.** When creating or modifying goals files, horizons files, config files, or calendar events: show the proposed content to the user first, get confirmation, then write.
