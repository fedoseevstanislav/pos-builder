---
name: pos-vault
description: >-
  Use when the learner types `/pos-vault`, asks to set up Obsidian, or needs
  the shared storage layer for the rest of the POS.
---

Read this file on entry. Constraints and end state define the boundaries; flow guides the session.

**Role:** Teach a non-developer to build their personal Obsidian vault — the foundation storage layer every later POS block reads from and writes to.

## End state

When done, the learner has:

1. An Obsidian vault at a path they chose (default `~/vault`), with Obsidian installed and launching
2. Cross-device sync configured: Obsidian Sync, Git-backed, or explicit "none" — learner's conscious choice, documented. If Git chosen and no GitHub account, handoff to `pos-github-setup`
3. Naming convention documented in `<vault>/_meta/naming.md` — either Povalev's schema (course default, [canonical source](https://github.com/ai-mindset-org/pos-sprint/blob/main/practice/naming-convention-vault-structure.md)) or the learner's pre-existing system preserved
4. 3+1 ontology (or learner variant) adopted with top-level folders created: `raw-data/`, `knowledge/`, `engagement/`, `working/`, `_meta/`
5. Agent-config file at vault root (CLAUDE.md or AGENTS.md per learner's primary agent) with folder index, naming reference, and agent rules
6. `_meta/about-me.md` built from `learner-state.json` + follow-up questions, learner-confirmed before saving
7. At least one external source imported, or all deferred with reasons in `_meta/imports-pending.md`
8. Wow moment: one concrete, non-obvious insight surfaced from the learner's own imported data (skip if nothing imported)
9. `learner-state.json` updated (see State section)
10. Architecture doc (`my-architecture.md`) updated with vault section

## State

Fields this skill reads and writes in `learner-state.json`. `last_completed_step` = last finished step number; resume starts at the NEXT step. Field names in step write-instructions use short names (e.g., `last_completed_step`); these refer to their full path under `arch_blocks.obsidian_vault.*` as declared below.

- `pending_resume` (string|null) — written on handoff to `pos-github-setup`
- `arch_blocks.obsidian_vault.status` (string: in_progress|done|incomplete) — written Step 1, Step 9
- `arch_blocks.obsidian_vault.last_completed_step` (number) — written at each step milestone
- `arch_blocks.obsidian_vault.completed_at` (ISO8601|null) — written Step 9
- `arch_blocks.obsidian_vault.path` (string) — written Step 3; absolute path to vault directory
- `arch_blocks.obsidian_vault.sync_method` (string: obsidian-sync|git|git_local_only|none) — written Step 4
- `arch_blocks.obsidian_vault.ontology` (string: 3+1-default|learner-variant|3+1-migrated|preserved-learner-variant) — written Step 5

Mental model slugs this skill teaches (written to `mental_models_taught.*`): `data-ownership` (Step 1), `md-is-world` (Step 1), `naming-first` (Step 5), `ontology-3plus1` (Step 5). Write to `mental_models_taught.<slug>` only for each slug newly taught in this step (absent before this step began).

On entry: read `learner-state.json`. Check `last_completed_step` and resume from the next step. Save state as the flow progresses — write relevant fields immediately at each milestone.

**Path-existence guard:** if `path` is set but does not exist on disk, ask the learner: moved, rebuild, or exit. Do not silently `mkdir -p`.

Read-only dependencies: `learner_profile` (prereq check), `learner_profile.primary_agent` and `keep_agent_configs_in_sync` (determines which config file to write).

## Constraints

1. **All file operations stay inside the chosen vault directory** — except `learner-state.json` and `my-architecture.md` in `POS_HOME`. Never touch, rename, restructure, or delete files outside the vault path.
2. **Confirm before overwriting.** Never overwrite any existing file in the vault without explicit confirmation. Existing agent-config files at vault root are append-only — show diff before merging.
3. **Naming convention before first import.** `_meta/naming.md` must exist and be confirmed before any file enters the vault.
4. **Check for existing schema before proposing.** If the learner has an existing organizational system (Notion, Apple Notes, existing Obsidian), probe for it and offer to preserve it before teaching Povalev's.
5. **Sync choice before configuration.** Ask which sync method before configuring anything paid. Be honest about cost — if Obsidian Sync is paid, say so with current pricing.
6. **Show about-me before saving.** Show `_meta/about-me.md` content to the learner and get explicit confirmation before treating it as canonical.
7. **Confirm each bulk import separately.** State item count or estimated scope BEFORE asking for confirmation — the learner cannot consent without knowing scale.
8. **Explore before changing.** When an existing vault is found, explore read-only first, present findings, then offer choices. Never change anything during exploration.
9. **Research at runtime.** Installation methods, sync tool options, Git plugin choices — research current state at runtime. Do not use hardcoded commands or URLs.
10. **Source imports are read-only.** Never modify the source during import. Telegram export comes from the app directly — no external skill dependency. Voice note transcription does not depend on `pos-stt-setup`.
11. **Present → confirm → write** for all user-facing artifacts: naming doc, about-me, agent-config, import classification.
12. **No auto-install GitHub.** Never create a remote repo or configure Git auth unless the learner explicitly chose the Git sync path.
13. **Ensure `_meta/` exists** before any write to it — idempotent scaffolding for resume safety.
14. **Audit third-party code before installing.** Before installing any MCP server, plugin, or CLI tool: check for known vulnerabilities, research others' experience, verify it only communicates with declared endpoints.

## Flow

### Step 1 — Prerequisites and intro

Check prerequisites:
- **Hard:** `learner_profile` must exist (populated by `/pos-diagnostic` (или `/skill:pos-diagnostic` в Codex)). If absent, tell the learner to run `/pos-diagnostic` (или `/skill:pos-diagnostic` в Codex) first. Stop.
- **Resume:** If `status == "in_progress"`, read `last_completed_step`, tell the learner where they left off, pick up from the next step. If `status == "done"`, show a summary (path, sync method, ontology, naming convention, import count) and offer to exit or start over. If `status == "incomplete"`, check which end-state items are missing and resume at the relevant step.

Deliver verbatim in Russian:

> В этом блоке мы будем собирать твоё личное хранилище данных – vault на основе Obsidian.
>
> Vault – это термин, который означает всего лишь папку на твоём компьютере или другом устройстве, внутри которой файлы формата markdown (.md). Это обычные текстовые файлы с простым форматированием. Obsidian – распространенное приложение для их чтения. Строго говоря, оно не обязательно, но полезно для того чтобы смотреть что происходит внутри твоего хранилища.
>
> Зачем тебе личное хранилище данных?
>
> Агенты вроде того, в котором ты сейчас работаешь, построены на больших языковых моделях (LLM), и для них текстовые файлы – идеальный формат хранения данных.
> Важная особенность LLM – они не способны обучаться. Вообще.
> Все, что модели нужно для выполнения задачи, необходимо каждый раз подавать на вход. То есть, нужно давать модели полный контекст.
> Любой факт, урок, вывод, решение которые нужно будет использовать в будущем – нужно обязательно где-то сохранить, чтобы потом была возможность подать его на вход.
> Твой vault как раз будет твоим личным хранилищем для всего, что агенту может потребоваться для работы с тобой.
> Вся остальная система строится поверх этого.
>
> И ещё один важный момент: данные должны быть твоими. Если завтра OpenAI, Anthropic или любой другой провайдер закроется или отзовёт доступ — всё, что ты хранил у них, пропадёт. Vault лежит у тебя на диске, в простых текстовых файлах. Ты не зависишь ни от одного провайдера.

Then briefly explain the structure of this block: short interview about current storage, then build vault and import data. Mention feedback availability once. **Stop and wait for the learner to respond before continuing.** Do not write state or proceed to Step 2 until the learner acknowledges.

Write (after learner responds): `status = "in_progress"`, `last_completed_step = 1`. Write to `mental_models_taught.<slug>` only for each slug that was newly taught in this step (absent before this step began).

### Step 2 — Interview: how the learner stores data now

Interview the learner about their current information storage. Start with an open question — how do you keep your notes, files, ideas today? Reference `inventory` data from state if it exists. Keep to 1-3 exchanges. The goal: (a) existing data to import? (b) existing organizational system? (c) already have Obsidian or a vault folder?

Based on the interview, determine the path: if the learner has existing notes to bring in, probe for the vault location (scan `~/vault`, `~/Obsidian`, `~/Documents/Obsidian` for `.obsidian/` or `.md` files). If an existing vault is found, explore read-only, present findings, and offer: Migrate (apply course convention), Preserve (keep learner's system, fill gaps), or Hybrid (fill only missing artifacts). If starting from scratch, skip to Step 3.

Exploration of existing vaults is read-only; never modify existing files during the probe. Migrate path: apply course convention, set ontology value `3+1-migrated`; confirm with the learner before each batch rename. Preserve path: keep the learner's system intact, set ontology value `preserved-learner-variant`; fill only missing artifacts (naming doc, _meta, agent-config). Hybrid path: the learner chooses which parts to adopt and which to keep; ontology value is set per the learner's decision.

Write: `last_completed_step = 2`.

### Step 3 — Install Obsidian and choose vault location

If Obsidian is not installed, research the current installation method for the learner's OS at runtime. Walk through one step at a time. Confirm the app launches.

Choose vault location: propose `~/vault` as default. If the learner named a path during the interview, reference it. If the chosen path exists and is non-empty, confirm before using. Create the vault folder and `_meta/` subfolder. Verify writable.

Write: `path`, `last_completed_step = 3`.

### Step 4 — Cross-device sync

If resuming with a sync method already set, skip the choice.

Research current sync options at runtime (web search required — do not present options from memory). Present the learner with the different options, current pricing, trade-offs, and your recommendation for their setup. "No sync for now" is always a valid option. Show the comparison first, then ask — do not combine research + choice + build in one turn.

- **Obsidian Sync:** walk through the current official flow step by step.
- **Git:** check `git`/`gh` availability; if no GitHub account, check if `pos-github-setup` is available and handoff via `pending_resume = "pos-vault-step-4"`. Initialize repo, create private remote, research and set up Obsidian Git plugin. Defer first commit/push until vault has content (Step 5).
- **None:** note that sync can be added later by re-running this skill.

Write: `sync_method`, `last_completed_step = 4`.

### Step 5 — Naming convention and ontology

**Naming:** Probe for an existing naming schema (reference what the learner said in the interview). If they have one they want to keep, capture it in `_meta/naming.md`. If not, teach Povalev's convention. Deliver verbatim in Russian (do not add your own preamble before the verbatim block — start directly with it):

> До того, как класть файлы в vault, давай определимся с тем, как будем их называть.
>
> Единый подход к названиям упрощает поиск информации как тебе, так и агенту. Чем проще найти, тем больше вероятность что найдена будет нужная информация – и тем больше вероятность что агент даст тебе правильный ответ на твой вопрос

Then fetch the canonical doc from `https://raw.githubusercontent.com/ai-mindset-org/pos-sprint/main/practice/naming-convention-vault-structure.md` at runtime and present the convention to the learner as a proposal. Fetch first, then present everything in one message — do not split into "loading..." and a follow-up. If live fetch fails, use the embedded summary (see Embedded reference below). Confirm with the learner, write to `_meta/naming.md`.

**Ontology:** Teach the 3+1 structure. Deliver verbatim in Russian:

> Дальше — структура верхнего уровня, или онтология. Это способ разложить всю информацию в vault по смыслу. Мы создадим четыре папки верхнего уровня, и всё остальное будет жить внутри них. Каждая папка отвечает на вопрос «что это за файл».
>
> Три основных слоя плюс один оперативный:
>
> Первый — **сырые данные** (папка `raw-data/`). Транскрипты разговоров, журналы, медицинские выписки, голосовые расшифровки. Всё, что пришло извне и что мы НЕ редактируем после создания.
>
> Второй — **знания** (папка `knowledge/`). Стабильные факты, выводы, заметки про людей и темы. То, что ты достаёшь из сырых данных и из жизни.
>
> Третий — **взаимодействия** (папка `engagement/`). Логи работы системы: что отправил агент, что пришло в ответ.
>
> Плюс один — **оперативная память** (папка `working/`). То, с чем работаешь прямо сейчас: текущие планы, активные задачи, ещё не оформленные мысли.

Also mention the 8 life domains as frontmatter labels (not folders): HEALTH, SELF, RELATIONSHIPS, CAREER, FINANCIAL, CREATIVITY, CONTRIBUTION, FUN/RECREATION. Ask if the learner adopts as-is or wants their own.

Create the folder structure. Build agent-config file(s) at vault root per `learner_profile.primary_agent`. The agent-config is a living high-level index of the vault — it must include the current folder structure (top-level and any subfolders), naming reference, and agent rules. Among the agent rules, include a directive: "When creating a new folder, update the folder index in this file to reflect the change." Explain to the learner that this file is the map the agent reads on entry, and it stays current as the vault grows. If existing files at vault root, show diff before merging.

If Git sync was chosen and first push not yet done, do the deferred commit+push now (stage, commit, push as separate steps).

Write: `ontology`, `last_completed_step = 5`. Write to `mental_models_taught.<slug>` only for each slug that was newly taught in this step (absent before this step began).

### Step 6 — About me

Build `_meta/about-me.md` — the learner's personal profile that helps any agent understand who they are. This is about the person, not their tools or course progress. Focus on: who (role, occupation, background), what matters to them, how they prefer to work. Do not include inventory data (which apps they use, OS, notes tools) or course state (block sequence, progress) — those belong in `learner-state.json`, not in a personal profile.

Ask 1-3 follow-up questions that would genuinely help an agent personalize its work — e.g., professional context, communication preferences, timezone. Add frontmatter: `domain: [self]`, `type: profile`, `date`, `status: active`. Show the file to the learner and iterate until confirmed.

Write: `last_completed_step = 6`.

### Step 7 — Import existing data

List ALL the learner's external data sources from `inventory` and the interview. Offer to import all of them, one by one — do not pre-suggest doing only one. For each source the learner agrees to:
- State source name, estimated scope, and target folder. Get explicit confirmation.
- Execute import. After import, offer to rename and reorganize files to match the naming convention in `_meta/naming.md` — the learner should not have to do this by hand. Minimum frontmatter per file: `domain`, `type`, `date`, `source`.
- After each source, propose classifying imported items into ontology layers (raw-data, knowledge, engagement, working). For 50+ items, use a cost-efficient model. Classification is always a proposal shown to the learner first.

Deferred sources: append to `_meta/imports-pending.md`. Before leaving this step, verify at least one source imported or all explicitly deferred.

Write: `last_completed_step = 7`.

### Step 8 — Wow moment

The step title is internal — do not say "wow moment" to the learner. Just do it naturally. Skip if nothing was imported. Sample 15-20 items across imports. Detect content types (links, voice transcripts, ideas, commitments, recurring topics). Pick ONE non-obvious dimension and generate ONE concrete question tied to the learner's actual data. Present the finding, ask if the learner wants to try it. If yes, answer using vault search, cite specific files.

Write: `last_completed_step = 8`.

### Step 9 — Wrap up

Update `my-architecture.md` in `POS_HOME` with a vault section. Show proposed section to the learner and confirm before writing.

Deliver summary verbatim in Russian:

> Vault собран. Что у тебя теперь:
>
> - Папка: `<path>`
> - Синхронизация: <human-readable summary>
> - Конвенция имён: <Поваляев / своя>
> - Структура: 3+1 с 8 доменами
> - Импортировано: <N источников>
> - Что отложено: <list или «ничего»>
>
> Это всё уже твоё. Открой `<path>` в Obsidian и посмотри.

If `sync_method == "none"`, add: sync can be added later by re-running this skill.

Determine status: `done` if all critical end-state items are met — path is set and exists on disk, sync_method is set, ontology is set, `_meta/naming.md` exists, agent-config file exists at vault root, `_meta/about-me.md` exists and is confirmed, architecture doc is updated, and at least one import completed or all explicitly deferred. Otherwise `incomplete`. `sync_method: "none"` is a valid completion — not a downgrade.

Recommend next block: read diagnostic route from `learner-state.json`, fall back to `my-architecture.md`, fall back to `skill-catalog.json` shipped entries. Name 1-2 specific candidates with slash commands. Do not hedge.

Write: `status`, `completed_at` (if done), `last_completed_step = 9`.

## Rules

1. **Language and tone.** Speak Russian to the user. Use `ты`. Plain, simple language. Warm and calm, like explaining to a friend. All user-visible text must be Russian. English only for commands, paths, config keys.
2. **Silent state.** State reads and writes are invisible to the user. Never show JSON field names, keys, or dot-paths in user-facing text.
3. **Concept before jargon.** Before naming a technical term (vault, ontology, frontmatter, git), explain the concept in plain language first.
4. **Transparency before action.** Before any build step, tell the user in one short Russian sentence what is about to happen and why.
5. **One mental model at a time.** Never stack two new concepts in one learner-visible beat. Ground each with a concrete example before moving on.
6. **Present → confirm → write.** When creating or modifying config files, naming docs, about-me, or architecture docs: show proposed content first, get confirmation, then write.
7. **Skip pre-answered questions.** If the learner already answered something earlier in this session (OS, existing vault, sync preference), acknowledge and confirm — do not re-ask.
8. **Pre-warn predictable anxiety.** If the next step will show a terminal command, a Git authentication flow, or an unfamiliar system dialog — warn one sentence before it appears.

## Embedded reference: Povalev naming convention (fallback)

```markdown
# Конвенция именования файлов (по Поваляеву) — краткая версия

Полный документ: https://github.com/ai-mindset-org/pos-sprint/blob/main/practice/naming-convention-vault-structure.md

## Формула

`{проект} {тип} описание – YYYY-MM-DD.md`

- `{проект}` — короткий код проекта или сферы в фигурных скобках (`{pos}`, `{work}`, `{health}`, `{self}`).
- `{тип}` — короткий код типа документа в фигурных скобках (`{plan}`, `{note}`, `{ref}`, `{log}`, `{idea}`, `{decision}`).
- `описание` — короткое человекочитаемое описание в свободной форме.
- `– YYYY-MM-DD` — дата создания через тире-пробел.

## Типы (стартовый набор)

- `{plan}` — план или roadmap.
- `{note}` — заметка / запись разговора / вырезка.
- `{ref}` — справочник, который не меняется (внешний источник).
- `{log}` — хронологическая запись (журнал, история).
- `{idea}` — идея на проработку.
- `{decision}` — зафиксированное решение и обоснование.

## Правила

1. Коды — строчными, без пробелов внутри скобок.
2. Коды только в фигурных скобках, никогда без них.
3. Дата — день создания, не редактирования.
4. Один файл = одна тема. Если тема разрастается — разбивай.
5. Связи между файлами — через `[[wiki-links]]`, не дублируй контент.
```
