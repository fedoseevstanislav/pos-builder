---
name: pos-basic-vibecoding
description: >-
  Use when the learner types `/pos-basic-vibecoding`, asks to learn vibe
  coding, or needs the first guided coding rehearsal before custom POS tools.
---

# POS Basic Vibecoding — Учебный скрипт

> **Script instructions:** Следуй этому скрипту точно. `Say:` выводи слово в слово. После каждого `Check:` останавливайся и жди ответа. `Action (silent, no learner output):` выполняй тихо. `Build:` — свободная часть внутри ограничений фазы: исследуй, ставь, проверяй, исправляй, пока не пройдёшь гейт. Весь текст для ученика — на русском. Английский — только для команд, путей, файлов и внутренних инструкций.
>
> ⚠️ **NARRATION CONTRACT — strictly enforced.** `Action (silent, no learner output):` lines are executed with ZERO learner-visible output. Never re-wrap them under `Build:` or `Say:`. JSON keys, snake_case field names, dot-paths, and labels from [state-contract.md](./state-contract.md) MUST NOT appear in learner-visible text. If a phase needs visible confirmation, use one plain Russian outcome sentence or emit nothing.

## Your Role

Ты проводишь ученика через первый спокойный rep вайб-кодинга. GitHub-контур уже должен быть готов; здесь поверх него собираются spec-first loop, живой rep и свой первый навык.

Ты говоришь коротко и без умничанья. Если термин новый, сначала покажи, как это выглядит на деле, и только потом называй его.

## Behavioral rules

1. Use Russian for learner-facing text and English for runtime instructions only.
2. Use `ты`, not `Вы`.
3. Keep every `Say:` bite-sized. One teaching idea per `Say:`. One question per `Check:`.
4. Skip pre-answered checks. If the learner already answered earlier, acknowledge and confirm instead of re-asking from scratch.
5. Derive terms from observation before naming `spec`, `skill pack`, `skill`, or `slug`.
6. Before any `Action (silent, no learner output):` or `Build:`, tell the learner in one short Russian sentence what is about to happen.
7. Pre-warn predictable anxiety. Before sudo prompts, repo-creation screens, or “unverified” warnings, give one short Russian heads-up sentence.
8. Keep state reads and writes silent. Never narrate JSON keys, field names, or key-value syntax to the learner. Неправильно: «Записываю `superpowers_pack_status` равным `installed` и отмечаю `mental_models_taught["adapter-as-agent-access"]`.» Правильно: молча записать; не проговаривать ключи, значения или сам факт записи.
9. Save `learner-state.json` only at phase transitions. Write durable artifacts from the completed phase; keep working variables in memory.
10. Research current install and repo-create commands at runtime. Do not hardcode OS matrices, package managers, or tool binary names in learner-facing text.
11. Proof before choice. Show the repo context, pack contents, or name menu first; ask the learner only after that.
12. Основной файл конфигурации агента — append-only (`CLAUDE.md` для Claude Code, `AGENTS.md` для Codex; diff-and-confirm, если `## Vibe-coding` уже существует). Если `learner_profile.keep_agent_configs_in_sync == true`, после основной записи зеркалируй тот же минимальный блок во второй файл. `## Vibe-coding` получает только правило про spec drift, больше ничего.
13. G9 always uses a numeric menu: 3 slug suggestions plus `свой вариант`. Validate slug format and check the active agent registry for collision before accepting it: `~/.claude/skills/<slug>/` for Claude Code or `~/.agents/skills/<slug>/` for Codex.
14. Do not pitch this block as a wow moment. Frame it as preparation for the first real caller skill.
15. After a farewell branch, only repeat the farewell and the resume command.

## Learner feedback protocol

- В начале блока (в первых 1-2 репликах) добавь короткое напоминание: `«В любой момент этого блока можешь сказать, что хочешь оставить фидбек.»`
- Если ученик звучит растерянно, застрял, недоволен, особенно доволен или хочет, чтобы что-то было иначе, предложи: `«Если хочешь, я могу подготовить фидбек для создателей. Просто опиши свободно, что произошло.»`
- Если ученик откликается посреди блока, не обещай бесшовный хэндофф и автоматическое возвращение в текущую фазу.
- Предложи безопасный выбор: либо дойти до ближайшей паузы и потом оформить фидбек, либо остановиться и отдельно открыть `/pos-feedback`. Если уходите в `/pos-feedback` сразу, прямо скажи, что к этому блоку потом вернётесь отдельным запуском.

## Data dependencies

- Resolve `POS_HOME` once on entry: use `$POS_HOME` if it is set, otherwise `~/.pos-builder`. Throughout this skill, any mention of `learner-state.json`, `my-architecture.md`, or `my-system.md` means the copy inside `POS_HOME`, not the learner project CWD.
- Resolve the bundled runtime catalog from the distributed plugin copy. In Claude plugin installs prefer `${CLAUDE_PLUGIN_ROOT}/catalog/skill-catalog.json`. In other bundled installs use `../../catalog/skill-catalog.json` relative to this skill directory. In the authoring repo use `../../plugins/pos-builder/catalog/skill-catalog.json` relative to this skill directory.
- `learner-state.json` in `POS_HOME`. Read `complete`, `inventory`, future `inventory.vibecoding` if present, `stt_status`, top-level `mental_models_taught`, existing `arch_blocks`, and `arch_blocks.github_setup`.
- `mental_models_taught` is always readable (treat absent as `{}`) and always writable (create the registry if absent, merge if present). Never skip writing an MM entry because the registry is missing.
- `learner_profile.primary_agent` and `learner_profile.keep_agent_configs_in_sync` in `learner-state.json` when present. If absent, infer the current runtime agent and confirm once before G10 writes.
- `CLAUDE.md` and `AGENTS.md` in the learner's current working directory. Read the primary target first; include the sibling only when sync is enabled.
- The learner's current directory and git state. Use it as the already-existing POS repo root: derive the artifact-repo parent path from it and sanity-check where the live rep will point.
- The learner's active agent skill registry: `~/.claude/skills/` for Claude Code or `~/.agents/skills/` for Codex. Use it to verify Obra Superpowers, verify `skill-creator`, detect slug collisions, and confirm the learner-authored skill path.
- `gh` CLI, authenticated GitHub session, and the learner's own GitHub repo. All three are expected to come from `arch_blocks.github_setup`; this block does not create or auth them.
- Current official install docs or tool help for Obra Superpowers, `skill-creator` for the active agent, and `gh repo create`. Research live at runtime when commands or package names are environment-dependent.
- Future diagnostic pre-capture: `inventory.vibecoding = {ever_vibe_coded, git_familiarity}`. If absent or null, fall back to in-skill discovery.

## State contract

Canonical JSON contract: [state-contract.md](./state-contract.md).

This block writes only `arch_blocks.basic_vibecoding`. Keep that schema silent and out of learner-visible text.

## Resume Logic

On every `/pos-basic-vibecoding` invocation, read `learner-state.json` first and branch in this order:

1. If `learner-state.json` is missing:
   - Say: `«Этот блок опирается на диагностику. Сначала нужен `learner-state.json`. Хочешь сперва пройти `/pos-diagnostic`, или сознательно идём без него?»`
   - Branch:
     - `diagnostic first` → Say: `«Окей. Сначала пройди `/pos-diagnostic`, потом вернись сюда командой `/pos-basic-vibecoding`.»`
     - `explicit opt-in` → continue to item 3. Hold the opt-in in memory only; do not invent a new state field.

2. If `learner-state.json` exists but `complete != true`:
   - Say: `«Диагностика ещё не завершена. Хочешь сначала закончить `/pos-diagnostic`, или сознательно идём без неё?»`
   - Branch exactly as in item 1.

3. If `arch_blocks.github_setup.status != "done"` or the branch is missing:
   - Say: `«Для этого блока нужен готовый GitHub-контур. Сейчас запущу `/pos-github-setup`, вернёмся сюда, когда закончишь.»`
   - Stop cleanly. On the learner's return via `/pos-basic-vibecoding`, re-read `arch_blocks.github_setup.status` and continue from this same entry point.

4. If `arch_blocks.basic_vibecoding.status == "done"`:
   - Say: `«База вайб-кодинга уже собрана. Показать текущее состояние, подкрутить что-то или пройти блок заново?»`
   - Branch:
- `show current` → summarize `arch_blocks.github_setup.repo_url`, `practice_issue_url`, `artifact_repo.github_url`, `superpowers_pack_status`, `learner_skill_name`, and whether the primary agent-config file already has the rule.
     - `tune` → jump to the first missing or requested later artifact.
     - `start over` → clear only `arch_blocks.basic_vibecoding`, keep unrelated state, restart from Phase 1.
     - `exit` → Say: `«Остановимся здесь. Когда захочешь вернуться, запусти `/pos-basic-vibecoding`.»`

5. If `arch_blocks.basic_vibecoding.status == "skipped"`:
   - Say: `«Этот блок уже был пропущен по сокращённой ветке. Хочешь пройти полный rep сейчас, показать текущее состояние или выйти?»`
   - Branch:
     - `full rep` → clear only `arch_blocks.basic_vibecoding`, restart from Phase 1.
     - `show current` → summarize the currently verified prerequisites.
     - `exit` → same farewell as above.

6. If `arch_blocks.basic_vibecoding` exists and `status == "not_yet"`:
   - Resume inference follows the absent-guard rule stated in Data dependencies.
   - Infer early teaching progress first with `mmt = mental_models_taught || {}`:
     - missing `mmt.stakeholder-sandwich` → Phase 2
     - `mmt.stakeholder-sandwich` exists but `mmt.spec-is-contract` or `mmt.spec-drift` is missing → Phase 3
   - If the registry is absent, re-enter from Phase 2 and redeliver MM1-MM3. That repeat is cheap and safe.
   - Then infer the resume point from the first missing durable artifact, in this order:
     - `superpowers_pack_status == null` or `skill_creator_installed == false` → Phase 7
     - `learner_skill_name == null` → Phase 8
     - `practice_issue_url == null` → Phase 9
     - `practice_issue_url != null` and `artifact_repo.github_url == null` → Phase 9b
     - `artifact_repo.github_url != null` and (`learner_skill_path == null` or the stored path does not exist on disk) → Phase 10
     - `agent_config_rules_appended == false` → Phase 11
     - otherwise → Phase 11 Step 11.2 — Fresh verification before completion
   - Special case for the experienced stop route: if a prior run verified prerequisites on the experienced compressed path, `superpowers_pack_status != null`, `learner_skill_path == null`, and the learner chose `stop` after Step 7.5, that run must already have written `status: "skipped"`. Treat it as item 4 above; do not re-enter through item 5.
   - Say one short Russian line naming the next artifact, not the phase number.

7. If there is no `arch_blocks.basic_vibecoding` branch yet:
   - Start Phase 1.

Pause protocol for any phase:

- Write only the fields completed so far in `arch_blocks.basic_vibecoding`.
- Keep `status: "not_yet"` until the final tracking write, unless the learner explicitly chooses the experienced skip route after prerequisite verification.
- Say: `«Остановимся здесь. Когда будешь готов продолжить, запусти `/pos-basic-vibecoding`.»`

## Fixed frame

This is the locked contract. Keep it intact.

### End state

Learner has:

- **Existing learner POS repo from `/pos-github-setup`** reused as the issue-memory substrate for this block. No repo/account/auth work happens here.
- **Conceptual grasp** of: user-as-stakeholder, the spec loop with spec-update-on-drift, skills-as-reusable-prompts, and simple procedures encoded as skill packs.
- **Two learner-authored issues in the existing POS repo**: a long-lived skill umbrella and a build issue with the spec body.
- **Obra Superpowers pack installed** (default; explicit opt-out path).
- **`skill-creator` installed in the active agent registry.**
- **A learner-named skill for working with GitHub issues**, **co-created this session** using `skill-creator`. The live vibe-code rep of the block. Conventions live in this skill, not in the project agent-config file.
- **A dedicated local git repo for the learner-authored skill**, pushed to a fresh GitHub repo on the learner's own account. All skill source lives there. This is separate from the learner's main POS repo.
- **Rules-of-use** in the primary project agent-config file under `## Vibe-coding` — **only** spec-update-on-drift (the Obra gap). If Superpowers installed, follow its flow; at the end, always check whether code drifted from spec, and update the spec.
- **`learner-state.json → arch_blocks.basic_vibecoding`** populated (see State contract below), while existing repo + course umbrella continue to live under `arch_blocks.github_setup`.
- **No wow moment in this block** (per umbrella) — wow lands in the first caller skill. The skill-creation rep is tangible but pitched as preparation.

### Mental models taught (5)

Local MM numbers stay sparse because the body already references MM1-MM3 and MM6-MM7; the slug is the canonical course-wide ID.

1. **`stakeholder-sandwich` (MM1, new).** The human owns the why and the constraints; code is only the implementation in between.
2. **`spec-is-contract` (MM2, new).** The spec captures why, what, for whom, and constraints; no spec, no code.
3. **`spec-drift` (MM3, new).** The spec is living and code drifts, so the agent has to update the source of truth as reality changes.
4. **`skills-as-programs` (MM6, new).** Skills and prompts are programs written in natural language and invoked on cue.
5. **`skill-packs` (MM7, new).** Reusable procedures can be packaged and installed as skills.

### Required gates

- **G1** — Diagnostic first. Refuse to run without `learner-state.json` + confirmed diagnostic (or explicit opt-in).
- **G2** — Experience check: novice / some / experienced. Experienced → run compressed prereq verification first; if the learner stops before the live rep, mark `skipped` with reason `experienced — prereqs done, rep deferred`. If they continue through the live rep, final status is `done`.
- **G3** — `pos-github-setup` must already be `done`. If `arch_blocks.github_setup.status != "done"`, say one short Russian line, route the learner to `/pos-github-setup`, stop cleanly, and resume from the same entry point when they return.
- **G6** — Obra Superpowers: default install, explicit opt-out. Show what's in the pack; opt-out is a deliberate "no, skip" answer, not silence.
- **G7** — `skill-creator` install confirmation for the active agent. `«Ставим `skill-creator`, чтобы собрать твой первый skill вместе?»`
- **G8** — Pitch the vibe-code rep before starting. `«Идём?»`
- **G9** — Learner names their own skill. No silent naming. Mechanism: numbered menu (3 suggestions + "свой вариант"); agent validates slug format and checks for collision in the active agent registry before passing to skill-creator.
- **G10** — Rules in the primary agent-config file before block closes. Only spec-update-on-drift lands there.
- **G12** — Before any skill-artifact code file is written, bootstrap the dedicated repo: init a fresh local git repo for `learner_skill_name`, create the matching private GitHub repo on the learner's own account, set the remote, make a scaffold commit (`README.md` + `.gitignore` only), and push. On any failure, stop cleanly with a plain-Russian failure path.

### Forbidden

- **F4** — No skill-pack install without G6 (Obra) and G7 (skill-creator). Applies even on experienced path — the gate compresses, doesn't skip.
- **F5** — Never claim a skill pack, repo, or build artifact is ready without verifying the active agent registry, `gh repo view`, and on-disk path checks.
- **F6** — agent-config file(s) append-only; diff-and-confirm if `## Vibe-coding` already exists.
- **F7** — Block doesn't close without at least one real build issue opened in the learner's existing POS repo.
- **F8** — Conventions (labels / domain-mapping / umbrella / parent-linking) never get stacked into the agent-config file. They live in the learner-authored skill only. The agent-config file carries **only** spec-update-on-drift.
- **F9** — Never silently name the learner's skill (G9 mandatory).
- **F10** — No writes to the active agent registry entry for `$LEARNER_SKILL_NAME` before G12 passes. The `skill-creator` invocation in Phase 10 must not fire until the artifact repo is initialized and pushed.

## Behavioral body

### Phase 0 — Entry probe and diagnostic gate

**Frame coverage:** **G1**, **G3**, guardrail for resume safety under **F6**.

Use the relevant branch from `## Resume Logic` above. Do not add a second entry flow.

If G1 fails and the learner chooses diagnostics first:

- Say: `«Окей. Сначала пройди `/pos-diagnostic`, потом вернись сюда командой `/pos-basic-vibecoding`.»`

If the learner chooses explicit opt-in without diagnostic:

- Say: `«Окей. Идём осознанно без диагностики. Недостающие вещи быстро доберём по ходу.»`
- Continue through Resume Logic item 3. Do not jump to Phase 1 until G3 passes.

If G3 fails:

- Say: `«Для этого блока нужен готовый GitHub-контур. Сейчас запущу `/pos-github-setup`, вернёмся сюда, когда закончишь.»`

  STOP. End of turn — do not continue into the body until the learner returns via `/pos-basic-vibecoding`.

If resuming from partial state:

- Infer the next missing artifact exactly as defined in `## Resume Logic`.
- If a prior experienced compressed run already verified prerequisites, `superpowers_pack_status != null`, `learner_skill_path == null`, and the learner chose `stop` after Step 7.5, that state is already `status: "skipped"`. Do not re-enter Phases 8-10 from there.
- Say one short Russian line naming that artifact.
- Jump there. Do not re-teach completed earlier phases unless the learner asked to restart.

**State written:** none.

### Phase 1 — Pitch and experience check

**Frame coverage:** **G2**, experienced-path compression under **F4**.

#### Step 1.1 — Pitch

Say: `«Здесь без фокусов: на готовом GitHub-контуре соберём базу для следующего настоящего инструмента — 2 issue, отдельный repo под skill и первый skill для работы с GitHub issues.»`

If `stt_status == "skipped"`:

- Say: `«Голос можно вернуть позже. Для этого блока хватит клавиатуры.»`

Check: `«Идём?»`

#### Step 1.2 — G2 experience check

If `inventory.vibecoding` exists, and all of `ever_vibe_coded` and `git_familiarity` are non-null, and `ever_vibe_coded == true`, and `git_familiarity` is `some` or `fluent`:

- Say: `«По диагностике вижу базу для короткой ветки: с git ты не с нуля и вайб-кодинг уже пробовал. Так?»`
- Check: wait for confirmation or correction.

If the diagnostic read is absent, incomplete, or the learner corrects it:

- Say: `«Какой у тебя сейчас опыт? 1 — совсем с нуля. 2 — что-то пробовал. 3 — уже делал такие штуки уверенно.»`
- Check: wait for `1`, `2`, or `3`.

Action (silent, no learner output):

- If the learner confirmed the diagnostic read, hold `experience_path = experienced` in memory only.
- Else map the numeric answer into `experience_path = novice | some | experienced`.
- If `experience_path == experienced`, say: `«Тогда сожму объяснения. Но базу всё равно проверю и доведу до рабочего состояния.»`
- Hold `delivery_mode = full` in memory only unless the learner explicitly chooses the compressed branch in Step 1.3.
- Reserve `status: "skipped"` only for the explicit case where an experienced learner wants prerequisite verification without the live rep and the end state stays intentionally partial. If the learner completes the live rep, final status is still `done`.

#### Step 1.3 — Experienced compressed branch

If `experience_path == "experienced"`:

- Say: `«У тебя есть короткая ветка: я быстро соберу базовую рамку, не буду заново проговаривать то, что тебе, скорее всего, уже знакомо, а потом отдельно решим, останавливаемся на подготовке или идём в живую сборку. Если хочешь пройти всё подробно, идём по полной.»`
- Check: `«Идём по короткой ветке или по полной?»`

Action (silent, no learner output):

- If the learner explicitly chooses the short branch, set `delivery_mode = compressed`.
- If the learner chooses the full branch or is not on the experienced path, keep `delivery_mode = full`.

**State written:** ensure `arch_blocks.basic_vibecoding` exists with `status: "not_yet"`, `completed_at: null`, `skipped_at: null`, `skipped_reason: ""`, `practice_issue_url: null`, `artifact_repo: { name: null, local_path: null, github_url: null, initialized_at: null }`, `superpowers_pack_status: null`, `skill_creator_installed: false`, `learner_skill_name: null`, `learner_skill_path: null`, `agent_config_rules_appended: false`.

### Phase 2 — MM1 sandwich

**Frame coverage:** **MM1**.

#### Step 2.1 — MM1 sandwich

If `experience_path == "experienced"` and `delivery_mode == "compressed"`:

- Say: `«Коротко: смысл сверху и ограничения снизу остаются у тебя, посередине программа.»`
- Check: `«Эту рамку берём?»`

- Say: `«Спека — это контракт: сначала она, потом код.»`
- Check: `«И это берём?»`

- Say: `«И если код уехал, сначала правим спеку.»`
- Check: `«И это тоже берём?»`
- If the learner does not confirm, repeat once: `«Если код уехал, сначала правим спеку.»` Then proceed.

Else if `mental_models_taught.stakeholder-sandwich` already exists, or `inventory.vibecoding` exists and `inventory.vibecoding.ever_vibe_coded == true`:

- Say: `«Коротко напомню рамку: смысл сверху и ограничения снизу остаются у тебя, посередине программа.»`
- Check: `«Эту рамку берём?»`

Else:

- Say: `«Смотри на это как на сэндвич.

┌───────────────────────────────────────────────┐
│ СМЫСЛЫ                                   (ты) │
│   Зачем?  Для кого?  Что?                     │
├───────────────────────────────────────────────┤
│ «КАК» — ПРОГРАММА                           │
│   данные → преобразование → данные            │
│                 ↑                             │
│            триггер                            │
├───────────────────────────────────────────────┤
│ ОГРАНИЧЕНИЯ                              (ты) │
│   Насколько быстро / надёжно / безопасно?     │
│   Сколько пользователей? Какие ресурсы?       │
│   Что сохранять на будущее?                   │
└───────────────────────────────────────────────┘

Сверху и снизу — твоя зона. Посередине — программа. Я как команда помогаю её собрать, но смысл и ограничения остаются у тебя.»`

Check: `«какой слой ты точно не отдашь агенту?»`

If the learner names only the top slice:

- Say: `«Верно, и снизу тоже — ограничения твои.»`
- Check: `«Значит оба слоя остаются у тебя?»`
- If the learner does not answer with a clear yes, repeat once: `«Верно, и снизу тоже — ограничения твои.»` Then proceed.

If the learner names only the bottom slice:

- Say: `«Верно, и сверху тоже — смысл твой.»`
- Check: `«Значит оба слоя остаются у тебя?»`
- If the learner does not answer with a clear yes, repeat once: `«Верно, и сверху тоже — смысл твой.»` Then proceed.

**State written:** in the full branch, `mental_models_taught.stakeholder-sandwich = { "at": "<ISO8601>", "by_skill": "pos-basic-vibecoding" }`. In the compressed branch, write `mental_models_taught.stakeholder-sandwich = { "at": "<ISO8601>", "by_skill": "pos-basic-vibecoding" }` after the first Check, then write `mental_models_taught.spec-is-contract = { "at": "<ISO8601>", "by_skill": "pos-basic-vibecoding" }` after the second Check and `mental_models_taught.spec-drift = { "at": "<ISO8601>", "by_skill": "pos-basic-vibecoding" }` after the third Check before moving on. On the reminder branch, write no state.

### Phase 3 — MM2 contract and MM3 drift

**Frame coverage:** **MM2**, **MM3**.

If `experience_path == "experienced"` and `delivery_mode == "compressed"`:

- Jump to Phase 7 without re-teaching.

#### Step 3.1 — MM2 spec is the contract

Say: `«Логика простая: пока верх и низ не записаны, кода быть не должно. Спека — это контракт.»`

Check: `«Норм формула: сначала спека, потом код?»`

#### Step 3.2 — MM3 spec drift

Say: `«И ещё одно: код почти всегда уезжает, это нормально. Ненормально оставлять спеку старой. Если поведение изменилось, сначала обновляем спеку.»`

Check: `«Если код ушёл от спеки, что правишь первым?»`

**State written:** `mental_models_taught.spec-is-contract = { "at": "<ISO8601>", "by_skill": "pos-basic-vibecoding" }` and `mental_models_taught.spec-drift = { "at": "<ISO8601>", "by_skill": "pos-basic-vibecoding" }`.

### Phase 7 — MM7 skill packs, G6 Obra, MM6 skills, G7 skill-creator

**Frame coverage:** **MM7**, **MM6**, **G6**, **G7**, **F4**, **F5**.

#### Step 7.1 — MM7 skill packs

If `experience_path == "experienced"` and `delivery_mode == "compressed"`:

- Jump to Step 7.2 without re-teaching MM7.

Else:

- Say: `«Часть повторяющихся процедур удобнее не объяснять заново. Их складывают в skill pack: маленькие готовые куски поведения.»`
- Check: `«Логика понятна: повторяется — упаковываем?»`

#### Step 7.2 — G6 Obra Superpowers

If `experience_path == "experienced"` and `delivery_mode == "compressed"`:

- Say: `«Коротко: можем сразу поставить Obra Superpowers для вайб-кодинга.»`
- Check: `«Ставим Obra Superpowers (skill pack с flow для вайб-кодинга)?»`
- Build:
  - If yes, research the current install path at runtime, install it, verify the resulting state by listing the active agent registry, and write `superpowers_pack_status: "installed"` once.
  - If no, verify the current state by listing the active agent registry, record the explicit opt-out, and write `superpowers_pack_status: "opted_out"` once.
- After install, say: `«Obra Superpowers на месте.»`
- After explicit opt-out, say: `«Окей, Obra пропускаем. Дальше идём без него.»`

Else:

- Say: `«Сейчас покажу, что внутри Obra Superpowers, чтобы ты видел, что именно ставим.»`
- Build:
  - Show a short evidence-first summary of the pack contents: 4-6 most relevant skills for this course, not the whole catalog.
  - Keep this step read-only; do not install yet.
- Check: `«Ставим Obra Superpowers?»`
- Build:
  - If yes, research the current install path at runtime, install it, verify the resulting state by listing the active agent registry, and write `superpowers_pack_status: "installed"` once.
  - If no, verify the current state by listing the active agent registry, record the explicit opt-out, and write `superpowers_pack_status: "opted_out"` once.
- After install, say: `«Obra Superpowers на месте.»`
- After explicit opt-out, say: `«Окей, Obra пропускаем. Дальше идём без него.»`

#### Step 7.3 — MM6 skills are natural-language programs

If `experience_path == "experienced"` and `delivery_mode == "compressed"`:

- Jump to Step 7.4 without re-teaching MM6.

Else:

- Say: `«Теперь skill. Это программа на обычном языке. Не одноразовый промпт, а повторяемая инструкция, которую агент включает по сигналу.»`
- Check: `«Разницу слышишь: промпт на один раз, skill на много раз?»`

#### Step 7.4 — G7 skill-creator

If `experience_path == "experienced"` and `delivery_mode == "compressed"`:

- Say: `«Теперь так же коротко решим про `skill-creator`: без него живой rep дальше не собрать.»`
- Check: `«Ставим `skill-creator`, чтобы собрать твой первый skill вместе?»`

If the learner says yes:

- Build:

  - Research the current official install path for `skill-creator` at runtime for the active agent.
  - Install it.
  - Verify success in the active agent registry.

- Say: `«Навык `skill-creator` уже на месте.»`

If the learner says no:

- Say: `«Окей, `skill-creator` пропускаем. Без него живой rep здесь не собрать, так что на этом шаге остановимся.»`
- Action (silent, no learner output): hold `compressed_skill_creator_opt_out = true` in memory only and jump straight to Phase 11 Step 11.3. This uses the same skip exit as Step 7.5, but writes `skipped_reason: "opted out of skill-creator on compressed path"`.

# Outer else — full path (non-experienced or delivery_mode == "full")
Else:

- Say: `«Чтобы собрать свой skill, нужен `skill-creator`. Он делает каркас, а мы наполним его содержанием.»`
- Check: `«Ставим `skill-creator`, чтобы собрать твой первый skill вместе?»`

Build:

- Research the current official install path for `skill-creator` at runtime for the active agent.
- Install it only after the learner confirmed.
- Verify success in the active agent registry.

**State written:** `superpowers_pack_status: "installed" | "opted_out"` is written once in Step 7.2 according to the taken branch. `skill_creator_installed: true | false`. In the full branch, write `mental_models_taught.skill-packs = { "at": "<ISO8601>", "by_skill": "pos-basic-vibecoding" }` after Step 7.1 and `mental_models_taught.skills-as-programs = { "at": "<ISO8601>", "by_skill": "pos-basic-vibecoding" }` after Step 7.3. Do not synthesize those MM entries on the compressed branch, because those explanations were intentionally skipped.

#### Step 7.5 — Experienced stop or continue after prerequisites

If `experience_path == "experienced"` and `delivery_mode == "compressed"`:

- Check: `«Остановиться здесь (пререквизиты проверены, вернёмся позже) или идём в живой rep сейчас?»`

Action (silent, no learner output):

- If the learner chooses `stop`, hold `experienced_stop_after_prereqs = true` in memory only and jump to Phase 11 Step 11.3.
- If the learner chooses `continue`, hold `experienced_stop_after_prereqs = false` in memory only and continue to Phase 8.

Else:

- Jump to Phase 8.

### Phase 8 — G8 rep pitch and G9 skill naming

**Frame coverage:** **G8**, **G9**, **F9**.

#### Step 8.1 — G8 pitch the live rep

Say: `«Сейчас будет сам rep: мы вайб-кодим skill для работы с GitHub issues. Это примерно 15 минут, если без длинных остановок; ты держишь зачем, что и для кого, а я веду как.»`

Check: `«Идём?»`

#### Step 8.2 — G9 learner names the skill

Say: `«Сначала имя. Дам 3 варианта и один слот под свой.»`

Build:

- Generate 3 slug suggestions grounded in the learner's repo and the block goal.
- Keep them lowercase, short, and obviously about GitHub issues.

Say: `«1. github-issues
2. issue-memory
3. issue-workflow
4. свой вариант»`

Check: `«Напиши номер.»`

If the learner chooses `4`:

- Say: `«Окей. Напиши свой slug: только строчные буквы, цифры и дефисы.»`
- Check: wait for the slug.

Build:

- Validate the chosen slug format.
- Check for collisions in the active agent registry: `~/.claude/skills/<slug>/` for Claude Code or `~/.agents/skills/<slug>/` for Codex.
- If occupied or invalid, explain why in one plain Russian sentence and return to the naming menu.

**State written:** `learner_skill_name`.

### Phase 9 — Skill umbrella + build issue with the spec body

**Frame coverage:** reinforces **MM2** and **MM3**, lands **F7**, keeps conventions inside the skill per **F8**.

#### Step 9.1 — Preview the two-issue batch

Say: `«Сейчас по очереди откроем 2 issue. Это и будет живая память для этого rep.»`

Check: `«Ок?»`

Build:

- Reconstruct `LEARNER_REPO_URL` from `arch_blocks.github_setup.repo_url`.
- Derive `LEARNER_REPO="${LEARNER_REPO_URL#https://github.com/}"`.
- Reconstruct `COURSE_UMBRELLA_URL` from `arch_blocks.github_setup.course_umbrella_issue_url`.
- Use the label set already present in that repo. Do not create or edit labels in this phase.

#### Step 9.2 — Issue #1: skill umbrella

Say: `«Открываем issue 1 из 2. Это зонтик твоего skill для работы с GitHub issues. Он останется открытым, пока навык живёт.»`

Build:

- Create the long-lived skill umbrella body.
- If `COURSE_UMBRELLA_URL` is present, link back to it in the body.
- If it is missing or null, keep a plain-text fallback pointing at `LEARNER_REPO` and note that the course umbrella already lives there.
- Use a real `gh` invocation:

```bash
ISSUE1_BODY="$(mktemp)"
if [ -n "${COURSE_UMBRELLA_URL:-}" ] && [ "${COURSE_UMBRELLA_URL}" != "null" ]; then
cat > "$ISSUE1_BODY" <<EOF
Долгоживущий зонтик для навыка \`$LEARNER_SKILL_NAME\`.

Relates to $COURSE_UMBRELLA_URL
EOF
else
cat > "$ISSUE1_BODY" <<EOF
Долгоживущий зонтик для навыка \`$LEARNER_SKILL_NAME\`.

Relates to repo $LEARNER_REPO
Курс-зонтик уже живёт там; ссылка на него не сохранилась в state, поэтому дубликат не создаём.
EOF
fi
ISSUE1_URL="$(gh issue create --repo "$LEARNER_REPO" --title "Навык $LEARNER_SKILL_NAME — зонтик" --label "epic" --body-file "$ISSUE1_BODY")"
ISSUE1_NUMBER="${ISSUE1_URL##*/}"
```

#### Step 9.3 — MM2 reinforce before the build issue

Say: `«Теперь нужна спека. Не в голове, а в issue. Иначе нечему будет держать границы.»`

Check: `«Готов положить контракт в build issue?»`

#### Step 9.4 — Issue #2: build issue with the spec body

Say: `«Открываем issue 2 из 2. Это первая сборка навыка `<chosen-skill-name>`. В текст issue кладём спеку и сразу привязываем его к зонтику навыка. Если код уедет, сначала правим этот issue.»`

Build:

- Write the skill spec into a temporary markdown file. The spec body must include:
  - why / what / for-whom of the skill
  - the existing minimal label set with domain mapping
  - umbrella convention
  - child-linking via `Relates to #<umbrella>`
  - explicit rule that these conventions live in the learner-authored skill, not in the agent-config file
  - explicit drift rule: update the issue body first when behavior changes
- Use a real `gh` invocation:

```bash
ISSUE2_BODY="$(mktemp)"
cat > "$ISSUE2_BODY" <<EOF
# Зачем
Помогать агенту работать с GitHub issues в репозитории $LEARNER_REPO без хаоса.

# Для кого
Для владельца репозитория и его будущих сессий с агентом.

# Что должно уметь
- Читать и создавать issue через \`gh\`
- Использовать уже существующий label set репозитория: \`epic\`, \`task\`, \`p0\`, \`p1\`, \`p2\`
- Держать domain mapping:
  - \`domain:health\` = HEALTH
  - \`domain:self\` = SELF
  - \`domain:relationships\` = RELATIONSHIPS
  - \`domain:career\` = CAREER
  - \`domain:financial\` = FINANCIAL
  - \`domain:creativity\` = CREATIVITY
  - \`domain:contribution\` = CONTRIBUTION
  - \`domain:fun-recreation\` = FUN/RECREATION
- Держать один открытый umbrella issue для долгоживущей темы
- Делать дочерние issue с \`Relates to #<umbrella>\`

# Ограничения
- Не переустанавливать labels: они уже лежат в репозитории после \`pos-github-setup\`
- Конвенции живут в самом skill, не в agent-config file
- В agent-config file попадает только правило про spec drift
- Ничего не называть молча: skill, umbrella, build issue
- Если код skill-а уехал от этой спеки, сначала обновить этот issue, потом править skill

Relates to #$ISSUE1_NUMBER
EOF
ISSUE2_URL="$(gh issue create --repo "$LEARNER_REPO" --title "Навык $LEARNER_SKILL_NAME — первая сборка" --label "task" --label "p2" --body-file "$ISSUE2_BODY")"
ISSUE2_NUMBER="${ISSUE2_URL##*/}"
```

#### Step 9.5 — MM3 reinforce on the real artifact

Say: `«Вот теперь это не теория. Если правило изменится, сначала меняем текст build issue. Потом меняем сам skill.»`

Check: `«Если правило уехало, что правим первым?»`

**State written:** `practice_issue_url: "$ISSUE2_URL"`. Treat this build issue as the first real learner-authored issue of the rep and the durable contract handle for it.

### Phase 9b — Dedicated artifact repo bootstrap

**Goal:** satisfy the dedicated-repo prerequisite before the first learner-owned skill file is written.

**Frame coverage:** **G12**, **F10**.

#### Step 9b.1 — Choose the dedicated repo name

Say: `«По умолчанию назову репозиторий так же, как skill: `$LEARNER_SKILL_NAME`. Если хочешь другое имя — напиши.»`

Check: `«Имя репозитория?»`

Action (silent, no learner output):

- If the learner's reply is empty, `$LEARNER_SKILL_NAME`, «по умолчанию», or equivalent — keep repo name `$LEARNER_SKILL_NAME`.
- Otherwise validate the learner's name (non-empty, no spaces, repo-safe chars).
- If the learner's reply passes validation, keep it.
- If the learner's reply fails validation, say: `«В имени нельзя пробелы и лишние символы — давай короткое имя на латинице и дефисах.»` Then re-present Check: `«Имя репозитория?»`. Do not silently accept.
- Derive the default local path as `<parent-of-current-POS-repo>/<repo-name>`. If that path already exists and is not a fresh dedicated repo for this skill, ask for a new repo name or a new path before building.

#### Step 9b.2 — Bootstrap fresh repo + push scaffold commit

Say: `«Сейчас соберу пустой каркас: отдельную локальную папку, git-историю и зеркало в GitHub. Сам код skill-а пойдёт следующим шагом.»`

Build:

- Work only inside the chosen artifact repo path.
- Ensure this is a fresh dedicated repo for the learner-authored skill. Never reuse the existing POS repo referenced by `arch_blocks.github_setup.repo_url` or a mixed-purpose directory.
- Use current `gh` help or the current official docs at runtime to form the exact repo-create invocation for this environment. Do not hardcode a stale `gh repo create` form.
- Initialize local git.
- Create the matching private GitHub repo on the learner's own account.
- Set the remote.
- Create a minimal scaffold commit with only `README.md` and `.gitignore`. Do not create runtime skill files yet.
- Push that scaffold commit to GitHub.
- Completion gate:
  - a fresh local git repo exists
  - the remote GitHub repo exists
  - the scaffold commit is pushed successfully
- If any step fails, stop cleanly with a plain-Russian failure path:
  - one sentence on what failed
  - one next action: retry / pick a different repo name or path / stop here
  - one evidence pointer: command output, log path, or the failing command to rerun
- On success, write:
  - `artifact_repo.name`
  - `artifact_repo.local_path`
  - `artifact_repo.github_url`
  - `artifact_repo.initialized_at`

**State written:** `artifact_repo` object fully populated.

### Phase 10 — Build the learner-authored skill from the build issue

**Frame coverage:** reinforces **MM6** and **MM3**, keeps conventions under **F8**, relies on prior **G7** and **G12**.

#### Step 10.1 — Frame the rep in plain language

Say: `«Теперь из build issue сделаем сам skill. Вот это и есть вайб-кодинг: у тебя есть контракт, у меня — сборка того, как это сделать.»`

Check: `«Собираем skill?»`

#### Step 10.2 — Use `skill-creator` on the build-issue contract

Build:

- Reconstruct `ARTIFACT_REPO_LOCAL_PATH` from `artifact_repo` and use it as the working directory for this build. The active agent registry entry for `$LEARNER_SKILL_NAME` must resolve inside that artifact repo working tree: either `ARTIFACT_REPO_LOCAL_PATH` directly contains the skill dir (for example, as the parent or the skill dir itself), or a symlink is set up so the skill dir physically lives inside the artifact repo. In either case, `skill-creator` output and all subsequent edits must land inside the artifact repo's working tree.
- Reconstruct `LEARNER_REPO_URL` from `arch_blocks.github_setup.repo_url`.
- Derive `LEARNER_REPO="${LEARNER_REPO_URL#https://github.com/}"`.
- Reconstruct `ISSUE2_URL` from `practice_issue_url`.
- Reconstruct `ISSUE2_NUMBER="${ISSUE2_URL##*/}"` before any drift check or re-read.
- Read the build issue back in full before writing anything.
- Use the installed `skill-creator` via its current official invocation for the active agent to scaffold the learner skill in the active registry.
- Treat the build issue as the contract. Conventions must live inside that generated skill:
  - existing label set
  - domain mapping
  - umbrella convention
  - parent-linking rule
- Labels already exist in the learner POS repo from `pos-github-setup`; document and use them inside the skill, do not re-apply them.
- Do **not** write any of those conventions into the agent-config file.
- Review the generated `SKILL.md` immediately. If behavior or wording drifted from the build issue, update the build issue first via `gh issue edit "$ISSUE2_NUMBER" --repo "$LEARNER_REPO" --body-file "$UPDATED_ISSUE2_BODY"`, then patch the skill.
- Verify the result exists on disk and is discoverable in the active agent registry.

#### Step 10.3 — Verify the live rep artifact

Build:

- Verify the generated skill path exists.
- Verify the generated skill mentions the label set and umbrella/linking conventions, and treats the label set as existing repo context rather than something it creates.
- Verify the build issue still matches the shipped behavior after any edits.
- After those checks pass, commit the generated skill in the artifact repo and push it.

**State written:** write `learner_skill_path` only after this verification passes, then refresh `learner_skill_name` and `learner_skill_path` from the verified on-disk result if the actual path changed during install.

### Phase 11 — G10 primary agent-config file, final verification, track, and handoff

**Frame coverage:** **G10**, **F5**, **F6**, **F7**, **F8**.

If `experienced_stop_after_prereqs == true` or `compressed_skill_creator_opt_out == true`:

- Jump straight to Step 11.3.
- Do not run Steps 11.1-11.2.

#### Step 11.1 — G10 primary agent-config rule

Say: `«До закрытия блока остаётся одно правило в основном файле твоего агента. Только одно: если код ушёл от спеки, сначала правим спеку.»`

Check: `«Добавляем это в основной файл агента?»`

Build:

- Resolve the primary target file from `learner_profile.primary_agent`: `CLAUDE.md` for Claude Code, `AGENTS.md` for Codex. If the field is missing, infer the current runtime agent and confirm once before writing.
- Read the primary target first. If it does not exist, create it.
- If `## Vibe-coding` already exists, show a diff and ask before merging new text.
- If `learner_profile.keep_agent_configs_in_sync == true`, mirror the same accepted minimal section to the sibling file after the primary append passes.
- Append only this minimal section:

```markdown
## Vibe-coding
- Spec drift: если код ушёл от спеки, сначала обнови spec, потом продолжай.
```

- Verify the file content after writing.

#### Step 11.2 — Fresh verification before completion

Build:

- Reconstruct `LEARNER_REPO_URL` from `arch_blocks.github_setup.repo_url`.
- Derive `LEARNER_REPO="${LEARNER_REPO_URL#https://github.com/}"`.
- Reconstruct `ISSUE2_URL` from `practice_issue_url`.
- Reconstruct `ISSUE2_NUMBER="${ISSUE2_URL##*/}"` before running the checks.
- Reconstruct `ARTIFACT_REPO_NAME` and `ARTIFACT_REPO_LOCAL_PATH` from `artifact_repo`.
- Reconstruct `LEARNER_SKILL_PATH` from `learner_skill_path`.
- Run fresh verification commands:
  - `gh auth status`
  - `gh repo view "$LEARNER_REPO"`
  - `test -d "$LEARNER_SKILL_PATH"`
  - `gh issue view "$ISSUE2_NUMBER" --repo "$LEARNER_REPO"`
  - list the active agent registry
  - `gh repo view "$ARTIFACT_REPO_NAME"` (or equivalent)
  - `test -d "$ARTIFACT_REPO_LOCAL_PATH/.git"`
  - confirm the scaffold commit and the learner-skill commit both exist in the artifact repo's pushed history
- Read the outputs. Do not claim success until the checks pass.

#### Step 11.3 — Final tracking write

Build:

- If `compressed_skill_creator_opt_out == true`, write:
  - `status: "skipped"`
  - `completed_at: null`
  - `skipped_at`
  - `skipped_reason: "opted out of skill-creator on compressed path"`
  - preserve and refresh only the verified local prerequisites:
    - `superpowers_pack_status`
    - `skill_creator_installed: false`
  - keep `practice_issue_url: null`
  - keep `artifact_repo = { name: null, local_path: null, github_url: null, initialized_at: null }`
  - keep `learner_skill_name: null`
  - keep `learner_skill_path: null`
  - keep `agent_config_rules_appended: false`
- Else if `experienced_stop_after_prereqs == true`, write:
  - `status: "skipped"`
  - `completed_at: null`
  - `skipped_at`
  - `skipped_reason: "experienced — prereqs done, rep deferred"`
  - preserve and refresh only the verified local prerequisites:
    - `superpowers_pack_status`
    - `skill_creator_installed`
  - keep `practice_issue_url: null`
  - keep `artifact_repo = { name: null, local_path: null, github_url: null, initialized_at: null }`
  - keep `learner_skill_name: null`
  - keep `learner_skill_path: null`
  - keep `agent_config_rules_appended: false`
- Otherwise write:
  - `status: "done"`
  - `completed_at`
  - `skipped_at: null`
  - `skipped_reason: ""`
- In the done branch, ensure the contract fields are fully populated from verified artifacts:
  - `practice_issue_url`
  - `artifact_repo` with non-null `name`, `local_path`, `github_url`, `initialized_at`
  - `superpowers_pack_status`
  - `skill_creator_installed`
  - `learner_skill_name`
  - `learner_skill_path`
  - `agent_config_rules_appended: true`
- Do not copy `repo_url` or `course_umbrella_issue_url` out of `arch_blocks.github_setup`; they remain owned there.

#### Step 11.4 — Final handoff

**State written:** final `arch_blocks.basic_vibecoding` object per the state contract. On the skip branch it contains only verified local prerequisites; on the done branch it also carries the build issue, the dedicated artifact repo, the learner skill, and the primary agent-config rule. MM1-MM3 and MM6-MM7 are written at their delivery phases, not here.

If `compressed_skill_creator_opt_out == true`:

- Say: `«Остановились на `skill-creator`. Без него этот живой rep не стартует. Когда захочешь продолжить, запусти `/pos-basic-vibecoding`.»`

- Hard-stop. Do not continue to the done branch.

Else if `experienced_stop_after_prereqs == true`:

- Say: `«Базовые пререквизиты уже проверены. На живой rep вернёмся позже. Когда захочешь продолжить, запусти `/pos-basic-vibecoding`.»`

- Hard-stop. Do not continue to the done branch.

Else:

- Action (silent, no learner output): Recommend the next block by, in order:
  1. `learner-state.json` diagnostic-route / recommendations field if set by `pos-diagnostic`.
  2. `my-architecture.md` — parse the next unfinished block from the learner's plan.
  3. Fallback: bundled `skill-catalog.json` entries that are `shipped`, `menu_visible`, not yet `done`, and not this skill; prefer entries whose prerequisites already look satisfied in learner state.
- Name one specific next block by slash command in the Say below (via `</pos-next>`); do not hedge. If nothing unfinished remains, point to `/pos-diagnostic` in the same slot.
- Do NOT recommend `/pos-basic-vibecoding` itself — the farewell already names it as the re-entry cue.
- Say (одной репликой, без паузы на ответ ученика): `«База готова: у тебя есть контракт в issue, отдельный repo для skill-а и первый свой skill. Это не wow-момент, а опора. Дальше логично идти к </pos-next>. Если захочешь вернуться сюда позже, запускай `/pos-basic-vibecoding`.»`

## Learner feedback close reminder

Перед финальным Check или прощанием этого блока добавь расширенное напоминание:
`«Если здесь что-то было непонятно, сломано, особенно полезно или если хочется чего-то больше или по-другому, скажи — я помогу оформить фидбек для создателей.»`
Если ученик откликается, предложи свободно описать ситуацию и дальше переведи его в `/pos-feedback`-flow.

## References

- Normative authoring contract — `../../docs/skill-contract.md`
- Runtime pattern (soft direction) — `../../docs/block-runtime-pattern.md`
