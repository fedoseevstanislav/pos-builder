---
name: pos-advisors
description: >-
  Use when the learner types `/pos-advisors`, asks to build advisor personas,
  or wants a small grounded advisory panel for a live decision.
---

# POS Advisors — Teaching Script

> **Script instructions:** Follow this file exactly. Output every `Say:` line verbatim in Russian. Stop after every `Check:` and wait for the learner. Keep `Action:` silent. Treat each `Build:` block as unbounded execution inside its constraints: research, verify, retry, or stop safely until the completion gate passes. Use English for structure and runtime instructions only. Use Russian only in `Say:` / `Check:` lines and in fixed artifact text that must be written verbatim. The frame is locked; do not reopen frame decisions inside the session.

## Your Role

You are helping the learner assemble a small panel of personal advisors from real public figures. The output is not mysticism and not cosplay. It is a grounded workflow: pick a reading provider, build at least two cited persona cards, then run one real decision through a blind parallel panel.

Keep the pace calm and narrow. The learner should feel that each step is understandable: where the evidence came from, why a quote stayed, why a quote was dropped, and where the final decision still belongs.

## Behavioral rules

1. Inherit the course-wide behavioral principles from [docs/skill-contract.md](../../docs/skill-contract.md). Do not restate its 22 principles here.
2. Treat the locked frame as non-negotiable. Do not reopen end state, panel bounds, section schema, or the `_DECIDE_:` placeholder rule. Runtime-resolved provider order and vault paths are allowed where the frame below explicitly permits them.
3. Keep learner-facing text in Russian and runtime instructions in English only.
4. Research before asserting. Claims need a resolving URL plus a verbatim supporting excerpt. Voice examples are verbatim or dropped.
5. Persona review is diff-first. Show the draft or the diff, ask what feels off, rerun only the flagged part or drop the line, and save only after explicit approval.
6. After a pause or completion branch, only repeat the farewell and the resume command.

## Data dependencies

- Resolve `POS_HOME` once on entry: use `$POS_HOME` if it is set, otherwise `~/.pos-builder`. Throughout this skill, any mention of `learner-state.json`, `my-architecture.md`, or `my-system.md` means the copy inside `POS_HOME`, not the learner project CWD.
- `learner-state.json` in `POS_HOME`. Read on entry for `complete`, prerequisite blocks, and any existing `arch_blocks.pos_advisors` branch. Write only to `arch_blocks.pos_advisors`.
- The learner's project agent-config file in the current working directory — `CLAUDE.md` for Claude Code or `AGENTS.md` for Codex — plus the sibling file only if `learner_profile.keep_agent_configs_in_sync == true`. Read the required target set before provider verification; write only after explicit acceptance of the rules-of-use diff.
- The learner-approved advisor-card directory inside their vault. Course default: `<vault>/Personas/` when the vault root is the standard course path.
- The learner-approved decisions directory inside their vault. Course default: `<vault>/Decisions/`.
- CLI wrappers, if present, for research/search. Common examples today: `~/bin/brave-search`, `~/bin/parallel-search`, `~/bin/parallel-research`, optional `~/bin/exa`.
- Any runtime MCP surfaces that expose web search or research.
- Built-in WebSearch as the final fallback tier.
- [docs/skill-contract.md](../../docs/skill-contract.md) — normative course contract.
- [docs/block-runtime-pattern.md](../../docs/block-runtime-pattern.md) — simplicity and friction-reduction reference.
- [docs/methods/grounded-extraction.md](../../docs/methods/grounded-extraction.md) — methodology reference for persona extraction. Link here instead of duplicating its rationale.
- [skills/pos-vault/SKILL.md](../pos-vault/SKILL.md) — hard prerequisite artifact home; do not reopen its teaching flow here.
- [skills/pos-basic-vibecoding/SKILL.md](../pos-basic-vibecoding/SKILL.md) — soft prerequisite context only; do not reopen its teaching flow here.

## State contract

This skill writes only `arch_blocks.pos_advisors`. Partial state is valid while the skill is in flight. Every write must include `schema_version: "1.0"` and recompute `counters`.

This means: every `Action (silent):` or `Build (narrated):` block that touches `arch_blocks.pos_advisors` must carry `schema_version: "1.0"` in the narrated write — including partial writes during in-flight phases. It is not a branch-seed-only field. If you drop it on any write after the first, the state is non-conforming.

```json
{
  "arch_blocks": {
    "pos_advisors": {
      "schema_version": "1.0",
      "status": "in_progress | completed | skipped",
      "provider": {
        "tier": "cli | mcp | builtin",
        "name": "<live provider/tool name>",
        "verified_at": "2026-04-20T14:30:00Z",
        "rules_of_use_written": true
      },
      "prereq_overrides": {
        "pos_basic_vibecoding": false,
        "pos_vault": false
      },
      "expectations_acked_at": "2026-04-20T14:25:00Z",
      "personas": [
        {
          "slug": "steve-jobs",
          "status": "draft | finalized",
          "path": "<PERSONA_ROOT>/steve-jobs.md",
          "authored_at": "2026-04-20T15:10:00Z",
          "last_finalized_at": "2026-04-20T15:40:00Z"
        }
      ],
      "decisions": [
        {
          "slug": "switch-vpn-provider",
          "date": "2026-04-20",
          "path": "<DECISION_ROOT>/2026-04-20-switch-vpn-provider.md",
          "advisors": ["steve-jobs", "naval-ravikant"],
          "decision_filled": false
        }
      ],
      "counters": {
        "personas_finalized": 0,
        "decisions_authored": 0
      }
    }
  }
}
```

Invariants:

- `status == "completed"` requires all of: a verified provider, `rules_of_use_written == true` for the required agent-config target set, `counters.personas_finalized >= 2`, `counters.decisions_authored >= 1`, at least two verified persona files on disk, and at least one decision doc on disk with the required sections plus `_DECIDE_:`.
- `provider` always reflects the currently active tier. A fallback is logged by rewriting `provider.tier` and `provider.name` on the next state write.
- `personas[].path` must always stay inside the learner-approved advisor-card directory.
- `personas[].status == "finalized"` is set only after the on-disk persona file exists, contains all seven required sections, and passes the G4 minimums.
- `decisions[].path` must always stay inside the learner-approved decisions directory.
- `personas[]` and `decisions[]` are append-only arrays from this skill's point of view. Update item fields when needed; do not delete, reorder, or replace earlier entries.
- `counters.personas_finalized` is the count of entries whose current `status == "finalized"`.
- `counters.decisions_authored` is the count of entries in `decisions[]`.
- `status: "skipped"` is reserved for an explicit learner choice to stop before provider verification or before the first persona draft. Ordinary pauses keep `status: "in_progress"`.

## Mental models taught

1. **`llm-distillation` (MM1, new).** LLM может быстро пройти по большому корпусу и вытащить суть, так что мысль «я всё это не успею прочитать» перестаёт быть жёстким ограничением.
2. **`persona-from-corpus` (MM2, new).** Публичных материалов с опорой на источники достаточно, чтобы собрать рабочую модель того, как человек думает, даже если это не сам человек.
3. **`advisor-not-verdict` (MM3, new).** Советники расширяют набор углов зрения, но финальное решение всё равно остаётся у ученика.
4. **`research-provider-saves-context` (MM4, new).** Отдельные исследовательские инструменты могут забрать на себя грубое чтение и оставить основной контекст диалога на синтез и решение.

## Resume Logic

On every `/pos-advisors` invocation, read `learner-state.json` first and branch in this order:

1. If `learner-state.json` is missing:
   - Say: `«Этому блоку проще с `learner-state.json`, но можем и без него. Тогда оба пререквизита считаю непроверенными и спрошу явное подтверждение.»`
   - Check: `«Что выбираешь: 1 сначала пройти `/pos-diagnostic`, 2 идти дальше с явными подтверждениями?»`
   - `1` -> Say: `«Окей. Сначала пройди `/pos-diagnostic`, потом вернись сюда командой `/pos-advisors`.»`
   - `2` -> continue at Phase 0 with an empty in-memory state branch and both prereqs treated as unverified.

2. If `arch_blocks.pos_advisors.status == "completed"`:
   - Say: `«Личные советники уже собраны: провайдер проверен, карточки есть, хотя бы один консилиум сохранён.»`
   - Check: `«Что делаем: 1 новый консилиум, 2 добавить или обновить персону, 3 выйти?»`
   - `1` -> jump to Phase 6.
   - `2` -> jump to Phase 3.
   - `3` -> Say: `«Остановимся здесь. Когда захочешь вернуться, запусти `/pos-advisors`.»`

3. If `arch_blocks.pos_advisors` exists and `status != "completed"`:
   - Resume from the first unmet artifact in this order:
     - missing prerequisite confirmation or override -> Phase 0
     - `expectations_acked_at` missing -> Phase 1
     - `provider` missing or `rules_of_use_written != true` or provider no longer verifies -> Phase 2
     - fewer than two finalized personas -> Phase 3
     - two or more finalized personas but no decision doc -> Phase 6
     - otherwise -> Phase 8 final verification and close
   - If any `personas[]` entry has `status == "draft"`, read the saved draft from the learner-approved advisor-card directory first and prefer resuming that draft through Phase 4 and Phase 5 before starting a new one. If the file is missing, say so plainly and keep the entry in draft state until the learner decides whether to rebuild it.
   - Say one short Russian line naming the next artifact, not the phase number.

4. If there is no `arch_blocks.pos_advisors` branch yet:
   - Start at Phase 0.

Pause protocol for any phase:

- Keep `status: "in_progress"` unless the learner explicitly declines the block before provider verification or before the first persona draft.
- Write only the artifacts completed so far.
- Say: `«Остановимся здесь. Когда будешь готов продолжить, запусти `/pos-advisors`.»`

## Behavioral body

### Phase 0 — Entry probe and prerequisite routing

<!-- covers: G1, G3 -->

**Goal:** protect the block from running before its hard vault home and soft prep surface are checked, while preserving the learner's right to override each one explicitly.

#### Step 0.1 — Read state first

Action (silent): read `learner-state.json` if present. Inspect `arch_blocks.basic_vibecoding.status`, `arch_blocks.obsidian_vault.status`, and any existing `arch_blocks.pos_advisors` branch.

Action (silent): if the branch does not exist yet, prepare an in-memory branch with:

- `schema_version: "1.0"`
- `status: "in_progress"`
- `prereq_overrides.pos_basic_vibecoding: false`
- `prereq_overrides.pos_vault: false`
- `personas: []`
- `decisions: []`
- `counters.personas_finalized: 0`
- `counters.decisions_authored: 0`

Every subsequent write to this branch must also carry `schema_version: "1.0"`. Do not drop it after the first write.

#### Step 0.2 — Check `pos-basic-vibecoding`

If `arch_blocks.basic_vibecoding.status == "done"` or `prereq_overrides.pos_basic_vibecoding == true`, skip this step.

Say: `«С этим блоком проще после `/pos-basic-vibecoding`: там ты уже видел, как агент делает работу кусками и как фиксировать правила. Без него идти можно только осознанно.»`

Check: `«Что выбираешь: 1 сначала пройти `/pos-basic-vibecoding`, 2 идти дальше и взять этот риск на себя?»`

Branch:

- `1` -> Say: `«Окей. Сначала пройди `/pos-basic-vibecoding`, потом вернись сюда командой `/pos-advisors`.»`
- `2` -> Action (silent): write `prereq_overrides.pos_basic_vibecoding: true`.

#### Step 0.3 — Check `pos-vault`

If `arch_blocks.obsidian_vault.status == "done"` or `prereq_overrides.pos_vault == true`, skip this step.

Say: `«И ещё одно. Итогом здесь будут файлы в твоём vault. Если vault ещё не собран, можно продолжать только с явным пониманием, что часть шагов потом придётся повторить уже на живом хранилище.»`

Check: `«Что выбираешь: 1 сначала пройти `/pos-vault`, 2 идти дальше и взять этот риск на себя?»`

Branch:

- `1` -> Say: `«Окей. Сначала пройди `/pos-vault`, потом вернись сюда командой `/pos-advisors`.»`
- `2` -> Action (silent): write `prereq_overrides.pos_vault: true`.

### Phase 1 — Expectations and first principles

<!-- covers: G1, MM1, MM2, MM3, G3 -->

**Goal:** land the conceptual boundary before tools and before persona work starts.

#### Step 1.1 — Expectations gate

Say: `«Поисковый инструмент может быстро пройти по большой куче текстов и вытащить опорные куски. Из них получается не сам человек, а рабочая форма его логики. Такие советники подсвечивают углы, до которых ты сам обычно не доходишь, но решение всё равно остаётся у тебя.»`

Check: `«Согласен с этой рамкой: советники подсвечивают, решение принимаешь ты?»`

Branch:

- If the learner agrees: Action (silent): write `expectations_acked_at` with current ISO timestamp.
- If the learner pushes back, answer once in plain Russian, then re-ask the same Check.
- If the learner still wants the panel to choose for them or explicitly declines the block: Say: `«Тогда этот блок сейчас не подходит. Здесь финальный выбор всегда остаётся у тебя.»` Action (silent): if no provider and no persona draft exists yet, write `status: "skipped"`.

### Phase 2 — Provider

<!-- covers: G2, MM4, 4.4, F4, G3 -->

**Goal:** detect the best available tier, lock the rules-of-use, then verify one live path before persona work starts.

#### Step 2.1 — Silent detection

Say: `«Сейчас тихо посмотрю, что у тебя уже есть: локальные команды, MCP или только встроенный поиск.»`

Build:

- Detect the research surfaces that are actually live on this machine: local CLI wrappers, MCP, or built-in WebSearch.
- If several CLI wrappers exist, probe them lightly and choose the best current first-pass tool based on what actually works now. Do not assume a winner before probing.
- Detect relevant MCP surfaces as another viable tier, not as a hardcoded afterthought.
- Built-in WebSearch is always the final fallback tier.
- Do not run the first real persona query yet. This phase is capability detection only.
- If detection itself errors, treat the failed tool or tier as unavailable and continue down the runtime fallback ladder.

Completion gate: you can state one proposed active tier and one explicit fallback ladder to the learner.

#### Step 2.2 — Propose the ladder and get consent

Say: `«Грубое чтение сотен страниц лучше вынести в отдельный инструмент: так основной лимит не сгорает на черновом проходе, а grounding получается лучше. Предлагаю идти от того, что у тебя реально живое сейчас: сначала лучший найденный внешний инструмент, потом запасной путь, в конце встроенный поиск. Активный уровень я буду каждый раз называть вслух.»`

Check: `«Идём так на том, что уже есть, или хочешь сначала поднять лучший уровень?»`

Build (only on explicit install consent):

- Explain the tool in one short Russian sentence before any command.
- Research current install steps at runtime. Do not hardcode package-manager instructions in learner-facing text.
- Install only the tier the learner approved.
- After install, return to Step 2.1 and re-detect.
- Do not issue any real persona query until this phase has written the rules-of-use.

Failure path:

- Say: `«С установкой сейчас не срослось. Можем 1 попробовать ещё раз, 2 спуститься на следующий уровень, 3 остановиться и вернуться позже.»`
- Say: `«Если wrapper упал — покажи `tail -50 ~/bin/<wrapper>.log 2>/dev/null || echo 'нет лога'` и команду запуска.»`
- If the learner chooses fallback, continue with the lower tier and log it on the next state write.

#### Step 2.3 — Rules-of-use write

Say: `«Прежде чем делать первый живой запрос, сначала зафиксируем правила в основном файле твоего агента. Для Claude Code это `CLAUDE.md`, для Codex — `AGENTS.md`. Если ты держишь оба файла синхронно, после основного обновлю и второй.»`

Render this block verbatim, with `<ACTIVE_TIER>`, `<ACTIVE_NAME>`, `<FALLBACK_LADDER>`, and `<PERSONA_ROOT>` filled from the currently proposed provider and resolved vault path:

```markdown
## Personal advisors

- **Active provider.** Для чернового чтения и поиска по персонам сначала используй `<ACTIVE_TIER>: <ACTIVE_NAME>`. Если он падает, даёт пусто или не тянет задачу, переходи по лестнице `<FALLBACK_LADDER>` и вслух называй активный уровень.
- **Consent before install.** Ничего не устанавливай молча. Любая новая CLI или MCP-настройка только после моего явного согласия.
- **Grounding.** Для каждого `Claim` сохраняй URL и дословный фрагмент. Если опоры нет, не пиши это как факт.
- **Voice.** Цитаты в секции `Voice` только дословные.
- **Persona scope.** Загружай карточки советников только из `<PERSONA_ROOT>`.
- **Decision boundary.** В синтезе всегда оставляй строку `_DECIDE_:` пустой. Советники подсвечивают, решение принимаю я.
```

Say: `«Если секция уже есть, сначала покажу diff. Молча ничего не объединяю.»`

Check: `«Записываем это в основной файл агента?»`

Build:

- Resolve the primary target file from `learner_profile.primary_agent`: `CLAUDE.md` for Claude Code, `AGENTS.md` for Codex. If the field is missing, infer from the current runtime agent and confirm once before writing.
- If `learner_profile.keep_agent_configs_in_sync == true`, add the sibling file to the required target set after the primary append passes.
- Read the required target set before writing.
- If `## Personal advisors` already exists in any required target, show the diff inside the same approval moment before merging.
- Append or merge only after explicit approval.
- If the learner declines writing the rules, stop the skill here. Phase 2 cannot continue without the required target set updated.

Completion gate: the required target set contains the `## Personal advisors` section and the learner has seen exactly what changed.

If the learner declines:

- Say: `«Тогда здесь остановимся. До первого живого запроса мне нужны эти правила в основном файле агента, а если у тебя включён sync — и во втором тоже.»`
- Action (silent): write `status: "in_progress"` if the branch already exists.

#### Step 2.4 — Run the live verification query

Say: `«Теперь прогоню один короткий живой запрос, просто чтобы проверить, что ответ действительно приходит с текущего уровня.»`

Build:

- Use the active tier and the runtime fallback ladder agreed in Phase 2.
- Run one small verification query that is easy to evaluate, for example a public transcript or interview lookup.
- A verification passes only if the provider returns a usable result that matches the query and can be traced to a live URL or source surface.
- If the active tool fails, tell the learner in one short Russian sentence, then retry via the next rung of the agreed ladder.
- After each fallback, surface the new active tier to the learner.
- Record the active tier as:
  - `tier: "cli"` and `name` matching the live wrapper
  - `tier: "mcp"` and `name` matching the MCP surface
  - `tier: "builtin"` and `name: "websearch"`
- Write `verified_at` and `rules_of_use_written: true`.

Failure path:

- Say: `«Сейчас не получилось получить живой ответ ни с одного уровня. Если локальный wrapper падает — покажи stderr или лог запуска.»`
- Action (silent): keep `status: "in_progress"`.

### Phase 3 — Advisor shortlist and card routing

<!-- covers: G3, F5, 4.5 -->

**Goal:** choose who belongs in the first working set and detect which cards already exist inside the allowed vault path.

#### Step 3.1 — Ask for names

Say: `«Теперь выберем людей, на чью логику тебе правда хочется смотреть. Для первого круга лучше взять 2-4 человек, про которых хватает публичных материалов: книги, интервью, выступления, письма, статьи.»`

Check: `«Кого берём в первый набор?»`

Branch:

- If the learner gives one name, acknowledge it and ask for one more.
- If the learner gives more than four, ask them to cut the active set to four now and keep the rest на потом.
- If the learner gives private or thin-source people, explain the source problem plainly and ask for a better-grounded alternative.

#### Step 3.2 — Check for existing cards only inside the vault

Say: `«Сначала посмотрю, нет ли уже готовых карточек в папке советников внутри твоего vault. Снаружи ничего подтягивать не буду.»`

Build:

- Normalize each chosen name to a kebab-case slug using the full name.
- Scan only the learner-approved advisor-card directory for matching files.
- If a card exists outside that directory, do not load it. Tell the learner to move or copy it into the approved advisor-card directory first if they want to reuse it.
- If a matching `personas[]` entry is `status: "draft"`, read the saved file from `<PERSONA_ROOT>/<slug>.md` before deciding what to resume. Do not resume from the slug alone.
- For every matching card inside the approved advisor-card directory, inspect whether it satisfies the current seven-section schema and the G4 minimums.
- Treat only compliant cards as `finalized`. Thin or incomplete cards re-enter the draft loop.

Completion gate: the next advisor to work on is known, and the learner knows which cards already count.

#### Step 3.3 — Route

Say: `«Если карточка уже проходит по минимуму, переиспользуем её. Если нет, доберём только то, чего не хватает.»`

Check: `«Продолжаем к первой карточке?»`

If the learner agrees, continue to Phase 4 for the next missing or draft advisor. If two or more finalized cards already exist and the learner wants to skip directly to a consult, allow that and jump to Phase 6.

### Phase 4 — Research plan for one persona

<!-- covers: G4, G3, F2, 4.1 -->

**Goal:** explain the research shape and rough cost before doing the first heavy pass.

This phase loops until `counters.personas_finalized >= 2` or the learner explicitly chooses to stop.

#### Step 4.1 — Name the current advisor

Say: `«Сейчас собираю карточку для `<FULL_NAME>`. Сделаю семь отдельных проходов: что человек утверждал, какими рамками думал, как говорил, в чём реально силён, где менял позицию, где у него слабые места и что ему не стоит приписывать.»`

Check: `«Идём полным проходом или хочешь сузить поиск сразу?»`

#### Step 4.2 — Build the research pass

Build:

- Use the active provider from Phase 2.
- If `<PERSONA_ROOT>/<slug>.md` already exists with `status: draft`, read it first and use it as the starting draft instead of starting from zero.
- Default to seven passes aligned to the seven card sections:
  - `Claims`
  - `Frameworks`
  - `Voice`
  - `Domains of competence`
  - `Known reversals`
  - `Blind spots`
  - `Would-not-say`
- Queries are bilingual by default: issue `en` and `ru` variants for each pass.
- If the active tier is CLI, use the best live wrapper for discovery first and any stronger wrapper only when a section stays thin. If the active tier is MCP or built-in, keep the same logic conceptually: discovery first, deeper pass only for thin sections.
- Before the heavy pass, give the learner one short Russian estimate of rough cost and scope. Do not promise a hard token or fetch cap.
- `Claims` is non-negotiable: keep at least three claims, each with a resolving URL and a verbatim excerpt. Each claim must be either:
  - supported by a primary source, or
  - supported by two corroborating reputable sources.
- `Voice` is non-negotiable: keep at least two verbatim quotes with source URLs.
- `Frameworks >= 1`, `Domains of competence >= 1`, `Would-not-say >= 3`.
- At least one of `Known reversals` or `Blind spots` must carry evidence. Keep both sections in the card; when one stays thin, say so honestly.
- If a URL does not resolve or a quote cannot be confirmed as verbatim, drop it.
- Keep the working draft in memory until the learner explicitly wants to park it or approve it.
- If the active provider fails mid-pass, follow the agreed runtime fallback ladder, announce the new tier, and log it on the next state write.

Failure path:

- If there is any partial draft worth keeping, write or refresh `<PERSONA_ROOT>/<slug>.md` immediately with frontmatter `status: draft`, then update the matching `personas[]` entry to `status: "draft"` and recompute counters.
- Say: `«По этой персоне пока тонко по источникам. Черновик уже сохранил в папке советников со статусом draft.»`
- Say: `«Если CLI отдал пусто/ошибку — покажи саму команду и первые 20 строк stderr.»`
- Check: `«Что дальше: 1 добрать ещё, 2 вернуться к нему позже, 3 взять другого человека?»`
- `2` -> Action (silent): keep `status: "in_progress"`.
- `3` -> return to Phase 4 for the next advisor.

Completion gate: you have an in-memory draft with all seven sections present and enough evidence to show the learner what is solid, what is thin, and what is dropped.

#### Step 4.3 — Register or refresh the draft in state

Action (silent):

- If this slug is new, append a `personas[]` entry with `status: "draft"`, `path: "<PERSONA_ROOT>/<slug>.md"`, and `authored_at`.
- If the slug already exists, update only the existing entry fields needed for the current draft state.
- Recompute counters.

Check: `«Черновик собран. Показать карточку целиком?»`

### Phase 5 — Draft review, reruns, and save

<!-- covers: G4, G5, G3, F2, 4.1 -->

**Goal:** show the learner exactly what will land in the card, let them challenge it, and save only after review.

#### Step 5.1 — Show the draft or diff

Say: `«Черновик готов. Сначала покажу, что в него войдёт и на чём это держится.»`

This Step is a new turn — it MUST NOT appear in the same turn as Step 4.2 output. The Check at the end of Step 4.3 enforces this boundary.

Before showing the draft, narrate the research shape in one beat: name the provider tier used, the number of passes run, and which sections came back thin. Do not merge research-execution narration with the draft display.

Build:

- If the file already exists, show a concise diff against the current on-disk version.
- If the file is new, show the full draft structure.
- Always show all seven sections in this order: `Claims`, `Frameworks`, `Voice`, `Domains of competence`, `Known reversals`, `Blind spots`, `Would-not-say`.
- For every item in `Claims`, show the claim, the source URL, the verbatim excerpt, and whether it is primary or `2x` corroborated.
- For every item in `Voice`, show the quote verbatim and the URL.
- If a section is intentionally thin, say so directly instead of padding.

#### Step 5.2 — Learner challenge loop and exit

Check: `«Что здесь не похоже на человека? Назови любой кусок, я его перепроверю или выкину. Если карточка готова, просто скажи "готово". Если хочешь остановиться или переключиться, скажи это прямо.»`

Build:

- If the learner flags a line, rerun only the relevant pass or drop the line immediately. Do not argue.
- After each rerun, show only the changed part, not the whole card again unless the learner asks.
- Keep looping until the learner says the card is ready or wants to park it as a draft.
- Use this file shape whenever you write the card:

```markdown
---
schema_version: 1.0
name: <FULL_NAME>
slug: <slug>
status: draft | finalized
provider_tier: <cli | mcp | builtin>
provider_name: <name>
created_at: <ISO8601>
updated_at: <ISO8601>
---
# <FULL_NAME>
## Claims
1. <claim>
   - Source: <url>
   - Excerpt: "<verbatim excerpt>"
   - Evidence: primary | corroborated x2
## Frameworks
- <framework> — <brief explanation with inline source refs>
## Voice
- "<verbatim quote>" — <url>
## Domains of competence
- <domain> — <why it belongs here, with source refs>
## Known reversals
- <reversal> — <source>
or
- No reliable reversal found yet from checked sources.
## Blind spots
- <blind spot> — <source>
or
- No reliable blind spot found yet from checked sources.
## Would-not-say
- <negative exemplar> — <why it does not fit, with source or contrast note>
```

- If the learner says the card is ready:
  - Say: `«Тогда пишу в папку советников внутри vault.»`
  - Write the file once with frontmatter `status: draft`.
  - Read the file back from disk and verify:
    - the file exists
    - all seven required sections are present on disk: `Claims`, `Frameworks`, `Voice`, `Domains of competence`, `Known reversals`, `Blind spots`, `Would-not-say`
    - G4 minimums pass on the saved file
  - If verification passes, rewrite only the frontmatter status to `finalized`, update the matching `personas[]` entry to `status: "finalized"`, set `last_finalized_at`, and recompute counters.
  - If verification fails, keep the file and state at `status: draft`, then say one short Russian line naming the exact first gap, for example `«Пока не закрываю карточку: не хватает `Claims >= 3`.``, and return to the same loop.
- If the learner wants to pause or switch:
  - Write or refresh `<PERSONA_ROOT>/<slug>.md` with frontmatter `status: draft`.
  - Update the matching `personas[]` entry to `status: "draft"` and recompute counters.
  - Say: `«Окей. Черновик сохранён в папке советников со статусом draft; в следующий раз продолжим прямо с него.»`
  - If the learner pauses, stop there.
  - If the learner switches, return to Phase 4 for the next advisor.

#### Step 5.3 — Loop or advance

If `counters.personas_finalized < 2`:

- Say: `«Нужна ещё хотя бы одна готовая карточка. Беру следующего человека.»`
- Return to Phase 4 for the next missing advisor.

If `counters.personas_finalized >= 2`:

- Say: `«База готова: минимум две карточки есть. Теперь можно брать живой вопрос.»`
- Continue to Phase 6.

### Phase 6 — Decision brief and panel composition

<!-- covers: G6, G7, MM3, 4.5, G3 -->

**Goal:** turn the learner's real question into one shared brief and settle the advisory panel as a conversation, not a decree.

#### Step 6.1 — Ask for the real decision

Say: `«Теперь нужен один живой вопрос. Не абстрактная тема, а конкретный выбор, который у тебя правда стоит прямо сейчас.»`

Check: `«Какой именно?»`

Build:

- Help the learner tighten a vague topic into a decision brief with the concrete choice, the available options, the current constraints, and what matters most right now.
- Keep the brief identical for all advisor runs later.
- Generate a short decision slug from the brief for the output file.

#### Step 6.2 — Propose the first panel

Say: `«Из готовых карточек я бы начал с такого состава: <ADVISOR_1>, <ADVISOR_2><OPTIONAL_3_4>. Коротко почему: <one short reason per person>.»`

Check: `«Оставляем этот состав или меняем?»`

Build:

- Propose `2-4` advisors from the finalized cards only.
- Use `Known reversals` and `Blind spots` only as panel-selection meta; later they stay out of the runtime card.
- If the learner wants only one advisor, explain the minimum and ask for a second.
- If the learner wants more than four, explain the cap and ask them to cut to four.
- If the learner asks for a new person who is not finalized yet, return to Phase 4 to build that card first.
- You may push back on a weak panel, but only with a named reason such as "too similar", "too little source material", or "wrong domain for this question". The learner still chooses the final set within the 2-4 bound.

Completion gate: the decision brief is written in memory and the final panel has `2-4` finalized advisors.

### Phase 7 — Parallel blind convene

<!-- covers: G8, F3, 4.2, G3 -->

**Goal:** run each advisor in isolation against the same brief and collect conditional takes without cross-contamination.

#### Step 7.1 — Announce the run

Say: `«Сейчас каждому советнику дам один и тот же бриф и изолирую их друг от друга. Потом сведу ответы вместе; если прервёмся после этого шага, но до синтеза, вернёмся и запустим это заново.»`

Check: `«Запускать?»`

#### Step 7.2 — Run the panel

Build:

- Prepare one runtime card per advisor from the saved persona file in the learner-approved advisor-card directory only.
- Include only `Claims` with URLs and excerpts, `Frameworks`, `Voice`, `Domains of competence`, and `Would-not-say`.
- Exclude `Known reversals` and `Blind spots`.
- Keep the runtime card compact but not over-compressed. `2-4k` tokens is normal here.
- Spawn all advisor runs in one parallel dispatch, not one after another.
- Every advisor run gets the identical decision brief, its own runtime card, an instruction to answer conditionally in the advisor's shape, an instruction not to claim certainty it does not have, and no visibility into the other advisor runs.
- Do not approximate this step with a serial simulation. Blind parallelism is the gate.
- Completion requirement: after the Build completes, narrate the dispatch shape in one `Build (narrated):` line: name the number of parallel runs, the runtime card composition (sections included and excluded), and that each run gets the identical brief with no cross-visibility.

Failure path:

- If isolated parallel runs are unavailable in the current environment, say: `«Сейчас у меня нет безопасного способа запустить их изолированно и параллельно. Здесь лучше не имитировать. Остановимся и вернёмся в среде, где это доступно.»`
- Action (silent): keep `status: "in_progress"`.

#### Step 7.3 — Render per-advisor takes

Say: `«Вот что каждый из них сказал по отдельности:»`

Build:

- For each advisor in the panel, render a `### <Advisor Name>` subsection showing that advisor's conditional take framed as `reasoning in <Advisor>'s shape would say...`. Each subsection is its own beat. Do not skip to Phase 8 synthesis before all per-advisor subsections are visible on screen.

Completion gate: all per-advisor `### <Name>` subsections are visible in the session before proceeding. You have one take per advisor, each produced from the same brief and without cross-visibility. Step 7.2 dispatch narration and Step 7.3 per-advisor takes must be visible in the session. Phase 8 save is invalid without them.

### Phase 8 — Synthesis render, save, and close

<!-- covers: F1, F2, G3, 4.3, MM3 -->

**Goal:** show the advisory result where the learner is already looking, preserve it to disk, and close with the decision still open.

#### Step 8.1 — Render in terminal first

Say: `«Вот результаты работы твоих личных советников:»`

Build:

- Render this structure in the terminal before any file write:

```markdown
## Decision brief
- <brief>
## Consensus
| Theme | Shared take | Why it matters |
| --- | --- | --- |
| ... | ... | ... |
## Disagreements
| Theme | Advisor | Take | Tension |
| --- | --- | --- | --- |
| ... | ... | ... | ... |
_DECIDE_:
```

- Keep the advisor takes framed conditionally: `reasoning in X's shape would say...`
- Do not fill `_DECIDE_:`

#### Step 8.2 — Save gate

Say: `«Если ок, сохраню полный разбор в папке решений внутри твоего vault. Если файл уже есть, сначала покажу diff.»`

Check: `«Сохранять?»`

Build:

- If the learner says yes, write only to `<DECISION_ROOT>/<YYYY-MM-DD>-<slug>.md`.
- Save this structure:

```markdown
---
schema_version: 1.0
date: <YYYY-MM-DD>
slug: <slug>
panel: [<advisor-slug-1>, <advisor-slug-2>]
decision_filled: false
---
# <Decision title>
_DECIDE_:
## Decision brief
- <brief>
## Advisor takes
### <Advisor 1>
reasoning in <Advisor 1>'s shape would say...
### <Advisor 2>
reasoning in <Advisor 2>'s shape would say...
## Consensus
| Theme | Shared take | Why it matters |
| --- | --- | --- |
| ... | ... | ... |
## Disagreements
| Theme | Advisor | Take | Tension |
| --- | --- | --- | --- |
| ... | ... | ... | ... |
```

- If the learner says no, keep the synthesis in memory, write `status: "in_progress"`.
- If the learner says yes, read the saved file back and verify:
  - the file exists on disk
  - frontmatter includes `schema_version: 1.0`
  - it contains `Decision brief`
  - it contains one advisor subsection per selected advisor
  - it contains `Consensus`, `Disagreements`, and `_DECIDE_:`
- If verification fails, do not update `decisions[]`; say one short Russian line naming the exact first gap and keep `status: "in_progress"`.
- If verification passes, update `decisions[]` with `slug`, `date`, `path`, `advisors`, `decision_filled: false`, then recompute counters.

#### Step 8.3 — Final verification and state close

Action (silent):

- Verify:
  - `provider.rules_of_use_written == true`
  - provider has a valid `verified_at`
  - at least two `personas[]` entries are `status: "finalized"`
  - verify each finalized persona file still exists on disk
  - at least one decision doc exists on disk
  - the saved decision doc contains `Decision brief`, per-advisor sections, `Consensus`, `Disagreements`, and `_DECIDE_:`
- If any check fails, keep `status: "in_progress"`, say one short Russian line naming the exact first gap, and route back to the first missing artifact.
- If all pass, write `status: "completed"`.

Say: `«Готово. У тебя есть готовые карточки и первый консилиум по живому вопросу. В строке `_DECIDE_:` я оставил место под твой выбор. Когда захочешь новый вопрос или нового советника, снова запусти `/pos-advisors`.»`
