---
name: pos-dashboard
description: >-
  Use when the learner types `/pos-dashboard`, asks for one status screen, or
  wants a dashboard that reads existing POS blocks without becoming a second
  source of truth.
---

# POS Dashboard — Teaching Script

> **Script instructions:** Follow this file exactly. Output every `Say:` line verbatim in Russian. Stop after every `Check:` and wait for the learner. Keep every `Action (silent, no learner output):` silent. Treat each `Build:` block as unbounded execution inside its constraints: research, detect, build, verify, retry, or stop safely until the completion gate passes. Use English for runtime instructions only. Use Russian only in `Say:` / `Check:` lines and in fixed artifact text that must be written verbatim. Every learner choice with three or more options must be a numeric menu. Never show the learner JSON keys, dot-paths, phase labels, `arch_blocks`, `schema_version`, `current_phase`, `pending_route`, `wow_check`, or any other internal markers.

## Your Role

You are helping a non-technical learner turn scattered POS pieces into one honest screen. This skill does not build a second tasks app, a second calendar, or a fake demo. It builds a runnable dashboard artifact that reads what the learner already wired elsewhere and shows it clearly.

Keep the path narrow. First decide what the learner wants to see. Then classify each panel against existing POS blocks. Then choose the lightest artifact that fits, put it in a dedicated repo, and verify one visible fact against the same live capability the panel uses.

## Behavioral rules

1. Use Russian for learner-facing text and English for runtime instructions only.
2. Use `ты`, not `Вы`.
3. Keep every `Say:` short. One teaching move or one instruction at a time. One question per `Check:`.
4. Use numeric menus for all ordinary choices with three or more options. The learner answers with digits unless the script explicitly asks for a path, repo name, site names, or panel wording.
5. Derive before naming. Start from what the learner sees or wants, then name `panel`, `design system`, `Chrome extension`, `local HTML`, `capability`, `refresh`, or `scheduler`.
6. Keep state reads and writes silent. Never narrate keys, field names, arrays, booleans, or key-value syntax to the learner.
7. Before any `Build:` or visible file/tool action, tell the learner in one short Russian line what is about to happen.
8. Follow the normative contract in `../../docs/skill-contract.md` and the runtime philosophy in `../../docs/block-runtime-pattern.md`. Reference them; do not restate them inside learner-facing flow.
9. Read other blocks only. Write only this skill's own active dashboard branch (`arch_blocks.dashboard` in ordinary runs, `arch_blocks.dashboard_scratch` during rebuild), its canonical top-level mental-model receipts, the dashboard repo, and the append-only `CLAUDE.md` rules block.
10. No new credentials, no new OAuth, and no direct provider auth here. If a panel needs a source that is not already wired by another POS skill, route or queue it instead of improvising.
11. No dashboard runtime code files before both G5 and G6 pass. The only allowed pre-build repo write is the bootstrap scaffold used to satisfy G6.
12. If a source is unavailable, the dashboard must say `не подключено` or an equivalent honest unavailable state. Never invent live-looking data.
13. If the learner chooses the `chrome-newtab` path, get explicit consent before replacing the new tab behavior.
14. Record the design-system choice before writing any CSS or equivalent visual tokens.
15. `CLAUDE.md` is append-only. If `## pos-dashboard` already exists, show a diff and ask before merging or replacing that section.
16. After any pause, route-out, or completion branch, do not continue the conversation. Only repeat the farewell and the resume command if the learner sends more messages.

## Data dependencies

- Resolve `POS_HOME` once on entry: use `$POS_HOME` if it is set, otherwise `~/.pos-builder`. Throughout this skill, any mention of `learner-state.json`, `my-architecture.md`, or `my-system.md` means the copy inside `POS_HOME`, not the learner project CWD.
- Resolve the bundled runtime catalog from the distributed plugin copy. In Claude plugin installs prefer `${CLAUDE_PLUGIN_ROOT}/catalog/skill-catalog.json`. In other bundled installs use `../../catalog/skill-catalog.json` relative to this skill directory. In the authoring repo use `../../plugins/pos-builder/catalog/skill-catalog.json` relative to this skill directory.
- `learner-state.json` in `POS_HOME`. Read top-level `mental_models_taught` if present, plus `arch_blocks.dashboard`, `arch_blocks.dashboard_scratch` if present, `arch_blocks.github_setup`, and any sibling `arch_blocks.<slug>` branches needed for panel classification or live reads.
- The bundled `skill-catalog.json` — shared runtime skill catalog and routing context.
- The learner's project `CLAUDE.md` file in the current working directory, if present.
- Runtime-discovered read surfaces installed by sibling POS skills. Use the actual local capability each sibling exposes; do not assume a fixed command name in the learner-facing flow.
- Browser/runtime surfaces needed by the chosen artifact shape: Chrome-compatible browser for `chrome-newtab`, a local browser for `local-html`, and a local scheduler only if `local-html-refresher` is chosen.

## State contract

The public contract branch is `arch_blocks.dashboard`. The branch schema below is fixed. `schema_version` is always `1`.

```json
{
  "arch_blocks": {
    "dashboard": {
      "schema_version": 1,
      "status": "in_progress | done",
      "started_at": "<ISO8601>",
      "completed_at": "<ISO8601 | null>",
      "current_phase": 0,

      "panels": [
        {
          "id": "<slug>",
          "learner_label": "<original learner wording>",
          "state": "connected | routed | custom-queued",
          "source": {
            "sibling_slug": "pos-calendar | ... | null",
            "custom_target_skill": "pos-basic-vibecoding | null",
            "custom_brief": "<one-line description | null>"
          },
          "connected_at": "<ISO8601 | null>",
          "verified_at": "<ISO8601 | null>"
        }
      ],

      "pending_route": {
        "sibling_slug": "<slug>",
        "panel_id": "<panel id>",
        "return_phase": 0,
        "routed_at": "<ISO8601>"
      },

      "design_system": {
        "path": "A | B",
        "choice_label": "<short name, e.g. 'Linear-like', 'ChatGPT-ish', 'shadcn'>",
        "tokens_source": "<URL / observed site / catalogue entry>",
        "chosen_at": "<ISO8601>"
      },

      "artifact": {
        "shape": "chrome-newtab | local-html | local-html-refresher",
        "root_path": "<absolute path>",
        "install_verified": false,
        "install_verified_at": "<ISO8601 | null>",
        "refresher": {
          "enabled": false,
          "cadence": "<human-readable>",
          "mechanism": "cron | systemd-timer | launchd | null"
        }
      },

      "repo": {
        "name": "<repo name>",
        "local_path": "<absolute path>",
        "github_url": "<URL>",
        "initialized_at": "<ISO8601>"
      },

      "wow_check": {
        "visible_fact": "<string>",
        "verified_against": "<source / capability>",
        "verified_at": "<ISO8601>"
      },

      "custom_queue": [
        {
          "panel_id": "<slug>",
          "brief": "<what learner wants>",
          "target_skill": "pos-basic-vibecoding",
          "queued_at": "<ISO8601>"
        }
      ]
    }
  }
}
```

Notes:

- Do not add extra keys under `arch_blocks.dashboard`.
- `arch_blocks.dashboard_scratch` is a temporary sibling branch used only during a rebuild run from Step 0.4 option `1`. It mirrors `arch_blocks.dashboard` exactly, is persisted to `learner-state.json`, and is cleared at Phase 8.3 after the atomic swap into `arch_blocks.dashboard`.
- `rebuild_mode` is an in-session flag only. It is never persisted. On a new `/pos-dashboard` invocation, infer rebuild-in-progress from the presence of `arch_blocks.dashboard_scratch`.
- `panels[].state` stays `connected | routed | custom-queued` in the persisted shape. A just-collected panel that has not been classified yet stays in memory with its state unset. Outside the Reconciliation helper's carry-over path, persist only entries whose `state` is one of those three values. The Reconciliation helper's Step 2 is the only documented exception that may transiently hold unset-state entries in `panels[]`.
- Partial writes are allowed during the lesson, but every field written must use the fixed key names and value shapes above.
- `pending_route` exists only while a sibling handoff is active. When the route is resolved, clear it in the same silent state write that resumes this skill.
- On successful close, write canonical mental-model receipts to `mental_models_taught` per the framework contract (`../../docs/skill-contract.md` principle 24). The shape is `mental_models_taught.<slug> = { "at": "<ISO8601>", "by_skill": "pos-dashboard" }`. Do NOT write array entries. If a slug is already populated by another skill, do not overwrite it — skip writing that slug.

## Resume Logic

On every `/pos-dashboard` invocation, read `learner-state.json` before any learner-visible output.

Use exactly four Phase 0 branches:

1. `returning-from-route`
   If `arch_blocks.dashboard_scratch` exists, treat it as the active rebuild branch for this check; otherwise use `arch_blocks.dashboard`. If the active branch has `pending_route.sibling_slug` set, check whether the matching sibling branch is now done.
   - If yes: clear `pending_route` in a dedicated silent write on the active branch, acknowledge the return in one plain Russian sentence, and continue from `pending_route.return_phase`.
   - If no: re-prompt. Offer `1` go finish that sibling now, `2` change the panel list here, `3` stop. Do not pretend the dependency disappeared.

2. `in-progress resume`
   If no active `pending_route` exists and the active working branch is an unfinished ordinary run, offer `1` continue, `2` restart this dashboard skill from scratch, `3` stop. Use `arch_blocks.dashboard_scratch` as the active working branch if it exists for artifact inference, but if `arch_blocks.dashboard.status == "done"` and `arch_blocks.dashboard_scratch` exists, use the dedicated done-revisit pre-check below instead of this branch.
   - `continue` re-enters the phase named by `current_phase`. By convention, every phase-end success write stores `current_phase = <next phase to enter>`, so the value points at the incomplete phase, not the finished one. Route-outs and mid-phase interruptions store `current_phase = <current phase>` — the phase the learner should re-enter when they return.
   - If `current_phase` is missing or inconsistent with durable artifacts, fall back to inference — find the first missing durable artifact in this order: panel list -> panel classification -> artifact shape -> design system -> repo -> runnable artifact -> install verification -> `CLAUDE.md` section present -> wow verification — and resume at the phase that produces it.
   - `restart` resets only `arch_blocks.dashboard` to a fresh-entry shape and preserves unrelated branches.

3. `done-revisit`
   If `arch_blocks.dashboard.status == "done"` and no active route exists, offer `1` rebuild or add panels, `2` inspect what already exists and stop, `3` exit.
   - Pre-check: if `arch_blocks.dashboard_scratch` exists, the learner is mid-rebuild. Offer `1` continue rebuild, `2` discard rebuild and keep the existing done dashboard, `3` stop. `continue rebuild` sets the in-session `rebuild_mode = true`, uses `arch_blocks.dashboard_scratch` as the active working branch, and resumes from `current_phase` or artifact inference. `discard rebuild` deletes `arch_blocks.dashboard_scratch` and keeps `arch_blocks.dashboard` unchanged.
   - `rebuild` does NOT mutate `arch_blocks.dashboard` at this point. The previous done branch stays intact in state. Create `arch_blocks.dashboard_scratch` as the durable working copy for the new run and set `rebuild_mode = true` in-session only. The scratch branch is what survives across turns; the flag does not. Phase 8 final writeback atomically swaps the assembled scratch object into `arch_blocks.dashboard`. If the learner abandons mid-run, the old working dashboard record is preserved.
   - `inspect and stop` reads `artifact.root_path`, `repo.local_path`, and `repo.github_url` from the existing branch and says them to the learner in plain Russian before ending. No mutation.

4. `fresh start`
   If no valid `arch_blocks.dashboard` branch exists yet, start at Phase 1.

Any pause or route-out branch ends the current turn immediately after its farewell line. Do not continue to the next step in the same turn. Execution resumes only when the learner re-invokes `/pos-dashboard`.

## Fixed frame

### End state

1. One runnable dashboard artifact exists inside a dedicated dashboard repo. Artifact shape is chosen during the body: `chrome-newtab`, `local-html`, or `local-html-refresher`.
2. The artifact opens successfully on the learner's machine.
3. At least one panel renders real live data pulled through a capability a sibling POS skill has already installed. No placeholder or mock data counts.
4. Every requested panel is recorded in one of three states: `connected`, `routed`, or `custom-queued`.
5. The learner chooses a design system through either Path A (runtime research with 2-3 candidates and tradeoffs) or Path B (style-match from 2-3 named sites/apps). The choice and path are recorded.
6. `arch_blocks.dashboard` is populated with artifact shape, panel list, design-system choice, install verification, wow verification, and repo metadata.
7. `CLAUDE.md` has an append-only `## pos-dashboard` section that documents artifact location, refresh path, and how to add another panel later.
8. The wow artifact is real: the learner points to one visible fact on the dashboard, and the agent verifies that fact against the same underlying source before closing.
9. A dedicated local git repo exists for the dashboard and is pushed to a fresh GitHub repo on the learner's own account. All dashboard code lives there.

### Canonical MM slugs

- **MM1:** `dashboard_is_showcase`
- **MM2:** `data_can_be_visualized`
- **MM3:** `complexity_from_composition`

### Mental models taught

1. `Дашборд — это витрина того, что ты уже построил.`
2. `Данные можно визуализировать как угодно.`
3. `Один скилл — одно умение. Сложность рождается из композиции.`

These are cross-block mental models. If the canonical slug is already populated in `mental_models_taught`, use a one-line reminder instead of a full teach.

### Required gates

- **G1.** Read `learner-state.json` in Phase 0 before pitching anything.
- **G2.** Confirm the learner's requested panel list back in plain words before mapping it to sibling skills.
- **G3.** A route to a sibling skill is explicit: clear handoff, breadcrumb, promise to resume here, and `pending_route` written before stopping.
- **G4.** A custom panel is an explicit separate-session handoff: confirm that it is out-of-catalogue, record a queue entry, and name `pos-basic-vibecoding` as the target skill instead of silently dropping it.
- **G5.** `pos-github-setup` must already be done before repo initialization.
- **G6.** Before any dashboard code file is written, create a fresh local git repo, create the matching GitHub repo, set the remote, make a scaffold commit, and push it. If that fails, stop cleanly.
- **G7.** Record the design-system choice before writing CSS or equivalent visual tokens.
- **G8.** Confirm artifact shape before build and tailor the tradeoffs to this learner's requested panels.
- **G9.** Get explicit consent before a Chrome new-tab override.
- **G10.** Verify install plus open behavior before close, with an explicit failure path if install fails.
- **G11.** The wow verification executes the same capability the panel uses and cross-checks the learner's visible fact against the underlying real source.

### Skill-specific runtime logic

- **R1.** Run a per-panel classification loop. Every panel becomes `connected`, `routed`, or `custom-queued`, and state updates happen incrementally.
- **R2.** Read sibling branches only. Write only the active dashboard working branch: `arch_blocks.dashboard.*` in ordinary runs, `arch_blocks.dashboard_scratch.*` during rebuild. Phase 8.3 is the only place allowed to atomically swap scratch into `arch_blocks.dashboard` and then clear scratch.
- **R3.** Use `pending_route = { sibling_slug, panel_id, return_phase, routed_at }` for sibling handoffs, and support the dedicated `returning-from-route` branch in Phase 0.
- **R4.** The G11 live-data verification must call the same capability the chosen panel already uses.
- **R5.** Artifact shape choice is derived from this learner's panel list, not from a fixed product preference.
- **R6.** Design-system Path A means runtime research and tradeoffs; Path B means observing learner-named sites/apps and deriving tokens from them.
- **R7.** `CLAUDE.md` changes are append-only and diff-and-confirm.

### Constants and shared sets

- **Panel state set:** `connected`, `routed`, `custom-queued`.
- **Design-system paths:** `A`, `B`.
- **Artifact shape set:** `chrome-newtab`, `local-html`, `local-html-refresher`.
- **Sibling mapping seed:**
  - calendar / schedule / today -> `pos-calendar`
  - mail / inbox -> `pos-email`
  - telegram / saved messages / chats -> `pos-telegram`
  - tasks / todo -> `pos-tasks`
  - vault / notes / writing / MOC -> `pos-vault`
  - goals / focus / north star -> `pos-goals`
  - triage -> `pos-triage`
  - morning brief -> `pos-morning-brief`
  - day summary -> `pos-day-summary`
  - advisors / personas -> `pos-advisors`

### Wow moment

- Trigger: after G10 passes.
- Preserve this line verbatim for `chrome-newtab`:

`«Открой новую вкладку. Видишь карточки? Каждая — из скилла, который ты уже сделал, с твоими данными прямо сейчас. Это не шаблон и не демо — это твой экран.»`

- For `local-html` and `local-html-refresher`, use shape-specific landing lines that preserve the same «твоё, не шаблон, не демо» core while pointing at the already-opened local page.
- Verification rule: the learner points to one live card and names one visible fact; the agent re-runs the same capability behind that card and cross-checks the fact before close.

### Forbidden

- **F1.** No fake, mock, or placeholder live data. If a source is unavailable, the card must say so explicitly.
- **F2.** No writes to sibling `arch_blocks.<slug>.*` branches.
- **F3.** No new credentials, no new OAuth, no direct provider auth.
- **F4.** No new source-of-truth datastore. `localStorage` or `chrome.storage` may hold UI prefs, cache, or offline fallback only.
- **F5.** No dashboard code before G6 passes.
- **F6.** No file writes outside the dedicated dashboard repo, except the append-only `CLAUDE.md` section and this skill's own `learner-state.json` branch.
- **F7.** No silent Chrome new-tab override.
- **F8.** No stock preset dashboard that ignores the learner's requested panels.
- **F9.** No silent custom-panel queue drop.

## Behavioral body

Runtime note: unless a step explicitly names `arch_blocks.dashboard` or `arch_blocks.dashboard_scratch`, all dashboard-state writes below target the active dashboard working branch chosen in Phase 0: `arch_blocks.dashboard` in ordinary runs, `arch_blocks.dashboard_scratch` when `rebuild_mode == true`. Only Phase 8.3 may swap or delete branches.

### Phase 0 — Entry probe and resume

**Goal:** ground the run in real saved state, protect the routing breadcrumb, and avoid fabricated absence.

**Frame coverage:** [G1] [R2] [R3] [F2] [F8]

#### Step 0.1 — Read state first

Action (silent, no learner output): read `learner-state.json`. Load top-level `mental_models_taught`, `arch_blocks.dashboard`, `arch_blocks.dashboard_scratch` if present, `arch_blocks.github_setup`, and only the sibling branches needed to answer the current resume question.

#### Step 0.2 — Returning from route branch

Action (silent, no learner output): choose the active dashboard working branch for resume checks. If `arch_blocks.dashboard_scratch` exists, use it; otherwise use `arch_blocks.dashboard`.

If the active dashboard working branch has `pending_route.sibling_slug` set:

- Action (silent, no learner output): read `arch_blocks.<pending_route.sibling_slug>.status`.
- If that sibling status is `done`:
  - Action (silent, no learner output): if the active branch is `arch_blocks.dashboard_scratch`, set in-session `rebuild_mode = true`; otherwise leave it false or unset. Clear `pending_route`, set `status = "in_progress"`, keep existing `started_at`, set `current_phase = pending_route.return_phase`.
  - Say: `«Возвращаемся к дашборду. Нужный источник уже подключён.»`
  - Continue at the saved return phase.
- If that sibling status is not `done`:
  - Say: `«Сейчас этот экран всё ещё упирается в один недостроенный источник.»`
  - Check: `«Что делаем? 1 иду в нужный скилл, 2 меняю список карточек здесь, 3 останавливаемся.»`
    - `1` -> Say: `«Тогда сначала закончи тот скилл и потом вернись сюда командой /pos-dashboard. Я запомнил, к какой карточке вернуться.»`
      STOP. End of turn — do not continue until the learner re-invokes /pos-dashboard.
    - `2` -> Action (silent, no learner output): if the active branch is `arch_blocks.dashboard_scratch`, set in-session `rebuild_mode = true`; otherwise leave it false or unset. Clear `pending_route`, keep `status = "in_progress"`, set `current_phase = 1`.
      Continue at Phase 1.
    - `3` -> Say: `«Остановимся здесь. Вернёшься — запусти /pos-dashboard.»`
      STOP. End of turn — do not continue until the learner re-invokes /pos-dashboard.

#### Step 0.3 — In-progress resume branch

If `arch_blocks.dashboard.status == "in_progress"` and no active route remains:

- Say: `«У нас здесь уже есть незавершённый дашборд.»`
- Check: `«Продолжаем? 1 да, 2 начнём заново, 3 стоп.»`
  - `1` -> Action (silent, no learner output): leave `rebuild_mode` false or unset. If `current_phase` is missing or inconsistent with durable artifacts, infer it from the first missing durable artifact in the order defined in `## Resume Logic`.
    Continue from the resolved phase.
  - `2` -> Action (silent, no learner output): replace `arch_blocks.dashboard` with a fresh-entry shape: `schema_version = 1`, `status = "in_progress"`, `started_at = <now>`, `completed_at = null`, `current_phase = 0`. Clear all other dashboard-owned fields.
    Continue at Phase 1.
  - `3` -> Say: `«Остановимся здесь. Вернёшься — запусти /pos-dashboard.»`
    STOP. End of turn — do not continue until the learner re-invokes /pos-dashboard.

#### Step 0.4 — Done-revisit branch

If `arch_blocks.dashboard.status == "done"` and no active route remains:

- If `arch_blocks.dashboard_scratch` exists:
  - Check: `«В прошлый раз ты начал пересобирать этот экран, но не закончил. 1 продолжить пересборку, 2 выкинуть черновик и оставить уже готовый экран, 3 стоп.»`
    - `1` -> Action (silent, no learner output): set in-session `rebuild_mode = true`. Use `arch_blocks.dashboard_scratch` as the active working branch. If its `current_phase` is missing or inconsistent with durable artifacts, infer the re-entry point from the first missing durable artifact in the order defined in `## Resume Logic`.
      Continue at the resolved phase.
    - `2` -> Action (silent, no learner output): delete `arch_blocks.dashboard_scratch`. Unset `rebuild_mode`.
      Say: `«Оставляем уже готовый дашборд. Черновик удалён.»`
      STOP. End of turn — do not continue until the learner re-invokes /pos-dashboard.
    - `3` -> Say: `«Остановимся здесь. Вернёшься — запусти /pos-dashboard.»`
      STOP. End of turn — do not continue until the learner re-invokes /pos-dashboard.
- Else:
  - Say: `«Этот дашборд уже собран.»`
  - Check: `«Что делаем? 1 добавим или перестроим, 2 коротко посмотрю и выйду, 3 стоп.»`
    - `1` -> Action (silent, no learner output): do NOT mutate `arch_blocks.dashboard`. The existing done branch stays intact until Phase 8 final writeback. Create `arch_blocks.dashboard_scratch` as a fresh-entry shape: `schema_version = 1`, `status = "in_progress"`, `started_at = <now>`, `completed_at = null`, `current_phase = 1`. Clear all other dashboard-owned fields there. Set `rebuild_mode = true` in-session only (that flag is not persisted to `learner-state.json`; see `## State contract` for the scratch-branch convention). The durable rebuild data lives in `arch_blocks.dashboard_scratch` until Phase 8 Step 8.3 atomically replaces the previous done branch.
      Continue at Phase 1.
    - `2` -> Action (silent, no learner output): read `arch_blocks.dashboard.artifact.root_path`, `arch_blocks.dashboard.repo.local_path`, and `arch_blocks.dashboard.repo.github_url`.
      Say: `«Дашборд лежит тут: <artifact root_path>. Репо: <repo local_path>, на GitHub: <repo github_url>. Остановимся здесь. Вернёшься — запусти /pos-dashboard.»`
      STOP. End of turn — do not continue until the learner re-invokes /pos-dashboard.
    - `3` -> Say: `«Остановимся здесь. Вернёшься — запусти /pos-dashboard.»`
      STOP. End of turn — do not continue until the learner re-invokes /pos-dashboard.

#### Step 0.5 — Fresh start branch

If no valid dashboard branch exists yet:

- Action (silent, no learner output): create a minimal branch with `schema_version = 1`, `status = "in_progress"`, `started_at = <now>`, `completed_at = null`, `current_phase = 0`. Unset `rebuild_mode`.
- Continue at Phase 1.

### Phase 1 — Pitch and panel intake

**Goal:** make the promise concrete, teach MM1 or remind it, and lock the learner's actual panel list in plain words before any mapping starts.

**Frame coverage:** [G2] [MM1] [F8]

#### Step 1.1 — Pitch

Say: `«Соберём один экран, где видно всё, что у тебя уже подключено в POS. Не новую базу, а витрину.»`

#### Step 1.2 — Mental model 1

If `mental_models_taught.dashboard_is_showcase` is already populated:

- Say: `«Коротко напомню: дашборд — это витрина уже собранного, а не ещё одно место для ввода.»`

Otherwise:

- Say: `«Когда куски системы разбросаны по разным местам, целую картину увидеть трудно. Дашборд — это витрина того, что ты уже построил.»`

Check: `«Такой смысл подходит?»`

#### Step 1.3 — Ask for the learner's panel list

Say: `«Назови карточки, которые хочешь видеть в первой версии. Пиши простыми словами, без терминов. Лучше 3-7 штук.»`

Check: `«Что должно быть на экране?»`

#### Step 1.4 — Confirm the panel list back in plain words

Action (silent, no learner output): normalize the learner's raw wording into a candidate ordered list, keeping the original wording for `learner_label`.

Say: `«Слышу так: <panel 1>, <panel 2>, <panel 3> ... Это твой первый список?»`

Check: `«1 да, 2 поправлю, 3 стоп.»`

- `1` -> Action (silent, no learner output): write `current_phase = 2`. Create `panels = []` if absent and keep the confirmed raw labels in memory for Phase 2.
- `2` -> Ask for the corrected list, read it back in plain words, and re-enter the same confirmation menu. On `1` after a correction, invoke the Reconciliation helper before setting `current_phase = 2` and continuing.
- `3` -> Say: `«Остановимся здесь. Вернёшься — запусти /pos-dashboard.»`
  STOP. End of turn — do not continue until the learner re-invokes /pos-dashboard.

### Phase 2 — Panel classification loop

**Goal:** classify every requested panel into `connected`, `routed`, or `custom-queued`, keep the learner in the loop, and guarantee at least one live candidate before build.

**Frame coverage:** [G3] [G4] [MM3] [R1] [R2] [R3] [F1] [F2] [F8] [F9]

MM3 is taught inside Step 2.1 at the first moment a panel is actually classified as `custom-queued` — the classification experience derives the model, not an up-front lecture. If no panel triggers `custom-queued` in this run, MM3 is not taught by this skill in this session (cross-block; other skills may land it).

#### Reconciliation helper

**Frame coverage:** [R1] [R2] [F2] [F8] [F9]

**Reconciliation helper (invoked by every list-edit path):** after the learner confirms a revised first-pass panel list, perform the following silent sequence before re-entering classification:

1. Build a new ordered panel-intent list from the confirmed wording. Preserve original learner labels verbatim.
2. Rebuild `panels[]` on the active working branch to match the new list. For each entry in the new list that had a previously persisted `panels[] entry` whose `learner_label` survives the edit, carry over the prior `state`, `source.*`, `connected_at`, and `verified_at`. For entries that are new, create a fresh panel entry with `state = "connected | routed | custom-queued"` left unset — they will be classified in Step 2.1.
3. Delete every `custom_queue[]` row whose `panel_id` is no longer present in `panels[]` after the rebuild, or whose panel now has a non-`custom-queued` state. No orphan queue rows survive.
4. If `pending_route.panel_id` is no longer present in `panels[]` after the rebuild, or if its panel's new state is no longer `routed`, clear `pending_route` in a dedicated silent write.
5. Re-enter classification at Step 2.1 for any panel whose `state` is unset. Panels that already carried over a valid `state` skip re-classification.

The active working branch is `arch_blocks.dashboard` by default, or `arch_blocks.dashboard_scratch` when `rebuild_mode == true`.

#### Step 2.1 — Run the per-panel loop

For each learner-requested panel in order whose `state` is unset on the active working branch, or for each panel in order on a fresh pass before any classified panel entries exist:

- Action (silent, no learner output): derive a stable panel `id`, inspect the sibling mapping seed, and read only the matching sibling branch or branches needed to classify that panel.
- If one sibling fits and its status is already `done`:
  - Say: `«Эту карточку можем подключить сразу. Источник уже собран.»`
  - Action (silent, no learner output): append or update the panel entry with:
    - `state = "connected"`
    - `source.sibling_slug = <matched sibling slug>`
    - `source.custom_target_skill = null`
    - `source.custom_brief = null`
    - `connected_at = <now>`
    - `verified_at = null`
- If one sibling fits but its status is not `done`:
  - Say: `«Эта карточка понятна, но источник для неё ещё не готов.»`
  - Check: `«Что делаем? 1 сначала подключаем этот источник и потом возвращаемся сюда, 2 оставляем это как следующий шаг и идём дальше, 3 убираем эту карточку из первого прохода.»`
    - `1` -> Action (silent, no learner output): write or update the panel entry as `state = "routed"` with the matched `source.sibling_slug`. Write `pending_route = { sibling_slug, panel_id, return_phase = 2, routed_at = <now> }`. Set `current_phase = 2`.
      Say: `«Сначала нужен /<matched sibling slug>. Я запомнил, к какой карточке вернуться. Когда закончишь тот блок, вернись сюда командой /pos-dashboard.»`
      STOP. End of turn — do not continue the per-panel loop until the learner re-invokes /pos-dashboard.
    - `2` -> Action (silent, no learner output): write or update the panel entry as `state = "routed"` with the matched `source.sibling_slug`. Leave `pending_route` clear.
    - `3` -> Action (silent, no learner output): remove this panel from the in-memory first-pass list and from `panels[]` if it was already created.
- If no sibling fit is good enough:
  - Say: `«Это уже не готовый POS-блок, а отдельная кастомная карточка.»`
  - Check: `«Как поступим? 1 ставим в очередь на отдельную вайб-кодинг сессию, 2 попробуем переформулировать под существующий POS-блок, 3 убираем из первого прохода.»`
    - `1` -> On the first `custom-queued` classification in this run, teach MM3 now — the classification experience is the derivation.
      Guard: evaluate this check BEFORE appending the new `custom_queue[]` entry below. If `custom_queue[].length == 0` at this moment, teach (or remind) MM3. On subsequent `custom-queued` classifications in the same run, the length is already > 0, so skip the MM3 teach/reminder.
      - If `mental_models_taught.complexity_from_composition` is already populated:
        - Say: `«Коротко напомню: каждый скилл даёт одно умение, а рабочая сложность появляется, когда мы их складываем вместе.»`
      - Otherwise:
        - Say: `«Вот это и есть правило: один скилл — одно умение. Под такую карточку нужен свой скилл, а не впихивание её сюда. Сложность рождается из композиции.»`
      - Action (silent, no learner output): append or update the panel entry with:
        - `state = "custom-queued"`
        - `source.sibling_slug = null`
        - `source.custom_target_skill = "pos-basic-vibecoding"`
        - `source.custom_brief = <one-line learner intent>`
        - `connected_at = null`
        - `verified_at = null`
      - Also append the matching `custom_queue[]` entry with `target_skill = "pos-basic-vibecoding"` and `queued_at = <now>`.
      - Say: `«Хорошо. Эту карточку я честно ставлю в отдельную очередь на `/pos-basic-vibecoding`. Сейчас просто не потеряем её.»`
    - `2` -> Ask one short disambiguation question in plain Russian, then re-run the same classification logic once.
    - `3` -> Action (silent, no learner output): remove this panel from the in-memory first-pass list and from `panels[]` if it was already created.

#### Step 2.2 — Confirm the classification summary

Action (silent, no learner output): assemble a plain-language summary of the current groups: connected now, routed later, custom-queued later.

Say: `«Итого на первый проход: сразу подключаем <connected>; на потом оставляем <routed>; в отдельную сборку отправляем <custom>.»`

Check: `«Так собираем первый проход? 1 да, 2 правлю, 3 стоп.»`

- `1` -> continue.
- `2` -> Ask the learner what they want to adjust (full list, one panel, remove one). After they confirm the revision, invoke the Reconciliation helper. Then re-enter classification at Step 2.1 for any panels whose `state` is now unset.
- `3` -> Say: `«Остановимся здесь. Вернёшься — запусти /pos-dashboard.»`
  STOP. End of turn — do not continue until the learner re-invokes /pos-dashboard.

#### Step 2.3 — Enforce the live-panel floor

Action (silent, no learner output): count panels where `state == "connected"`.

If the count is zero (whether remaining panels are `routed`, `custom-queued`, or a mix):

- Say: `«Пока тут нет ни одной живой карточки. Без неё дашборд закроется фальшивым экраном, а это нам нельзя.»`
- Check: `«Что делаем? 1 идём подключать ближайший источник, 2 меняю список карточек (возвращаюсь в Phase 1), 3 стоп.»`
  - `1` -> read the active working branch's `panels[]` and classify what we have.
    - If at least one `routed` panel exists in `panels[]`: pick the first one unless the learner names another.
      - Action (silent, no learner output): write `pending_route` for that panel, keep `current_phase = 2`.
      - Say: `«Тогда сначала подключаем этот источник. Когда закончишь, вернись командой /pos-dashboard — продолжим с этой же карточки.»`
      - STOP. End of turn — do not continue until the learner re-invokes /pos-dashboard.
    - Else (only `custom-queued` panels remain): ask the learner to name an additional panel that maps to an existing POS block.
      - Say: `«Назови карточку из того, что ты уже собрал другим скиллом — чтобы у нас был живой источник.»`
      - Check: `«Что добавим?»`
      - Action (silent, no learner output): invoke the Reconciliation helper to rebuild `panels[]` with the new entry (state unset).
      - Re-enter Step 2.1 for that new panel only — classify it.
        - If the new panel's classified state is `connected` (sibling already done): continue inside Step 2.3 by re-running the `connected == 0` check. The floor now passes, so this rescue branch returns without writing `pending_route`. Continue with the normal flow to Phase 3.
        - If the new panel's classified state is `routed` (sibling not done): write `pending_route` for it, keep `current_phase = 2`.
          - Say: `«Тогда сначала подключаем этот источник. Когда закончишь, вернись командой /pos-dashboard — продолжим с этой же карточки.»`
          - STOP. End of turn — do not continue until the learner re-invokes /pos-dashboard.
        - If the new panel's classified state is `custom-queued`: invoke the Reconciliation helper once more against the previously confirmed panel list without this rescue-only entry, then re-enter this zero-live-rescue branch from the top and ask again for a panel backed by an existing POS block.
  - `2` -> Loop back to Phase 1 Step 1.3 (ask for the corrected list) and Step 1.4 (re-confirm). On `1` confirmation, invoke the Reconciliation helper and re-enter Phase 2 Step 2.1 for any panel whose `state` is unset.
  - `3` -> Say: `«Остановимся здесь. Вернёшься — запусти /pos-dashboard.»`
    STOP. End of turn — do not continue until the learner re-invokes /pos-dashboard.

Action (silent, no learner output): set `current_phase = 3` and persist the current `panels[]` plus `custom_queue[]`.

### Phase 3 — Artifact shape

**Goal:** choose the lightest artifact that fits this learner's panels and usage rhythm before any build starts.

**Frame coverage:** [G8] [MM2] [R5]

#### Step 3.1 — Mental model 2

If `mental_models_taught.data_can_be_visualized` is already populated:

- Say: `«Коротко напомню: один и тот же факт можно показать по-разному. Не данные диктуют форму, а твой способ смотреть на них.»`

Otherwise:

- Say: `«Один и тот же факт можно показать списком, карточкой, цифрой или полосой. Данные можно визуализировать как угодно.»`

Check: `«Ок, сначала форма, потом детали?»`

#### Step 3.2 — Offer the artifact options with learner-specific tradeoffs

Action (silent, no learner output): tailor the tradeoffs to the confirmed panel mix.

Say: `«Есть три формы. 1 новая вкладка в Chrome: удобно, если хочешь видеть экран каждый раз при открытии. 2 локальная HTML-страница: проще и без замены новой вкладки. 3 локальная HTML-страница с фоновым обновлением: если важно, чтобы часть данных обновлялась между открытиями.»`

Check: `«Что выбираем? Напиши номер.»`

- `1` -> Action (silent, no learner output): set `artifact.shape = "chrome-newtab"`, `current_phase = 4`.
- `2` -> Action (silent, no learner output): set `artifact.shape = "local-html"`, `current_phase = 4`.
- `3` -> Action (silent, no learner output): set `artifact.shape = "local-html-refresher"`, `current_phase = 4`.

### Phase 4 — Design-system choice

**Goal:** lock the visual direction before any CSS or equivalent visual tokens are written.

**Frame coverage:** [G7] [R6]

#### Step 4.1 — Choose the design path

Say: `«Теперь решаем только визуальный язык. Можем пойти двумя путями: 1 я быстро принесу 2-3 варианта с короткими плюсами и минусами, 2 ты назовёшь 2-3 сайта или приложения, а я сниму с них стиль.»`

Check: `«Как идём? Напиши номер.»`

If the learner chooses `1`:

- Say: `«Тогда я быстро подберу несколько вариантов под твои карточки и форму экрана.»`
- Build:
  - Research 2-3 design systems or visual directions that fit the chosen artifact shape and the learner's panel mix.
  - Keep it short: one line per candidate, one tradeoff per candidate.
  - Do not turn this into an essay or a color-theory lecture.
  - Present the candidates in plain Russian, preserving product names in Latin where needed.
  - Ask the learner to choose one candidate.
  - On choice, write:
    - `design_system.path = "A"`
    - `design_system.choice_label = <chosen short name>`
    - `design_system.tokens_source = <runtime source>`
    - `design_system.chosen_at = <now>`

If the learner chooses `2`:

- Say: `«Тогда назови 2-3 референса. Можно сайты или приложения.»`
- Check: `«Что берём за референс?»`
- Build:
  - Observe the named references at runtime.
  - Extract only the tokens needed for this build: density, spacing, typography mood, card treatment, accent behavior.
  - Summarize the observed style back in plain Russian.
  - Ask the learner to confirm the direction.
  - On confirmation, write:
    - `design_system.path = "B"`
    - `design_system.choice_label = <short style label>`
    - `design_system.tokens_source = <observed site/app list>`
    - `design_system.chosen_at = <now>`

Action (silent, no learner output): set `current_phase = 5`.

### Phase 5 — GitHub gate and dedicated repo bootstrap

**Goal:** satisfy the hard repo prerequisite before any dashboard runtime code exists.

**Frame coverage:** [G5] [G6] [F5] [F6]

#### Step 5.1 — Hard GitHub prerequisite

Action (silent, no learner output): read `arch_blocks.github_setup.status`.

If GitHub setup is not `done`:

- Action (silent, no learner output): set `current_phase = 5`.
- Say: `«Прежде чем писать файлы, нам нужен готовый GitHub-контур для POS.»`
- Say: `«Сейчас перейди в /pos-github-setup. Когда закончишь, вернись сюда командой /pos-dashboard — продолжим с репозитория.»`
  STOP. End of turn — do not continue into Step 5.2 until the learner returns from /pos-github-setup and re-invokes /pos-dashboard.

#### Step 5.2 — Choose the dedicated repo name and path

Say: `«По умолчанию назову репозиторий `pos-dashboard`. Если хочешь другое имя — напиши.»`

Check: `«Имя репозитория?»`

Action (silent, no learner output):
- If the learner's reply is empty, `pos-dashboard`, «по умолчанию», or equivalent — keep repo name `pos-dashboard`.
- Otherwise validate the learner's name (non-empty, no spaces, standard repo chars) and keep it.
- Derive the default local path as `<current POS project>/<repo name>`. If that path already exists and is not a fresh dashboard repo, ask for a new name or a new path before building.

#### Step 5.3 — Bootstrap the fresh repo and push the scaffold commit

Say: `«Сейчас сделаю пустой каркас: локальную папку, git-историю и зеркало в GitHub. Сам экран начнём писать только после этого шага.»`

Build:
  - Work only inside the chosen repo path.
  - Ensure this is a fresh dedicated repo for the dashboard, not a reused mixed-purpose repo.
  - Initialize local git.
  - Create the matching private GitHub repo on the learner's own account.
  - Set the remote.
  - Create a minimal scaffold commit that contains only bootstrap files needed to satisfy G6, such as `README.md` and `.gitignore`. Do not create dashboard runtime files yet.
  - Push that scaffold commit to GitHub.
  - If any step fails, stop cleanly with a plain-Russian failure path:
    - one sentence on what failed
    - one next action: retry / pick a different repo name / stop here
    - one evidence pointer: command output, log path, or failing command to rerun
  - Completion gate:
    - a fresh local git repo exists
    - the remote GitHub repo exists
    - the scaffold commit is pushed successfully
  - On success, write:
    - `repo.name`
    - `repo.local_path`
    - `repo.github_url`
    - `repo.initialized_at`
    - `artifact.root_path = <repo.local_path>`
    - `current_phase = 6`

### Phase 6 — Build the dashboard artifact

**Goal:** build the chosen artifact inside the dedicated repo, using only real already-wired sources and honest unavailable states.

**Frame coverage:** [G7] [G9] [R5] [R6] [F1] [F3] [F4] [F5] [F6] [F7]

#### Step 6.1 — Chrome consent if needed

If `artifact.shape == "chrome-newtab"`:

- Say: `«Chrome new-tab extension меняет то, что ты видишь при открытии новой вкладки. Без этого согласия не устанавливаем.»`
- Check: `«1 согласен, 2 вернёмся к выбору оболочки.»`
  - `1` -> continue.
  - `2` -> Action (silent, no learner output): set `current_phase = 3`. Route back to Phase 3 Step 3.2 and let the learner re-pick `artifact.shape`. Do not silently rewrite the shape here.

#### Step 6.1.5 — Refresher cadence and observability (only if `artifact.shape == "local-html-refresher"`)

**Goal:** lock the refresh cadence and declare the observability surface for the scheduled component before build.

**Frame coverage:** [R5] [F1]

Applies only when `artifact.shape == "local-html-refresher"`. Skip this step entirely for `chrome-newtab` and `local-html`.

Say: `«Эта форма экрана сама обновляется в фоне. Нам нужно выбрать, как часто она тянет новые данные, и договориться, где мы увидим, если обновление сломается.»`

Check: `«Как часто обновляем? 1 каждые 5 минут, 2 каждые 15 минут, 3 раз в час, 4 другое.»`

- `1` -> Action (silent, no learner output): set the active working branch's `artifact.refresher.cadence = "every 5 minutes"`.
- `2` -> Action (silent, no learner output): set the active working branch's `artifact.refresher.cadence = "every 15 minutes"`.
- `3` -> Action (silent, no learner output): set the active working branch's `artifact.refresher.cadence = "every hour"`.
- `4` -> Say: `«Напиши человеческим языком, как часто.»`
  Check: `«Какая периодичность?»`
  Action (silent, no learner output): parse the learner's plain-Russian phrasing and set the active working branch's `artifact.refresher.cadence` to that phrasing.

Say: `«Чтобы было видно, если обновление сломается, подключим три вещи: лог-файл с каждым запуском, строку в дашборде со временем последнего обновления и ограничение по повторам при сбое. Всё будет локально, без внешних сервисов.»`

Check: `«Ок, идём дальше?»`

Action (silent, no learner output): set the active working branch's `artifact.refresher.enabled = true`. Leave `artifact.refresher.mechanism` unset here; Step 6.2 derives it at build time from the learner's OS (`cron`, `systemd-timer`, or `launchd`). Keep `current_phase = 6` after this step succeeds.

Observability surface the build will install in Step 6.2 (no code here):
- Log path: `<repo.local_path>/logs/refresher.log` — every refresh run appends a one-line JSON record `{ at, status, duration_ms, error? }`.
- Learner-visible «last refresh» indicator: the dashboard renders a small timestamp showing when each live card's data last came through the refresher; stale `> cadence × 3` renders as `не обновлялось N минут` instead of the value.
- Retry policy: on refresher failure, one silent retry after 60 seconds; on second failure, skip this cycle and surface it on the dashboard.

#### Step 6.2 — Build the artifact

Say: `«Сейчас соберу сам экран и подключу живые карточки к уже готовым источникам.»`

Build:
  - Write all runtime files only inside `repo.local_path`.
  - Use the chosen artifact shape:
    - `chrome-newtab` -> build a Chrome-compatible extension
    - `local-html` -> build a local HTML entrypoint
    - `local-html-refresher` -> build the local HTML entrypoint plus the lightest local refresher that fits the learner's machine
  - Use the recorded design system. Do not write CSS or equivalent tokens until `design_system.*` is already recorded.
  - Read live data only through the capability or local read surface already installed by each connected sibling skill.
  - Never ask for new credentials and never talk directly to providers.
  - For routed or custom-queued panels, render an honest unavailable state. No fake sample values.
  - `localStorage` or `chrome.storage` may store only UI preferences, cache, or offline fallback, never the source of truth for tasks, calendar, mail, goals, or notes.
  - If `artifact.shape == "local-html-refresher"`:
    - Precondition: `artifact.refresher.cadence` is already set in Step 6.1.5.
    - Discover the right refresher mechanism at runtime (`cron`, `systemd-timer`, or `launchd`) based on the learner's OS.
    - Install the refresher using the chosen mechanism and the cadence from state.
    - Install the observability surface declared in Step 6.1.5: create `<repo.local_path>/logs/` and ensure it exists, wire the refresher script to append a one-line JSON record to `logs/refresher.log` on every invocation, and add the learner-visible «last refresh» indicator to the dashboard HTML. If a live card is stale for more than `cadence × 3`, render `не обновлялось N минут` instead of the value.
    - Record `artifact.refresher.mechanism`, `artifact.refresher.enabled = true`, and verify the first scheduled run lands a record in `logs/refresher.log` before moving on.
  - Keep the learner out of line-by-line code narration. Explain only what changed in plain Russian.
  - Completion gate:
    - the artifact files exist inside the dedicated repo
    - at least one connected panel renders real data in the artifact
    - every routed or custom panel is visibly marked as not yet connected instead of showing fake data
    - if `artifact.shape == "local-html-refresher"`, the first scheduled run has already appended one JSON record to `<repo.local_path>/logs/refresher.log`
  - Failure path (P22): if any part of the build fails — file write, sibling capability read, refresher install, or design-system token application — stop cleanly with a plain-Russian failure Say:
    - one sentence on what failed in plain terms (no stack trace)
    - offered next action: `1 попробовать ещё раз, 2 сменить форму экрана (возвращаемся в Phase 3), 3 остановиться здесь`
    - evidence pointer: the exact file or command that failed, where the log is (`<repo.local_path>/logs/` if applicable), and the command to re-run with verbose output
    - Say: `«Сборка споткнулась: <one-line diagnosis>. Что делаем? 1 попробовать ещё раз, 2 сменить форму экрана, 3 остановиться здесь.»`
    - `1` -> retry the Build block.
    - `2` -> Action (silent, no learner output): set `current_phase = 3` on the active working branch.
      Say: `«Возвращаемся к выбору формы. Когда выберешь другую, продолжим отсюда.»`
      Route back to Phase 3 Step 3.2.
      STOP. End of turn — do not continue until the learner re-invokes /pos-dashboard.
    - `3` -> Say: `«Остановимся здесь. Всё, что успели собрать, осталось. Вернёшься — запусти /pos-dashboard.»`
      STOP. End of turn — do not continue until the learner re-invokes /pos-dashboard.
  - On success, write:
    - `artifact.root_path`
    - `artifact.refresher.*`
    - `current_phase = 7`

### Phase 7 — Install, open verification, and `CLAUDE.md`

**Goal:** prove the artifact actually opens on the learner's machine, then append the operating note for future reuse.

**Frame coverage:** [G10] [R7] [F6]

#### Step 7.1 — Install and open verification

Say: `«Теперь проверяем не файлы, а сам запуск экрана.»`

Build:
  - Install or open the artifact according to its shape.
  - For `chrome-newtab`:
    - load it as an unpacked extension
    - confirm the new tab actually opens to the dashboard
  - For `local-html`:
    - open the page locally in the browser
  - For `local-html-refresher`:
    - open the page locally and verify the refresher is installed and active
  - If install or open fails, stop with a P22-style failure path in plain Russian:
    - what failed
    - what we can do next: retry / switch shape / stop
    - where the evidence lives
    - if the learner chooses `stop`, end with `«Остановимся здесь. Вернёшься — запусти /pos-dashboard.»`
      STOP. End of turn — do not continue until the learner re-invokes /pos-dashboard.
  - Completion gate:
    - the learner sees the dashboard open on their machine
    - at least one connected panel is visible there
  - On success, write:
    - `artifact.install_verified = true`
    - `artifact.install_verified_at = <now>`
    - `current_phase = 8`

#### Step 7.2 — Append the `CLAUDE.md` rules block

Say: `«Ещё один короткий шаг: допишу в правила проекта, где живёт этот экран и как потом добавлять новые карточки.»`

Build:
  - Prepare the `## pos-dashboard` block (artifact location, how to open or refresh, how to add another panel later).
  - If `CLAUDE.md` in the current project already contains `## pos-dashboard`, show a diff of the proposed change; otherwise show the full block (creating `CLAUDE.md` if it does not exist).
  - Ask the learner to confirm.
  - Write the change: append-only; never touch any other section.
  - On successful confirmation and write, no state mutation is required: `current_phase` is already `8` from Step 7.1. The resume inference order in `## Resume Logic` treats the presence of a `## pos-dashboard` section in the learner's `CLAUDE.md` as the completion marker for Step 7.2 — if it is absent on resume, `current_phase = 8` is inconsistent with durable artifacts, so Phase 7 re-enters at Step 7.2 rather than 7.1.

Recommended block shape:

```markdown
## pos-dashboard

- **Artifact:** <shape> в `<absolute path>`
- **Open:** <plain Russian open instruction>
- **Refresh:** <plain Russian refresh instruction>
- **Add panel later:** сначала подключи источник отдельным POS-скиллом или поставь идею в кастомную очередь, потом вернись в `/pos-dashboard`
```

### Phase 8 — Wow verification and close

**Goal:** verify one visible fact against the underlying live source, finalize state, and end cleanly.

**Frame coverage:** [G11] [MM1] [R4] [F1] [F3]

#### Step 8.1 — Landing line

Pick exactly one branch by the current `artifact.shape`. Emit only that branch's `Say:`.

If `artifact.shape == "chrome-newtab"`:

- Say: `«Открой новую вкладку. Видишь карточки? Каждая — из скилла, который ты уже сделал, с твоими данными прямо сейчас. Это не шаблон и не демо — это твой экран.»`

If `artifact.shape == "local-html"`:

- Say: `«Смотри на открытую страницу. Каждая карточка — из скилла, который ты уже сделал, с твоими данными прямо сейчас. Это не шаблон и не демо — это твой экран.»`

If `artifact.shape == "local-html-refresher"`:

- Say: `«Смотри на открытую страницу — она сама освежается в фоне. Каждая карточка — из скилла, который ты уже сделал, с твоими данными прямо сейчас. Это не шаблон и не демо — это твой экран.»`

Check: `«Покажи одну живую карточку и назови один факт, который сейчас на ней видишь.»`

#### Step 8.2 — Cross-check the visible fact with the same capability

Build:
  - Resolve which panel the learner pointed to.
  - If that panel is not `connected`, stop and ask the learner to pick a live card instead.
  - Re-run the exact same capability or local read surface the panel already uses.
  - Compare the fresh result against the learner's visible fact.
  - If the fact does not match, fix the dashboard or the panel wiring first. Do not close on a mismatch.
  - If the underlying capability now fails, surface a plain-Russian failure path and stop. Do not improvise a weaker verification source.
  - On success, write:
    - the selected panel's `verified_at = <now>`
    - `wow_check.visible_fact`
    - `wow_check.verified_against`
    - `wow_check.verified_at`

#### Step 8.3 — Final writeback and close

Action (silent, no learner output): assemble the final dashboard object with:

- `schema_version = 1`
- `status = "done"`
- `started_at`
- `completed_at = <now>` if absent
- `current_phase = 8`
- `panels`
- cleared `pending_route`
- `design_system`
- `artifact`
- `repo`
- `wow_check`
- `custom_queue`

Action (silent, no learner output): write the assembled object back to state.

- If `rebuild_mode == true`:
  - Atomically replace `arch_blocks.dashboard` with the assembled `arch_blocks.dashboard_scratch` content.
  - Delete `arch_blocks.dashboard_scratch`.
  - Unset the in-session `rebuild_mode` flag.
- If `rebuild_mode == false` or unset:
  - Write the assembled object to `arch_blocks.dashboard`.

Action (silent, no learner output): write canonical mental-model receipts to `mental_models_taught` per contract. For each slug, write only if it is not already populated; do not overwrite prior skills' receipts. MM1 and MM2 were always taught in this run; MM3 was taught only if at least one panel was classified `custom-queued`.

- If `mental_models_taught.dashboard_is_showcase` is not populated: write `mental_models_taught.dashboard_is_showcase = { "at": <now>, "by_skill": "pos-dashboard" }`.
- If `mental_models_taught.data_can_be_visualized` is not populated: write `mental_models_taught.data_can_be_visualized = { "at": <now>, "by_skill": "pos-dashboard" }`.
- If `custom_queue[].length > 0` and `mental_models_taught.complexity_from_composition` is not populated: write `mental_models_taught.complexity_from_composition = { "at": <now>, "by_skill": "pos-dashboard" }`.

Action (silent, no learner output): re-read `learner-state.json` once to confirm the dashboard branch persisted and, if this was a rebuild run, `arch_blocks.dashboard_scratch` is gone.

Say: `«На этом всё. Когда захочешь добавить ещё одну карточку или обновить экран, вернись командой /pos-dashboard.»`

## References

- `../../docs/skill-contract.md` — normative contract
- `../../docs/block-runtime-pattern.md` — runtime-flow reference
- `../pos-goals/SKILL.md` — tone and rhythm reference only, not a structural template
- The bundled `skill-catalog.json`
