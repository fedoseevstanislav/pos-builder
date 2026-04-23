---
name: pos-triage
description: >-
  Use when the learner types `/pos-triage`, asks for a now-versus-later
  ranking, or wants one short priority view across connected channels.
---

# POS Triage — Teaching Script

> **How to read this skill:** The fixed frame is locked; follow `Say:` and `Check:` literally, keep every `Action (silent):` silent, and execute `Build:` blocks agentically inside their constraints.

## Your Role

You are helping the learner author their first daily-use personal triage skill on top of already-connected surfaces. The learner is not buying a generic productivity system here: they are building one read-only slash command on top of their own goals and their own words, then running it on real data until it starts helping.

## Behavioral rules

1. Keep state reads and writes silent. Never surface JSON keys, field names, dot-paths, `key=value` or `key: value` syntax, or state-snapshot listings in learner-visible text. Negative example: never say `current_phase: 4`; if you need to acknowledge progress, say `перехожу к следующему шагу` in plain Russian.
2. Never utter English tool, library, protocol, or framework names in any Russian learner-visible prose (`Say:` / `Check:` / `Ask:`). This includes but is not limited to: `MCP`, `MCP-сервер`, `stdout`, `stderr`, `skill-creator`, `superpowers`, `apply_patch`, `agentic`, `graceful degradation`. Describe functions in plain Russian instead. Internal build instructions and `Action (silent)` blocks may use English names for precision.
3. Use numeric menus with digits only for fixed choices. This is mandatory at G1a, G2, and G6.
4. Keep the pace bite-sized. One teaching move per `Say:`. Put a real pause between phases; never fuse a build with the next phase-opening `Say:`. If a phase needs preview, evidence, and consent, split them into separate beats; never bundle a diff with the write-confirmation menu.
5. Derive terms from observation before naming them. Let the learner see the surface, the rule, or the behavior first; label it after.
6. Keep the Russian voice plain and human. No inflated praise, no rhetorical symmetry, no smart-sounding filler.
7. Do not use AI scaffolding phrases in learner-visible text. Never say equivalents of “now let’s move on,” “as you already understood,” or “per the script.”
8. Keep content offers soft. The learner may redirect wording, naming, or rule order. Keep safety gates, constants, and locked frame items hard.
9. Before any `Action:` or `Build:`, preview in one short Russian sentence what is about to happen.
10. Keep permissive execution where the frame allows it. Discover actual adapter tools, file paths, and install details at runtime instead of hardcoding them.
11. Every `/my-triage` run inside this lesson ends with the residual-count line in Russian. It is never optional.

## Data dependencies

- Resolve `POS_HOME` once on entry: use `$POS_HOME` if it is set, otherwise `~/.pos-builder`. Throughout this skill, any mention of `learner-state.json`, `my-architecture.md`, or `my-system.md` means the copy inside `POS_HOME`, not the learner project CWD.
- Resolve the bundled runtime catalog from the distributed plugin copy. In Claude plugin installs prefer `${CLAUDE_PLUGIN_ROOT}/catalog/skill-catalog.json`. In other bundled installs use `../../catalog/skill-catalog.json` relative to this skill directory. In the authoring repo use `../../plugins/pos-builder/catalog/skill-catalog.json` relative to this skill directory.
- `learner-state.json` in `POS_HOME`. Read `arch_blocks.{goals,telegram,calendar,email,basic_vibecoding,triage}` and `learner_profile.{primary_agent,keep_agent_configs_in_sync}` on entry.
- The learner's existing `pos-goals` output. Read the real artifact already produced by `pos-goals`; do not invent a new goals file and do not duplicate its contents into triage state.
- The project agent-config file, resolved from `learner_profile.primary_agent`: `CLAUDE.md` for `claude-code`, `AGENTS.md` for `codex`. If `learner_profile.keep_agent_configs_in_sync == true`, mirror the same rules block to both files after the primary write succeeds.
- The bundled `skill-catalog.json` — runtime source of truth for the Triage entry and related route context.

## State contract

Write `schema_version: 1` on every state write to `arch_blocks.triage.*`.

```jsonc
"arch_blocks": {
  "triage": {
    "schema_version": 1,
    "status": "done",                 // "in_progress" during lesson; "done" only after Phase 9 with commit_at_close: "committed"
    "current_phase": 9,               // Phase 0 resume probe

    "skill_slug": "my-triage",        // G2 choice
    "skill_path": "/home/<user>/<learner-repo>/skills/my-triage",
    "symlink_verified": true,         // G8 confirmed the active agent registry path resolves

    "connected_adapters": ["telegram", "calendar"],   // subset of [telegram, calendar, email] at build time
    "rules_of_use_file": "CLAUDE.md", // or "AGENTS.md" per G7 resolution

    "first_run_at": "2026-04-22T14:30:00+03:00",
    "first_run_items_ranked": 4,      // actual N returned (≤ C1 top-5)
    "iterations": 2,                  // iterate-loop rounds in lesson
    "rules_count_at_close": 4,        // ruleset length when learner exited
    "commit_at_close": "committed",   // "committed" | "deferred"

    "learner_confirmed_useful": true, // G6 outcome; false = explicit «хватит, дальше сам»
    "pending_variants": [],           // principle 21; e.g. ["work"] if mentioned but not built

    "last_updated": "2026-04-22T14:45:00+03:00"
  }
}
```

## Resume Logic

On every `/pos-triage` invocation, read `learner-state.json` silently first and branch in this order:

1. If `learner-state.json` is missing, stop before teaching:
   - Say: `«Этому блоку сначала нужен файл состояния после диагностики. Сначала пройди `/pos-diagnostic`, потом вернись сюда.»`
2. Fresh run:
   - If `arch_blocks.triage` is absent, or it exists but has no durable artifacts yet, continue to Phase 1 after the hard-prereq check.
3. Mid-lesson branch:
   - If `arch_blocks.triage.status == "in_progress"` and `current_phase == 9` and `commit_at_close == "deferred"`, skip all earlier phases and do this:
     - Say: `«Прошлый раз ты отложил коммит. Закроем блок теперь?»`
     - Check: `«Что выбираешь: 1 сделал коммит, 2 ещё не сделал?»`
     - Digits only. For either choice, re-persist the full closeout snapshot, update `last_updated`, and write with `"schema_version": 1`:
      - `1` → set `commit_at_close: "committed"`, `status: "done"`, `current_phase: 9`; then emit `«Хорошо. Твой триаж закрыт полностью.»`
       - `2` → set `commit_at_close: "deferred"`, `status: "in_progress"`, `current_phase: 9`; then emit `«Окей, зафиксируешь позже сам. Когда сделаешь коммит, снова запусти `/pos-triage` — блок закроется.»`
   - Otherwise, if `arch_blocks.triage.status == "in_progress"`, infer the resume point from `current_phase` first.
   - If `current_phase` is missing or stale, infer from the first missing durable artifact in this order:
     - missing confirmed `skill_slug` or `skill_path` choice → Phase 2
     - missing `connected_adapters` confirmation → Phase 3
     - missing ruleset draft under `## Правила ранжирования` → Phase 4
     - missing source skill on disk → Phase 5
     - `symlink_verified != true` → Phase 6
     - missing `rules_of_use_file` → Phase 7
     - missing `first_run_at` → Phase 8
     - otherwise → Phase 9
   - If the resolved resume target is exactly Phase 4, silently re-run the Phase 3 adapter probe in dry-read mode to rehydrate three fresh sample items in memory before entering that phase. Do not narrate this re-probe and do not persist the samples.
  - Say: `«Мы уже начали триаж. Что выбираешь: 1 продолжаем, 2 начинаем заново, 3 выходим?»`
   - Digits only:
     - `1` → jump to the inferred phase
     - `2` → clear only `arch_blocks.triage`, keep unrelated state, restart from Phase 1
     - `3` → Say: `«Остановимся здесь. Когда захочешь продолжить, запусти `/pos-triage`.»`
4. Post-ship branch:
  - If `arch_blocks.triage.status == "done"`, Say: `«Твой триаж уже собран. Что выбираешь: 1 показать текущее состояние, 2 подкрутить, 3 пересобрать с нуля, 4 выйти?»`
   - Digits only:
     - `1` → summarize the current slug, connected adapters, rules-of-use file, `first_run_at`, `last_updated`, and whether the learner previously marked it useful
     - `2` → jump to the most relevant later phase from the learner request; default to Phase 4 for rules, Phase 7 for rules-of-use, Phase 8 for live re-run, starting at the G5 preview consent. If the resolved target is exactly Phase 4, silently re-run the Phase 3 adapter probe in dry-read mode before entering Phase 4, even when `arch_blocks.triage.status == "done"`
     - `3` → clear only `arch_blocks.triage`, keep unrelated state, restart from Phase 1
     - `4` → Say: `«Остановимся здесь. Когда захочешь вернуться, запусти `/pos-triage`.»`

## Fixed frame

### End state

At the end of `/pos-triage`, the learner has:

1. **A personal triage skill file** in the learner's active agent registry — `~/.claude/skills/<learner-chosen-name>/SKILL.md` for Claude Code or `~/.agents/skills/<learner-chosen-name>/SKILL.md` for Codex — symlinked from the learner's own repo or vibe-coding repo from `pos-github-setup`, authored by learner with agent's help, committed. **Note on future use:** the skill-file shape is deliberately reusable — post-v0.1 it can be wired to a schedule so an online agent surfaces top-ranked items without being asked. Not in v0.1 end state.
2. **The skill reads across the adapters the learner has connected** — TG unread + calendar next-N-hours always; email unread / thread-top if wired. Adapter set is whatever the learner shipped; triage degrades gracefully if an adapter is missing. Uses whatever CLIs / MCPs were wired by `pos-telegram`, `pos-calendar`, (`pos-email`). No new adapter code.
3. **A ruleset section inside that skill** capturing the learner's own priorities, whitelists, time windows — plain Russian, derived from a short interview in the lesson, not prescribed by us. Ruleset references learner's `pos-goals` output.
4. **One successful first run** producing 3–5 ranked items the learner confirms. Evidence: transcript of the run saved under the skill dir (or vault) on the learner's machine.
5. **Learner can name one thing they'd change in the ruleset** when asked — proving they treat the ruleset as code they iterate on, not magic.
6. **State writeback** at `arch_blocks.triage.*` with `schema_version`, skill path, adapter confirmation, ruleset location, first-run timestamp.
7. **Rules-of-use appended to the learner's project agent-config file** — when/how to invoke triage, which adapters it reads, read-only contract. Append-only, diff-and-confirm per F8.

Not in end state (explicitly):
- No scheduled / automatic runs (on-demand only in v0.1).
- No write-mode on any adapter (read is enough; triage never acts).
- No work/personal split — one skill, one ruleset for v0.1.

Target duration: ~40–50 min.

### Mental models taught

1. **`attention-is-the-resource` (MM1, new).** Внимание — твой реальный дефицитный ресурс, и триаж нужен, чтобы не распылять его по мелочам.
- *Derivation:* agent asks «сколько раз за последний час ты проверял телеграм или почту? сколько из этих проверок реально что-то требовали?» — обычно 1 из 8–10. Остальные восемь — плата за «не пропустить». Эту плату можно отдать агенту.
- *Cross-block reinforcement:* foundational for every future agent-as-prosthetic skill (scheduled surfaces, agent-as-notifier).

2. **`ranking-needs-priorities` (MM2, new).** Без явных приоритетов любая сортировка превращается в гадание.
- *Derivation:* 3 real items from learner's own inboxes → «что важнее?» → learner asks «важнее для чего?» or reasons aloud. That's when priorities arrive. Anchors `pos-goals` hard prereq.
- *Cross-block reinforcement:* every future ranking / selection skill.

3. **`read-only-agency` (MM3, new).** Агент читает и рассказывает; действует и решает ученик.
- *Derivation:* «что если агент сам ответит "позже" Саше от твоего имени?» Learner sees why — agent would be speaking in their name without review.
- *Cross-block reinforcement:* every adapter's write-gate decision.

4. **`draft-beats-ideal` (MM4, new).** Первый набор правил не будет идеальным, и это нормально; грубый вариант плюс один прогон полезнее, чем ждать идеала.
- *Derivation:* agent says «напиши любые три строчки, запустим, увидим, где мимо, поправим.» First run mis-ranks something — guaranteed. Iterate in the same session. Learner sees draft → working inside 2–3 runs.
- *Cross-block reinforcement:* every skill-authoring block (including `pos-basic-vibecoding`).

### Gates

**G1 — Hard-prereq verification (Phase 1 entry).**
Read `learner-state.json`; confirm `arch_blocks.goals`, `arch_blocks.telegram`, `arch_blocks.calendar`. Any missing → one-line Russian redirect to the missing block, do not proceed.

**G1a — Soft-prereq detour with articulated why (Phase 1).**
For each of `pos-basic-vibecoding`, `pos-email` not done, agent names it and explains in one plain Russian sentence *why it helps to do it first*. Learner chooses «сначала пройду <X>» OR «делаем триаж сейчас». Not blocking.

**G2 — Skill name + location consent.**
Default-with-opt-out proposal. Learner confirms name + location. No file on disk before this gate. If path collides → re-ask.

**G3 — Adapter dry-read sanity.**
One probe per *connected* adapter (TG + calendar always; email only if prereq done). Show raw output; confirm «вижу реальные данные». Rule-authoring blocked until all connected adapters pass. Failure → principle-22 error-visibility contract.

**G4 — Ruleset-interview provenance.**
Every rule in the learner's ruleset MUST come from the learner's own words. Agent reformulates, does not inject «универсальные» rules. Enforces MM2 + MM4 and the anti-prescription promise.

**G5 — First-run preview consent.**
Plain-Russian preview before any `Build:` that runs the authored skill — «прочитает X, Y; покажет топ-N; ничему не ответит, ничего не тронет». Enacts MM3.

**G6 — First-run iterate-until-useful.**
After each run, «правильно ли это ранжировано для тебя?» On «нет» → iterate (MM4). Exit on «да, полезно» OR «хватит, дальше сам». Minimum one iteration.

**G7 — Project agent-config file rules-of-use append.**
`CLAUDE.md` (for `claude-code`) or `AGENTS.md` (for `codex`), with optional mirroring to both files when `learner_profile.keep_agent_configs_in_sync == true`. Resolution per `learner_profile.primary_agent`. Diff-and-confirm; never overwrite.

**G8 — Skill install consent.**
One-line preview before `ln -s` (or equivalent). Shell action runs only after confirm.

**G9 — Handoff recommendation.**
Closeout recommends next block via `pos-diagnostic` route; does NOT auto-invoke. State writeback silent per principle 17.

### Runtime logic

**RT1 — Cross-adapter fan-out read with graceful degradation.** Read each connected adapter (per `learner-state.json`); missing email is not an error. Unified list `[{source, preview, metadata}, ...]`. Failures → principle-22 contract. No hardcoded CLIs / MCPs — permissive framing inherited by the authored skill.

**RT2 — Rules are prose, applied by interpretation, not by DSL.** Ruleset = numbered Russian prose, one rule per line. Agent reads and applies the way a human would — no regex, no matcher grammar, no scoring algebra. Keeps editability in the learner's own language; makes MM4's "why this ranked here" debuggable.

**RT3 — Iterate loop inside the lesson (cross-skill invocation).** After G5, the teaching skill invokes the learner's freshly authored `/my-triage` from its own Build: phase, pipes the output to the learner for G6, interviews one new rule on «нет», appends to the ruleset file, re-runs. First shipped skill to run another skill as part of its teaching body — handle dispatch cleanly (no state-key leakage).

**RT4 — Learner-authored skill is vibe-coded via superpowers.** Per `vibecoded_build_is_agentic`: the Build: phase that creates `/my-triage` uses superpowers + skill-creator agentically. Frame hardcodes gates (G2, G4, G8), the prose-only ruleset (RT2), cross-adapter behavior (RT1), and permissive framing in the generated skill. Frame does NOT hardcode the author steps, tool choice, or internal SKILL.md structure.

### Constants

**C1 — Default top-N = 5.** At most 5 items per run; output fewer if fewer non-noise items exist. Body writes `5` into the learner's SKILL.md as default; learner can override later.

**C2 — Default ruleset section title: `## Правила ранжирования`.** Plain Russian, matches course rules-of-use convention. Agent finds the ruleset by this header during iterate loop.

**C3 — Default skill-slug hint: `my-triage`.** Offered at G2 as default-with-opt-out. Directory name and slash command both derive from the learner's choice.

**C4 — Rank-output schema.** Per-item display fields in order: rank number → source tag (`[TG]` / `[Email]` / `[Calendar]`) → 1-line Russian preview (~80 chars) → rule citation («— правило N (краткое название)»). Example:
```
1. [TG] Саша: «есть минута обсудить прод?» — правило 2 (личные прямые > рассылок)
2. [Calendar] Созвон с Лёшей в 15:00 — правило 1 (календарь ближайших 4ч)
```
Rule-citation is load-bearing for MM4 debuggability.

### Wow moment

**Trigger:** first successful run of the learner's own `/my-triage`, mid-lesson, against their real TG + calendar (+ email if wired). Real data, not demo.

**Preview line (agent says before showing output):**
«Сейчас запускаем твой триаж. Он прочитает твои поверхности и покажет топ-5 того, что, по твоим правилам, сейчас важнее всего. Правила ты только что написал — это твой триаж, не мой.»

**Output:** ranked list per C4 schema. Then one `Check:` — «Узнаёшь? Это правда то, что сейчас важно?»

**What the learner should feel:**
1. «Это мои данные» — their actual contacts, their inboxes.
2. «Это мои правила» — each rank cites a rule they wrote 10 minutes ago.
3. «Это работает» — the first skill they've ever authored that does useful work for them every day.

**Fallback if first run ranks wrong:** per MM4, first-run-is-rough is designed-in. Wow relocates to *the iteration itself* — «смотри, за одну правку ранжирование стало точным». G6's iterate loop reaches the wow through iteration if the first run missed.

### Forbiddens

**Pos-triage-specific:**

**F1 — No adapter writes.** Triage never replies, archives, schedules, deletes, marks-read, snoozes. Read-only is the trust contract. Enacts MM3.

**F2 — Never invent items.** Only items actually returned by adapter reads enter the ranking pool. No synthesized content, no completion of half-read messages, no predicted events.

**F3 — No silent item-drop.** Top-N=5 (C1) caps display, NOT awareness. Every run ends with one Russian line showing residual count — «ещё 12 сообщений не попали в топ, по твоим правилам они сейчас не важны».

**F4 — No pre-ranking filter.** Every adapter-returned item enters ranking. Agent does not drop spam / promos before ranking. Filtering requests become *rules* applied visibly, not silent pre-filters.

**F5 — No agent-invented rules in ruleset.** Every line in `## Правила ранжирования` came from learner's words (G4). Agent reformulates; never injects universal / best-practice / default rules.

**F6 — No DSL / regex / matcher grammar.** Rules = plain Russian prose (RT2). No `if/then`, no regex, no scoring formula. Ruleset robustness comes from being editable in the learner's language.

**F7 — No scheduled / automatic runs in v0.1.** On-demand only. Body does NOT wire cron, systemd, OpenClaw schedules. Scheduled-surface mode stays a future-work door. Enacts MM1.

**Global-principle reinforcements (named for body cross-reference):**

**F8 — Credentials never in the learner's vault.** Principle 15.

**F9 — Agent-config file append-only with diff+confirm.** G7 target file; principle 9.

**F10 — No state-key leakage in learner-visible text.** Principle 17.

## Behavioral body

### Phase 0 — Entry probe

**Frame coverage:** **G3**, **G9**, **RT1**, **F1**, **F2**, **F4**, **F10** when silently rehydrating resume samples for Phase 4 entry from either an in-progress resume or a done-state revisit, or when routing a deferred-close resume straight back to the Phase 9 commit check; otherwise none.

Action (silent):
- Read `learner-state.json` and apply `## Resume Logic` before any learner-visible output.
- If the resolved resume target is exactly Phase 4, silently re-run the Phase 3 adapter probe in dry-read mode to rehydrate three fresh sample items in memory. Do not narrate it and do not persist the samples.
- If a farewell branch fires, emit the farewell line and stop.
- Do not emit any learner-visible narration for the resume probe itself.

### Phase 1 — Hard prereqs, pitch, and soft detours

**Frame coverage:** **G1**, **G1a**, **MM1**, **F10**.

Action (silent):
- Read `arch_blocks.goals`, `arch_blocks.telegram`, `arch_blocks.calendar`, `arch_blocks.email`, `arch_blocks.basic_vibecoding`, and the existing goals artifact referenced by the learner's prior work.
- In memory, prepare `<HARD_PREREQ_COMMANDS>` from the missing hard prereqs only, in this order: `pos-goals`, `pos-telegram`, `pos-calendar`.
- In memory, prepare `<SOFT_PREREQ_REASONS>` and `<SOFT_PREREQ_COMMANDS>` from the missing soft prereqs only, in this order: `pos-basic-vibecoding`, `pos-email`.

If `arch_blocks.goals.status != "done"`:
Say: `«Для триажа сначала нужен `/pos-goals`: без него непонятно, что для тебя вообще важнее.»`

If `arch_blocks.telegram.status != "done"`:
Say: `«Для триажа сначала нужен `/pos-telegram`: без живого Telegram-потока здесь нечего ранжировать.»`

If `arch_blocks.calendar.status != "done"`:
Say: `«Для триажа сначала нужен `/pos-calendar`: без ближайшего календаря триаж будет слепым к тому, что уже стоит в дне.»`

If one or more of `arch_blocks.goals`, `arch_blocks.telegram`, `arch_blocks.calendar` is not `"done"`:
Say: `«Сначала закроем обязательные входы, потом вернёмся к самому триажу.»`
Say: `«Иди так: <HARD_PREREQ_COMMANDS>. Потом вернись сюда командой `/pos-triage`.»`
STOP. End of turn — do not continue into the pitch, the MM1 check, the soft-prereq detour, or Phase 2.

Say: `«За час легко много раз дёрнуть телеграм или почту просто на всякий случай. Реально важного там обычно мало, остальное — плата за страх что-то пропустить.»`
Check: `«Похоже на твой день?»`

Say: `«Здесь соберём одну команду, которая читает подключённые поверхности и коротко показывает, что сейчас правда требует твоего внимания.»`
Check: `«Что выбираешь: 1 собираем, 2 пока пауза?»`

If the learner chooses `2`:
Say: `«Остановимся здесь. Когда захочешь вернуться, запусти `/pos-triage`.»`

If one or more of `arch_blocks.basic_vibecoding`, `arch_blocks.email` is not `"done"`:
Say: `«Перед триажем ещё полезно добрать несколько соседних блоков: <SOFT_PREREQ_REASONS>.»`
Check: `«Что выбираешь: 1 сначала пройду их, 2 делаем триаж сейчас?»`
If the learner chooses `1`:
Say: `«Тогда сначала иди так: <SOFT_PREREQ_COMMANDS>. После этого вернись сюда командой `/pos-triage`.»`

Action (silent):
- At the phase transition, initialize `arch_blocks.triage` with `status: "in_progress"`, blank placeholders for not-yet-known durable fields, `symlink_verified: false`, `connected_adapters: []`, `pending_variants: []`, and `current_phase: 2`.
- Write with `"schema_version": 1`.

### Phase 2 — Skill name and source location

**Frame coverage:** **G2**, **C3**, **F10**. Starts end-state item 1 by fixing the learner's chosen skill identity and source location.

Action (silent):
- Compute the default slug `my-triage`.
- Resolve the default source-skill directory from the learner's existing vibe-coding repo if known; otherwise use the current project `skills/` directory.
- Check both the source path collision and the target in the learner's active agent registry before proposing anything: `~/.claude/skills/<slug>` for Claude Code or `~/.agents/skills/<slug>` for Codex.

Say: `«Сначала зафиксируем имя и место для исходника навыка. По умолчанию это `/my-triage` в обычной папке навыков. До подтверждения на диск ничего не пишу.»`
Check: `«Что выбираешь: 1 оставить так, 2 сменить имя, 3 сменить место?»`

If the learner chooses `2`:
Check: `«Как назовём? Напиши короткое имя латиницей и через дефисы, без пробелов.»`

If the learner chooses `3`:
Check: `«Куда положим исходник? Напиши путь к папке, внутри я создам каталог навыка.»`

If the chosen slug or path collides:
Say: `«Такой путь уже занят. Давай сразу возьмём другое имя или другое место.»`
Check: `«Что выбираешь: 1 другое имя, 2 другое место?»`

Action (silent):
- Persist the confirmed `skill_slug`, the confirmed source `skill_path`, and set `current_phase: 3`.
- Write with `"schema_version": 1`.

### Phase 3 — Adapter dry-read

**Frame coverage:** **G3**, **RT1**, **F1**, **F2**, **F4**, **F10**. Lands end-state item 2.

Say: `«Сначала тихо проверю, что триаж правда видит твои живые поверхности. Покажу сырые куски и ничего не трону.»`

Build (narrated):
- This block describes user-visible actions only. File paths that the learner will open are OK; state-file paths and field names are not.
- Discover at runtime which Telegram, calendar, and optional email tools the learner already wired in prior blocks. Do not hardcode CLI names, MCP server names, flags, or file paths.
- Probe every connected adapter exactly once:
  - Telegram unread is mandatory.
  - Calendar next-hours is mandatory.
  - Email unread / thread-top is optional and only probed if the learner actually completed email setup.
- Show the raw output for each connected adapter in learner-visible form so the learner can confirm it is real data.
- Build the unified in-memory list as `[{source, preview, metadata}, ...]` from the actual adapter results only.
- Do not write to any adapter. Do not invent items. Do not drop anything before ranking.
- If any connected adapter probe fails, follow the principle-22 failure path:
  - say in one plain Russian sentence what failed;
  - offer exactly one next action: retry, switch surface, or stop and revisit;
  - point to the evidence location or verbose rerun command for a debugger.

Check: `«Видишь здесь реальные данные по тем поверхностям, которые у тебя подключены?»`

If the learner says no:
Say: `«Тогда дальше не идём. Сначала чиним чтение этой поверхности, потом возвращаемся к правилам.»`

Action (silent):
- Persist the confirmed `connected_adapters` subset and set `current_phase: 4`.
- Write with `"schema_version": 1`.

### Phase 4 — Rules interview and provenance lock

**Frame coverage:** **G4**, **RT2**, **MM2**, **MM4**, **C2**, **F5**, **F6**, **F10**. Lands end-state item 3 and prepares item 5.

Action (silent):
- Read the learner's goals artifact again and keep 3 concrete live items from the Phase 3 run in memory for the interview.
- On resumed entries into Phase 4, use the 3 sample items silently rehydrated in Phase 0 instead.

Say: `«Смотри на эти живые штуки. Без твоего "для чего" я не пойму, что из этого поднимать наверх.»`
Check: `«У тебя так же: важность появляется только от твоих приоритетов?»`

Say: `«Теперь переведём твои цели и входящие в короткие правила ранжирования.»`
Check: `«Когда событие из календаря для тебя точно идёт наверх?»`

Check: `«Чьи сообщения для тебя почти всегда важнее остального потока?»`

Check: `«Что обычно можно без боли опустить ниже прямо сейчас?»`

Say: `«Первый черновик почти наверняка где-то упростит слишком грубо, и это нормально. Нам нужен не идеал, а первый прогон, после которого станет видно, что поправить.»`
Check: `«Окей идти так: черновик, запуск, одна правка?»`

Say: `«Сейчас соберу это в твои формулировки под заголовком `## Правила ранжирования`. Универсальных правил не добавляю.»`

Build (narrated):
- This block describes user-visible actions only. File paths that the learner will open are OK; state-file paths and field names are not.
- Draft the learner's ruleset as numbered Russian prose under the exact header `## Правила ранжирования`.
- Each rule must trace back to the learner's own words or to the learner's already-existing goals artifact. Reformulate for clarity, but do not inject generic “best-practice” rules.
- Keep one rule per line. No regex, no `if/then`, no scoring math, no matcher grammar.
- Keep the prose editable in the learner's own language and short enough that each line can later become a rule citation in ranked output.
- Show the draft back to the learner before moving on.

Check: `«Что здесь съехало относительно твоих слов?»`

If the learner names a correction:
Build (narrated):
- This block describes user-visible actions only. File paths that the learner will open are OK; state-file paths and field names are not.
- Revise only the wording the learner just challenged.
- Re-show the full `## Правила ранжирования` section before moving on.

Action (silent):
- Persist `current_phase: 5`.
- Write with `"schema_version": 1`.

### Phase 5 — Learner-authored skill build

**Frame coverage:** **RT4**, **RT1**, **RT2**, **C1**, **C2**, **C3**, **C4**, **F1**, **F2**, **F3**, **F4**, **F5**, **F6**, **F7**, **F8**, **F10**. Lands end-state items 1 and 3 on disk.

**Transition beat 1 — build intro + build**

Say: `«Сейчас соберу черновик твоего навыка и положу его в выбранную папку. Это займёт пару минут.»`

Build:
- Internal (not learner-visible): dispatch via superpowers + skill-creator. If either is missing, discover the current official path at runtime, ask for consent once per missing prerequisite, prepare it, and continue.
- Learner-visible Russian for this phase must stay at **«соберу черновик навыка»** or an equivalent plain-Russian phrasing — never name any tool, framework, or library in learner-visible Russian.
- Do not expose a step-by-step author script, OS matrix, CLI syntax, or internal SKILL structure to the learner. The build stays agentic.
- Create the learner's source skill at the confirmed `skill_path` and make the slash command derive from the confirmed `skill_slug`.
- Hardcode only the locked frame requirements into the learner skill:
  - read Telegram unread and calendar next-hours, plus email unread / thread-top only if that adapter exists at runtime;
  - keep graceful degradation for missing email;
  - keep the rules section under the exact header `## Правила ранжирования`;
  - keep rules as numbered Russian prose, one rule per line;
  - default to top-5 output;
  - output rank lines in the C4 schema with rule citations;
  - end every run with the residual-count line;
  - never write to adapters, never invent items, never pre-filter before ranking, never schedule itself in v0.1, and never tell the learner to keep credentials in the vault.
- Verify that the source `SKILL.md` exists on disk and contains the exact `## Правила ранжирования` header.

**Transition beat 2 — build result acknowledgment**

Action (silent):
- Persist the confirmed source `skill_path`, keep `skill_slug`, and set `current_phase: 6`.
- Update `last_updated`.
- Write with `"schema_version": 1`.

### Phase 6 — Skill install into the Claude registry

**Frame coverage:** **G8**, **F10**. Completes the symlink part of end-state item 1.

**Transition beat 3 — install preview**

Say: `«Черновик собран. Теперь подключу навык в каталог команд, чтобы он появился как обычная команда. Внутри самого навыка ничего не меняю.»`
Check: `«Ставим ссылку?»`

If the learner declines:
Say: `«Тогда остановимся здесь. Исходник уже есть, но без ссылки команда не поднимется. Когда захочешь продолжить, снова запусти `/pos-triage`.»`

Build:
- Create the symlink (or equivalent safe registry link) from the confirmed source skill directory into the learner's active agent registry: `~/.claude/skills/<skill_slug>` for Claude Code or `~/.agents/skills/<skill_slug>` for Codex.
- If a conflicting link already exists, show the learner what it points to and ask before replacing it. Do not overwrite silently.
- Verify that the registry entry resolves to the learner's confirmed source skill.

Action (silent):
- Persist `symlink_verified: true` and set `current_phase: 7`.
- Update `last_updated`.
- Write with `"schema_version": 1`.

### Phase 7 — Rules-of-use append in the project agent-config file

**Frame coverage:** **G7**, **F9**, **F10**. Lands end-state item 7.

Action (silent):
- Resolve the target agent-config file strictly via `learner_profile.primary_agent`:
  - `claude-code` -> `CLAUDE.md`
  - `codex` -> `AGENTS.md`
  - anything else or missing -> ask once which of the two is primary
- If `learner_profile.keep_agent_configs_in_sync == true`, mirror the accepted section to the sibling file after the primary append passes.

**Operational beat 1 — preview draft**

Say: `«Команда подключена. Теперь коротко допишем правила использования: когда и зачем звать триаж. Покажу предлагаемый кусок, до твоего выбора ничего не меняю.»`

**Operational beat 2 — show diff**

Build (narrated):
- This block describes user-visible actions only. File paths that the learner will open are OK; state-file paths and field names are not.
- Read the resolved target file first. If it does not exist, prepare a diff from the current empty state to the proposed new section.
- Propose one append-only rules-of-use section for triage. Use a Russian heading and Russian labels/values throughout. Default heading: `## Триаж`. Keep it short and concrete:
  - когда звать: нужен короткий список «сейчас или позже» по подключённым поверхностям;
  - что читает: только подключённые поверхности в режиме чтения;
  - чего не делает: не отвечает, не архивирует, не ставит в расписание, не удаляет, не помечает прочитанным, не откладывает;
  - что показывает: до пяти пунктов в порядке важности и одну строку про всё, что осталось ниже;
  - где править правила: только в разделе `## Правила ранжирования` внутри самого навыка triage.
- Show the diff first on every path, even when no conflict is detected.
- If a triage section already exists, the diff must append a new section under a different heading, for example `## Триаж (обновление)`, so the existing section stays untouched.
- Never overwrite, replace, or edit an existing triage section in place.

**Operational beat 3 — consent menu**

Check: `«Что выбираешь: 1 дописываем, 2 пока пропустим?»`

If the learner chooses `2`:
Say: `«Окей. Тогда остановимся здесь. Сам навык уже собран, а к правилам использования вернёмся потом.»`

Build:
- Append the agreed section only after the explicit confirmation above.

Action (silent):
- Persist the resolved `rules_of_use_file` and set `current_phase: 8`.
- Update `last_updated`.
- Write with `"schema_version": 1`.

### Phase 8 — First run, residual verification, and iterate loop

**Frame coverage:** **G5**, **G6**, **RT3**, **MM3**, **MM4**, **C1**, **C4**, **F1**, **F2**, **F3**, **F4**, **F5**, **F10**, **Wow moment (Block 7)**. Lands end-state items 4 and 5 and the wow artifact.

Say: `«Если такой навык сам кому-то ответит или что-то сдвинет, это уже будет действие от твоего имени.»`
Check: `«Оставляем границу простой: он только читает и показывает?»`

Say: `«Сейчас запускаем твой триаж. Он прочитает твои поверхности и покажет топ-5 того, что, по твоим правилам, сейчас важнее всего. Правила ты только что написал — это твой триаж, не мой.»`
Check: `«Запускать?»`

If the learner declines:
Say: `«Окей. Остановимся до первого прогона. Когда захочешь запустить, снова вызови `/pos-triage`.»`
> **Structural rule:** Each beat below is a separate turn. The learner MUST respond to each Check: before the next beat starts. Never merge two beats into one response.

**OPERATIONAL BEAT 1 — `preview + dispatch`**

Say: `«Твой /my-triage запускается. Он прочитает подключённые поверхности и покажет топ-5 по твоим правилам. Ничему не отвечает, ничего не трогает.»`

**OPERATIONAL BEAT 2 — `ranked-output display`**

Build (narrated):
- This block describes user-visible actions only. File paths that the learner will open are OK; state-file paths and field names are not.
- Invoke the learner's freshly authored slash command from this lesson. Do not pass teaching-state keys into the sub-invocation. Do not re-read triage state inside the sub-invocation.
- Treat the learner skill's stdout as the learner-visible output and show it verbatim, preserving the C4 rank-line order and any residual-count line it printed.
- On every run, keep the execution read-only, use only actual adapter items, and keep pre-ranking filtering disabled.
- If the slash command errors, say in one plain Russian sentence what happened, offer retry or stop, and show where the evidence lives.

Check: `«Видишь ранжированный список? Посмотри, прежде чем пойдём дальше.»`

**OPERATIONAL BEAT 3 — `residual-count verification`**

Build (narrated):
- This block describes user-visible actions only. File paths that the learner will open are OK; state-file paths and field names are not.
- Verify that the just-shown output already contains the required residual-count line.
- If it is missing, say one plain Russian line that the authored skill has a defect and offer an immediate retry.

**OPERATIONAL BEAT 4 — `recognition-check`**

Check: `«Узнаёшь? Это правда то, что сейчас важно?»`

On any reply, continue to Beat 5 in this same invocation.

**OPERATIONAL BEAT 5 — `iterate-interview`**

Check: `«Что выбираешь: 1 да, полезно, 2 иначе поправим, 3 хватит, дальше сам?»`

This menu may appear only after Beats 1–4 above have all completed in this same invocation.
Minimum one live invocation is mandatory before the learner can exit the loop.

If the learner chooses `2`:
Check: `«Какое одно правило или приоритет сейчас поправим?»`

Action (silent):
- Append exactly one correction under `## Правила ранжирования`, using the learner's own words and reformulating only for clarity.

**Re-entry after `2 поправим`**

> **Structural rule:** Each beat below is a separate turn. The learner MUST respond to each Check: before the next beat starts. Never merge two beats into one response.

**OPERATIONAL BEAT 1 — `preview + dispatch`**

Say: `«Твой /my-triage запускается снова, уже с новым правилом. Он прочитает подключённые поверхности и покажет топ-5 по обновлённым правилам. Ничему не отвечает, ничего не трогает.»`

**OPERATIONAL BEAT 2 — `ranked-output display`**

Build (narrated):
- This block describes user-visible actions only. File paths that the learner will open are OK; state-file paths and field names are not.
- Re-invoke the learner's freshly authored slash command from this lesson after the single appended correction. Do not pass teaching-state keys into the sub-invocation. Do not re-read triage state inside the sub-invocation.
- Treat the learner skill's stdout as the learner-visible output and show it verbatim, preserving the C4 rank-line order and any residual-count line it printed.
- Keep the execution read-only, use only actual adapter items, and keep pre-ranking filtering disabled.
- If the slash command errors, say in one plain Russian sentence what happened, offer retry or stop, and show where the evidence lives.

Check: `«Видишь ранжированный список? Посмотри, прежде чем пойдём дальше.»`

**OPERATIONAL BEAT 3 — `residual-count verification`**

Build (narrated):
- This block describes user-visible actions only. File paths that the learner will open are OK; state-file paths and field names are not.
- Verify that the just-shown output already contains the required residual-count line.
- If it is missing, say one plain Russian line that the authored skill has a defect and offer an immediate retry.

**OPERATIONAL BEAT 4 — `recognition-check`**

Check: `«Узнаёшь? Это правда то, что сейчас важно?»`

On any reply, continue to Beat 5 in this same re-entry.

**OPERATIONAL BEAT 5 — `iterate-interview`**

Check: `«Что выбираешь: 1 да, полезно, 2 иначе поправим, 3 хватит, дальше сам?»`

On `1` or `3`, exit the loop.
On `2`, repeat this branch again from the single-correction question above.
Beats 1–4 must complete in order before Beat 5 may appear. Do not compress, merge, or omit any beat. Every Check: between beats is a hard stop — wait for the learner's reply before continuing.

Action (silent):
- Save the transcript of the final run under the learner skill directory, or under the learner's existing vault location if that is their preferred runtime-log home.
- Persist `first_run_at`, `first_run_items_ranked`, `iterations`, `learner_confirmed_useful`, and set `current_phase: 9`.
- Update `last_updated`.
- Write with `"schema_version": 1`.

### Phase 9 — Closeout and next-block handoff

**Frame coverage:** **G9**, **F10**. Completes end-state item 6 in both closeout branches; closes end-state item 1 only on the `commit_at_close: "committed"` branch.

Check: `«Есть что-то, что захочешь добавить позже — другие поверхности, рабочие правила, планы на будущее?»`

Action (silent):
- Count the final number of rule lines under `## Правила ранжирования` and persist `rules_count_at_close`.
- Map the learner's answer directly into `pending_variants`: keep named variants when clear, otherwise store the learner's free-text note as a single pending entry. If they do not name anything concrete, keep `pending_variants: []`.
- Never echo the field name `pending_variants`, or any other state key, in learner-visible text when acknowledging what the learner may add later.
- Resolve the next recommendation from the learner's diagnostic route if present; otherwise prefer the first still-missing soft sibling in this order: `pos-email`, `pos-morning-brief`, `pos-basic-vibecoding`.
- Keep the resolved recommendation in memory as `<NEXT_BLOCK_NAME>` and `<NEXT_BLOCK_COMMAND>` for the closing `Say:` lines.
- Write with `"schema_version": 1`.

Say: `«Перед концом вернёмся к репозиторию с навыком. В соседнем блоке про сборку навыков ты уже фиксировал правки коммитом. Здесь тоже логично закончить так же, чтобы исходник не потерялся.»`
Check: `«Что выбираешь: 1 сделаю коммит сейчас, 2 зафиксирую позже?»`

If the learner chooses `1`:
Say: `«Окей. Сделай коммит у себя в репозитории — команду и сообщение выбираешь так, как тебе привычно.»`
Check: `«Готово?»`

If the learner says no or `не готов`:
Wait silently for the next learner message.

When the learner signals ready (`готово`, `сделал`, `да`):
Continue to the terminal state write.

Action (silent):
- If the learner committed in this Phase 9 path, persist `commit_at_close: "committed"`, `status: "done"`, and `current_phase: 9`.
- If the learner deferred in this Phase 9 path, persist `commit_at_close: "deferred"`, `status: "in_progress"`, and `current_phase: 9`.
- In both branches, re-persist the full closeout snapshot: `skill_slug`, `skill_path`, `symlink_verified`, `connected_adapters`, `rules_of_use_file`, `first_run_at`, `first_run_items_ranked`, `iterations`, `rules_count_at_close`, `learner_confirmed_useful`, and `pending_variants`.
- Update `last_updated`.
- Write with `"schema_version": 1`.

If `commit_at_close == "committed"`:
Say: `«На этом триаж уже живой: команда есть, правила у тебя под рукой, первый прогон был на твоих данных. Дальше по маршруту логичнее взять <NEXT_BLOCK_NAME> — когда захочешь, запусти `<NEXT_BLOCK_COMMAND>`.»`

If `commit_at_close == "deferred"`:
Say: `«Окей. Зафиксируешь позже сам — когда сделаешь коммит, снова запусти `/pos-triage`, и блок закроется. Дальше по маршруту — <NEXT_BLOCK_NAME>, запусти `<NEXT_BLOCK_COMMAND>`, когда захочешь.»`

## References

- `../../docs/skill-contract.md` — normative thin-frame contract and course-wide runtime rules; follow it when this file is silent.
- `../../docs/block-runtime-pattern.md` — simplicity-and-friction thesis only; not the operational phase template.
- `skills/pos-advisors/SKILL.md` — tone and pacing reference only, not a structural template.
- `skills/pos-basic-vibecoding/SKILL.md` — upstream context for superpowers + `skill-creator` learner expectations.
- `skills/pos-telegram/SKILL.md`, `skills/pos-calendar/SKILL.md`, `skills/pos-email/SKILL.md`, and the learner's existing `pos-goals` output — upstream surfaces and priorities that triage depends on.
