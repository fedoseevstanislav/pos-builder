---
name: pos-dashboard
description: >-
  Use when the user types `/pos-dashboard`, asks for one status screen, or
  wants a dashboard that reads existing POS blocks without becoming a second
  source of truth.
---

Read this file on entry. Constraints and end state define the boundaries; flow guides the session.

**Role:** Help a non-developer turn scattered POS pieces into one honest screen that reads what they already wired elsewhere.

## End state

When done, the user has:

1. One runnable dashboard artifact (`chrome-newtab`, `local-html`, or `local-html-refresher`) inside a dedicated git repo, pushed to a fresh GitHub repo on their own account
2. The artifact opens on the user's machine; at least one panel renders real live data pulled through a capability an existing POS skill already installed
3. Every requested panel is recorded as `connected` (source done), `routed` (source not done yet, queued for another skill), or `custom-queued` (no matching POS block, queued for `/pos-basic-vibecoding`)
4. A design system chosen via Path A (agent researches 2-3 candidates) or Path B (user names 2-3 reference sites/apps and agent derives tokens)
5. One verified wow fact: the user points to a visible value on the dashboard and the agent re-runs the same capability to cross-check it
6. If `local-html-refresher`: a local scheduler refreshes data at user-chosen cadence, with a log file, a "last refresh" indicator on the dashboard, and a retry-then-surface policy for failures
7. The dashboard repo has its own agent-config file (resolved from `learner_profile.primary_agent`) with an append-only `## pos-dashboard` section documenting artifact location, refresh path, and how to add a panel later. If `learner_profile.keep_agent_configs_in_sync == true`, the sibling file is mirrored in the same repo.
8. `learner-state.json` updated (see State section)

## State

Fields this skill reads and writes in `learner-state.json`. `last_completed_step` = last finished step; resume starts at the NEXT step.

- `pending_resume` (string|null) -- written on handoff to sibling skills (Step 1, Step 3, Step 6)
- `arch_blocks.dashboard.status` (string: in_progress|done) -- written Step 1, Step 9
- `arch_blocks.dashboard.started_at` (ISO8601) -- written Step 1
- `arch_blocks.dashboard.completed_at` (ISO8601|null) -- written Step 9
- `arch_blocks.dashboard.last_completed_step` (number) -- written at each step milestone
- `arch_blocks.dashboard.panels[]` (array of {id, learner_label, state, source, target_skill, brief}) -- written Step 2 (id + learner_label + state:"requested"), Step 3 (state updated to connected/routed/custom-queued; source, target_skill, brief populated as applicable)
- `arch_blocks.dashboard.pending_route_sibling` (string|null) -- written Step 3; cleared on return in Step 1 resume
- `arch_blocks.dashboard.pending_route_panel_id` (string|null) -- written Step 3; cleared on return in Step 1 resume
- `arch_blocks.dashboard.design_system_path` (string) -- written Step 5
- `arch_blocks.dashboard.design_system_label` (string) -- written Step 5
- `arch_blocks.dashboard.design_system_chosen_at` (ISO8601) -- written Step 5
- `arch_blocks.dashboard.artifact_shape` (string) -- written Step 4
- `arch_blocks.dashboard.artifact_root_path` (string) -- written Step 6
- `arch_blocks.dashboard.artifact_install_verified` (boolean) -- written Step 8
- `arch_blocks.dashboard.config_written` (boolean) -- written Step 8; true after the agent-config section is successfully appended
- `arch_blocks.dashboard.artifact_refresher_cadence` (string|null) -- written Step 4; null if not refresher
- `arch_blocks.dashboard.repo_name` (string) -- written Step 6
- `arch_blocks.dashboard.repo_local_path` (string) -- written Step 6
- `arch_blocks.dashboard.repo_github_url` (string) -- written Step 6
- `arch_blocks.dashboard.wow_verified_at` (ISO8601) -- written Step 9
- `mental_models_taught.dashboard_is_showcase` -- written Step 2
- `mental_models_taught.data_can_be_visualized` -- written Step 4
- `mental_models_taught.complexity_from_composition` -- written Step 3 (only if any panel classified as custom-queued)

Read-only dependencies: `arch_blocks.github_setup.status`, `arch_blocks.basic_vibecoding.status`, `arch_blocks.calendar.status`, `arch_blocks.email.status`, `arch_blocks.telegram.status`, `arch_blocks.tasks.status`, `arch_blocks.vault.status`, `arch_blocks.goals.status`, `arch_blocks.triage.status`, `arch_blocks.morning_brief.status`, `arch_blocks.day_summary.status`, `arch_blocks.advisors.status`, `learner_profile`.

Read-write dependency: `mental_models_taught` -- read to check which MMs are already taught; written when new MMs are taught during this flow.

**Rebuild convention.** When the user revisits a done dashboard and chooses to rebuild, create `arch_blocks.dashboard_scratch` as the working branch mirroring the same fields. All writes target the scratch branch during rebuild. Step 9 atomically swaps scratch into `arch_blocks.dashboard` and deletes scratch. If the user abandons mid-rebuild, the old done branch is preserved.

On entry: read `learner-state.json`. Check `last_completed_step` and resume from the next step. If `dashboard_scratch` exists, infer rebuild-in-progress. Save state at step transitions.

## Constraints

1. **Read only, never write sibling state.** Read sibling `arch_blocks` branches for panel classification and live reads. `learner-state.json` writes are limited to `arch_blocks.dashboard` (or `dashboard_scratch` during rebuild), `pending_resume`, and `mental_models_taught`. Exception: the append-only agent-config section (Step 8) writes outside the dashboard repo.
2. **No fake data.** If a source is unavailable, the card must say `не подключено` or equivalent. No mock values, no placeholder live data.
3. **No new credentials or auth.** No new OAuth, no direct provider auth. If a panel needs an unwired source, route to the relevant POS skill or queue for `/pos-basic-vibecoding`.
4. **No dashboard code before repo bootstrap.** The repo must be initialized, pushed to GitHub, and scaffold-committed before any dashboard runtime files are written.
5. **GitHub setup required.** `pos-github-setup` must be done before repo initialization. If not done, hand off.
6. **All code inside the dedicated repo.** No file writes outside the dashboard repo, except the learner's agent-config file, `learner-state.json`, and scheduler/observability artifacts for the refresher shape (e.g. systemd units, launchd plists, log files).
7. **Design system before CSS.** Record the design-system choice before writing any CSS or visual tokens.
8. **Chrome consent explicit.** Get explicit consent before replacing the Chrome new-tab behavior.
9. **Agent-config is append-only.** Resolve target from `learner_profile.primary_agent`. If `## pos-dashboard` already exists, show a diff and get confirmation before merging. If `learner_profile.keep_agent_configs_in_sync == true`, mirror to the sibling file.
10. **No silent panel drops.** Every panel the user named must end up `connected`, `routed`, or `custom-queued`. Never silently remove a panel.
11. **No stock preset dashboard.** The artifact is built from the user's actual panel list, not a generic template.
12. **Live verification uses the same capability.** The wow check re-runs the exact same read surface the panel already uses. No weaker substitute.
13. **Present then write.** Config files, agent-config sections, and design tokens: show proposed content, get confirmation, then write.
14. **At least one live panel required.** The build cannot proceed with zero `connected` panels. If all panels are routed or custom-queued, the user must either wire a source first or add a panel backed by an already-done POS block.
15. **No source-of-truth in localStorage.** `localStorage` or `chrome.storage` may hold UI prefs, cache, or offline fallback only.
16. **Audit third-party code before installing.** Check for known vulnerabilities, research others' experience, inspect source for unexpected network calls.

## Sibling mapping seed

Use to classify user-requested panels against existing POS blocks:

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

## Mental models

Hardcoded Russian text. Teach/remind per the two-branch pattern: full teach if slug not in `mental_models_taught`, short reminder if already present.

1. **`dashboard_is_showcase`** -- Дашборд -- это витрина того, что ты уже построил. (Taught Step 2.)
2. **`data_can_be_visualized`** -- Данные можно визуализировать как угодно. (Taught Step 4.)
3. **`complexity_from_composition`** -- Один скилл -- одно умение. Сложность рождается из композиции. (Taught Step 3, only when a panel is classified as custom-queued.)

## Flow

### Step 1 -- Prerequisites and entry

Check prerequisites:
- **Hard:** `learner_profile` must exist (from `/pos-diagnostic`). If absent, stop and direct user to run `/pos-diagnostic` (или `/skill:pos-diagnostic` в Codex) first.
- **Hard:** `arch_blocks.github_setup.status == "done"` -- the dashboard must live in its own git repo pushed to GitHub (Step 6). If not done, write `pending_resume = "pos-dashboard"`, route to `/pos-github-setup` (или `/skill:pos-github-setup` в Codex), stop.
- **Soft:** `arch_blocks.basic_vibecoding.status` -- needed for the pipeline build (dashboard panels, refresher scripts are vibe-coding work). If not done, nudge: the build steps will create automation scripts, which is easier after learning the basics. If the learner opts in, write `pending_resume = "pos-dashboard"`, route to `/pos-basic-vibecoding` (или `/skill:pos-basic-vibecoding` в Codex), stop. If they decline, proceed -- the agent handles the build using whatever coding skills/superpowers are available.
- **Soft:** Any sibling skill whose block the user wants on the dashboard (`pos-calendar`, `pos-telegram`, `pos-email`, `pos-tasks`, `pos-vault`, `pos-goals`, `pos-morning-brief`, `pos-day-summary`, `pos-triage`, `pos-advisors`).
- **Resume:** If `status == "in_progress"`, read `last_completed_step`, tell the user where they left off, resume from the next step. If `dashboard_scratch` exists and `status == "done"`, the user was mid-rebuild -- offer to continue rebuild, discard it, or stop. If `pending_route_sibling` is set, check whether that sibling is now done: if done, clear `pending_route_sibling` and `pending_route_panel_id` to null, update the related panel's state from `routed` to `connected`, and resume; if still not done, re-prompt: finish sibling, keep as routed, or remove.

Write (fresh start only): `status = "in_progress"`, `started_at`, `last_completed_step = 1`.

### Step 2 -- Intro and panel intake

Deliver verbatim in Russian:

> Соберём одну страницу, которая читает твои источники и показывает главное. Ты выбираешь, откуда брать данные и по каким правилам их фильтровать.

Teach MM `dashboard_is_showcase` (or remind) — one short sentence inline, no separate paragraph.

Then list the learner's done blocks as a compact comma-separated sentence (read from arch_blocks at runtime, only status "done"). Keep the whole intake to three beats — no padding between them: (1) the done-blocks list, (2) one sentence asking what they want to see when they open the computer in the morning or switch between deep work — 3-7 items in their own words, (3) 2-3 short examples drawn from their actual done blocks. Confirm the panel list back in plain words before any classification.

Write: `panels[]` as `{id, learner_label, state: "requested"}` for each named panel, `mental_models_taught.dashboard_is_showcase`, `last_completed_step = 2`.

### Step 3 -- Panel classification

For each panel, classify against the sibling mapping seed and update the panel object:
- **Sibling done** -> set `state: "connected"`, `source: <capability path>`. Tell the user this card can be wired immediately.
- **Sibling exists but not done** -> set `state: "routed"`, `source: <sibling slug>`. Offer: (1) go finish that sibling first and return, (2) keep it as "later" and continue, (3) remove from this pass. If the user chooses to go finish: write `pending_route_sibling`, `pending_route_panel_id`, `pending_resume = "pos-dashboard"`, say farewell with `/pos-dashboard` resume command, stop.
- **No matching sibling** -> set `state: "custom-queued"`, `target_skill: "pos-basic-vibecoding"`, `brief: <one-line description>`. On the first custom-queued panel in this run, teach MM `complexity_from_composition` (or remind) and write `mental_models_taught.complexity_from_composition`. Queue for `/pos-basic-vibecoding` (или `/skill:pos-basic-vibecoding` в Codex). Tell the user honestly.

After classifying all panels, show a summary: which are connected now, which are routed, which are custom-queued. Confirm. If the user edits the list, reclassify changed panels (carry over previously classified panels that survived the edit; clean up orphaned route entries).

Enforce: at least one panel must be `connected`. If zero, the user must wire a source or add a panel backed by a done block before the build can proceed.

Write: `panels[]`, `pending_route_sibling` and `pending_route_panel_id` if applicable, `last_completed_step = 3`.

### Step 4 -- Artifact shape

Teach MM `data_can_be_visualized` (or remind). Write `mental_models_taught.data_can_be_visualized`.

Recommend `chrome-newtab` as the default — the dashboard is most useful when the learner sees it every time they open a tab. If the learner wants something different, offer two alternatives:
1. **Local HTML page** -- simpler, no browser override
2. **Local HTML with background refresh** -- data updates between opens

If the user picks `local-html-refresher`, choose refresh cadence (offer 5m / 15m / 1h / custom). Explain the observability surface: log file, "last refresh" indicator, retry policy.

Write: `artifact_shape`, `artifact_refresher_cadence` (null if not refresher), `last_completed_step = 4`.

### Step 5 -- Design system

Offer two paths:
- **Path A:** Agent researches 2-3 design systems/visual directions that fit the shape and panels, presents with one tradeoff each, user picks.
- **Path B:** User names 2-3 reference sites/apps, agent observes them at runtime and derives tokens (density, spacing, typography, card treatment, accent).

Write: `design_system_path`, `design_system_label`, `design_system_chosen_at`, `last_completed_step = 5`.

### Step 6 -- Repo bootstrap

If `pos-github-setup` is not done, hand off now. This is the hard gate. Write `pending_resume = "pos-dashboard"` before stopping.

Confirm repo name (default: `pos-dashboard`). Create a fresh local git repo, create the matching private GitHub repo, set remote, make a scaffold commit (README + .gitignore only), push. No dashboard code yet.

Write: `repo_name`, `repo_local_path`, `repo_github_url`, `artifact_root_path`, `last_completed_step = 6`.

### Step 7 -- Build the dashboard

Build the artifact inside the dedicated repo using the chosen shape and design system. Read live data only through capabilities already installed by sibling POS skills. For routed/custom-queued panels, render honest "not connected" state.

The build is vibe-coding work. Use superpowers or any installed coding skills available in the agent's runtime for building the pipeline scripts and configs. The learner watches and confirms, but the agent does the coding.

If `chrome-newtab`: get explicit consent before replacing the new tab. Build a Chrome-compatible extension.
If `local-html`: build a local HTML entry point.
If `local-html-refresher`: build local HTML plus the lightest local scheduler for this OS (cron / systemd-timer / launchd). Install the observability surface. Verify the first scheduled run lands a log record.

**Commit gate.** After the build completes and the learner confirms they can see the artifact: check whether the build produced new file types that should be ignored (e.g., `node_modules/`, `dist/`, `*.log`, build output dirs). If so, add them to the existing `.gitignore` in the repo before staging — show the proposed additions and get confirmation. Then commit all built artifacts: `git add .` followed by `git commit -m "Build dashboard artifact"`. Show the commands, get confirmation, run them. Push to GitHub (remote already exists from Step 6) and verify with `git log --oneline`. Only after the commit is verified does the flow advance to Step 8.

Write (after commit verified): `last_completed_step = 7`.

### Step 8 -- Install and verify

Open the artifact on the user's machine. Verify it actually renders with at least one live panel visible.

Resolve the agent-config target from `learner_profile.primary_agent`. Write the agent-config file inside the dashboard repo itself (e.g. `CLAUDE.md` in the repo root), not the learner's global agent-config. The dashboard config is only relevant when working inside the dashboard repo. Content: artifact location, how to open/refresh, how to add a panel later. Present first, confirm, then write. If `learner_profile.keep_agent_configs_in_sync == true`, mirror to the sibling file in the same repo.

Write: `artifact_install_verified`, `config_written = true`, `last_completed_step = 8`.

### Step 9 -- Wow verification and close

Pick the landing line by artifact shape. For `chrome-newtab` deliver verbatim:

> Открой новую вкладку. Видишь карточки? Каждая -- из скилла, который ты уже сделал, с твоими данными прямо сейчас. Это не шаблон и не демо -- это твой экран.

For `local-html` and `local-html-refresher`, adapt the line to point at the open page, preserving the core: this is yours, not a template, not a demo.

Ask the user to point to one live card and name one visible fact. Re-run the same capability behind that card to cross-check. If mismatch, fix before closing.

If rebuild: swap scratch into `arch_blocks.dashboard`, delete scratch.

Tell the user: to add panels or rebuild later, run `/pos-dashboard` (or `/skill:pos-dashboard` in Codex).

Set `status = "done"` only if `config_written` is true. If `config_written` is false, set `status = "incomplete"` and tell the user the agent-config section was not written — offer to complete it before closing.

Write: `status`, `completed_at` (if done), `wow_verified_at`, `last_completed_step = 9`.

## Rules

1. **Language and tone.** Speak Russian to the user. Use `ты`. Plain, warm, calm. All user-visible text must be Russian.
2. **Silent state.** State reads and writes are invisible. Never show JSON keys, dot-paths, or field names.
3. **Concept before jargon.** Explain the idea before naming `design system`, `Chrome extension`, `scheduler`, `panel`, or any other term.
4. **Transparency before action.** Before any build step, one short Russian sentence on what's about to happen.
5. **Present then write.** Config files, agent-config sections, design tokens: show proposed content, get confirmation, write.
6. **One mental model at a time.** Never stack two new MMs in one beat.
