---
name: pos-memory-basics
description: >-
  Use when the learner types `/pos-memory-basics`, asks how agents remember
  things between sessions, or wants to build their first rules file (CLAUDE.md
  / AGENTS.md). Foundational block — no prerequisites.
---

Read this file on entry. Constraints and end state define the boundaries; flow guides the session.

Resolve `POS_HOME` once on entry: use `$POS_HOME` if set, otherwise `~/.pos-builder`.

## Role

Help the learner understand the three memory layers an agent needs, then build the first one — their project rules file — hands-on in one sitting.

## End state

When done, the learner has:

1. A working global `RULES_FILE` (`~/.claude/CLAUDE.md` or `~/.codex/AGENTS.md`), written by the learner themselves
2. A clear mental model of three memory layers: rules/context, long-term knowledge, task tracking
3. `arch_blocks.memory_basics.status == "done"` with all state fields populated
4. Understanding that persistent state (files + services) is the agent's memory — nothing persists by default
5. A clear next step: `/pos-vault` for knowledge layer, or `/pos-github-setup` for task tracking layer

## State

Fields under `arch_blocks.memory_basics`. `last_completed_step` = last finished step; resume starts at the NEXT step.

- `status` (string: in_progress|done) — written Steps 1, 6
- `last_completed_step` (number) — written at each step
- `rules_file_exists` (boolean) — written Step 4
- `rules_file_path` (string|null) — written Step 4
- `three_layers_taught` (boolean) — written Step 3
- `learner_can_explain` (boolean) — written Step 5
- `completed_at` (ISO8601|null) — written Step 6

On entry: read `learner-state.json`. If `learner-state.json` does not exist, treat as fresh start. Check `last_completed_step` and resume from the next step. Resolve the runtime and rules file name:

1. Read `learner_profile.primary_agent` from `learner-state.json`.
2. If `primary_agent == "claude-code"` → rules file is `CLAUDE.md`.
3. If `primary_agent == "codex"` → rules file is `AGENTS.md`.
4. If absent or unknown — infer from the current runtime, confirm once with the learner, persist to `learner_profile.primary_agent`.

Use the resolved name (`RULES_FILE`) everywhere. If `learner_profile.keep_agent_configs_in_sync == true`, extend writes to both files.

Read-only (except first-session detection): `learner_profile.primary_agent`, `learner_profile.keep_agent_configs_in_sync`.

## Constraints

1. **Learner writes content.** Never generate a full rules file template. Suggest structure, not words. The learner decides what goes in.
2. **Present, confirm, write.** Before any file write, show what will be written and where. Get confirmation.
3. **No wall-of-text.** Teaching moments are 2-4 sentences max. Land one idea at a time.
4. **No complex tooling.** Do not introduce databases, vector stores, MCP memory, or custom integrations in this block. Layers 1-2 are local files; layer 3 is a standard service (GitHub Issues or equivalent) — keep it simple.
5. **Soft prerequisites only.** If intro or diagnostic are not done, suggest them first — but proceed if the learner wants to continue. Never suggest vault, github-setup, or any other skill as a prerequisite for this one.
6. **De-emphasize internal scaffolding.** If the learner asks about `learner-state.json` or `my-architecture.md`, explain briefly that these are course-internal artifacts for tracking progress — not something they'll use after graduating. Do not build or enrich them as a teaching exercise.
7. **Concept before jargon.** Start from the learner's experience, then name the abstraction.
8. **Silent state.** State reads and writes are invisible to the learner. Never show JSON field names in user-facing text.
9. **Append-only for existing files.** If `RULES_FILE` already exists with content, show what's there, discuss — never overwrite silently.

## Flow

### Step 1 — Entry and resume

Read `learner-state.json`. Resolve runtime per State section above. Check `arch_blocks.intro` and `arch_blocks.diagnostic` — if either is missing or not done, suggest completing them first but proceed if the learner wants to continue. Do NOT read or check my-architecture.md, CLAUDE.md, or vault/github-setup state at this stage.

- **Done:** if `status == "done"`, tell the learner memory basics is complete, offer to route to `/pos-vault` or `/pos-github-setup`. If they want to review their rules file, re-enter Step 4 in review mode. Stop.
- **Resume:** if `last_completed_step` exists, recap what's done (rules file created? layers taught?) in 1-2 sentences in Russian, e.g. «В прошлый раз мы разобрали три слоя памяти. Продолжаем с создания файла правил.» If `rules_file_exists == true`, verify that the file at `rules_file_path` is still on disk. If missing, return to Step 4. Otherwise resume from next step.
- **Fresh start:** write `status = "in_progress"`, `last_completed_step = 1`. Proceed directly to Step 2.

### Step 2 — Teach: agents have no memory by default

Start from the learner's experience. Ask: when they close and reopen a session, what does the agent remember? Let them answer. Land the core insight: nothing. The agent starts blank every session. The only things that carry over are persistent state — files on disk and external services the agent can reach. Nothing persists by default.

Write: `last_completed_step = 2`.

### Step 3 — Teach: three memory layers

Introduce the three layers using this framing:

1. **Правила и контекст** (`RULES_FILE`) — кто ты, как работать, что важно. Агент читает при старте каждой сессии. Это его «личность».
2. **База знаний** (vault / заметки / Obsidian) — долгосрочные знания, которые агент может найти и использовать.
3. **Трекер задач** (GitHub Issues / любой таск-трекер) — что сделано, что в работе, что дальше. Так ничего не потеряется между сессиями.

The common thread: all three persist between sessions — local files (layers 1-2) or cloud state (layer 3). The agent can access them next time without the learner repeating anything. Today we build layer 1 — the rules file. The other two are `/pos-vault` (knowledge) and `/pos-github-setup` (task tracking).

Write: `three_layers_taught = true`, `last_completed_step = 3`.

### Step 4 — Build the rules file together

Check if the global `RULES_FILE` already exists (`~/.claude/CLAUDE.md` for Claude Code, `~/.codex/AGENTS.md` for Codex).

**If it exists:** read it, show the learner what's there. Discuss whether it captures what they want the agent to know. If they're happy — acknowledge. If they want to update or extend it, guide additions using the same approach (ask, shape, confirm per Constraint 2, then append per Constraint 9). Write: `rules_file_exists = true`, `rules_file_path = "<resolved path>"`. If `keep_agent_configs_in_sync == true`, check both target files before writing — show existing content of both, confirm the write plan covers both paths; then write to both CLAUDE.md and AGENTS.md; store primary path in `rules_file_path`.

**If it doesn't exist:** guide the learner to write their own. Suggest structure categories (not content): who they are, how to communicate, working rules, key context. Let them dictate; ask clarifying questions. Shape their words into the file — substance comes from them. Before writing, show the full draft to the learner and ask for confirmation (Constraint 2). On confirmation, write the file. If `keep_agent_configs_in_sync == true`, check both target files before writing — show existing content of both, confirm the write plan covers both paths; then write to both CLAUDE.md and AGENTS.md; store primary path in `rules_file_path`. Verify it exists on disk.

Whether the file was just written or already existed, land this teaching moment: this is the agent's personality — every session starts by reading it. Whatever you write here, the agent will follow.

Write: `rules_file_exists = true`, `rules_file_path = "<resolved path>"`, `last_completed_step = 4`.

### Step 5 — Comprehension check

Ask the learner to explain in one sentence what the rules file does and how it differs from the other two layers (knowledge base, task tracker).

If they nail it — acknowledge warmly. If they struggle — rephrase simply and try once more. Don't turn it into a test; it's a confirmation that the mental model landed.

Write: `learner_can_explain = true/false`, `last_completed_step = 5`.

### Step 6 — Close and route

Summarize in Russian: the agent now has personality — a rules file it reads every session. Two more layers ahead:

> «Теперь у твоего агента есть „личность" — файл правил, который он читает при каждом старте. Это первый слой памяти. Впереди ещё два: база знаний (чтобы агент мог находить нужную информацию) и трекер задач (чтобы не терять, что сделано и что дальше). Когда будешь готов — `/pos-vault` для базы знаний, `/pos-github-setup` для трекера задач.»

Write: `status = "done"`, `completed_at`, `last_completed_step = 6`.

## Rules

1. **Language and tone.** Speak Russian to the learner. Use `ты`. Plain, warm, like explaining to a friend.
2. **One mental model per step.** Don't stack concepts. Land one, move on.
3. **Learner writes, agent structures.** Ask questions to draw content out. Never dictate what should be in their rules file.
4. **Concrete over abstract.** Show the learner their own files, their own agent behavior. Don't lecture about memory in general.
5. **Short session.** This block should take 10-15 minutes. Don't pad. If the learner gets it, move forward.
6. **Clear next steps.** Always end with concrete routing: `/pos-vault` or `/pos-github-setup`.
