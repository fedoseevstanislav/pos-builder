---
name: pos-tasks
description: >-
  Use when the learner types `/pos-tasks`, asks to connect a task tracker, or
  needs one task system as the agent's primary working tracker.
---

Read this file on entry. Constraints and end state define the boundaries; flow guides the session.

**Role:** Teach a non-developer to connect their task trackers to the agent, choose a main agent tracker, set scopes, create per-tracker operational skills, and verify a live write with visible attribution.

## End state

When done, the learner has:

1. All declared tracker(s) connected to the agent -- or explicit deferral per tracker with a recorded reason
2. One tracker designated as the main agent's tracker (`is_agent_memory_home: true`) -- chosen consciously from the learner's real trackers
3. Per tracker: provider chosen (no silent default), connection method chosen (`mcp` / `cli` / `vibe-coded-adapter` / `native-gh`), tracker class (`personal` / `work` / `shared`), scope starting at `read`, credential location (`keyring` / `env-file` / `tool-native`) never inside Obsidian vault
4. Per write-enabled tracker: baseline snapshot (or explicit `"none"` with reason), attribution marker documented, one live write verified with visible attribution
5. Today-view shown for the main agent's tracker as natural "it works" confirmation
6. Thin routing pointer appended to agent-config file (which trackers exist, what for, which is primary -- 2-4 lines max)
7. Per write-enabled tracker: operational skill created via superpowers skill-authoring (detailed rules, scopes, attribution, project layout)
8. `my-architecture.md` updated with a Tasks section
9. `learner-state.json` `arch_blocks.tasks` populated

## State

Fields this skill reads and writes in `learner-state.json`. `last_completed_step` = last finished step; resume starts at the NEXT step. Save state as the flow progresses -- write relevant fields immediately at each milestone, not batched to the end.

### Top-level resume fields

- `pending_resume` (string|null) -- `"pos-tasks-after-github-setup"` | `"pos-tasks-after-vibecoding"` | `"pos-tasks-write-path"` | `"pos-tasks-loop"` | null
- `pending_resume_tracker_slug` (string|null) -- `declared_trackers[].slug` for loop/write-path pause

### `arch_blocks.tasks.*`

- `status` (string: deferred_no_tracker|partial|done) -- written Step 1, Step 7
- `last_completed_step` (number) -- written at each step milestone
- `completed_at` (ISO8601|null) -- written Step 7 when done
- `no_tracker_reason` (string|null) -- written Step 1 if no tracker
- `declared_trackers[]` (array of objects) -- written Step 2; each entry: `slug`, `label`, `resolution_status` (pending|wired|deferred), `is_agent_memory_home` (boolean)
- `connected[]` (array of objects) -- written Step 3 per tracker; each entry:
  - `tracker_slug`, `provider`, `provider_label` (for other), `connection_method`, `mcp_server_name` (string|null), `cli_tool` (string|null), `adapter_skill_name` (string|null)
  - `tracker_class` (personal|work|shared), `projects_connected[]` (id, label, role)
  - `scopes_granted[]` (read, create, edit, comment, link, label, close, archive), `hard_delete_enabled` (boolean, default false)
  - `credentials_location`, `credentials_path`, `infosec_consent_granted` (boolean)
  - `attribution_prefix` (string), `attribution_style` (issue-title-prefix|comment-sig|label-suffix|other)
  - `backup` (object: enabled, status, reason, format, location, cadence, last_run)
  - `action_log_path` (string|null) -- written Step 5; path to append-only JSONL action log
  - `is_agent_memory_home` (boolean)
  - `live_write_verified` (boolean) -- written Step 6
- `tracker_skills_created[]` (array of objects) -- written Step 5; each entry: `tracker_slug`, `skill_name`, `skill_path`
- `routing_pointer_status` (string: appended|deferred_read_only|null) -- written Step 5
- `routing_pointer_appended_to` (object: claude_md_path, agents_md_path, section_name) -- written Step 5
- `wow_moment_today_view_shown` (boolean) -- written Step 3
- `routing_pointer_skipped` (boolean) -- written Step 5
- `my_architecture_skipped` (boolean) -- written Step 7
- `gaps[]` (array of string or {slug, reason, deferred_at_phase}) -- written as gaps arise

### Read-only dependencies

- `learner_profile` (prereq check, repo context, primary_agent, keep_agent_configs_in_sync)
- `inventory.task_trackers` (starting hint, not final truth)
- `arch_blocks.github_setup.status`
- `arch_blocks.basic_vibecoding.status` (soft dep -- skill authoring requires superpowers)
- `arch_blocks.obsidian_vault.path` (credential-location safety check; missing vault state does not block)

On entry: read `learner-state.json`. Check `last_completed_step` and resume from the next step. Save state as the flow progresses.

## Mental models

None. This block does not teach new mental models. The learner already knows `adapter`, `agent-as-assistant`, `personal-vs-corporate`, and `scopes-risk` from earlier skills.

## Constraints

1. **No write before skill.** No live write without a per-tracker operational skill created and registered.
2. **No write without baseline.** No write without a baseline snapshot or recorded `"none"` gap with reason per tracker.
3. **No write without attribution.** Every agent-created trace (titles, comments, labels) must carry the agreed attribution marker.
4. **No silent close/archive/edit.** Per-operation confirmation named in the tracker skill.
5. **Hard delete blocked.** Unless explicit opt-in in the tracker skill.
6. **Scope starts read.** Write unlocks per-tracker only after explicit learner choice.
7. **Work-tracker infosec warning.** Before any auth flow on a work-class tracker, warn the learner to verify their company policy allows this connection.
8. **Show scopes before auth.** Display requested permissions on-screen before any auth/token flow.
9. **Per-project opt-in.** For multi-project trackers, never connect all projects/workspaces silently.
10. **Account separation.** Personal token never used for work, vice versa. Confirm which account when ambiguous.
11. **Credentials safe.** Never inside Obsidian vault. Never plaintext (repo, chat output, anywhere visible).
12. **Agent-config stays thin.** The `## Tasks` section in CLAUDE.md/AGENTS.md is a routing pointer only -- which trackers exist, what each is for, which is primary. Detailed operational rules live in per-tracker skills. Keep the pointer as short as possible (aim for 2-5 lines depending on tracker count).
13. **Agent issues = real work only.** Agent opens issues only for actual work, not scratch state or memory.
14. **Tracker is a work surface.** Never pitch a scratch issue or separate memory issue.
15. **Research live.** Provider-specific commands, packages, adapter names, auth flows -- research at runtime. Do not hardcode.
16. **Connection method = explicit choice.** No silent default.
17. **Credential location = explicit choice.** Offer system keyring, env-file outside repo, or tool-native storage. Verify outside vault path.
18. **One main tracker.** Exactly one declared tracker has `is_agent_memory_home: true`.
19. **Audit third-party code before installing.** Before installing any MCP server, CLI tool, or library: check for known security vulnerabilities (CVEs, advisories, disclosed incidents), research others' experience with it, and inspect the source code to verify it only communicates with its declared endpoints -- no telemetry, no third-party data exfiltration, no unexpected network calls. Report findings to the learner as part of the tool choice.
20. **Deterministic action log.** Every task tracker write (creates, edits, comments, labels, closes, archives) must be logged to an append-only JSONL action log. The log must be produced by the tool layer (wrapper script or logging proxy), not by the agent. For CLI tools: build a wrapper that appends to the log on every call. For MCP servers: build a thin logging proxy that intercepts tool calls, logs them, and forwards to the real server. Read-only access before the log is set up is allowed.
21. **Superpowers soft dependency.** Creating per-tracker skills requires the superpowers skill-authoring tooling. If `arch_blocks.basic_vibecoding.status != "done"`, offer to hand off to `/pos-basic-vibecoding` first, or proceed if the learner confirms they have superpowers installed another way.

## Flow

### Step 1 -- Prerequisites and entry

Check prerequisites:
- **Hard:** `learner_profile` must exist (populated by `/pos-diagnostic`). If absent, tell the user to run `/pos-diagnostic` first. Stop.
- **Resume:** Handle resume branches:
  - `pending_resume == "pos-tasks-after-github-setup"` → GitHub setup is now done; continue from Step 2.
  - `pending_resume == "pos-tasks-after-vibecoding"` → Vibecoding setup is now done; continue from Step 5.
  - `pending_resume == "pos-tasks-write-path"` → Validate `pending_resume_tracker_slug` against `declared_trackers[].slug`. If the slug is not found or the tracker was deferred, clear `pending_resume` and fall through to status-based re-entry. Otherwise resume at Step 6 for that tracker. If `routing_pointer_status` is unset, resume at Step 5 first.
  - `pending_resume == "pos-tasks-loop"` → Validate `pending_resume_tracker_slug` against `declared_trackers[].slug`. If the slug is not found or the tracker was deferred, clear `pending_resume` and fall through to status-based re-entry. Otherwise resume the unresolved phase for that tracker.
  - Unknown `pending_resume` → clear and fall through.
- **Status-based re-entry:**
  - `deferred_no_tracker` → Ask if the learner wants to try now or come back later. If later, exit gracefully.
  - `done` → Inform the learner this block is already complete. Offer: show current state, adjust something, or start from scratch. If "start from scratch": reset all `arch_blocks.tasks` fields to initial values (preserving `started_at`) and jump to Step 1 intro.
  - Other `arch_blocks.tasks` exists → Offer: continue from where they left off, start over, or exit.

**Intro (fresh start):**

Explain in plain language what this block does: give the agent access to the learner's task tracker so it can see tasks, remind about them, help prioritize, and sometimes even execute them. Convey this warmly and concisely. Ask if ready to begin.

If the learner declines, do not write state; exit gracefully with an invitation to return.

Write: `status = "partial"`, `last_completed_step = 1`.

### Step 2 -- Confirm tracker list and choose main tracker

Read `inventory.task_trackers`. Parse into `declared_trackers`. Filter to actual task trackers only -- set aside note-taking apps, calendars, wikis with a brief mention they belong to a later block.

If no trackers declared: ask what the learner uses. If none at all, ask what is blocking them, set `status = "deferred_no_tracker"` with reason, jump to Step 7.

Show the filtered list. Ask if it looks right or needs adjusting.

**Choose main agent's tracker:** Build a numbered menu. GitHub Issues always first if it is in the declared trackers list and github-setup is done or learner profile suggests dev work. Recommend GitHub Issues in one sentence -- the place where the agent will track its own tasks and operational work. Reserve a final option for a custom choice. If the learner picks custom: ask for the tracker name, add it to `declared_trackers` with `resolution_status = "pending"`, then continue the main-tracker choice with the updated list. The learner picks.

Mark exactly one entry as `is_agent_memory_home: true`.

**GitHub routing gate:** If main tracker is GitHub Issues and `arch_blocks.github_setup.status != "done"`, explain the handoff, write `pending_resume = "pos-tasks-after-github-setup"` and `last_completed_step = 2`, invoke `/pos-github-setup`. Stop here.

Write: `declared_trackers`, `last_completed_step = 2`.

### Step 3 -- Per-tracker connect loop

Run once per declared tracker. For each tracker:

**Announce** which tracker comes next by its human label.

**Provider confirm.** If the name maps directly to a known provider, confirm in one sentence. If ambiguous, present numbered matches plus a custom option.

**Personal or work.** Three-option choice: personal, work, shared/team. If work: deliver Constraint 7 warning and ask whether to proceed. If declined: defer the tracker (`resolution_status = "deferred"`). If this was the main tracker (`is_agent_memory_home: true`): prompt the learner to pick a new main from remaining non-deferred trackers. If no trackers remain, set `status = "deferred_no_tracker"` with reason and jump to Step 7. Otherwise loop to next.

**Connection method.** Consult `tracker-landscape.md` for the shortlist, then research the provider's current adapter options at runtime. Audit per Constraint 19 before recommending -- report findings to the learner as part of presenting options. Present only real, currently working candidates. One pro/con line each. Recommend if one is clearly better. For GitHub Issues with existing github-setup, note native `gh` reuses what is configured. For vibe-coded adapter: frame it as the agent writing a small custom adapter. Ask to pick by number. Include a final option for a detailed comparison.

**Auth, credentials, and first read.** Research auth path at runtime. Walk through step by step. Show requested permissions before auth (Constraint 8). For multi-project providers: show projects as a numbered list, ask which to connect (Constraint 9). Offer credential storage options (Constraint 17). Verify outside vault. Never print secrets into chat. Perform first read only after explicit confirmation. On failure: name what happened, offer retry, change method, or pause (save `pending_resume = "pos-tasks-loop"` with tracker slug).

**Today-view (main tracker only).** If this is the main tracker: generate a ranked view of today's tasks (urgent, waiting, deferrable). Mark `wow_moment_today_view_shown = true`.

**Loop transition.** Mark tracker as wired. If pending trackers remain, ask if continue or take a break. On pause: save `pending_resume = "pos-tasks-loop"` with tracker slug.

Write per tracker: `connected[]` entry, `scopes_granted = ["read"]`, `resolution_status = "wired"`, `last_completed_step = 3`.

### Step 4 -- Read-to-write choice

Run per wired tracker. Tell the learner they can stay read-only or open actions (create, edit, comment, link, label, close, archive). Delete is a separate conversation. Ask naturally.

If read-only: move to next tracker. If write: set write scopes, `hard_delete_enabled = false`.

Write: `scopes_granted`, `hard_delete_enabled`, `last_completed_step = 4`.

### Step 5 -- Routing pointer, per-tracker skills, baseline, and attribution

**Superpowers gate.** Check `arch_blocks.basic_vibecoding.status`. If not `"done"`, explain that creating per-tracker operational skills requires the superpowers skill-authoring tooling (installed during the vibecoding block). Offer: hand off to `/pos-basic-vibecoding` (write `pending_resume = "pos-tasks-after-vibecoding"`), or confirm superpowers are already available by another path. If handoff: stop here.

**Context-efficiency teaching moment.** Before writing anything, explain the two-layer architecture to the learner:
- The agent-config file (CLAUDE.md / AGENTS.md) is loaded every single session, even when the agent is doing something completely unrelated to tasks. Anything written there costs context tokens every time. So it should contain only a minimal routing pointer: which trackers exist, what each is for, which is primary.
- Detailed operational rules (what scopes are granted, how attribution works, what needs confirmation, project layout, priority labels) belong in a separate skill file that loads only when the agent actually works with that tracker. This keeps the main config light and the detailed rules always available on demand.

This is a practical demonstration of context efficiency -- the same principle applies to everything the learner writes into CLAUDE.md going forward.

**Baseline snapshot (per write-enabled tracker).** Before live write, offer a snapshot as a safety net. If declined, record a one-phrase reason as a conscious-choice gap. If accepted, research provider-appropriate export and create it. On failure: offer retry or skip.

Write: `backup.enabled`, `backup.status`, `backup.reason`.

**Action log setup (per write-enabled tracker, Constraint 20).** Configure a tool-layer logging wrapper or proxy that appends every tracker access to a timestamped JSONL log. For CLI tools: build a wrapper script. For MCP servers: build a thin logging proxy that intercepts tool calls, logs them, and forwards to the real server. Verify the log works by running a test read through the logging layer and confirming an entry appears. Do not proceed to live write until verified.

Write: `action_log_path`.

**Attribution pattern (per write-enabled tracker).** Research 1-3 attribution patterns natural for this tracker. Present options plus a custom choice. Learner picks.

Write: `attribution_prefix`, `attribution_style`.

**Write-path approval gate.** Before creating skills or writing the pointer, confirm with the learner that they want to proceed with the write path. Explain: we are about to create operational skills and write a routing pointer to the agent config. If the learner declines: downgrade all scopes to `["read"]`, set `routing_pointer_skipped = true`, set `routing_pointer_status = "deferred_read_only"`, skip to Step 7.

**Per-tracker operational skill (per wired tracker with write enabled).** Using superpowers skill-authoring, create a skill for this tracker. The skill should contain:
- Tracker identity: provider, connection method, projects connected
- Scopes: what the agent is allowed to do (read, create, edit, comment, link, label, close, archive)
- Hard delete rule (blocked unless explicitly opted in)
- Attribution: prefix and style for this tracker
- Confirmation rules: which operations require per-action confirmation from the learner
- Work-tracker caution (if applicable)
- Project layout: which projects/boards/spaces are connected and their roles
- Priority system: how priorities are represented in this tracker
- Action log: path to the JSONL log for this tracker

Name the skill `pos-tracker-<slug>` (e.g., `pos-tracker-github-issues`, `pos-tracker-todoist`). Register it in the skill directory. Show the draft to the learner, get confirmation, then write.

Write: `tracker_skills_created[]` entry.

**Routing pointer (once, after all tracker skills are created).** Draft a thin `## Tasks` section for the agent-config file:
- Which tracker is the agent's primary task home
- One line per connected tracker: name + what it's for
- A note to invoke the per-tracker skill for detailed rules

Keep the pointer as short as possible (Constraint 12). Resolve target file from `learner_profile.primary_agent`. Show draft. Offer: approve or edit.

If approved: append to target file. If sync enabled, mirror to sibling.

After writing the pointer, tell the learner to restart the agent so the new section becomes active. Save state before prompting the restart. On re-entry, `/pos-tasks` resumes at Step 6.

Write: `routing_pointer_status`, `routing_pointer_appended_to`, `last_completed_step = 5`.

### Step 6 -- Live write verification

Run per write-enabled tracker. Skip if no write-enabled trackers remain.

Ask if the learner is ready for the first live write.

If declined: save `pending_resume = "pos-tasks-write-path"` with tracker slug. Stop here -- do not proceed to Step 7.

If agreed: explain this is one small real action to see the attribution in practice. Offer options: comment on a task, create a small sub-task, add a label, or a custom action. Show dry-run preview (target, action, payload, attribution marker). Confirm before executing. Read result back, verify marker is visible and action log recorded this write. If marker absent, re-run attribution menu and repeat. On failure: offer retry or change action.

Write: `connected[].live_write_verified = true` for this tracker.

Write: `last_completed_step = 6`.

### Step 7 -- Wrap up

**Architecture doc update.** Compose a `## Tasks` section for `my-architecture.md` with: main tracker name, connected trackers with classes, scopes summary, credentials location summary, pointer to tracker skills, date configured.

If no `## Tasks` section exists: show draft, ask to append. If it exists: show diff, ask. If declined: mark `my_architecture_skipped = true`.

**State cleanup.** Determine `status`:
- `deferred_no_tracker` if set in Step 1
- `done` if every declared tracker is wired or deferred, routing pointer resolved, tracker skills created for write-enabled trackers, today-view shown for main tracker, and for every write-enabled tracker: action log exists (action_log_path set) AND live write verified (`live_write_verified == true`)
- `partial` otherwise

If done: set `completed_at`. Clear `pending_resume`.

**Summary and next block.** Summarize what was accomplished: which trackers are connected, which is main, whether write is enabled, which skills were created. If deferred trackers exist: name them, say they can be connected by running `/pos-tasks` again. If gap notes: mention loose ends.

**Security handoff.** If status is done and `arch_blocks.security.status` is not `done`, recommend `/pos-security` as the priority next step — the learner just connected an inbound surface that handles untrusted external content. If security is already done, recommend the next block from the skill catalog. Do not recommend `/pos-tasks` itself. Tell the learner which block comes next. Ask if they want to jump or take a break. On pause: everything is saved.

Write: `status`, `completed_at` (if done), `gaps[]`, `last_completed_step = 7`.

## Rules

1. **Language and tone.** Speak Russian to the user. Use informal address. Plain, simple language. Warm and calm, like explaining to a friend. All user-visible text must be Russian -- no English leaking in status updates, fix-up messages, or wrap-up.
2. **Silent state.** State reads and writes are invisible to the user. Never show JSON field names, keys, or dot-paths in user-facing text.
3. **Concept before jargon.** Before naming a technical term, explain the concept in plain language first.
4. **Transparency before action.** Before any build step, tell the user in one short sentence what is about to happen and why.
5. **Clear choices.** When presenting choices, explain each option and recommend the best one if there is one. Use numbered menus only for real multi-option routing; for binary confirmations, ask naturally.
6. **Present → confirm → write.** When creating or modifying config files, skill files, or architecture docs: show the proposed content to the user first, get confirmation, then write.
7. **All text runtime-generated.** No hardcoded user-facing phrases in this skill file. The agent generates all learner-facing text at runtime based on the directives above.
