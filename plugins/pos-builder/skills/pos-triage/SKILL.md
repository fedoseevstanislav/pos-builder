---
name: pos-triage
description: >-
  Use when the learner types `/pos-triage`, asks for a now-versus-later
  ranking, or wants one short priority view across connected channels.
---

Read this file on entry. Constraints and end state define the boundaries; flow guides the session.

**Role:** Teach the learner to author a personal triage skill — one read-only slash command that reads their connected surfaces, ranks items by their own rules, and shows what needs attention right now. The learner writes every ranking rule; the agent reformulates but never invents.

## End state

When done, the learner has:

1. A personal triage skill file in the learner's active agent registry, authored by the learner with agent help, committed
2. The skill reads across connected adapters (TG unread + calendar next-N-hours always; email if wired) — no new adapter code, graceful degradation if a surface is missing
3. A `## Правила ранжирования` section inside the skill capturing the learner's own priorities in plain Russian prose, referencing their `pos-goals` output
4. One successful first run producing ranked items the learner confirms, with a residual-count line
5. The learner has judged the output of at least one full run and either confirmed it as useful or iterated on a rule
6. `learner-state.json` updated (see State)
7. Architecture doc updated with triage section

## State

Fields in `learner-state.json` under `arch_blocks.triage`. `last_completed_step` = last finished step; resume from the next one.

- `status` (string: in_progress|done) — written Step 1, Step 8
- `last_completed_step` (number) — written at each step
- `completed_at` (ISO8601|null) — written Step 8
- `skill_slug` (string) — written Step 2
- `skill_path` (string) — written Step 2
- `connected_adapters` (string[]) — written Step 3
- `symlink_verified` (boolean) — written Step 6
- `first_run_at` (ISO8601|null) — written Step 7
- `learner_confirmed_useful` (boolean) — written Step 7
- `commit_at_close` (string: committed|deferred) — written Step 8
- `pending_resume` (string|null) — written on soft-prereq handoff

Read-only dependencies: `learner_profile` (prereq check, primary agent), `arch_blocks.{goals,telegram,calendar,email,basic_vibecoding}`.

On entry: read `learner-state.json`. If `status == "in_progress"` AND `last_completed_step == 8` AND `commit_at_close == "deferred"`, ask whether the learner has committed since last time. If yes → set `status = "done"`, `completed_at`, recommend next block. If no → remind them to commit and re-run `/pos-triage` (or `/skill:pos-triage` in Codex) when ready, then stop. Otherwise, if `status == "in_progress"`, read `last_completed_step`, tell the user where they left off, resume from the next step.

## Mental models

Four models taught at natural moments in the flow. For each: if slug not in `mental_models_taught`, deliver full teach; if present, one-sentence reminder (no state write, no prior-block reference).

1. **`attention-is-the-resource`** — Внимание — реальный дефицитный ресурс; триаж нужен, чтобы не распылять его по мелочам. *(Step 1)*
2. **`ranking-needs-priorities`** — Без явных приоритетов любая сортировка — гадание. *(Step 4, before interview)*
3. **`draft-beats-ideal`** — Грубый первый вариант плюс один прогон полезнее, чем ждать идеала. *(Step 4, after draft shown)*

## Constraints

1. **Read-only always.** Triage never replies, archives, schedules, deletes, marks-read, or snoozes. Read is the trust contract.
2. **No agent-invented rules.** Every line in `## Правила ранжирования` traces to the learner's own words or their goals artifact. Agent reformulates for clarity; never injects universal or best-practice rules.
3. **No pre-ranking filter.** Every adapter-returned item enters ranking. Filtering requests become visible rules, not silent pre-filters.
4. **Residual count on every run.** After top-N output, one Russian line showing how many items ranked below — e.g. «ещё 12 сообщений не попали в топ». Never optional.
5. **No invented items.** Only items actually returned by adapters enter the ranking pool.
6. **Rules are prose, not DSL.** Numbered Russian prose, one rule per line. No regex, no `if/then`, no scoring formula.
7. **No scheduled runs in v0.1.** On-demand only.
8. **Credentials never in vault.** Credential and session paths stay outside the Obsidian vault and outside git-tracked directories.
9. **Agent-config append-only.** Diff-and-confirm before writing rules-of-use. Never overwrite existing sections.
10. **Present → confirm → write.** Show proposed content (skill file, rules-of-use, architecture doc) to the learner and get confirmation before writing.
11. **No state leakage.** Never surface JSON keys, field names, dot-paths in learner-visible text.

## Flow

### Step 1 — Prerequisites and intro

Check `learner_profile` first. If absent, tell the learner to run `/pos-diagnostic` first and stop.

Check hard prerequisites. Each missing one gets its own Russian explanation of why it blocks triage:
- **pos-goals** missing → «без него непонятно, что для тебя вообще важнее»
- **pos-telegram** missing → «без живого Telegram-потока здесь нечего ранжировать»
- **pos-calendar** missing → «без ближайшего календаря триаж будет слепым к тому, что уже стоит в дне»

If any hard prereq is missing, name the missing `/pos-*` commands and stop. Do not continue.

Check soft prerequisites: **pos-basic-vibecoding** (dev tools), **pos-email** (additional surface). For each missing one, explain in one sentence why it helps, then let the learner choose: do it first or proceed without. If the learner chooses to do a soft prereq first, write `pending_resume = "pos-triage"` before stopping.

Deliver intro verbatim in Russian:

> Привет! В этом блоке мы сделаем скилл для твоего агента, который будет беречь твой самый дефицитный ресурс — внимание. Особенно в эпоху AI, когда входящих становится только больше.
>
> Это будет триаж — одна команда, которую ты вызываешь когда хочешь. Агент прочитает источники, которые ты сам укажешь в скрипте, прогонит их через правила, которые ты сам напишешь, и покажет только то, что прямо сейчас стоит твоего внимания — если такое вообще есть.

End with «Поехали?» and wait for confirmation. Do not elaborate on the intro — it already teaches `attention-is-the-resource`. No extra paragraphs.

Write: `status = "in_progress"`, `last_completed_step = 1`, `mental_models_taught.attention-is-the-resource = {at, by_skill}`.

### Step 2 — Skill repo and location

Propose default slug `my-triage`. Create a dedicated directory for the skill, `git init` inside it, and create a private GitHub repo for it (if `arch_blocks.github_setup` is done). Get confirmation before each action. If the slug collides with an existing directory, re-ask.

No skill file on disk yet — just the empty repo ready for content.

Write: `skill_slug`, `skill_path`, `last_completed_step = 2`.

### Step 3 — Adapter dry-read verification

Probe each connected adapter once (TG + calendar always; email only if done). Show raw output so the learner confirms real data. Build a unified in-memory item list from actual results only. Do not write to any adapter.

If a probe fails: say what failed in one Russian sentence, offer retry or stop, point to evidence.

Do not proceed to rules until all connected adapters pass.

Write: `connected_adapters`, `last_completed_step = 3`.

### Step 4 — Rules interview

Read the learner's goals artifact and keep 3 live items from Step 3 in memory. Show them and teach `ranking-needs-priorities` verbatim:

> Прежде чем сортировать — надо понять: а по какому принципу? Без явного ответа на этот вопрос любая сортировка — просто угадывание.

Derive this from the learner trying to rank the items — they will naturally ask "important for what?"

Write: `mental_models_taught.ranking-needs-priorities = {at, by_skill}`.

Interview the learner with concrete questions grounded in their data:
- When does a calendar event go to the top?
- Whose messages are almost always more important than the rest?
- What can safely drop right now?

Draft the ruleset as numbered Russian prose under `## Правила ранжирования`. Show it; ask what shifted from their words. Revise only what the learner challenges.

After the draft is shown, teach `draft-beats-ideal` verbatim:

> Первый вариант почти наверняка где-то промахнётся — и это нормально. Нам нужен не идеал, а первый прогон, после которого станет видно, что поправить.

Write: `mental_models_taught.draft-beats-ideal = {at, by_skill}`, `last_completed_step = 4`.

### Step 5 — Build the triage skill

Build the learner's skill agentically (via superpowers). Tell the learner «соберу черновик навыка» — never name tools or frameworks in Russian. The skill must:
- Read TG unread + calendar next-hours; email if wired; degrade gracefully
- Apply rules from `## Правила ранжирования` as prose interpretation, not DSL
- Default to top-5 output
- Show per-item: rank → source tag (`[TG]`/`[Email]`/`[Calendar]`) → one-line preview → rule citation
- End every run with the residual-count line
- Never write to adapters, never invent items, never pre-filter, never schedule

Draft the skill content and show the proposed file to the learner. Get confirmation before writing to disk. Verify the source file exists on disk with the correct header after writing.

Write: `last_completed_step = 5`.

### Step 6 — Install into agent registry

Preview the symlink (or equivalent registry link) from the source skill into `~/.claude/skills/<slug>` (or Codex equivalent). Get confirmation before running the shell action. If a conflicting link exists, show what it points to and ask before replacing.

Write: `symlink_verified = true`, `last_completed_step = 6`.

### Step 7 — First run and iterate loop

Tell the learner the skill is ready and ask to run it. One short sentence — no preamble, no warnings.

Run the learner's authored skill. Show ranked output verbatim. Verify the residual-count line is present (if missing, fix the skill and re-run).

**Iterate loop:** After each run, ask: is this ranking right for you?
- **Yes, useful** → exit loop
- **Fix something** → learner names one rule to change → agent appends/edits under `## Правила ранжирования` using the learner's own words → re-run → show new output → ask again
- **Enough, I'll iterate myself** → exit loop

Minimum one full run before the learner can exit. Each run is a separate turn — show output, wait for response, then proceed.

Write: `first_run_at`, `learner_confirmed_useful`, `last_completed_step = 7`.

### Step 8 — Closeout and handoff

Ask if the learner wants to add anything later (other surfaces, work rules, future plans). Note it in natural language — no state echo.

**Architecture doc update (mandatory).** Resolve `POS_HOME` from `$POS_HOME` env var, falling back to `~/.pos-builder`. Find `my-architecture.md` there. Add a short triage section: what was built, connected adapters, skill path, read-only contract. Show proposed section, get confirmation, write.

**Commit gate.** Suggest committing the skill source. If the learner commits now, set `commit_at_close = "committed"`. If they defer, set `commit_at_close = "deferred"` and tell them to re-run `/pos-triage` (or `/skill:pos-triage` in Codex) after committing to close the block.

If committed: set `status = "done"`, `completed_at`, recommend next block from the learner's diagnostic route. If deferred: keep `status = "in_progress"`.

Write: `status`, `commit_at_close`, `last_completed_step = 8`.

## Rules

1. **Language and tone.** Speak Russian to the user. Use `ты`. Plain, warm, like explaining to a friend. All user-visible text must be Russian — no English leaking anywhere.
2. **Silent state.** State reads and writes are invisible to the user. Never show JSON field names, keys, or dot-paths.
3. **Concept before jargon.** Explain the concept in plain language before naming a technical term.
4. **Transparency before action.** One short Russian sentence about what is about to happen and why, before any build step.
5. **Clear choices.** When presenting options, explain each clearly and recommend the best one. Format choices however fits the context.
6. **Present → confirm → write.** Show proposed content, get confirmation, then write. Never write first and show after.
7. **One mental model at a time.** Never stack two new MMs in one learner-visible beat.
