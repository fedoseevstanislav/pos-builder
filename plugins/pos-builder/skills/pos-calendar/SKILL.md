---
name: pos-calendar
description: >-
  Use when the learner types `/pos-calendar`, asks to connect a calendar, or
  needs calendar read and write access for later brief and planning skills.
---

Read this file on entry. Constraints and end state define the boundaries; flow guides the session.

**Role:** Teach a non-developer to connect their calendar to their agent so they can ask about their schedule and manage events by voice or text.

## End state

When done, the learner has:

1. A calendar adapter (MCP server or CLI tool) installed and authorized, with credentials stored outside the Obsidian vault and not in plaintext
2. At least one calendar connected, with a confirmed first read (weekly agenda shown to the learner)
3. If write access granted: a daily JSON backup script running and a deterministic action log (tool-layer, not agent-side) configured — both set up before write access is enabled
4. A standalone `calendar-rules` skill created and registered in the agent runtime, containing behavioral rules for calendar operations
5. A live moment: the learner asks a real calendar question and gets an answer; if write-enabled, they move or create an event by voice/text
6. `my-architecture.md` updated with a Calendar section (present-confirm-write)
7. `learner-state.json` updated (see State section)

## State

Fields this skill reads and writes in `learner-state.json`. `last_completed_step` = last finished step number; resume starts at the NEXT step. Field names in step write-instructions use short names (e.g., `last_completed_step`); these refer to their full path under `arch_blocks.calendar.*` as declared below.

- `arch_blocks.calendar.status` (string: in_progress|done|incomplete) — written Step 1, Step 9
- `arch_blocks.calendar.last_completed_step` (number) — written at each step milestone
- `arch_blocks.calendar.completed_at` (ISO8601|null) — written Step 9
- `arch_blocks.calendar.provider` (string: google|outlook|icloud|yandex|other) — written Step 2
- `arch_blocks.calendar.is_work_calendar` (boolean) — written Step 2
- `arch_blocks.calendar.connection_method` (string: mcp|gogcli|other-cli) — written Step 3
- `arch_blocks.calendar.calendars_connected` (array of {id, name, role}) — written Step 5
- `arch_blocks.calendar.scopes_granted` (string[]: ["read"] or ["read","write"] or ["read","write","delete"]) — written Step 4, updated Step 7
- `arch_blocks.calendar.credentials_path` (string|null) — written Step 4; directory or keyring service name
- `arch_blocks.calendar.credentials_location` (string: keyring|env-file|tool-native) — written Step 4; needed for rollback (revocation procedure differs by storage type)
- `arch_blocks.calendar.backup_enabled` (boolean) — written Step 6
- `arch_blocks.calendar.backup_location` (string|null) — written Step 6
- `arch_blocks.calendar.action_log_path` (string|null) — written Step 6; path to the append-only action log for agent calendar writes
- `arch_blocks.calendar.rules_skill_installed` (boolean) — written Step 8
- `arch_blocks.calendar.rules_skill_path` (string|null) — written Step 8; path to the standalone calendar-rules skill file
- `arch_blocks.calendar.live_moment_done` (boolean) — written Step 9; true after the learner completes the live calendar question

Mental model slugs this skill teaches (written to `mental_models_taught.*`): `adapter` (Step 2), `scopes-risk` (Step 2), `personal-vs-corporate` (Step 2 if work calendar detected), `time-awareness` (Step 5, first-read confirmation), `voice-agent-over-ui` (Step 9, live moment).

On entry: read `learner-state.json`. If `last_completed_step` exists, resume from the next step. If `status == "done"`, show a summary and offer to exit or start over. If `status == "incomplete"`, check which end-state items are missing and resume at the relevant step. Save state as the flow progresses — write relevant fields immediately at each milestone.

Read-only dependencies: `learner_profile` (prereq check).

## Constraints

1. **Credentials never in Obsidian.** Tokens, OAuth key files, and session files must be stored outside the Obsidian vault (`inventory.obsidian_path`) and outside any git-tracked directory. Verify location before proceeding past auth.
2. **Credentials never in plaintext.** Use system keyring, encrypted env file, or the tool's own secure storage. If the tool stores in plaintext by default, reconfigure or switch tools.
3. **No silent scope escalation.** Read access is the default. Write requires explicit consent after backup is in place. Delete requires separate explicit consent. Never request write/delete scopes without passing through the write-access gate (Step 7).
4. **Backup before write.** Daily JSON backup and deterministic action log must be confirmed working before write access is enabled. No exceptions. The action log must be produced by the tool layer (wrapper script or logging proxy), not by the agent.
5. **Work calendar warning.** If the calendar belongs to an employer, warn that the agent uses a cloud LLM and the learner should verify company policy. Warning only — the learner decides.
6. **Never mix accounts.** Personal and work tokens are separate authorizations. If the learner switches from work to personal mid-flow, verify the email addresses are different.
7. **Present-confirm-write.** Config files, rules sections, architecture doc updates — show proposed content, get confirmation, then write. Never write first.
8. **Rules are standalone.** Calendar rules must be a standalone skill file. Never write them into CLAUDE.md or AGENTS.md.
9. **Proof before tool choice.** When the learner must choose between connection methods (MCP servers, CLI tools), research current options at runtime first. No tool names from training data without live verification of maintenance status, last release, and token storage method.
10. **Scope strings from live sources only.** OAuth scope URLs come from the tool's output or current documentation, never from memory. If unavailable, tell the learner they will see the exact scopes on the consent screen.
11. **Recurring events always show the full rule.** Before any change to a recurring event, show the series rule and ask: this instance, all future, or all.
12. **Delete always confirmed.** Every deletion requires explicit per-event confirmation, enforced by the rules skill.
13. **Audit third-party code before installing.** Before installing any MCP server or CLI tool: check for known vulnerabilities, research others' experience, verify it only communicates with declared endpoints.

## Flow

### Step 1 — Prerequisites and entry

Check prerequisites:
- **Hard:** `learner_profile` must exist (populated by `/pos-diagnostic`). If absent, tell the learner to run `/pos-diagnostic` first. Stop.
- **Resume:** If `status == "in_progress"`, read `last_completed_step`, tell the learner where they left off, pick up from the next step. If `status == "incomplete"`, check which end-state items are missing and resume at the relevant step.

Read diagnostic hints: which calendar provider is in `inventory`, whether multiple providers are mentioned, personal vs work signals, whether the learner has a VPS, `stt_status`.

Write (fresh start only): `status = "in_progress"`, `last_completed_step = 1`.

### Step 2 — Intro and provider

Deliver verbatim in Russian:

> В этом блоке мы дадим агенту доступ к твоему календарю (или календарям), что позволит ему быть в курсе твоих планов и помогать тебе управлять твоим временем.

> В конце сможешь спросить «что у меня на этой неделе?» и получить ответ. Встречи тоже можно будет создавать и переносить словами, не открывая календарь.

If multiple providers found: «У тебя несколько календарей в разных сервисах. Начнём с одного, а остальные подключим по тому же шаблону отдельными запусками. С какого удобнее стартовать?»

Teach the adapter concept (if `mental_models_taught.adapter` is absent):

> Мы будем использовать адаптер (готовый или создадим новый) — это небольшая программа, которая позволяет агенту дотянуться до твоих данных, когда они лежат не в файлах на компьютере, а в облаке, не используя интерфейс. Календарь, почта, мессенджер, список задач — все это будет использовать специализированные адаптеры. Допустим, ты просишь агента «найди свободный час на этой неделе и поставь созвон с Аней». Агент через адаптер авторизуется в твой календарь, читает его, видит свободное окно, создаёт событие.

Confirm provider. If known from diagnostic, confirm in one sentence. If not, present a numbered menu (1 Google, 2 Outlook, 3 iCloud, 4 Yandex, 5 other, 6 no calendar yet). If no calendar yet: briefly compare popular options based on learner's ecosystem and recommend.

Classify personal vs work. If work calendar and the learner is not self-employed:

> Твоя Personal OS — личная система. Агент, которым ты пользуешься прямо сейчас, работает на основе LLM в облаке. Подключая рабочий календарь, убедись что ты не нарушишь правила и политики своей компании по доступу к данным.

Offer to continue with work or switch to personal. If switching, verify personal email differs from work email.

Write: `provider`, `is_work_calendar`, `last_completed_step = 2`. Write to `mental_models_taught.<slug>` only for each slug that was newly taught in this step (absent before this step began).

### Step 3 — Choose connection method

Research current connection methods for the learner's provider at runtime. For Google, research current options including MCP servers and gogcli, with an "alternatives" option. For other providers, research MCP servers, CLI tools, CalDAV options. Present 1-3 options with honest tradeoffs. Note: CLI tools are generally more context-efficient than MCP servers (run a command, get output, done) — mention this when comparing approaches.

Confirm the learner's choice. Record `connection_method`.

Write: `connection_method`, `last_completed_step = 3`.

### Step 4 — Install and authorize (read-only)

Before starting credential setup, check if the learner has time — on "not now", save state and tell them to resume with `/pos-calendar`.

Teach scopes concept (if `mental_models_taught.scopes-risk` is absent):

> Про уровни доступа. Чтение — агент видит твои встречи, но ничего не меняет; запись — может создавать и переносить события; удаление — может стирать. Если агент ошибётся на чтении, ты этого даже не заметишь. На записи — в календаре появится лишняя встреча. На удалении — встреча исчезнет. Принцип: даём минимум нужного и знаем, что именно сейчас разрешили.

Tell the learner: we start with read-only. Write is a separate decision later, after backup is in place.

> Сейчас подключим только чтение. Запись добавим отдельным шагом, когда убедимся, что чтение работает и у нас есть подстраховка на случай ошибки.

Walk through tool installation, credential storage setup, and read-only authorization. By the end of this step, the following must be true:

- The chosen adapter tool is installed and runnable
- The learner's OS is accounted for (if not already known); incompatible combinations have a fallback
- An OAuth client or equivalent auth entity exists for the learner's provider
- Token storage is configured: system keyring, encrypted env file, or tool-native — verified outside the Obsidian vault
- Scopes are shown to the learner before the consent screen
- OAuth (or equivalent auth) has completed successfully
- The token has landed in the agreed storage location
- Credentials are outside the Obsidian vault and outside any git-tracked directory

Pre-warn about "app not verified" if the learner created their own OAuth client.

Write: `scopes_granted = ["read"]`, `credentials_path`, `credentials_location`, `last_completed_step = 4`.

### Step 5 — Choose calendars and first read

Fetch the calendar list from the provider. Present calendars with labels (primary, shared, subscription). The learner chooses which to connect. Shared/team calendars only if explicitly named.

Read events for the next seven days. Display the schedule by day. Ask the learner to confirm everything looks correct. Debug if needed (missing events, time offset, API error).

> Теперь твой агент (я) видит твое расписание. Поздравляю!


Write: `calendars_connected`. If write path chosen: `last_completed_step = 5`. If read-only chosen: `last_completed_step = 7` (so resume skips Steps 6-7 and lands on Step 8). Write `mental_models_taught.time-awareness` if newly taught.

### Step 6 — Backup and action log

Set up two safety nets before write access:
- Daily JSON backup of the calendar. Choose host (VPS if available, laptop otherwise). Propose backup path outside Obsidian, confirm with learner. Build and schedule the backup script. Run once immediately.
- Deterministic action log for every calendar write (what, when, which event). The log must be produced by the tool layer, not by the agent remembering to write entries. For CLI tools: build a wrapper script that appends to the JSONL log on every call. For MCP servers: build a thin logging proxy that intercepts tool calls, logs them, and forwards to the real server. The agent must not be the one writing log entries — the tool writes them automatically. Propose log path, confirm, build.

Verify the backup file exists on disk, its size is non-zero, and it contains calendar data. Verify the action log works by running a test read through the logging layer and confirming an entry appears. Do not proceed to write access until both are verified.

Write: `backup_enabled`, `backup_location`, `action_log_path`, `last_completed_step = 6`.

### Step 7 — Write access

Ask the learner to choose write scope:

> Удаление — самый высокий риск. Ошибка агента или неправильно понятый им запрос может стереть данные, и без бэкапа их пришлось бы восстанавливать по памяти.

> Чтобы агент знал, что ему можно и нельзя, мы на следующем шаге создадим для него отдельный файл с правилами — скилл. Скилл — это инструкция, которую агент читает когда работает с конкретной темой. В отличие от CLAUDE.md, где лежат глобальные правила для всего, скилл применяется только к своей области — в нашем случае к календарю. Там и зафиксируем: агент переспросит тебя перед каждым удалением, пока ты сам не снимешь это требование.

> На техническом уровне "удаление" разрешено в правах доступа. Но в правилах агента (в скилле) зафиксируем, что каждое удаление агент обязан подтвердить с тобой — без этого он не удалит ничего.

Numbered menu: 1 создавать и редактировать, 2 создавать, редактировать и удалять.

For Google: explain that the OAuth scope technically covers deletion too, but deletion is blocked by a course rule in the agent-config file.

Re-authorize with new scopes. Show the new scope set before the consent screen. After success, propose a test event (title "POS test — можно удалить", today +1h, 15min). Create only after explicit confirmation.

Write: `scopes_granted` (updated), `last_completed_step = 7`.

### Step 8 — Rules-of-use skill

Teach what a skill is and why rules belong in a skill rather than CLAUDE.md: a skill is a self-contained instruction file the agent loads into context on demand — when not in use, it takes up zero context. CLAUDE.md is always loaded and always occupies context; a skill is loaded only when relevant and free when not in use. Calendar rules are specific to calendar work, so they belong in a skill — the agent will read them when doing calendar work and they won't cost context the rest of the time.

Create a standalone calendar-rules skill containing:
- **Reference info:** adapter tool, connection method, credentials location, backup path (if applicable)
- **Behavioral rules:** default calendar, read policy, create/edit/delete confirmation requirements, recurring event handling, shared calendar policy, personal/work separation — based on what the learner actually granted

If the learner declined write: note which actions are not currently granted. If a calendar-rules file already exists at the target location, show a diff and ask before overwriting.

Present the proposed skill content and target location. Get confirmation. Write and register in the agent runtime (e.g., `~/.claude/skills/` for Claude Code). Then add a one-line reference to the learner's runtime doc (CLAUDE.md for Claude Code, AGENTS.md for Codex — check `learner_profile.primary_agent`) telling the agent to load the calendar-rules skill before any calendar operation — skills in the registry are only discovered on demand, so without this pointer the agent won't know to load the rules automatically. Present the proposed addition, get confirmation, write. Verify discoverability.

If the learner refuses to write rules entirely: research the correct revocation procedure for the learner's credential storage type before executing rollback. Roll back write access to read-only, set `status = "incomplete"`, skip to Step 9.

After writing the skill and the CLAUDE.md reference, tell the learner to restart Claude Code so the new skill becomes available. The agent won't see newly created skills until the session restarts. Save state before prompting the restart. On re-entry, `/pos-calendar` resumes at Step 9.

Write: `rules_skill_installed = true`, `rules_skill_path`, `last_completed_step = 8`.

### Step 9 — Live moment, architecture doc, and wrap-up

**Live moment.** Prompt the learner to ask a real calendar question by voice (if STT set up) or text:

> Быстрее спросить агента про встречу или неделю, особенно голосом, чем делать это самостоятельно через интерфейс календаря. Попробуй прямо сейчас. Спроси меня голосом «что у меня на этой неделе?», если голосовой ввод настроен. Или напиши текстом, если пока без голосового ввода.

Re-read the calendar (fresh, not cached). Answer the question. If write-enabled and rules skill is installed: offer the learner to move or create an event. Before executing, show the relevant rule from the rules skill and get confirmation.

**Architecture doc update.** Add a Calendar section to `my-architecture.md`: provider, connection method, calendars connected, scopes, credentials location, backup info (if applicable), rules skill path, setup date. Present-confirm-write.

**State and handoff.** Determine status: `done` if all critical end-state items are met — rules skill is installed (rules_skill_installed), architecture doc is updated, adapter is authorized (credentials_path set), first read is done (calendars_connected non-empty), live moment completed (live_moment_done), and if write mode — backup configured (backup_enabled, backup_location set) and action log exists (action_log_path set). Otherwise `incomplete`.

**Security handoff.** If status is done and `arch_blocks.security.status` is not `done`, recommend `/pos-security` as the priority next step — the learner just connected an inbound surface that handles untrusted external content. If security is already done, recommend the next block from `skill-catalog.json` based on diagnostic route.

> Если хочешь оставить обратную связь создателям по поводу этого блока, скажи и я помогу тебе это сделать.

Write: `status`, `completed_at` (if done), `last_completed_step = 9`, `live_moment_done = true` (after live moment completed). Write `mental_models_taught.voice-agent-over-ui` if newly taught.

## Rules

1. **Language and tone.** Speak Russian to the learner. Use `ты`. Plain, warm, calm — like explaining to a friend. All user-visible text must be Russian. English only for commands, paths, config keys.
2. **Silent state.** State reads and writes are invisible to the learner. Never show JSON field names, keys, or dot-paths in user-facing text.
3. **Concept before jargon.** Before naming a technical term (OAuth, keyring, MCP, CalDAV, cron), explain the concept in one plain sentence first.
4. **Transparency before action.** Before any build step, one short Russian sentence about what will happen and why.
5. **Clear choices.** When presenting choices, explain each option and recommend if there is a clear best one. Use numbered menus for 3+ options.
6. **Skip pre-answered questions.** If the learner already stated their provider, OS, or preference earlier in this session, do not re-ask. Confirm and proceed.
7. **One mental model at a time.** Never stack two new concepts in one learner-visible beat.
8. **Pre-warn predictable anxiety.** If the next step will show an "app not verified" warning, a broader-than-expected scope list, or a terminal command after promising minimal terminal work — warn one sentence before it appears.
9. **After a pause farewell, stop.** Do not continue the skill flow; if the learner asks a new question, direct them to re-invoke `/pos-calendar`.
