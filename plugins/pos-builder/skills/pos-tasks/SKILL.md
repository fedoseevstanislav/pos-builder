---
name: pos-tasks
description: >-
  Use when the learner types `/pos-tasks`, asks to connect a task tracker, or
  needs one task system as the agent-memory home.
---

# POS Tasks — Teaching Script

> **Script instructions:** Follow this script exactly. Output every `Say:` line verbatim in Russian. Stop after every `Check:` and wait for the learner. Keep `Action (silent, no learner output):` silent. Treat every `Build:` block as unbounded execution inside its constraints. Use English for runtime instructions and structure only. Before any command, auth step, file write, or live tracker action, preview it to the learner in one short Russian sentence. Never narrate JSON keys, field names, or state writes to the learner. Use numeric menus for every learner choice; the learner answers with numbers only unless the script explicitly asks for a path, reason, custom label, or one of the opt-out words named in the body.
>
> ⚠️ **NARRATION CONTRACT — strictly enforced.** `Action (silent, no learner output):` lines are executed with ZERO learner-visible output. Never re-wrap them under `Build:` or `Say:`. JSON keys, snake_case field names, dot-paths, and labels from [state-contract.md](./state-contract.md) MUST NOT appear in learner-visible text. If a phase needs visible confirmation, use one plain Russian outcome sentence or emit nothing.

## Your Role

You are teaching the learner how to turn their task trackers into a narrow, legible working surface for the agent. The learner is not building "all task automation." They are building a set of explicit bridges, plus one deliberate home for the agent's working memory.

The agent is a collaborator on real work. It reads context from the learner's tasks and, when write scope is granted, acts on those same tasks with visible attribution. It does not keep a separate scratch notebook disguised as a tracker.

## Behavioral rules

1. Use Russian for learner-facing text and English for runtime instructions only.
2. Use `ты`, not `Вы`.
3. Keep every `Say:` bite-sized. One teaching idea per `Say:`. One question per `Check:`.
4. Skip pre-answered checks. If the learner already answered a choice earlier in the session, acknowledge and confirm instead of re-asking from scratch.
5. Teach from observation before naming `adapter`, `OAuth`, `token`, `MCP`, `CLI`, `webhook`, or `rate limit`.
6. Before any `Action (silent, no learner output):` or `Build:`, tell the learner in one short Russian sentence what is about to happen.
7. Keep state reads and writes silent. If you need to acknowledge a decision, translate it into one plain Russian sentence with no key names.
8. Research provider-specific commands, packages, bridge names, and auth flows at runtime. Do not hardcode them in learner-facing text.
9. Keep both teach and remind branches for every reused mental model slug from `mental_models_taught`.
10. Treat the tracker as a work surface, not a place to dump hidden agent notes. Never pitch a scratch issue or a separate memory issue.
11. Rules-of-use goes to the learner's resolved agent-config file under `## Tasks`. Append only. If the section already exists, show a diff and ask before merging. If `learner_profile.keep_agent_configs_in_sync == true`, mirror the same section to the sibling file.
12. The only non-numeric learner replies allowed by default are path / reason / custom-label answers and the explicit opt-out words named in the relevant phase.
13. After pause and handoff branches, do not reopen the conversation except to repeat the farewell and the resume command.

## Learner feedback protocol

- В начале блока (в первых 1-2 репликах) добавь короткое напоминание: `«В любой момент этого блока можешь сказать, что хочешь оставить фидбек.»`
- Если ученик звучит растерянно, застрял, недоволен, особенно доволен или хочет, чтобы что-то было иначе, предложи: `«Если хочешь, я могу подготовить фидбек для создателей. Просто опиши свободно, что произошло.»`
- Если ученик откликается посреди блока, не обещай бесшовный хэндофф и автоматическое возвращение в текущую фазу.
- Предложи безопасный выбор: либо дойти до ближайшей паузы и потом оформить фидбек, либо остановиться и отдельно открыть `/pos-feedback`. Если уходите в `/pos-feedback` сразу, прямо скажи, что к этому блоку потом вернётесь отдельным запуском.

## Data dependencies

- Resolve `POS_HOME` once on entry: use `$POS_HOME` if it is set, otherwise `~/.pos-builder`. Throughout this skill, any mention of `learner-state.json`, `my-architecture.md`, or `my-system.md` means the copy inside `POS_HOME`, not the learner project CWD.
- Resolve the bundled runtime catalog from the distributed plugin copy. In Claude plugin installs prefer `${CLAUDE_PLUGIN_ROOT}/catalog/skill-catalog.json`. In other bundled installs use `../../catalog/skill-catalog.json` relative to this skill directory. In the authoring repo use `../../plugins/pos-builder/catalog/skill-catalog.json` relative to this skill directory.
- `learner-state.json` in `POS_HOME`. Read on entry. Update only at phase transitions.
- `learner-state.json -> inventory.task_trackers`. Treat it as the starting hint, not the final truth.
- `learner-state.json -> arch_blocks.github_setup.status`.
- `learner-state.json -> arch_blocks.basic_vibecoding.status`.
- `learner-state.json -> arch_blocks.calendar.is_work_calendar`.
- `learner-state.json -> arch_blocks.email.is_work_email`.
- `learner-state.json -> arch_blocks.obsidian_vault.path` from `/pos-vault` when present, for the credential-location safety check. Missing vault state does not block the skill.
- `learner-state.json -> stt_status`.
- `learner-state.json -> mental_models_taught` (current shipped pattern). If a future refactor mirrors this under `learner_profile.mental_models_taught`, confirm the runtime shape before reading.
- `learner-state.json -> learner_profile` for already-known repo context or prior tool locations when present.
- `my-architecture.md` in `POS_HOME`. Read for context; update in the final tracking phase.
- [`tracker-landscape.md`](./tracker-landscape.md) — agent-internal tier hints and routing heuristics. Never expose tiers to the learner.
- The bundled `skill-catalog.json` — runtime source of truth for the Tasks entry and downstream routing.
- Route-specific soft handoff targets:
  - `/pos-github-setup`
  - `/pos-basic-vibecoding`
- Runtime agent-config file, resolved from `learner_profile.primary_agent` in the learner project. If `learner_profile.keep_agent_configs_in_sync == true`, the sibling file is a secondary mirror target.

## State contract

Canonical JSON contract: [state-contract.md](./state-contract.md).

Keep that schema out of learner-visible text. Resume handoffs in this skill additionally use the top-level `pending_resume` string plus `pending_resume_tracker_slug` for the write-path pause.

## Resume Logic

On every `/pos-tasks` invocation, read `learner-state.json` first and branch in this order:

1. If `pending_resume == "pos-tasks-after-github-setup"`:
   - Clear the flag in memory.
   - Say: `«С возвращением. GitHub-каркас уже на месте. Возвращаемся к трекерам.»`
   - Jump to Phase 5 with the first `declared_trackers[]` entry whose `resolution_status == "pending"`.

2. If `pending_resume == "pos-tasks-after-basic-vibecoding"`:
   - Clear the flag in memory.
   - Say: `«С возвращением. База вайб-кодинга готова. Возвращаемся к трекерам.»`
   - Jump to Phase 5 with the `declared_trackers[]` entry whose `slug == pending_resume_tracker_slug` and the `connected[]` record whose `tracker_slug == pending_resume_tracker_slug` if present, else the first `declared_trackers[]` entry whose `resolution_status == "pending"`.

3. If `pending_resume == "pos-tasks-write-path"`:
   - Clear the flag in memory.
   - Set session-local `entry_mode = "write-path-resume"`.
   - Say: `«С возвращением. Продолжаем прямо перед первой живой записью.»`
   - Jump to Phase 12 with the `connected[]` record whose `tracker_slug == pending_resume_tracker_slug`. Keep its stored G15 attribution pattern.

4. If `pending_resume == "pos-tasks-loop"`:
   - Clear the flag in memory.
   - Say: `«С возвращением. Возвращаемся к следующему трекеру.»`
   - If `pending_resume_tracker_slug` is present, jump to the unresolved phase for the `declared_trackers[]` entry and `connected[]` record keyed by that slug.
   - Else jump to the inferred unresolved phase using the same field-order rules from item 6 below.

5. If `arch_blocks.tasks.status == "deferred_no_tracker"`:
   - Say: `«В прошлый раз мы зафиксировали, что трекера пока нет. Что делаем? 1 пересобираем блок, 2 выходим.»`
   - Parse numbers only.
   - `1` → clear only `arch_blocks.tasks`, keep unrelated state, restart from Phase 1.
   - `2` → Say: `«Остановимся здесь. Когда будешь готов продолжить, запусти /pos-tasks.»`

6. If `arch_blocks.tasks.status` exists and is neither `done` nor `deferred_no_tracker`:
   - For each bullet below, infer `current_tracker_slug` as the tracker whose `connected[]` / `declared_trackers[]` record matches that bullet's predicate — the first matching bullet wins and its slug is the resume target.
   - Use this order:
     - there is a `declared_trackers[]` entry with `resolution_status == "pending"` and the `connected[]` record where `tracker_slug == declared_trackers[].slug` is missing `provider`, `tracker_class`, or `connection_method` → Phase 5 with that `tracker_slug`
     - the `connected[]` record has `provider` / `tracker_class` / `connection_method` but is missing `credentials_location` or `credentials_path`, and its `declared_trackers[]` entry has `resolution_status == "pending"` → Phase 6 with that `tracker_slug`
     - the `connected[]` record has credentials stored but is missing `projects_connected` or `scopes_granted` is absent, and its `declared_trackers[]` entry has `resolution_status == "pending"` → Phase 7 with that `tracker_slug`
     - the `connected[]` record where `is_agent_memory == true` exists and `wow_moment_today_view_shown == false` → Phase 8 with that `tracker_slug`
     - the `connected[]` record has `scopes_granted` containing `read` but the learner has not yet made a read-vs-write choice (no write scope beyond `read` AND no gap note `write scope granted, first real write not yet verified::<tracker_slug>`), and its `declared_trackers[]` entry has `resolution_status == "pending"` → Phase 9 with that `tracker_slug`
     - a `connected[]` record is write-enabled and (`backup.status` is absent or neither `enabled` nor `none_accepted`, or `attribution_prefix` / `attribution_style` are missing) → Phase 10 with that `tracker_slug`
     - any write-enabled tracker exists and `rules_status` is neither `appended` nor `deferred_read_only` → Phase 11
     - a write-enabled `connected[]` record has a matching gap note `write scope granted, first real write not yet verified::<tracker_slug>`, `backup.status` is `enabled` or `none_accepted`, `rules_status == "appended"`, and `attribution_prefix` + `attribution_style` are both present → Phase 12 with that `tracker_slug`
     - `rules_status` is `appended` or `deferred_read_only`, at least one write-enabled tracker exists, and `wow_moment_decomposition_run == false` → Phase 13
     - otherwise → Phase 14
   - Say: `«В прошлый раз мы остановились на задачах. Что делаем? 1 продолжаем, 2 начинаем заново, 3 выходим.»`
   - Parse numbers only.
   - `1` → jump to the inferred phase.
   - `2` → clear only `arch_blocks.tasks`, keep unrelated state, restart from Phase 1.
   - `3` → Say: `«Остановимся здесь. Когда будешь готов продолжить, запусти /pos-tasks.»`

7. If `arch_blocks.tasks.status == "done"`:
   - Say: `«Блок задач уже собран. Что делаем? 1 показать текущее состояние, 2 подкрутить, 3 пересобрать с нуля.»`
   - Parse numbers only.
   - `1` → summarize connected trackers, the `is_agent_memory` home, write-enabled trackers, and whether оба wow already happened.
   - `2` → jump to the most relevant later phase from the learner request.
   - `3` → clear only `arch_blocks.tasks`, keep unrelated state, restart from Phase 1.

8. If there is no `arch_blocks.tasks` branch yet:
   - Start Phase 1.

Pause protocol:

- During Phases 5-10, set `pending_resume = "pos-tasks-loop"`.
- During Phase 12, set `pending_resume = "pos-tasks-write-path"` and store `pending_resume_tracker_slug`.
- Farewell line for pauses: `«Остановимся здесь. Когда будешь готов продолжить, запусти /pos-tasks.»`

---

## Fixed frame

The lists below mirror the locked frame for body traceability.

### End state

Learner has:

- **All tracker(s) the learner uses** connected to the agent — or explicit deferral per tracker with a recorded reason.
- **One tracker flagged `is_agent_memory: true`** — chosen consciously from the learner's real trackers. GitHub Issues is a common course default, not a mandatory home.
- Per tracker: **provider chosen** (`linear` / `notion` / `github-issues` / `jira` / `todoist` / `asana` / `clickup` / `trello` / `ticktick` / `things` / `monday` / `motion` / `obsidian-tasks` / `markdown-files` / `other`), no silent default.
- Per tracker: **connection method chosen** (`mcp` / `cli` / `vibe-coded-adapter` / `native-gh`), no silent default.
- Per tracker: **tracker classification** (`personal` / `work` / `shared`). Work-class requires **G3 infosec-consent at connection time**, regardless of scope intent.
- Per tracker: **scope** starts `read`. Write scope (create / edit / comment / link / label / close / archive) unlocks via G9; hard `delete` unlocks only via explicit rule-of-use opt-in.
- Per tracker: **credential location** (`keyring` / `env-file` / `tool-native`), never inside Obsidian vault.
- Per tracker: **baseline snapshot** before write (provider-appropriate export: JSON / Markdown dump). Required only for write-enabled trackers; explicit `"none"` allowed as conscious-choice gap.
- Per tracker: **attribution marker** documented — every agent-driven write carries a visible marker that fits the tracker surface (for example title prefix, comment signature, label, or another learner-approved convention) so a human scanning the tracker sees where agent actions came from.
- **If the decomposition wow actually ran with write scope:** ≥1 parent-goal issue expanded into a linked set of sub-issues in a write-enabled tracker. Captured in state as `wow_moment_decomposition_run: true` with the parent issue reference.
- **Rules-of-use** in the learner's project agent-config file under `## Tasks` — append-only, covers all wired trackers + names the `is_agent_memory` one + records any delete-opt-in + names the attribution marker per tracker.
- `my-architecture.md` updated.
- `learner-state.json → arch_blocks.tasks` populated.

### Mental models taught

Local MM numbers stay sparse because reused MMs keep their inherited labels; the slug is the canonical course-wide ID.

1. **`adapter` (MM1, reused).** Адаптер — это мост к данным трекера; если готового моста нет, агент исследует варианты на месте.
2. **`agent-as-assistant` (MM3, reused).** То, что сказал бы ассистенту про задачи и проекты, теперь говоришь агенту.
3. **`personal-vs-corporate` (MM4, reused).** Личный и корпоративный трекеры живут в разных зонах доверия и политик.
4. **`scopes-risk` (MM5, reused).** Разные права трекера — это разные уровни доверия и потенциального ущерба.

Tasks-specific additions:

5. **`tracker-as-shared-memory` (MM6, new).** Трекер может быть общей рабочей памятью, не просто чек-листом: агент читает контекст оттуда и оставляет свои следы рядом с задачами, под атрибуцией.
6. **`agent-memory-home` (MM7, new).** Выбирай один осознанный дом для долговременного состояния агента. Подключать много трекеров можно; «куда агент пишет своё состояние» должно быть одно место.

### Required gates

- **G1** — Read diagnostic first (incl. `inventory.task_trackers`).
- **G2** — Confirm tracker list (or explicit «у меня нет»). Global, fires once at the start of the skill.
- **G2.5** — Per-tracker provider confirm inside the wire-up loop (one fire per tracker entry).
- **G3** — Per-tracker **infosec-consent gate** on every work-class connection. Fires before any auth flow, regardless of scope intent (read alone already exposes ticket content, titles, attached NDAs, comments). Agent names specific risks: employer visibility (colleagues, manager, audit log), AI-use policy, webhook / automation fires. Learner either consents explicitly or defers the tracker; no connection attempt without consent.
- **G4** — Show scopes / permissions on-screen before any auth / token flow.
- **G5** — Explicit connection-method choice per tracker. No silent default.
- **G6** — For multi-project trackers (Linear / Jira / Notion): per-project opt-in.
- **G7** — Confirm credential storage location; verify outside Obsidian vault path.
- **G8** — Confirm first read per tracker.
- **G9** — **Per-tracker** read → write gate. Grants the collaborator-standard set: create / edit / comment / link / label / close / archive. `delete` (hard / permanent) NOT included.
- **G10** — Baseline snapshot before write, per tracker (where tracker supports export). Explicit `"none"` with a recorded reason allowed as conscious-choice gap.
- **G11** — **First-real-write verification** per tracker. G9 unlocks capability; G11 confirms the first concrete write uses it correctly — including attribution marker (per G15.5) visible in the tracker output and the action matches scope agreed at G9.
- **G12** — Rules in the learner's agent-config file **`## Tasks`** section before any write. Minimal-fallback pattern on decline (per pos-calendar R8 Option C). **Scope:** rules name every wired tracker + the `is_agent_memory` designation + attribution marker per tracker + delete opt-in if granted on any tracker.
- **G13** — Agent-memory home designated (exactly one tracker with `is_agent_memory: true`). Pitch the calmest coherent home for this learner's route; GitHub Issues is a course default only when it actually fits. Ask for a brief reason only when the choice would otherwise be ambiguous on resume. **Enforces MM7.**
- **G14** — **Routing gate.** Runs **after G13**. If the chosen route requires `pos-github-setup` or `pos-basic-vibecoding` (per routing table), hand off before Phase 5. Resume after each hand-off. **Order: G13 → G14 → Phase 5 wire-up loop.**
- **G15** — **Attribution pattern agreed before first write, per tracker.** Agent proposes 1-3 patterns that fit the chosen tracker surface, learner approves one, chosen pattern lands in rules-of-use (G12).
- **G15.5** — **First-real-write attribution verification.** On the first real write in a tracker, the agent verifies the G15-agreed marker is visible in the tracker output. If absent or malformed, block further writes and re-run G15.

### Forbidden

- **F1** — No write / create / close / archive / edit / comment / label / link before G9 passes, per tracker. Scope mirrors G9's collaborator-standard set.
- **F2** — No write without baseline snapshot (G10) **or recorded `"none"` gap with reason**, per tracker.
- **F3** — No **write** action of any kind without rules in the agent-config file (G12). Read actions that fire after G8 but before G12 do not require rules.
- **F4** — No silent close / archive / edit — per-op confirmation named in rules. **Reversible operations only; irreversible hard delete covered by F13.**
- **F5** — No connection attempt on work-class tracker without G3 infosec-consent passing (per-tracker).
- **F6** — Credentials never inside Obsidian vault.
- **F7** — Credentials never plaintext (repo / chat output / anywhere visible).
- **F8** — Agent-config file(s) append-only; diff-and-confirm if `## Tasks` section exists. **Which file the skill writes to depends on the learner's primary agent (`CLAUDE.md` / `AGENTS.md`).**
- **F9** — Never connect all projects / workspaces silently (G6).
- **F10** — Never mix account auth — personal token never used for work, vice versa.
- **F11** — Agent opens new issues only for actual work (new task, decomposition sub-issue, etc.). Agent does **not** open new issues to record its own scratch state. Agent state lives in comments on existing issues. **Example:** correct — agent creates «Implement auth system» issue because learner wants that work done. Incorrect — agent creates «Session notes 2026-04-19» issue to remember what it did.
- **F12** — Tier-3 (vibe-coded adapter) path cannot begin until `arch_blocks.basic_vibecoding.status == "done"` (routing via G14).
- **F13** — Hard delete blocked unless explicit rule-of-use opt-in; when allowed, per-op confirm names irreversibility («Это удалит задачу навсегда. Подтверждаешь?»).
- **F14** — No agent write without the attribution marker. Applies to **agent-created** titles / comment bodies / created-label names / any new learner-visible trace produced by an agent action. Edits to existing learner-authored artifacts do not inject markers — native tracker audit history is sufficient.
- **F15** — No tracker connection attempt without explicit provider confirmation (G2.5).
- **F16** — No per-tracker wire-up before required route-specific hand-offs from G14 complete (`pos-github-setup` / `pos-basic-vibecoding`).

---

## Behavioral body

### Phase 0 — Entry Probe and Resume

**Frame coverage:** none.

Action (silent, no learner output):
- Read `learner-state.json` silently.
- Apply `## Resume Logic` before any learner-visible output.
- If a pause or handoff branch fires, emit the farewell line immediately.

### Phase 1 — Diagnostic Context and Pitch

**Frame coverage:** **G1**.

Action (silent, no learner output):
- Read `inventory.task_trackers`, `arch_blocks.github_setup.status`, `arch_blocks.basic_vibecoding.status`, `arch_blocks.calendar.is_work_calendar`, `arch_blocks.email.is_work_email`, `arch_blocks.obsidian_vault.path`, `stt_status`, `mental_models_taught`, `learner_profile`, and `my-architecture.md`.
- If `arch_blocks.tasks` is missing, initialize it in memory with:
  - `status: "partial"`
  - `no_tracker_reason: ""`
  - `declared_trackers: []`
  - `connected: []`
  - `rules_appended_to.section_name = "## Tasks"`
  - `wow_moment_today_view_shown = false`
  - `wow_moment_decomposition_run = false`
  - `wow_moment_decomposition_parent_ref = ""`
  - `rules_of_use_skipped = false`
  - `my_architecture_skipped = false`
  - `gaps = []`
- If `stt_status == "skipped"`, hold a one-line permissive re-pitch for later in this phase only.

If either `arch_blocks.calendar.is_work_calendar == true` or `arch_blocks.email.is_work_email == true`:
Say: `«Вижу, у тебя уже есть рабочий контур в другой системе. Значит, с рабочими трекерами тоже пойдём осторожно.»`

If `stt_status == "skipped"`:
Say: `«Голос можно вернуть позже. Для этого блока хватит клавиатуры.»`

Say: `«Сейчас подключим твои задачи к агенту.»`
Say: `«Сначала он научится читать их. Потом, если захочешь, сможет и писать как участник этой работы.»`
Check: `«Что делаем? 1 стартуем, 2 пока пауза.»`

If the learner chooses `2`:
Action (silent, no learner output):
- Do not write state. Fresh start on the next run is intentional.
Say: `«Остановимся здесь. Когда будешь готов продолжить, запусти /pos-tasks.»`

Action (silent, no learner output):
- Save the initialized `arch_blocks.tasks` branch at the phase transition.

### Phase 2 — Foundation Mental Models

**Frame coverage:** **MM1**, **MM3**, **MM6**, **F11**.

If `mental_models_taught.adapter` is present and `mental_models_taught.agent-as-assistant` is present:
Say: `«Мост к данным и логику ассистента ты уже видел. С задачами всё так же: агент подключается к рабочей поверхности и помогает вести реальную работу.»`
Check: `«Скажи, если есть вопрос. Иначе продолжаю.»`
Skip the split adapter / agent-as-assistant teach branches below and continue to the `tracker-as-shared-memory` block.

Else:

If `mental_models_taught.adapter` is absent:
Say: `«Сам по себе агент твои задачи не видит.»`
Say: `«Ему нужен мост к тому месту, где они живут.»`
Say: `«Такой мост здесь и есть адаптер.»`
Else:
Say: `«Мост к данным ты уже знаешь. С задачами это тот же принцип.»`

Check: `«Скажи, если есть вопрос. Иначе продолжаю.»`

If `mental_models_taught.agent-as-assistant` is absent:
Say: `«Думай об этом как об ассистенте.»`
Say: `«То, что попросил бы человека найти, связать или разложить по задачам, теперь просишь у агента.»`
Else:
If `mental_models_taught.adapter` is absent:
Say: `«Ассистентская логика та же: ты ставишь рабочую задачу, агент помогает её вести.»`

Check: `«Скажи, если есть вопрос. Иначе продолжаю.»`

If `mental_models_taught["tracker-as-shared-memory"]` is absent:
Say: `«Важно одно различие.»`
Say: `«Трекер здесь не отдельный блокнот для агента.»`
Say: `«Это общее место работы: агент читает контекст там же, где работа уже живёт, и пишет туда же, если ты разрешил.»`
Else:
Say: `«Помним: трекер здесь общий рабочий стол, а не скрытая тетрадка агента.»`
Check: `«Скажи, если есть вопрос. Иначе продолжаю.»`

Action (silent, no learner output):
- Mark absent mental-model slugs in memory.
- Save them only at the phase transition.

### Phase 3 — Tracker List, No-Tracker Branch, and Agent-Memory Home

**Frame coverage:** **G2**, **G13**, **MM4**, **MM7**.

Action (silent, no learner output):
- If `arch_blocks.tasks.declared_trackers` already exists, use it as the ordered baseline.
- Else parse `inventory.task_trackers` into an ordered `declared_trackers` list in memory with:
  - `slug`
  - `label`
  - `resolution_status = "pending"`
  - `is_agent_memory_home = false`

If `declared_trackers` is non-empty:
Say: `«Из диагностики я понял такие места, где у тебя живут задачи.»`
Action (silent, no learner output):
- Print the ordered list back as a numbered plain Russian list with human labels only.
Check: `«Что делаем? 1 список верный, 2 поправить список, 3 у меня сейчас нет трекера.»`
Else:
Say: `«В диагностике явного списка трекеров не вижу.»`
Check: `«Что делаем? 1 назову список сейчас, 2 у меня сейчас нет трекера.»`

If the learner chooses the edit-or-name branch:
Check: `«Напиши список через запятую. По одному короткому названию на трекер.»`
Action (silent, no learner output):
- Re-parse the learner answer into ordered `declared_trackers` entries with fresh slugs and `resolution_status = "pending"`.

If the learner chooses the no-tracker branch:
Check: `«Что сейчас мешает? Одной короткой фразой.»`
Action (silent, no learner output):
- Set:
  - `arch_blocks.tasks.status = "deferred_no_tracker"`
  - `arch_blocks.tasks.no_tracker_reason`
  - `arch_blocks.tasks.declared_trackers = []`
  - `arch_blocks.tasks.connected = []`
  - `arch_blocks.tasks.wow_moment_today_view_shown = false`
  - `arch_blocks.tasks.wow_moment_decomposition_run = false`
  - `arch_blocks.tasks.wow_moment_decomposition_parent_ref = ""`
- Save the phase transition.
- Jump to Phase 14.

If `mental_models_taught["personal-vs-corporate"]` is absent:
Say: `«Личный и рабочий трекер могут выглядеть одинаково.»`
Say: `«Но последствия разные: в рабочем обычно больше видимости и чужих правил.»`
Else:
Say: `«Напомню короткое: личный и рабочий контур с виду похожи, но риск разный.»`
Check: `«Скажи, если есть вопрос. Иначе продолжаю.»`

If `mental_models_taught["agent-memory-home"]` is absent:
Say: `«Подключить можно несколько трекеров.»`
Say: `«Но дом памяти у агента лучше один.»`
Say: `«Тогда ясно, где он читает контекст и где оставляет свой след.»`
Else:
Say: `«Подключений может быть много. Дом памяти — один.»`
Check: `«Скажи, если есть вопрос. Иначе продолжаю.»`

Say: `«Теперь выберем этот дом памяти.»`
Action (silent, no learner output):
- Build a numbered menu from `declared_trackers`.
- If a course-default home such as GitHub Issues is relevant for this learner's route and is not already in the list, add it as a menu-only suggested option. Do not append it to `declared_trackers` unless the learner picks it as the memory home.
- Reserve `9. Свой вариант`.
- Show the menu in Russian labels only.
Check: `«Куда ставим дом памяти агента? Напиши номер.»`

If the learner chooses `9`:
Check: `«Что это за трекер?»`
Action (silent, no learner output):
- Append the custom tracker to `declared_trackers` with `resolution_status = "pending"` and `is_agent_memory_home = false`.

If the learner chooses the menu-only `GitHub Issues` option and `declared_trackers` does not already contain it:
Action (silent, no learner output):
- Append `GitHub Issues` to `declared_trackers` with `resolution_status = "pending"` and `is_agent_memory_home = false`.

If the chosen home is not the course-default suggestion or the choice would otherwise be ambiguous on resume:
Check: `«Коротко: почему именно этот трекер?»`
Action:
- Keep the reason in memory and later record it only if needed for routing traceability.

Action (silent, no learner output):
- Mark exactly one `declared_trackers[]` entry as `is_agent_memory_home = true`.
- Reset every other `declared_trackers[]` entry to `is_agent_memory_home = false`.
- Mark absent `personal-vs-corporate` and `agent-memory-home` slugs in memory.
- Save the phase transition.

### Phase 4 — GitHub Readiness Routing

**Frame coverage:** **G14**, **F16**.

Action (silent, no learner output):
- Re-check:
  - `arch_blocks.github_setup.status`
  - chosen `declared_trackers[]` entry with `is_agent_memory_home = true`
  - `arch_blocks.obsidian_vault.path` from `/pos-vault` when present; it unlocks the exact G7 vault-location check later but never blocks routing.

If the chosen memory home is GitHub Issues and `github_setup.status != "done"`:
Say: `«Чтобы сделать GitHub домом памяти, сначала нужен базовый GitHub-каркас.»`
Say: `«Сейчас перекину тебя в /pos-github-setup, потом вернёмся сюда и продолжим с трекерами.»`
Action (silent, no learner output):
- Save `pending_resume = "pos-tasks-after-github-setup"`.
- Invoke `/pos-github-setup`.

If no handoff is needed:
Say: `«GitHub-порог ясен. Дальше идём по трекерам по одному.»`

### Phase 5 — Loop: Provider, Class, Scope Framing, and Connection Method

**Frame coverage:** **G2.5**, **G3**, **G5**, **MM5**, **G14**, **F12**, **F15**, **F16**.

Action (silent, no learner output):
- Build `pending_trackers` as every `declared_trackers[]` entry whose `resolution_status == "pending"`.
- If no pending trackers remain, jump to Phase 11.
- If Resume Logic carried an explicit `current_tracker_slug` into this phase and that slug matches a `pending_trackers` entry, pick that entry as `current_tracker`. Otherwise pick the first entry in `pending_trackers`.
- Set `current_tracker_slug = current_tracker.slug`.
- Look up `current_connection` as the `connected[]` record where `tracker_slug == current_tracker_slug`.
- If at least two `declared_trackers[]` entries are already `wired`, at least one pending tracker remains, and session-local `phase5_proactive_pause_prompt_shown != true`, set it to `true` and offer a one-time off-ramp before continuing.

If the proactive off-ramp condition fired:
Say: `«Два трекера собраны. Следующий сейчас или ставим паузу до следующего захода?»`
Check: `«Что делаем? 1 идём дальше, 2 пауза.»`

If the learner chooses `2`:
Action (silent, no learner output):
- Save `pending_resume = "pos-tasks-loop"`.
- Save `pending_resume_tracker_slug = current_tracker_slug`.
- Save the phase transition.
Say: `«Остановимся здесь. Когда будешь готов продолжить, запусти /pos-tasks.»`

Say: `«Берём следующий трекер: <current_tracker.label>.»`

If `current_connection` already has `provider`:
Say: `«Точное название уже зафиксировано. Просто сверю его с тобой.»`
Check: `«Что делаем? 1 оставляем как есть, 2 уточняем заново.»`
If the learner chooses `1`:
Action (silent, no learner output):
- Keep the stored provider and continue to the tracker-class step.
If the learner chooses `2` or `current_connection` is missing or has no `provider`:
Say: `«Сначала уточню точное название.»`
Action (silent, no learner output):
- Build a provider-confirm menu from the best current guesses plus common nearby matches.
- Always reserve `9. Свой вариант`.
- Show the menu with Russian labels only.
Check: `«Что это точнее? Напиши номер.»`

If the learner chooses `9`:
Check: `«Напиши точное название.»`
Action (silent, no learner output):
- Map the custom label to `provider = "other"` and keep the raw wording in `provider_label`.

If `current_connection` already has `tracker_class`:
Say: `«Контур уже помечен. Быстро сверю его.»`
Check: `«Что делаем? 1 оставляем как есть, 2 меняем контур.»`
If the learner chooses `1`:
Action (silent, no learner output):
- Keep the stored `tracker_class` and continue.
If the learner chooses `2` or `current_connection` is missing or has no `tracker_class`:
Say: `«Теперь уточню контур этого трекера.»`
Check: `«Что это за контур? 1 личный, 2 рабочий, 3 общий или командный.»`

If `tracker_class == "work"` or `current_connection.tracker_class == "work"`:
Say: `«Это рабочий трекер. Сейчас подключим агента — и это шаг другого класса, чем личное.»`
Say: `«Всё, что агент там прочитает и тем более сделает, видно твоему работодателю: коллегам в уведомлениях, руководителю в ленте, и навсегда в аудите аккаунта.»`
Say: `«У компании, скорее всего, есть политика про AI. Где-то такое прямо разрешено, где-то прямо запрещено, а часто — серая зона.»`
Say: `«Ответственность за подключение к рабочему трекеру — на тебе.»`
Check: `«Подключаем работу? 1 да, подключаем осознанно, 2 нет, не сейчас.»`

If the learner chooses `2`:
Action (silent, no learner output):
- Set `current_tracker.resolution_status = "deferred"`.
- Add a structured gap note: `{"slug": "<current_tracker_slug>", "reason": "инфосек не принят", "deferred_at_phase": "5"}`.
- Save the phase transition.
Say: `«<current_tracker.label> отложили. Причина: инфосек не принят.»`
- If pending trackers remain, loop back to the top of Phase 5.
- If all declared trackers are wired or deferred, continue to Phase 11.

If the learner chooses `1`:
Action (silent, no learner output):
- Set `current_connection.infosec_consent_granted = true`.

If `mental_models_taught["scopes-risk"]` is absent:
Say: `«У задач три уровня доверия.»`
Say: `«Посмотреть, поменять и удалить — это не одно и то же.»`
Say: `«Начинаем с минимума и поднимаем доступ только когда понятно зачем.»`
Check: `«Что делаем? 1 ясно, 2 есть вопрос.»`
Else:
Say: `«Напомню короткое: читать, менять и удалять — три разных уровня риска.»`
Check: `«Что делаем? 1 дальше, 2 есть вопрос.»`

If `current_connection` already has `connection_method`:
Say: `«Путь подключения уже выбран. Быстро сверю его.»`
Check: `«Что делаем? 1 оставляем как есть, 2 выбираем заново.»`
If the learner chooses `1`:
Action (silent, no learner output):
- Keep the stored `connection_method`.
If the learner chooses `2` or `current_connection` is missing or has no `connection_method`:
Say: `«Для самого моста есть несколько форм.»`
Say: `«Иногда это готовый мост внутри агента. Иногда отдельный инструмент. Иногда свой маленький адаптер.»`
Action (silent, no learner output):
- Read [`tracker-landscape.md`](./tracker-landscape.md).
- Build a connection-method menu for the confirmed provider from the tier hint plus runtime receipts.
- Use learner-facing labels:
  - `1. Готовый мост`
  - `2. Отдельный инструмент`
  - `3. Свой маленький адаптер`
  - `4. Родной GitHub путь` only when the provider is GitHub Issues
  - `9. Свой вариант`
Check: `«Какой путь берём? Напиши номер.»`

If the learner chooses `9`:
Check: `«Какой путь ты имеешь в виду?»`
Action (silent, no learner output):
- Normalize the answer into the closest internal `connection_method`.

If the learner is choosing a new method value:
Action (silent, no learner output):
- Map the learner-facing method choice to:
  - `mcp`
  - `cli`
  - `vibe-coded-adapter`
  - `native-gh`

Action (silent, no learner output):
- Create or update the current `connected[]` entry with:
  - `tracker_slug = current_tracker_slug`
  - `provider = newly chosen provider or current_connection.provider`
  - `provider_label = newly chosen provider_label or current_connection.provider_label`
  - `tracker_class = newly chosen tracker_class or current_connection.tracker_class`
  - `connection_method = newly chosen connection_method or current_connection.connection_method`
  - preserve `infosec_consent_granted` as set at G3 (only `true` on work-class trackers; `false` otherwise)
  - `is_agent_memory = true` only if `current_tracker.is_agent_memory_home == true`
- Mark absent `scopes-risk` in memory.

Say: `«Идём дальше с этим трекером или отложим?»`
Check: `«Что делаем? 1 продолжаем, 2 отложим на потом.»`

If the learner chooses `2`:
Check: `«Почему его откладываем?»`
Action (silent, no learner output):
- Set `current_tracker.resolution_status = "deferred"`.
- Add a structured gap note: `{"slug": "<current_tracker_slug>", "reason": "<learner raw phrase>", "deferred_at_phase": "5"}`.
- Save the phase transition.

Say: `«<current_tracker.label> отложили. Причина: <learner raw phrase>.»`

If the count of pending trackers is at least `2`:
Check: `«Следующий трекер сейчас или пауза? Скажи пауза, если нужна.»`

If the learner says `пауза`:
Action (silent, no learner output):
- Save `pending_resume = "pos-tasks-loop"`.
- Save `pending_resume_tracker_slug` as the first pending tracker slug.
- Save the phase transition.
Say: `«Остановимся здесь. Когда будешь готов продолжить, запусти /pos-tasks.»`

Action (silent, no learner output):
- If pending trackers remain, loop back to Phase 5.
- If all declared trackers are either wired or deferred, continue to Phase 11.

Action (silent, no learner output):
- If the chosen method resolves to `vibe-coded-adapter` and `basic_vibecoding.status != "done"`:
  - save `pending_resume = "pos-tasks-after-basic-vibecoding"`
  - save `pending_resume_tracker_slug = current_tracker_slug`
  - save the phase transition
  - invoke `/pos-basic-vibecoding`
- Save the phase transition.

### Phase 6 — Loop: Auth Surface and Credential Location

**Frame coverage:** **G4**, **G7**, **F6**, **F7**, **F10**.

Action (silent, no learner output):
- Use the `current_tracker_slug` selected by Phase 5 or Resume Logic.
- Look up `current_tracker` in `declared_trackers[]` and `current_connection` in `connected[]` where `tracker_slug == current_tracker_slug`.

If `current_connection` already has `credentials_location` and `credentials_path`:
Say: `«Место ключей по этому трекеру уже зафиксировано вне vault. Возьму его как базу.»`
Check: `«Что делаем? 1 оставляем как есть, 2 меняем путь хранения.»`
If the learner chooses `1`:
Action (silent, no learner output):
- Keep the stored credential location and skip the Build below.
If the learner chooses `2` or `current_connection` is missing `credentials_location` or `credentials_path`:
Say: `«Сейчас проверю, как этот трекер пускает агента внутрь.»`

Build:
- Research the provider's current auth path from primary sources at runtime.
- Before any auth, token, or permission flow, show the learner the requested permissions in plain Russian.
- Ask one explicit approval before the learner confirms anything on-screen.
- Enforce account separation:
  - if the current tracker is `work`, do not reuse a known personal credential path
  - if the current tracker is `personal`, do not reuse a known work credential path
- Offer learner-facing storage options in plain Russian and map them internally to:
  - `keyring`
  - `env-file`
  - `tool-native`
- Verify the exact credential location is outside `arch_blocks.obsidian_vault.path`.
- Never print secrets or raw tokens into chat output.
- If the Build fails, Say: `«Не получилось войти в этот трекер. Можно повторить позже или сменить путь. Детали — вывод последней команды или окно авторизации, которое сейчас видно.»`

Action (silent, no learner output):
- Write on the `connected[]` record where `tracker_slug == current_tracker_slug`:
  - `credentials_location`
  - `credentials_path`
- Save the phase transition.

### Phase 7 — Loop: Project Opt-In and First Read

**Frame coverage:** **G6**, **G8**, **F9**.

Action (silent, no learner output):
- Use the `current_tracker_slug` selected by the loop or Resume Logic.
- Look up `current_tracker` in `declared_trackers[]` and `current_connection` in `connected[]` where `tracker_slug == current_tracker_slug`.

If `current_connection` already has `projects_connected` and `scopes_granted` contains `read`:
Say: `«Первое чтение по этому трекеру уже подтверждено. Беру его как готовое.»`
Check: `«Что делаем? 1 идём дальше, 2 перечитываем поверхность заново.»`
If the learner chooses `1`:
Action (silent, no learner output):
- Keep the stored first-read result and skip the Build below.
If the learner chooses `2` or `current_connection` is missing `projects_connected` or `read` scope:
Say: `«Теперь проверю, есть ли внутри этого трекера несколько проектов или рабочих пространств.»`

Build:
- If the provider has multiple projects, spaces, repos, databases, or boards:
  - show them back as a numbered list
  - ask which ones подключаем
  - connect only the explicitly chosen ones
- If the provider has no such layer, say one plain Russian sentence and continue.
- Before the first read, show the learner the exact surface:
  - tracker name
  - selected project list if relevant
  - account class
- Ask:
  - `«Сейчас читаю только это. Что делаем? 1 продолжаем, 2 сужаем поверхность.»`
- On `2`, stay in this phase until the learner narrows or confirms the surface.
- Perform the first read only after the explicit `1`.
- If the Build fails, Say: `«Не получилось сделать первое чтение. Можно сузить поверхность или повторить позже. Детали — вывод последней команды или ответ API, который виден сейчас.»`

Action (silent, no learner output):
- Set the `connected[]` record where `tracker_slug == current_tracker_slug` to the chosen `projects_connected`.
- If `scopes_granted` is empty or absent, set it to `["read"]`.
- Save the phase transition.

### Phase 8 — Loop: Today-Ranked View

**Frame coverage:** **G8** and the first wow peak.

Action (silent, no learner output):
- Use the `current_tracker_slug` selected by the loop or Resume Logic.
- Look up `current_tracker` in `declared_trackers[]` and `current_connection` in `connected[]` where `tracker_slug == current_tracker_slug`.

If `current_connection` is not `is_agent_memory == true`:
Action (silent, no learner output):
- Skip to Phase 9 for this tracker.

If `arch_blocks.tasks.wow_moment_today_view_shown == true`:
Action (silent, no learner output):
- Skip to Phase 9 for this tracker.

Say: `«Теперь один полезный срез на сегодня.»`
Say: `«Покажу вид на сегодня, чтобы было видно: агент уже читает задачи по делу.»`

Build:
- Generate a plain Russian ranked view from the current agent-memory tracker only.
- Keep it read-only.
- Keep the shape small:
  - what горит сегодня
  - что ждёт ответа
  - что можно закрыть или архивировать позже
- Show the view directly in the chat.

Check: `«Что делаем? 1 полезно, идём дальше, 2 хочу уточнить, как это читать.»`

Action (silent, no learner output):
- Set `wow_moment_today_view_shown = true`.
- Save the phase transition.

### Phase 9 — Loop: Read-to-Write Gate

**Frame coverage:** **G9**, **MM5**, **F1**, **F13**.

Action (silent, no learner output):
- Use the `current_tracker_slug` selected by the loop or Resume Logic.
- Look up `current_tracker` in `declared_trackers[]` and `current_connection` in `connected[]` where `tracker_slug == current_tracker_slug`.

Say: `«С этим трекером можно остаться только на чтении.»`
Say: `«Или открыть рабочие действия: создавать, править, комментировать, связывать, ставить метки, закрывать и архивировать.»`
Say: `«Удаление сюда не входит.»`
Check: `«Что делаем? 1 оставляем только чтение, 2 открываем рабочие действия.»`

Action (silent, no learner output):
- If the learner stays on read:
  - keep `scopes_granted = ["read"]`
  - remove any gap note that exactly matches `write scope granted, first real write not yet verified::<current_tracker_slug>`
  - set `current_tracker.resolution_status = "wired"`
- If the learner opens write on a non-work tracker, set:
  - `["read", "create", "edit", "comment", "link", "label", "close", "archive"]`
- If write is opened, ensure there is a gap note for this tracker: `write scope granted, first real write not yet verified::<current_tracker_slug>`.
- `hard_delete_enabled` stays `false`.
- Save the phase transition.

If the learner stayed on read:
Say: `«С этим трекером готово.»`

  If the count of pending trackers is at least `2`:
  Check: `«Следующий трекер сейчас или пауза? Скажи пауза, если нужна.»`

    If the learner says `пауза`:
    Action (silent, no learner output):
    - Save `pending_resume = "pos-tasks-loop"`.
    - Save `pending_resume_tracker_slug` as the first pending tracker slug.
    - Save the phase transition.
    Say: `«Остановимся здесь. Когда будешь готов продолжить, запусти /pos-tasks.»`

  Action (silent, no learner output):
  - If pending trackers remain, loop back to Phase 5.
  - If all declared trackers are either wired or deferred, continue to Phase 11.

### Phase 10 — Loop: Baseline Snapshot, Attribution, and Loop Boundary

**Frame coverage:** **G10**, **G15**, **F2**, **F14**.

Action (silent, no learner output):
- Use the `current_tracker_slug` selected by the loop or Resume Logic.
- Look up `current_tracker` in `declared_trackers[]` and `current_connection` in `connected[]` where `tracker_slug == current_tracker_slug`.

If `current_connection` has write scope beyond `["read"]` and `backup.status` is neither `enabled` nor `none_accepted`:
Say: `«Перед первой живой записью по умолчанию сделаю снимок базы.»`
Say: `«Если хочешь пропустить, напиши пропустим. Тогда я зафиксирую как пропущено с причиной.»`
Check: `«Делаем снимок или пропустим?»`

If the learner types `пропустим`:
Check: `«Почему пропускаем?»`

Build:
- If the learner did not opt out, research the provider-appropriate export path and create the snapshot.
- If the learner opted out, store the learner's reason in `gaps`, set `backup.enabled = false`, and set `backup.status = "none_accepted"`.
- If the Build fails, Say: `«Не получилось сделать снимок базы. Можно повторить позже или зафиксировать как пропущено. Детали — вывод последней команды или экран экспорта, который открыт сейчас.»`

Action (silent, no learner output):
- If a snapshot exists, populate:
  - `backup.enabled = true`
  - `backup.status = "enabled"`
  - `backup.reason = null`
  - `backup.format`
  - `backup.location`
  - `backup.cadence = "on-demand"`
  - `backup.last_run`
- If the learner accepted `none`, populate:
  - `backup.enabled = false`
  - `backup.status = "none_accepted"`
  - `backup.reason`

If `current_connection` has write scope beyond `["read"]`:
If `current_connection` already has `attribution_prefix` and `attribution_style`:
Say: `«Паттерн атрибуции уже выбран. Быстро сверю его.»`
Check: `«Что делаем? 1 оставляем как есть, 2 меняем паттерн.»`
If the learner chooses `1`:
Action (silent, no learner output):
- Keep the stored attribution pattern.
If the learner chooses `2` or `current_connection` is missing `attribution_prefix` or `attribution_style`:
Say: `«Теперь договоримся, как именно будет видно, что это написал агент.»`
Action:
- Look at the chosen tracker surface and prepare 1-3 attribution patterns that are natural for it. Use a numeric menu only for the options that actually fit this tracker; always keep `9. Свой вариант`.
Check: `«Как помечаем действия агента в этом трекере? Напиши номер.»`

If the learner chooses `9`:
Check: `«Какой паттерн берём?»`

If the learner is choosing a new attribution style:
Action:
- Map the chosen menu item to the real pattern shown for this tracker. Typical outcomes may still be `issue-title-prefix`, `comment-sig`, `label-suffix`, or `other`, but do not assume the same menu shape for every tracker before you compare what the surface supports.
- Write `attribution_prefix` and `attribution_style` on the `connected[]` record where `tracker_slug == current_tracker_slug`.

Action (silent, no learner output):
- Set `current_tracker.resolution_status = "wired"`.

Say: `«С этим трекером готово.»`

If the count of pending trackers is at least `2`:
Check: `«Следующий трекер сейчас или пауза? Скажи пауза, если нужна.»`

If the learner says `пауза`:
Action (silent, no learner output):
- Save `pending_resume = "pos-tasks-loop"`.
- Save `pending_resume_tracker_slug` as the first pending tracker slug.
- Save the phase transition.
Say: `«Остановимся здесь. Когда будешь готов продолжить, запусти /pos-tasks.»`

Action (silent, no learner output):
- Save the phase transition.
- If pending trackers remain, loop back to Phase 5.
- If all declared trackers are either wired or deferred, continue to Phase 11.

### Phase 11 — Rules of Use in the Agent-Config File

**Frame coverage:** **G12**, **F3**, **F8**.

Say: `«Теперь зафиксируем правила, чтобы агент не додумывал их сам.»`
Say: `«Пишем их в файл конфигурации твоего основного агента, в секцию ## Tasks.»`

Build:
- Draft a `## Tasks` section that includes:
  - the `is_agent_memory` home
  - one block per wired tracker
  - read behavior
  - write behavior
  - hard delete rule
  - attribution marker per tracker
  - close / archive confirmation rule
  - work-tracker caution if any work tracker exists
- For read-only trackers, do not add an attribution-marker line; marker exists only for write-enabled trackers.
- Resolve the actual target file from `learner_profile.primary_agent`: `CLAUDE.md` for Claude Code, `AGENTS.md` for Codex.
- If `learner_profile.keep_agent_configs_in_sync == true`, mirror the same accepted section to the sibling file after the primary append passes.
- If `## Tasks` already exists, show a diff and ask before merging.
- Append only.

Action (silent, no learner output):
- Show the drafted block back to the learner in plain Russian and Markdown.

Check: `«Записываю как есть. Скажи поправить, если хочешь изменить строку; только чтение, если откладываем правила.»`

If the learner says `поправить`:
Check: `«Какую строку меняем?»`
Action (silent, no learner output):
- Edit the draft and repeat the same collapsed line once.

If the learner says `только чтение`:
Action (silent, no learner output):
- Downgrade every connected tracker to `scopes_granted = ["read"]`.
- Keep `hard_delete_enabled = false`.
- Remove any gap note that starts with `write scope granted, first real write not yet verified::`.
- Remove any gap note that starts with `decomposition wow pending`.
- Add a gap note explaining that write was left disabled because rules were declined.
- Set `rules_of_use_skipped = true`.
- Set `rules_status = "deferred_read_only"`.
- Save the phase transition.

If the learner keeps the draft as-is or confirms an edited draft:
Build:
- Append the agreed `## Tasks` section to the target agent-config file.
Action (silent, no learner output):
- Set `rules_of_use_skipped = false`.
- Set `rules_status = "appended"`.
- Write `rules_appended_to.claude_md_path` if the accepted target file is `CLAUDE.md`.
- Write `rules_appended_to.agents_md_path` if the accepted target file is `AGENTS.md`.
- If `learner_profile.keep_agent_configs_in_sync == true`, mirror the same accepted section to the sibling file and record that path in the other field.
- Save the phase transition.

### Phase 12 — First Real Write Verification Prelude and Pass

**Frame coverage:** **G11**, **G15.5**, **F4**, **F14**.

Action (silent, no learner output):
- Build `write_ready_trackers` as every `connected[]` record whose `scopes_granted` contains anything beyond `read` AND `gaps` contains a note exactly matching `write scope granted, first real write not yet verified::<connected[].tracker_slug>`, paired with the learner-facing label from `declared_trackers[]` by `tracker_slug`. Trackers without the gap note are already verified and skipped.
- If `write_ready_trackers` is empty, jump to Phase 13.
- If `pending_resume_tracker_slug` is present and its record is in `write_ready_trackers`, sort the record whose `tracker_slug == pending_resume_tracker_slug` first and clear the helper in memory after selecting it.

If `entry_mode != "write-path-resume"`:
Say: `«Теперь можно проверить первую живую запись.»`
Say: `«Если хочешь поставить паузу прямо перед ней, скажи пауза.»`
Check: `«Что делаем? 1 идём в первую живую запись, 2 пауза перед записью.»`

If the learner chooses `2`:
Action (silent, no learner output):
- Save `pending_resume = "pos-tasks-write-path"`.
- Save `pending_resume_tracker_slug = write_ready_trackers[0].tracker_slug`.
- Save the phase transition.
Say: `«Остановимся здесь. Когда будешь готов продолжить, запусти /pos-tasks.»`

For each tracker in `write_ready_trackers`:
Say: `«Для <tracker.label> проверим первую живую запись.»`
Say: `«Она будет маленькой и настоящей, чтобы было видно и действие, и маркер.»`
Check: `«Что берём? 1 короткий комментарий к существующей задаче, 2 новая маленькая подзадача, 3 новая метка на существующей задаче, 9 свой вариант.»`

If the learner chooses `9`:
Check: `«Что именно делаем?»`

Build:
- Pick the smallest supported action that creates a new learner-visible trace for this provider.
- The chosen action must be present in `scopes_granted` for this tracker.
- Avoid an unmarked edit as the first real write.
- Show the exact dry-run preview in plain Russian:
  - target
  - action
  - payload shape
  - attribution marker
- Ask:
  - `«Запускаем это в реальном трекере? 1 да, 2 нет.»`
- On `1`, execute the write.
- Read the result back immediately from the tracker output.
- Verify that the chosen marker is visible.
- If the marker is absent or malformed:
  - stop
  - rerun the G15 menu for this tracker
  - repeat the same first-write verification
- If the Build fails, Say: `«Не получилось сделать первую живую запись. Можно повторить позже или сменить маленькое действие. Детали — вывод последней команды или ответ трекера, который виден сейчас.»`

Action (silent, no learner output):
- If the learner chooses `2`, keep the existing gap note for this tracker: `write scope granted, first real write not yet verified::<tracker_slug>`.
- If the verification succeeds, remove any gap note that exactly matches `write scope granted, first real write not yet verified::<tracker_slug>`.
- Save the phase transition after each verified tracker.

### Phase 13 — Decomposition Wow

**Frame coverage:** the second wow peak, **F11**.

Action (silent, no learner output):
- Build `write_enabled_trackers` as every `connected[]` record whose `scopes_granted` contains anything beyond `read`, paired with the learner-facing label from `declared_trackers[]` by `tracker_slug`.
- Build `decomposition_candidates` from `write_enabled_trackers` that do not have a matching gap note `write scope granted, first real write not yet verified::<tracker_slug>`.
- If `write_enabled_trackers` is empty:
  - Say: `«Запись ни в одном трекере не открывали. Демонстрация декомпозиции переносится на следующий заход.»`
  - jump to Phase 14
- If `decomposition_candidates` is empty:
  - add a gap note: `decomposition wow pending — no verified write-enabled tracker`
  - save the phase transition
  - jump to Phase 14

Say: `«Теперь один живой проход с записью.»`
Say: `«Разложим одну настоящую цель на мелкие шаги и покажем это в реальном трекере.»`

If there is more than one candidate tracker:
Action (silent, no learner output):
- Show the candidate trackers as a numbered list.
Check: `«На каком трекере делаем разложение? Напиши номер.»`
Else:
Check: `«Делаем на <only candidate tracker.label>? 1 да, 2 отложим.»`

If the learner chooses `2` in the single-candidate branch:
Action (silent, no learner output):
- Add a gap note: `decomposition wow pending — learner deferred single-candidate run`
- Save the phase transition.
- Jump to Phase 14

Check: `«Откуда берём родителя? 1 уже есть задача, 2 назову цель сейчас.»`

If the learner chooses `1`:
Check: `«Пришли номер, ссылку или короткое название родительской задачи.»`

If the learner chooses `2`:
Check: `«Сформулируй цель одной короткой фразой.»`

Build:
- If no real parent task exists yet, create one because it is actual work, not scratch state.
- Propose a small decomposition preview:
  - parent
  - 3-5 child issues or sub-tasks
  - minimal labels or links
- Show the preview in plain Russian.
- Ask:
  - `«Что делаем? 1 создаём как показано, 2 правим, 3 отмена.»`
- On `1`, create the child artifacts and link them.
- On `2`, revise and repeat the preview.
- On `3`, add a gap note and stop short of live creation.
- If the Build fails, Say: `«Не получилось разложить цель в трекере. Можно повторить позже или оставить только превью. Детали — вывод последней команды или ответ трекера, который виден сейчас.»`

Action (silent, no learner output):
- If live creation happened:
  - set `wow_moment_decomposition_run = true`
  - set `wow_moment_decomposition_parent_ref` to the parent tracker-native URL or identifier
- Save the phase transition.

### Phase 14 — Track and Handoff

**Frame coverage:** end-state writeback and close discipline.

Action (silent, no learner output):
- Update `my-architecture.md` with a short `Tasks` section covering:
  - connected trackers
  - the `is_agent_memory` home
  - which trackers stayed read-only
  - which tracker got the decomposition path
- Compute final status:
  - keep `deferred_no_tracker` as-is if it was set in Phase 3
  - `done` if:
    - every declared tracker is either `wired` or `deferred`
    - exactly one tracker has `is_agent_memory: true`
    - `rules_status == "appended"` or `rules_status == "deferred_read_only"`
    - `wow_moment_today_view_shown == true`
    - `wow_moment_decomposition_run == true` or no `connected[]` record has scope beyond `["read"]`
    - there is no gap note that starts with `write scope granted, first real write not yet verified::`
    - there is no gap note for `decomposition wow pending`
  - `needs_verification` if only rules resolution, first live write, or decomposition on a write-enabled path remains open
  - `partial` otherwise
  - If status is `done`, set `completed_at`.
- Summarize in plain Russian:
  - connected trackers
  - the agent-memory home
  - whether write stayed off or was verified
  - whether decomposition already happened
  - if `status == "deferred_no_tracker"`, summarize the `no_tracker_reason` instead of wired trackers

If there are gap notes:
Say: `«Ещё остались хвосты. Их доберём этой же командой позже.»`

Action (silent, no learner output):
- Recommend the next block by, in order:
  1. `learner-state.json` diagnostic-route / recommendations field if set by `pos-diagnostic`.
  2. `my-architecture.md` — parse the next unfinished block from the learner's plan.
  3. Fallback: bundled `skill-catalog.json` entries that are `shipped`, `menu_visible`, not yet `done`, and not this skill; prefer entries whose prerequisites already look satisfied in learner state.
- Name one specific next block by slash command plus human name; do not hedge.
- If nothing unfinished remains, say so plainly and suggest `/pos-diagnostic` for re-prioritization.
- Do NOT recommend `/pos-tasks` itself as the next step — the pause farewell already carries `«запусти /pos-tasks»` as the re-entry cue for `partial` / `needs_verification` / `deferred_no_tracker`.

Say: `«Дальше логично идти к </pos-next>. Если нужна пауза — скажи.»`
Check: `«Если нужна пауза, скажи пауза.»`

If the learner asks for pause:
Say: `«Остановимся здесь. Когда будешь готов продолжить, запусти /pos-tasks.»`

Action (silent, no learner output):
- Hand off to the recommended next skill.

## Learner feedback close reminder

Перед финальным Check или прощанием этого блока добавь расширенное напоминание:
`«Если здесь что-то было непонятно, сломано, особенно полезно или если хочется чего-то больше или по-другому, скажи — я помогу оформить фидбек для создателей.»`
Если ученик откликается, предложи свободно описать ситуацию и дальше переведи его в `/pos-feedback`-flow.

## References

- `../../docs/skill-contract.md`
- `../../docs/block-runtime-pattern.md`
- The bundled `skill-catalog.json`
- [`tracker-landscape.md`](./tracker-landscape.md) — agent-internal provider and routing hints
