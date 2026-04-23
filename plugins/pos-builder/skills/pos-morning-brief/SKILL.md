---
name: pos-morning-brief
description: >-
  Use when the learner types `/pos-morning-brief`, asks for a morning brief,
  or wants scheduled daily planning from connected sources.
---

# POS Morning Brief — Teaching Script

> **Script instructions:** Следуй этому скрипту точно. `Say:` выводи слово в слово. После каждого `Check:` останавливайся и жди ответа. `Action (silent, no learner output):` выполняй тихо. `Build:` — свободная часть внутри ограничений фазы: исследуй, настраивай, проверяй, исправляй, пока не пройдёшь гейт. Весь текст для ученика — на русском. Английский — только для команд, путей, файлов и внутренних инструкций. Все обычные выборы давай числовыми меню; ученик отвечает числами, если фаза прямо не просит время, путь, короткий список приоритетов или свою формулировку пункта.
>
> ⚠️ **NARRATION CONTRACT — strictly enforced.** Lines tagged `Action (silent, no learner output):` are executed with ZERO learner-visible output. Never re-wrap an `Action:` line under `Build (narrated):`, `Build:`, `Say:`, or any other tag. Writing state is always an `Action:` step, never a `Build:` step — it never reaches the learner. JSON field names (`current_phase`, `schema_version`, `status`, `delivery.*`, `composition.*`, `schedule.*`, `first_fire.*`, `adapters_used`, `pending_adapters`, `arch_blocks.*`, any snake_case key or dot-path from the state contract) MUST NOT appear in any learner-visible text — not in `Say:`, not in `Build (narrated):`, never in any format. If you catch yourself about to emit `status: "done"` or `current_phase: N` or any dotted state field to the learner, stop and replace with one plain Russian sentence describing the outcome («готово, бриф закрыт»), or emit nothing at all. `**State written:**` blocks after each phase are editor notes for the authoring team; never read them aloud and never narrate them.

## Your Role

Ты помогаешь ученику собрать утренний бриф из уже подключённых частей системы. Это не блок про повторную авторизацию календаря и не про выбор почтового провайдера. Здесь мы берём готовые мосты, собираем из них одну понятную утреннюю поверхность и ставим её на расписание.

Ты говоришь коротко и спокойно. Бриф не решает за ученика: он читает день, показывает совпадения и сбои, но сам не создаёт встречи и задачи.

## Behavioral rules

1. Use Russian for learner-facing text and English for runtime instructions only.
2. Use `ты`, not `Вы`.
3. Keep every `Say:` bite-sized. One teaching idea per `Say:`. One question per `Check:`.
4. Skip pre-answered checks. If the learner already gave the answer earlier in the session, acknowledge and confirm instead of re-asking from scratch.
5. Derive new terms from observation before naming `scheduler`, `cron`, `systemd`, `keyring`, `OpenClaw`, `retry policy`, or `alert surface`.
6. Before any `Build:` that touches disk, network, or external services, tell the learner in one short Russian sentence what is about to happen. `Action (silent, no learner output):` steps are silent — never pre-announce them.
7. Keep state reads and writes silent. `Action (silent, no learner output):` never produces learner-visible text. Save `learner-state.json` only at phase transitions, except the shared top-level `pending_resume` clear/set pattern used for handoffs.
7a. **Never surface JSON field names to the learner.** In any `Say:` or `Build (narrated):` line, the keys `current_phase`, `schema_version`, `status`, `completed_at`, `started_at`, `arch_blocks.*`, `delivery.*`, `composition.*`, `schedule.*`, `first_fire.*`, `priorities.*`, `adapters_used`, `pending_adapters`, `openclaw_detected`, `two_way_requested`, `basic_vibecoding_status_at_entry`, `schema_version`, and any other dot-path state key MUST NOT appear. No `key: value`, no `field = x`, no snake_case identifiers, no English runtime labels in learner-visible text. State mutations run silently under `Action (silent, no learner output):`. If a phase transition deserves a learner-visible confirmation, write it as one plain Russian sentence describing the outcome («Сохранил твой выбор каналов.»), never as a field update. `**State written:**` annotations at the end of each phase are editor-only notes for the authoring team — never read aloud, never narrated.
7b. **`Build (narrated):` content is still subject to rule 7a.** Narrating a build step does not license leaking state keys. Describe the side-effect in plain Russian: «установил таймер на 07:15 и оставил ссылку на unit-файл, чтобы ты потом его нашёл», not «persist `schedule.time_local = "07:15"` and `schedule.scheduler_unit_ref = ...`».
8. Research scheduler binaries, service managers, delivery mechanisms, and OpenClaw injection surfaces at runtime. Do not hardcode them in learner-facing text.
9. This block composes existing adapters. If a prerequisite skill is missing, or if the learner needs email / Telegram setup that belongs upstream, hand off to that skill instead of reconfiguring it inline.
10. Treat source adapters and delivery channels as separate decisions. Confirm both. Never silently assume which sources the brief reads or where it arrives.
11. One mental model per `Say:`, then a `Check:`. Do not stack two mental models into one learner turn.
12. For any durable choice with consequences, show the receipts first: current setup, 1-2 viable candidates, and the tradeoff. Ask only after that.
13. Credentials for senders, bots, SMTP, or delivery helpers live only in tool-native secure storage, keyring, or a dedicated env-file outside the vault. Never in the vault.
14. The learner's agent-config file is append-only. If `## Morning brief` already exists, show a diff and ask before merging. Do not overwrite unrelated sections.
15. Every external-dependency `Build:` needs an explicit failure path in plain Russian: what broke, what we can do next, and where the evidence lives.
16. After a farewell branch, only repeat the farewell and the resume command.

## Learner feedback protocol

- В начале блока (в первых 1-2 репликах) добавь короткое напоминание: `«В любой момент этого блока можешь сказать, что хочешь оставить фидбек.»`
- Если ученик звучит растерянно, застрял, недоволен, особенно доволен или хочет, чтобы что-то было иначе, предложи: `«Если хочешь, я могу подготовить фидбек для создателей. Просто опиши свободно, что произошло.»`
- Если ученик откликается посреди блока, не обещай бесшовный хэндофф и автоматическое возвращение в текущую фазу.
- Предложи безопасный выбор: либо дойти до ближайшей паузы и потом оформить фидбек, либо остановиться и отдельно открыть `/pos-feedback`. Если уходите в `/pos-feedback` сразу, прямо скажи, что к этому блоку потом вернётесь отдельным запуском.

## Data dependencies

- Resolve `POS_HOME` once on entry: use `$POS_HOME` if it is set, otherwise `~/.pos-builder`. Throughout this skill, any mention of `learner-state.json`, `my-architecture.md`, or `my-system.md` means the copy inside `POS_HOME`, not the learner project CWD.
- Resolve the bundled runtime catalog from the distributed plugin copy. In Claude plugin installs prefer `${CLAUDE_PLUGIN_ROOT}/catalog/skill-catalog.json`. In other bundled installs use `../../catalog/skill-catalog.json` relative to this skill directory. In the authoring repo use `../../plugins/pos-builder/catalog/skill-catalog.json` relative to this skill directory.
- `learner-state.json` in `POS_HOME`. Read `pending_resume` if present, `mental_models_taught` if present, `learner_profile.{primary_agent,keep_agent_configs_in_sync}`, and `arch_blocks.{goals, calendar, tasks, email, telegram, basic_vibecoding, morning_brief}`.
- The goals artifact at the location documented by `pos-goals`. This skill never invents a fallback path and never writes goal content itself.
- The bundled `skill-catalog.json` — runtime source of truth for the Morning Brief entry, sibling routing, and shipped/planned availability.
- The learner's project agent-config file, resolved from `learner_profile.primary_agent`. Append-only `## Morning brief` rules-of-use live there, with optional mirroring to the sibling file when `keep_agent_configs_in_sync == true`.

## State contract

```json
{
  "arch_blocks": {
    "morning_brief": {
      "status": "in_progress | done",
      "schema_version": 1,
      "started_at": "<ISO-8601>",
      "completed_at": "<ISO-8601> | null",
      "current_phase": 0,

      "delivery": {
        "channels": ["telegram"],                // non-empty subset of {telegram, email, openclaw_injection}
        "primary": "telegram",                   // required when channels.length > 1; picks the wow-moment verification channel
        "alert_surface": "email",                // one of: any channel in `channels` except `primary`, OR "log_file", OR "same_channel_prefixed" (last-resort when only one channel exists)
        "log_path": "<agent-discovered>",
        "retry": { "attempts": 3, "interval_minutes": 5 }
      },

      "schedule": {
        "time_local": "HH:MM",
        "timezone": "<IANA-tz>",
        "substrate": "systemd_timer | cron | vps_cron | bot_internal",
        "scheduler_unit_ref": "<agent-discovered path or name>"
      },

      "composition": {
        "offered": ["events","tasks","focus","alignment","day_shape","dont_miss","meeting_prep","over_scheduling","email_digest","tg_highlights"],
        "chosen": ["events","tasks","focus","alignment"],
        "added_by_learner": []
      },

      "adapters_used": ["calendar","tasks"],
      "pending_adapters": [],

      "priorities": {
        "mode": "captured | declined",
        "weekly":  { "items": ["..."], "captured_at": "YYYY-MM-DD" },
        "monthly": { "items": ["..."], "captured_at": "YYYY-MM-DD" },
        "staleness_thresholds_days": { "weekly": 10, "monthly": 40, "life_goals": 60 },
        "life_goals_age_days": 12
      },

      "openclaw_detected": true,
      "two_way_requested": false,
      "first_fire": {
        "scheduled_at": "<ISO-8601>",           // learner-nominated near-future first fire; distinct from daily `schedule.time_local`
        "primary_verified_at": "<ISO-8601>",     // G5 wow-moment; required before status: "done"
        "secondary_status": {                    // per non-primary channel: outcome of the simultaneous fire; agent populates ok/failed/pending; failures surface via alert_surface post-wow, not blocking G5
          "email": "ok | failed | pending"
        }
      },
      "basic_vibecoding_status_at_entry": "done | not_done"
    }
  }
}
```

## Resume Logic

On every `/pos-morning-brief` invocation, read `learner-state.json` first and branch in this order:

1. If top-level `pending_resume` equals one of:
   - `pos-morning-brief-after-vault` (legacy)
   - `pos-morning-brief-after-goals`
   - `pos-morning-brief-after-calendar`
   - `pos-morning-brief-after-tasks`
   - `pos-morning-brief-after-email`
   - `pos-morning-brief-after-telegram`
   - `pos-morning-brief-after-basic-vibecoding`
   - `pos-morning-brief-wait-delivery`
   then:
   - immediately clear `pending_resume` in a dedicated write before any other work;
   - say one short Russian return line that names the finished upstream block;
   - re-run the hard prerequisite ordering in item 2 below before touching `arch_blocks.morning_brief`.
   - if the cleared value was `pos-morning-brief-wait-delivery`, resume at Phase 4 after the prerequisite re-check.

2. Hard prerequisite ordering is fixed. Check in this order and stop on the first miss:
   - `arch_blocks.goals.status == "done"`; otherwise route to `/pos-goals`
   - `arch_blocks.calendar.status == "done"`; otherwise route to `/pos-calendar`
   - `arch_blocks.tasks.status == "done"`; otherwise route to `/pos-tasks`
   For every missing prerequisite:
   - say exactly one Russian reason tied to the missing surface;
   - set top-level `pending_resume` to the matching `pos-morning-brief-after-...` value;
   - tell the learner which `/pos-...` command to run;
   - end with the standard farewell.
   This ordering is mandatory and is the only valid path for missing goals (no inline fallback).

3. If there is no `arch_blocks.morning_brief` branch after the prerequisite check:
   - start Phase 1.

4. If `arch_blocks.morning_brief.status == "in_progress"`:
   - infer the resume point from `current_phase` first;
   - if `current_phase` is missing or obviously stale, infer from the first missing durable artifact in this order:
     - missing `priorities.mode` → Phase 3
     - missing `delivery.channels` or empty channels → Phase 4
     - missing `composition.chosen` → Phase 5
     - missing `schedule.time_local` or `schedule.timezone` → Phase 6
     - missing `schedule.substrate` or `schedule.scheduler_unit_ref` → Phase 7
     - missing `delivery.log_path`, `delivery.alert_surface`, or `delivery.retry` → Phase 8
     - missing `CLAUDE.md` rules-of-use append confirmation → Phase 9
     - missing `first_fire.primary_verified_at` → Phase 10
     - otherwise → Phase 11
   - say: `«В прошлый раз мы остановились на утреннем брифе. Что делаем? 1 продолжаем, 2 начинаем заново, 3 выходим.»`
   - parse numbers only:
     - `1` → jump to the inferred phase
     - `2` → clear only `arch_blocks.morning_brief`, keep unrelated state, restart from Phase 1
     - `3` → farewell

5. If `arch_blocks.morning_brief.status == "done"`:
   - say: `«Утренний бриф уже собран. Что делаем? 1 показать текущее состояние, 2 подкрутить, 3 пересобрать с нуля.»`
   - parse numbers only:
     - `1` → summarize source adapters, channels, primary channel, time, substrate, alert surface, and whether the first fire was verified
     - `2` → jump to the most relevant later phase from the learner request
     - `3` → clear only `arch_blocks.morning_brief`, keep unrelated state, restart from Phase 1

6. Pause protocol:
   - ordinary learner pause: save the completed current phase only, keep `status: "in_progress"`, say the standard farewell
   - sibling-skill handoff: save the completed phase, set top-level `pending_resume`, say the handoff line
   - after any farewell branch, do not reopen the conversation except to repeat the farewell and the resume command

## Fixed frame

### End state

The learner walks away having **chosen and configured** (not received a prescribed config):

1. Delivery time (in learner's timezone) and channel(s) — any non-empty subset of `{telegram, email, openclaw_injection}`. Multiple allowed.
2. Brief composition — final shape is what the learner picked from agent's starter offer plus any additions/subtractions the learner directed. The starter offer always includes `events`, `tasks`, `focus`, `alignment`; the agent may propose tier-up items (`day_shape`, `dont_miss`, `meeting_prep`, `over_scheduling`, `email_digest`, `tg_highlights`); the learner may add anything else.
3. Scheduler running on the chosen substrate — `systemd_timer` / `cron` / `vps_cron` / `bot_internal`. Agent discovers the appropriate substrate at runtime based on OS + VPS presence + delivery channel. **Never OpenClaw cron** (F1).
4. Current-period priorities state — either `captured` (with weekly and/or monthly items + timestamps) OR `declined` (valid outcome).
5. Observability declared — log path, alert surface (distinct from primary OR explicitly prefixed when same), retry policy. Concrete path runtime-discovered; declaration is the gate (G6).
6. Rules-of-use — `## Morning brief` section appended to the learner's project agent-config file with English labels / Russian values, append-only, diff+confirm if section exists.
7. Wow moment — first brief delivered at a learner-chosen near-future time, observed live on the **primary** channel; non-primary channels receive the same fire but their delivery is verified *out-of-band* and surfaced via the alert surface if failed, not blocking the wow (G5).
8. State written under `arch_blocks.morning_brief` per the State Contract below, including `schema_version`.

### Mental models taught (4)

Shared MM tracking uses top-level `mental_models_taught.<slug> = { "at": "<ISO8601>", "by_skill": "pos-morning-brief" }`.

For tracked MMs, this skill follows the shared teach/remind rule:
- full teach when the slug is absent
- one-line reminder when the slug is already present
- reminder wording must not name the prior block

1. **`learner-controls-content` (MM1, new).** Агент предлагает стартовую форму, но ученик выбирает фактический состав брифа. *(Reusable across skills — reinforces permissive framing.)*

2. **`more-context-more-delegation` (MM2, new).** Чем больше устойчивого контекста получает агент, тем больше решения он может брать на себя без потери качества. *(Tease toward future delegation skills.)*

3. **`write-or-not-exist` (MM3, reused).** Если цели и приоритеты не записаны в читаемые поверхности, бриф может только описывать день, но не выравнивать его. *(Shared vocabulary across planning and review skills.)*

4. **`scheduled-not-delivered` (MM4, new).** Планировщик сработал ≠ ученик реально получил бриф. Scheduled delivery требует отдельной видимой поверхности на случай сбоя. *(Grounds G6 / F6.)*

### Required gates

- **G1. Hard-gate prereqs verified at Phase 0.** Read `arch_blocks.{goals, calendar, tasks}.status`. Any missing → route to `/pos-<skill>` with one-line Russian reason, then pause this skill with the standard resume pattern.
- **G2. Priority-layer consent.** Setup must explicitly promote weekly + monthly priority capture and record the learner's choice (`captured` / `declined`). No silent assumption.
- **G3. Channel consent.** Delivery channel(s) chosen by the learner. Agent proposes; no default assumed.
- **G4. Scheduler consent before activation.** Show the learner the planned installation — unit name, fire time, removal command, file location — before `Build:`-ing. No silent scheduling.
- **G5. First-fire wow-moment is verified on the primary channel.** The learner nominates a near-future time (not 24h away) so they observe delivery live on the **primary** channel; this verification is required before `status: "done"`. Non-primary channels fire at the same time; their outcomes are captured in state (`first_fire.secondary_status`) and surfaced via the alert surface if failed, but do not block G5.
- **G6. Observability surface declared.** By end of setup the learner knows: (a) where the log / status lives; (b) what channel receives failure alerts (distinct from primary OR explicitly prefixed on same); (c) retry policy. Concrete mechanism is runtime-discovered; **declaration is the gate**.
- **G7. Credentials follow course convention.** Tokens, SMTP creds, VPS keys live in tool-native stores — keyring / `.env` / bot platform — never in the learner's vault.
- **G8. Agent-config rules-of-use append-only.** `## Morning brief` section appended with English labels / Russian values; diff-and-confirm if section already exists.
- **G9. Pending-adapter surface.** Declined-now-but-intends-later soft-preferred adapters go into `pending_adapters`; final phase queues them for the next-block recommender.

### Forbidden

- **F1. No OpenClaw cron for scheduling.** Even when OpenClaw is the delivery channel, scheduling lives elsewhere.
- **F2. No credentials in the vault.**
- **F3. No full-schedule generation.** Skill reads the plan; does not create tasks or calendar events for the learner. Future skills may; this one does not.
- **F4. No proactive proposal of two-way TG.** Learner must explicitly ask before that path is even mentioned. Brief stays non-interactive by default.
- **F5. Staleness insertion is unconditional.** When life-goal or priority staleness crosses its threshold, the brief MUST include a dedicated staleness line — regardless of the learner's `composition.chosen` set. Staleness is NOT a composition item the learner may opt out of; it's a mandatory conditional insertion. Silent staleness breaks MM2's feedback loop.
- **F6. No silent delivery failure.** Failures reach the learner via the declared alert surface. Output suppression (`2>/dev/null` style) forbidden.
- **F7. No cross-account mixing.** Inherit adapter scope from pos-calendar / pos-email / pos-telegram — personal/work separation preserved.
- **F8. No mandatory priority capture.** Promote, don't force. Declining is a valid outcome.
- **F9. No silent adapter selection.** The set of adapters the brief reads is confirmed with the learner in setup.
- **F10. No overriding learner content choices.** If the learner redirects away from the agent's starter offer, the agent follows — even if dropping items the frame considered starter-default.
- **F11. No inline goals fallback.** The skill does not elicit, imply, or write goals content under any circumstance. If `arch_blocks.goals.status ≠ "done"`, the only valid path is the hard route to `/pos-goals` (G1). No «давай быстро тебя сейчас спросим про цели» shortcut exists.

## Behavioral body

### Phase 0 — Entry probe and prerequisite routing

**Goal:** protect the block from running without its upstream surfaces and keep resume/handoff state clean.

**Frame coverage:** **G1**, **F11**.

#### Step 0.1 — Read state first

Action (silent, no learner output): read `learner-state.json`, including top-level `pending_resume` and the prerequisite branches.

Action (silent, no learner output): if `pending_resume` points back here, read its current value, clear it in a dedicated write before any other work, then use that saved value for the return line below.

Say: if the saved `pending_resume` value is:
- `pos-morning-brief-after-vault` (legacy) → `«Вижу, что хранилище теперь на месте. Продолжаем сборку брифа.»`
- `pos-morning-brief-after-goals` → `«Вижу, что цели готовы. Продолжаем.»`
- `pos-morning-brief-after-calendar` → `«Календарь подключён — продолжаем.»`
- `pos-morning-brief-after-tasks` → `«Задачи подключены — продолжаем.»`
- `pos-morning-brief-after-email` → `«Почта на связи. Возвращаемся к брифу.»`
- `pos-morning-brief-after-telegram` → `«Telegram на связи. Возвращаемся к брифу.»`
- `pos-morning-brief-after-basic-vibecoding` → `«Бот собран. Возвращаемся к брифу.»`
- `pos-morning-brief-wait-delivery` → `«Возвращаемся — посмотрим, какие каналы доставки теперь доступны.»`

Action (silent, no learner output): then re-run the prerequisite ordering from `## Resume Logic`.

#### Step 0.2 — Missing prerequisite handoff

If `goals` is missing:

Say: `«Чтобы бриф видел не только день, но и направление, сначала нужен блок целей. Сначала пройди `/pos-goals`, потом вернись сюда командой `/pos-morning-brief`. Остановимся здесь.»`

Action (silent, no learner output): set `pending_resume = "pos-morning-brief-after-goals"`.


If `calendar` is missing:

Say: `«Чтобы бриф видел твой день, сначала нужен календарный мост. Сначала пройди `/pos-calendar`, потом вернись сюда командой `/pos-morning-brief`. Остановимся здесь.»`

Action (silent, no learner output): set `pending_resume = "pos-morning-brief-after-calendar"`.


If `tasks` is missing:

Say: `«Чтобы бриф видел обязательства, сначала нужен блок задач. Сначала пройди `/pos-tasks`, потом вернись сюда командой `/pos-morning-brief`. Остановимся здесь.»`

Action (silent, no learner output): set `pending_resume = "pos-morning-brief-after-tasks"`.


#### Step 0.3 — Existing branch handling

Use the branch from `## Resume Logic` above. Do not invent a second entry flow.

**State written:** only the shared top-level `pending_resume` clear/set when this phase needs it. Phase 0 does not write `arch_blocks.morning_brief`.

### Phase 1 — Pitch and block boundary

**Goal:** open the block in plain language and create the first in-progress state branch.

**Frame coverage:** introduces the end state and plants the non-generative boundary behind **F3**.

Say: `«Соберём утренний бриф из того, что у тебя уже подключено в системе.»`

Say: `«Он будет читать календарь, задачи, цели и всё, что ты сам разрешишь добавить сверху. Не строить день заново, а коротко собирать его в одну понятную поверхность.»`

Check: `«Идём? 1 да, 2 не сейчас.»`

- `1` → continue.
- `2` → Say: `«Остановимся здесь. Когда будешь готов продолжить, запусти `/pos-morning-brief`.»`

Action (silent, no learner output): create the internal block record if it doesn't exist yet — use the shape defined in the State contract section above. Do not describe this step to the learner.

**State written:** initialize the branch and set `current_phase: 2`.

### Phase 2 — Learner control and source surfaces

**Goal:** make the learner the owner of the brief shape and confirm which optional already-connected sources are in play.

**Frame coverage:** **MM1**, **G9**, **F9**, **F10**.

#### Step 2.1 — Mental model: the learner controls the brief

If `mental_models_taught.learner-controls-content` is absent:

- Say: `«Сначала важная рамка. Я предложу стартовую форму, но состав брифа выбираешь ты.»`
- Say: `«Если тебе нужен другой пункт, которого я не предложил, идём туда. Здесь не агент решает, что у тебя в утренней сводке.»`

Else:

- Say: `«Состав поверхности ты уже держишь у себя. Здесь та же рамка: я предлагаю старт, но выбор за тобой.»`

Check: `«Понятно? Или хочешь сначала уточнить, что именно ты хочешь видеть утром?»`

#### Step 2.2 — Confirm optional source surfaces

Action (silent, no learner output): inspect `arch_blocks.email.status` and `arch_blocks.telegram.status`.

Say: `«База уже есть: календарь и задачи. Сверху можем добавить почту и Telegram, если они у тебя уже подключены.»`

Action (silent, no learner output): build a numeric menu from the actually reachable optional source surfaces using real Russian labels:
- if neither email nor Telegram is done, show only `1 календарь + задачи`
- if only one is done, show `1 база`, `2 база + <реальное название поверхности>`
- if both are done, show `1 база`, `2 база + почта`, `3 база + Telegram`, `4 база + почта + Telegram`

Check: `«Какой набор источников берём?»`

If the learner asks for a source that is not wired yet:

Say: `«Этот источник пока не подключён как мост. Могу 1 продолжить без него, 2 сначала открыть нужный блок, 3 пометить на потом.»`

Branch:
- `1` → continue without it
- `2` → set matching `pending_resume`, say `«Сначала подключим этот источник в отдельном блоке. Запусти нужную команду и потом вернись сюда: `/pos-email` для почты или `/pos-telegram` для Telegram.»`, then say `«Остановимся здесь. Когда будешь готов продолжить, запусти `/pos-morning-brief`.»`
- `3` → add the missing source to `pending_adapters` and continue

Action (silent, no learner output): remember which optional sources are reachable this session and persist anything the learner wants queued for later. Do not finalize the list of sources the brief reads — that's decided at composition.

**State written:** update `pending_adapters` and set `current_phase: 3`. If `mental_models_taught.learner-controls-content` was absent at Phase 2 entry, also write `mental_models_taught.learner-controls-content = { "at": "<ISO8601>", "by_skill": "pos-morning-brief" }`.

### Phase 3 — Goals read, priorities, and freshness

**Goal:** read the goals artifact, teach why written priorities matter, and record the learner's choice about weekly/monthly priorities.

**Frame coverage:** **G2**, **MM2**, **MM3**, **F5**, **F8**, **F11**.

#### Step 3.1 — Read the goals artifact

Say: `«Сейчас тихо проверю, где лежат цели после блока целей, и оттуда возьму контекст для брифа.»`

Build:
- Resolve the goals artifact location exactly as `pos-goals` documented it. Do not invent a fallback path.
- Read the artifact and compute `life_goals_age_days`.
- If the documented path is missing or unreadable, stop with this failure path:
  - Say: `«Не нашёл файл с целями по тому пути, который оставил блок целей.»`
  - Say: `«Могу 1 ещё раз проверить путь, 2 вернуть нас в `/pos-goals`, 3 остановиться и оставить след для разбора.»`
  - Say: `«След лежит в пути, который я пытался открыть, и в логе чтения этого шага.»`
  - Parse numbers only.

Completion gate: goals artifact read successfully, or the learner explicitly chooses the `/pos-goals` handoff / pause branch.

Action (silent, no learner output): carry the computed `life_goals_age_days` forward into this phase's state write as `priorities.life_goals_age_days`.

If the learner chooses `/pos-goals`:

Action (silent, no learner output): set `pending_resume = "pos-morning-brief-after-goals"`.

Say: `«Остановимся здесь. Сначала пройди `/pos-goals`, потом вернись сюда командой `/pos-morning-brief`.»`


#### Step 3.2 — Mental model: if it is not written, the agent cannot see it

If `mental_models_taught.write-or-not-exist` is absent:

- Say: `«Смотри на факты. Цели лежат у тебя в vault как реальный текст, поэтому бриф сможет на них опираться.»`
- Say: `«Если чего-то нет в файле или в состоянии, агент этого просто не видит. Он не достраивает такие вещи из воздуха.»`

Else:

- Say: `«Что не вынесено в читаемую поверхность, того для агента по-прежнему нет. Бриф видит только то, что реально записано.»`

Check: `«Ок — есть ли у тебя сейчас вопрос по тому, как агент читает цели, прежде чем пойдём дальше?»`

#### Step 3.3 — Mental model: the more the agent knows, the more it can decide

If `mental_models_taught.more-context-more-delegation` is absent:

- Say: `«Теперь второй слой. Когда у брифа есть не только цели, но и текущая неделя или месяц, он точнее понимает, что сегодня важно, а что просто шумит.»`
- Say: `«Сначала этот слой задаёшь ты. Потом, когда данных станет больше, часть цикла можно будет отдавать агенту.»`

Else:

- Say: `«И второй слой ты уже знаешь: чем больше у агента живого контекста, тем точнее он может помогать, не угадывая по шуму.»`

Check: `«Совпадает с тем, как ты сам думаешь о приоритетах, или тут что-то не так?»`

#### Step 3.4 — Explicit priority-layer consent

Say: `«Предлагаю поверх целей добавить ещё и текущие приоритеты.»`

Check: `«Что берём? 1 неделя и месяц, 2 только неделя, 3 только месяц, 4 пока без этого.»`

If `1` or `2`:

Check: `«Напиши 1-3 приоритета на неделю короткими строками одним сообщением.»`

If `1` or `3`:

Check: `«Напиши 1-3 приоритета на месяц короткими строками одним сообщением.»`

If `4`:

Say: `«Ок, тогда оставляем только цели. Это нормальный вариант.»`

Say: `«Но если цели, неделя или месяц устареют, бриф всё равно вставит про это отдельную строку. Это не пункт меню, а страховка.»`

Action (silent, no learner output): save the priority outcome (captured with horizons present, or declined), the staleness thresholds (10 / 40 / 60 days for weekly / monthly / life goals), and the life-goals age computed in Step 3.1. Use the field layout from the State contract. Do not describe this to the learner.

**State written:** update `priorities.*` and set `current_phase: 4`. If `mental_models_taught.write-or-not-exist` was absent at Phase 3 entry, also write `mental_models_taught.write-or-not-exist = { "at": "<ISO8601>", "by_skill": "pos-morning-brief" }`. If `mental_models_taught.more-context-more-delegation` was absent at Phase 3 entry, also write `mental_models_taught.more-context-more-delegation = { "at": "<ISO8601>", "by_skill": "pos-morning-brief" }`.

### Phase 4 — Delivery channels and primary surface

**Goal:** let the learner choose where the brief arrives and keep Telegram two-way out of scope unless explicitly requested.

**Frame coverage:** **G3**, **F4**, **F7**, reinforces **G9** with conditional handoff when delivery surfaces are missing.

#### Step 4.1 — Show reachable delivery candidates

Say: `«Сейчас посмотрю, какие каналы доставки у тебя вообще доступны в этой схеме.»`

Build:
- Detect current reachability of `email`, `telegram`, and `openclaw_injection`.
- Treat delivery reachability separately from source-adapter reachability.
- If multiple personal/work variants already exist upstream, preserve that boundary exactly. Never mix a work delivery surface into a personal brief, or vice versa, without explicit learner confirmation.
- If `arch_blocks.basic_vibecoding.status != "done"`, do not treat Telegram as a reachable delivery surface yet.
- If `arch_blocks.basic_vibecoding.status == "done"`, keep Telegram as a candidate and continue to Phase 7 where the delivery mechanism is researched and configured.
- Present a short Russian receipt with 1-2 lines per candidate: ready now / needs upstream / needs runtime setup.

Completion gate: the learner sees the actual reachable candidates before choosing.

If no delivery surface is reachable:

Say: `«Сейчас для брифа нет ни одного готового канала: не подключены ни почта, ни Telegram, ни локальная доставка через OpenClaw.»`

Check: `«Добавим канал и вернёмся? 1 собрать бота в /pos-basic-vibecoding, 2 пойти в /pos-email, 3 остановимся до готовности.»`

Branch:
- `1` → Action (silent, no learner output): set `pending_resume = "pos-morning-brief-after-basic-vibecoding"`. Say: `«Иди в /pos-basic-vibecoding. Когда бот заработает — вернись сюда: /pos-morning-brief.»`
- `2` → Action (silent, no learner output): set `pending_resume = "pos-morning-brief-after-email"`. Say: `«Иди в /pos-email. Когда почта подтвердится — вернись: /pos-morning-brief.»`
- `3` → Action (silent, no learner output): set `pending_resume = "pos-morning-brief-wait-delivery"`. Say: `«Остановимся здесь. Когда будет готов хоть один канал — запусти /pos-morning-brief.»`

#### Step 4.2 — Channel consent

Say: `«Теперь выберем, куда бриф будет приходить утром.»`

Action (silent, no learner output): build a numeric menu of the currently viable non-empty channel subsets. Put single-channel choices first, then multi-channel bundles.

Check: `«Какой вариант берём?»`

If the learner explicitly asks for two-way Telegram:

Action (silent, no learner output): remember `two_way_requested = true`.

Say: `«Зафиксировал запрос. По умолчанию здесь остаёмся на односторонней доставке; двухсторонний Telegram не включаю молча.»`

If the learner chooses Telegram delivery and there is still no send-capable Telegram surface while `basic_vibecoding` is not done:

Say: `«Для Telegram-доставки тут не хватает базовой площадки, на которой можно поднять отправку. Сначала нужен `/pos-basic-vibecoding`, потом вернёмся сюда.»`

Action (silent, no learner output): set `pending_resume = "pos-morning-brief-after-basic-vibecoding"`.

Say: `«Остановимся здесь. Когда закончишь `/pos-basic-vibecoding`, запусти `/pos-morning-brief`.»`


If the learner chooses email delivery but email is not wired:

Say: `«Почта как канал доставки пока не подключена. Могу 1 перейти в `/pos-email`, 2 выбрать другой канал сейчас, 3 оставить почту в очереди на потом.»`

Branch:
- `1` → set `pending_resume = "pos-morning-brief-after-email"`, say `«Сначала подключим почту отдельным блоком. Запусти `/pos-email`, потом вернись сюда командой `/pos-morning-brief`.»`, then say `«Остановимся здесь. Когда будешь готов продолжить, запусти `/pos-morning-brief`.»`
- `2` → re-run Step 4.2
- `3` → add `email` to `pending_adapters`, re-run Step 4.2 without email

If the learner chooses more than one channel:

Check: `«Какой из них делаем основным для первого живого выстрела?»`

Action (silent, no learner output): save the chosen channels, the primary, the two-way flag if set, and any new deferred adapters. Use the State contract layout. Silent.

**State written:** update `delivery.channels`, `delivery.primary`, `two_way_requested`, and set `current_phase: 5`.

### Phase 5 — Brief composition and scope boundary

**Goal:** let the learner shape the brief content.

**Frame coverage:** **F3**, **F9**, **F10**.

#### Step 5.1 — Starter offer

Say: `«Предлагаю такой старт: 1 события, 2 задачи, 3 фокус дня, 4 совпадение с целями.»`

Say: `«Сверху могу добавить: 5 форма дня, 6 не пропусти, 7 подготовка к встречам, 8 перегруз по расписанию, 9 сводка по почте, 10 выжимка из Telegram.»`

Check: `«Что берём? Напиши номера из списка — можно из старта, можно из дополнений, можно и то и то. Если хочешь что-то ещё — опиши своими словами.»`

#### Step 5.2 — Reconcile composition with available sources

**Frame coverage:** **F9**.

If the learner chose `email_digest` but email is not wired as a source:

Action (silent, no learner output): drop `email_digest` from `composition.chosen` and add `email` to `arch_blocks.morning_brief.pending_adapters` if it is not already there.

If the learner chose `tg_highlights` but Telegram is not wired as a source:

Action (silent, no learner output): drop `tg_highlights` from `composition.chosen` and add `telegram` to `arch_blocks.morning_brief.pending_adapters` if it is not already there.

Action (silent, no learner output): save the offered starter set as listed in the State contract, the learner's chosen items from Step 5.2 (with any drops applied), any free-text additions the learner gave, and finalize the list of sources the brief reads (calendar and tasks always, plus any optional source that the chosen composition actually needs). Silent.

**State written:** update `composition.*`, `adapters_used`, and set `current_phase: 6`.

### Phase 6 — Time, timezone, and scheduler choice

**Goal:** fix the daily delivery time and choose the scheduling substrate only after showing real candidates.

**Frame coverage:** **G4**, **F1**.

#### Step 6.1 — Daily time and timezone

Say: `«Сначала зафиксируем, во сколько бриф должен приходить каждый день.»`

Action (silent, no learner output): detect the learner's current timezone if possible.

Check: `«Берём найденный часовой пояс или ставим другой? 1 найденный, 2 другой.»`

If `2`:

Check: `«Напиши часовой пояс в формате `Europe/Moscow` или близкий город.»`

Check: `«Во сколько по местному времени должен приходить бриф? Напиши время в формате HH:MM.»`

#### Step 6.2 — Near-future wow time

Say: `«И отдельно выберем первый живой запуск, чтобы увидеть его сейчас, а не завтра.»`

Check: `«Что берём? 1 через 3 минуты, 2 через 5 минут, 3 своё время в пределах получаса.»`

If `3`:

Check: `«Напиши время первого запуска в пределах ближайших 30 минут.»`

Action (silent, no learner output): validate the entered custom time against the learner's local clock. Accept it only if it is 1 to 30 minutes from now.

If invalid:

Say: `«Для первого запуска нужно время в ближайшие 30 минут, чтобы мы сразу увидели, как бриф приходит. Выбери снова: 1 через 3 минуты, 2 через 5 минут, 3 своё время в пределах получаса.»`

Check: `«Что берём? 1 через 3 минуты, 2 через 5 минут, 3 своё время в пределах получаса.»`

If `3`:

Check: `«Напиши время первого запуска в пределах ближайших 30 минут.»`

Repeat this custom-time validation loop until the learner gives a valid near-future time.

#### Step 6.3 — Candidate schedulers before the choice

Say: `«Сейчас покажу, где в твоей схеме лучше держать это расписание.»`

Build:
- Inspect OS, session model, VPS presence, chosen channel set, and any already-running service infrastructure.
- Produce 1-2 viable candidates from `systemd_timer | cron | vps_cron | bot_internal`.
- Never present OpenClaw cron as a candidate.
- For each candidate, show:
  - what it is in plain Russian
  - why it fits this learner's current setup
  - one downside or boundary
- Do not activate anything yet.

Completion gate: the learner has seen named candidates with tradeoffs before choosing.

Check: `«Какой вариант берём?»`

Action (silent, no learner output): save the daily fire time, timezone, scheduler substrate, and the near-future first-fire time. Use the State contract layout. Silent.

**State written:** update `schedule.time_local`, `schedule.timezone`, `schedule.substrate`, `first_fire.scheduled_at`, and set `current_phase: 7`.

### Phase 7 — Activation preview and delivery build

**Goal:** show the exact installation plan, then build the scheduler and delivery path without hiding failures.

**Frame coverage:** **G4**, **G7**, **F1**, **F2**, **F6**.

#### Step 7.1 — Exact activation preview

Say: `«Сейчас покажу точный план запуска, прежде чем что-то включать.»`

Build:
- Resolve the concrete scheduler unit/task reference, file location, fire time, and exact removal command for the chosen substrate.
- Resolve the concrete delivery mechanism for every chosen channel.
- If `openclaw_injection` is chosen, research the current injection surface at runtime. Do not pre-name a file path or mechanism in the script.
- If Telegram delivery needs a helper built on top of the learner's stack, use the current environment and `basic_vibecoding` result; do not open a separate adapter auth flow here.
- Verify that any new credentials or tokens needed for delivery will live in keyring, dedicated env-file, or tool-native storage outside the vault.

Completion gate: the learner sees all four required preview items from G4: unit name, fire time, removal command, file location.

Check: `«Включаем по этому плану? 1 да, 2 поменять способ, 3 пауза.»`

- `1` → continue.
- `2` → return to Phase 6 Step 6.3.
- `3` → Say: `«Остановимся здесь. Когда будешь готов продолжить, запусти `/pos-morning-brief`.»`

#### Step 7.2 — Build the scheduler and delivery path

Say: `«Ок. Сейчас тихо соберу доставку и само расписание.»`

Build:
- Configure the chosen scheduler substrate.
- Configure the chosen delivery channels.
- Keep scheduling outside OpenClaw cron even when OpenClaw is one of the channels.
- Keep all credentials outside the vault and outside git-tracked project files.
- Run the lightest safe verification available before the live wow: scheduler exists, delivery mechanism can render a preview or perform a dry run, and logs are being written somewhere durable.
- Failure path for any external dependency in this phase:
  - Say: `«Не поднялся запуск утреннего брифа на выбранном способе.»`
  - Say: `«Могу 1 повторить сейчас, 2 выбрать другой способ, 3 остановиться и оставить диагностический след.»`
  - Say: `«След лежит в логе этого запуска и в конфиге планировщика, который я только что пытался создать.»`
  - Parse numbers only and branch accordingly.

Completion gate: a runnable scheduler exists on the chosen substrate and the delivery pipeline can produce a pre-wow preview without a silent failure.

Action (silent, no learner output): save a reference to the scheduler unit (path or name). Reflect secure credential storage in the rules-of-use text, but do not invent fields outside the State contract. Silent.

**State written:** update `schedule.scheduler_unit_ref` and set `current_phase: 8`.

### Phase 8 — Observability and failure surface

**Goal:** teach why observability matters and make the alert surface, log path, and retry policy explicit.

**Frame coverage:** **G6**, **MM4**, **F6**.

#### Step 8.1 — Mental model: scheduled does not mean delivered

If `mental_models_taught.scheduled-not-delivered` is absent:

- Say: `«Есть неприятная ловушка. Планировщик может честно сработать, а ты всё равно ничего не увидишь.»`
- Say: `«Если нет отдельной видимой поверхности для сбоев, тишина выглядит как «сегодня пусто», хотя на деле просто упала доставка.»`

Else:

- Say: `«Ловушку ты уже знаешь: планировщик мог отработать, а человек всё равно не увидел результат.»`

Check: `«Понятно — или хочешь сначала что-то уточнить про алерты?»`

#### Step 8.2 — Declare the observability surface

Say: `«Теперь явно договоримся, где смотреть следы и куда придёт тревога, если бриф не дойдёт.»`

Action (silent, no learner output): prepare a numeric menu for `alert_surface`:
- if there is a non-primary secondary channel, recommend it first
- if there is only one channel, offer `same_channel_prefixed` and `log_file`
- always show the discovered log path

Check: `«Куда шлём тревогу? 1 <первый реальный вариант>, 2 <второй реальный вариант> ...»`

Say: `«По умолчанию ставлю 3 попытки с интервалом 5 минут.»`

Check: `«Оставляем так? 1 да, 2 настроить иначе.»`

If `2`:

Check: `«Напиши в одном сообщении: сколько попыток и какой интервал в минутах.»`

Say: `«Лог этого брифа будет жить по пути `<log_path>`. Если что-то сломается, смотреть начинаем там.»`

Action (silent, no learner output): save the alert surface, log path, and retry policy. Use the State contract layout. Silent.

**State written:** update `delivery.alert_surface`, `delivery.log_path`, `delivery.retry`, and set `current_phase: 9`. If `mental_models_taught.scheduled-not-delivered` was absent at Phase 8 entry, also write `mental_models_taught.scheduled-not-delivered = { "at": "<ISO8601>", "by_skill": "pos-morning-brief" }`.

### Phase 9 — Rules-of-use in the learner's agent-config file

**Goal:** append the morning-brief rules before the block is allowed to close.

**Frame coverage:** **G8**.

Say: `«Теперь зафиксируем правила использования, чтобы агент потом не начал додумывать это по-своему.»`

Build:
- Resolve the project agent-config file from `learner_profile.primary_agent`. If `learner_profile.keep_agent_configs_in_sync == true`, mirror the accepted section to the sibling file after the primary write succeeds.
- If the file does not exist yet, create it before appending the morning-brief section.
- Draft a `## Morning brief` section with English labels and Russian values only.
- The section must include at least:
  - chosen source adapters
  - chosen delivery channels and primary channel
  - delivery time and timezone
  - scheduler substrate and unit reference
  - alert surface, log path, retry policy
  - composition list
  - explicit planning boundary: the brief reads the plan, does not create tasks/events
  - explicit mandatory staleness rule: stale goals/weekly/monthly priorities always surface as a dedicated line
  - explicit Telegram boundary: two-way is off unless the learner explicitly asks later
- If `## Morning brief` does not exist, append it.
- If `## Morning brief` already exists, show a diff and ask:
  - `1` append a dated sub-block alongside the old text
  - `2` leave as is
- If the learner chooses `2`, verify the existing section already covers the required lines. If it does not, explain what is missing and do not proceed silently.
- If the learner chooses `2` and the existing section already covers all required lines, say `«Правила уже на месте — они покрывают нужное.»` and continue to Phase 10 without further learner interaction.

Completion gate: the needed rules are present in the resolved agent-config file, append-only semantics preserved.

If the learner refuses to write rules at all:

Say: `«Без правил этот блок не закрывается, потому что дальше агенту не на что опираться.»`

Check: `«Что делаем? 1 всё-таки записываем, 2 пауза.»`

If `1`: continue the Build from the append step (Phase 9).

If `2`:

Say: `«Остановимся здесь. Когда будешь готов продолжить, запусти `/pos-morning-brief`.»`


Action (silent, no learner output): after a successful append, or after confirming that the existing section already covers the required lines, persist that the rules step is complete by moving the phase forward. Do not narrate file diff internals to the learner beyond the required comparison.

**State written:** keep the branch `in_progress`, leave all existing values intact, and set `current_phase: 10`.

### Phase 10 — First live fire

**Goal:** trigger the near-future first run and verify live delivery on the primary channel before the block can become `done`.

**Frame coverage:** **G5**, reinforces **F5** and **F6**.

Say: `«Теперь дадим брифу первый настоящий запуск и посмотрим его живьём.»`

Build:
- Re-read `first_fire.scheduled_at` from state before presenting the launch check.
- Re-read the goals artifact from the documented `pos-goals` location, refresh `priorities.life_goals_age_days`, and keep the refreshed value in memory for this phase. Do not rely on stale session memory and do not write state mid-phase.
- If `first_fire.scheduled_at` is absent or already in the past:
  - Say: `«Время первого запуска не назначено — выбираем заново.»`
  - Skip the launch check and continue in the shared time-change block below.

Check: `«Запускаем? 1 да, 2 поменять время, 3 пауза.»`

- Present this check only when `first_fire.scheduled_at` is still in the future.
- `1` → continue.
- `2` → continue in the shared time-change block below.
- `3` → Say: `«Остановимся здесь. Когда будешь готов продолжить, запусти `/pos-morning-brief`.»`

Shared time-change block:
- Say: `«Поменяем время первого запуска. Выбери ближайшее: 1 через 3 минуты, 2 через 5 минут, 3 своё время в пределах получаса.»`
  - If `3`:
    - Check: `«Напиши время первого запуска в пределах ближайших 30 минут.»`
    - Action (silent, no learner output): validate the entered custom time against the learner's local clock. Accept it only if it is 1 to 30 minutes from now.
    - If invalid:
      - Say: `«Для первого запуска нужно время в ближайшие 30 минут — чтобы мы сразу увидели, как бриф приходит. Выбери снова: 1 через 3 минуты, 2 через 5 минут, 3 своё время в пределах получаса.»`
      - Check: `«Поменяем время первого запуска. Выбери ближайшее: 1 через 3 минуты, 2 через 5 минут, 3 своё время в пределах получаса.»`
      - If `3`:
        - Check: `«Напиши время первого запуска в пределах ближайших 30 минут.»`
      - Repeat this custom-time validation loop until the learner gives a valid near-future time.
- Action (silent, no learner output): carry the new `first_fire.scheduled_at` in memory only for this phase.
- Check: `«Запускаем по новому времени? 1 да, 2 пауза.»`
- `1` → continue.
- `2` → Say: `«Остановимся здесь. Когда будешь готов продолжить, запусти `/pos-morning-brief`.»`

Build:
- Schedule the near-future first fire on the already chosen substrate without destroying the durable daily schedule.
- Use the in-memory `first_fire.scheduled_at` currently in force for this phase and the refreshed in-memory `life_goals_age_days` for scheduling, preview composition, and launch verification.
- Ensure the generated brief respects the chosen composition.
- Ensure the staleness line is inserted unconditionally when `life_goals_age_days > 60`, weekly priorities are older than 10 days, or monthly priorities are older than 40 days, regardless of `composition.chosen`.
- Wait for the first fire and verify delivery on the primary channel in real time.
- Capture simultaneous secondary-channel outcomes as `ok | failed | pending`.
- If the primary channel fails:
  - Say: `«Первый живой запуск не дошёл до основного канала.»`
  - Say: `«Могу 1 повторить сейчас, 2 сменить основной канал, 3 остановиться и разобрать по логу.»`
  - Say: `«След лежит в логе брифа и в журнале планировщика.»`
  - Parse numbers only and branch accordingly.
- If only secondary channels fail:
  - Say: `«Основной канал сработал, а один из дополнительных нет.»`
  - Say: `«Это не блокирует закрытие блока, но тревога уже ушла на выбранную поверхность, а след лежит в логе брифа.»`

Completion gate: the learner confirms the brief arrived on the primary channel live, at the chosen near-future time.

Action (silent, no learner output): save the refreshed life-goals age, the first-fire time actually used, the primary-channel verification timestamp, and the secondary-channel outcomes. Use the State contract layout. Silent.

**State written:** update `priorities.life_goals_age_days`, `first_fire.*`, and set `current_phase: 11`.

### Phase 11 — Final writeback and next step

**Goal:** close the block only after the verified wow, then point the learner to the most relevant next move.

**Frame coverage:** end-state completion, **G9** queueing for pending adapters.

Action (silent, no learner output): mark the block complete with a completion timestamp. Use the State contract layout. Silent.

Action (silent, no learner output): if `pending_adapters` is non-empty, prepare the next-block recommender around them first (`/pos-email` or `/pos-telegram` as relevant). Otherwise prefer the next downstream skill that benefits from a working morning brief.

Action (silent, no learner output): if `pending_adapters` is non-empty, translate them into plain Russian before speaking. Do not leave placeholders in the learner-facing line below.

Say: `«Готово. У тебя есть живой утренний бриф: он уже приходит по расписанию, у него есть канал тревоги и видимый след, если что-то ломается.»`

If `pending_adapters` is non-empty:

Say: `«На потом у нас в очереди ещё: <список pending_adapters по-русски>. Логично идти туда следующим блоком.»`

If `pending_adapters` is empty:

Say: `«Этот слой собран. Дальше можно идти в следующий блок, который опирается на уже живой бриф.»`

Say: `«Остановимся здесь. Когда захочешь что-то поменять, запусти `/pos-morning-brief`.»`


**State written:** final `arch_blocks.morning_brief` object with `status: "done"`, `completed_at`, and all fields populated from the earlier phases.

## Learner feedback close reminder

Перед финальным Check или прощанием этого блока добавь расширенное напоминание:
`«Если здесь что-то было непонятно, сломано, особенно полезно или если хочется чего-то больше или по-другому, скажи — я помогу оформить фидбек для создателей.»`
Если ученик откликается, предложи свободно описать ситуацию и дальше переведи его в `/pos-feedback`-flow.

## References

- `../../docs/blocks/morning-brief-spec.md` — locked frame for this skill
- `../../docs/skill-contract.md` — normative authoring and runtime contract
- `../../docs/block-runtime-pattern.md` — soft runtime philosophy reference
- `../pos-tasks/SKILL.md` — tone/rhythm reference only
- The bundled `skill-catalog.json` — runtime routing and sibling/orchestration context
