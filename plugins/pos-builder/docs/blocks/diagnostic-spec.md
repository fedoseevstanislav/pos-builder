# Diagnostic Skill Spec

**Skill command:** `/pos`
**Type:** Teaching script (CLAUDE.md) for LLM — hybrid format
**Language:** Russian (learner-facing), English (internal LLM instructions)
**Duration:** 40-60 min, single skill with incremental state saves
**Prerequisites:** Claude Code or Codex installed and running

---

## Glossary

- **POS** — Personal Operating System. The learner's individual set of data, automations, and agents that improve their life.
- **POS-builder** — The interactive course that guides the learner through building their POS.
- **Inventory** — The process of mapping what tools, systems, and habits the learner already has.
- **Use case** — One of 24 defined life scenarios the course can address (e.g., morning brief, TG inbox, Obsidian vault). Canonical list: `scripts/data/use-cases/state.json`.
- **Architecture block** — A technical infrastructure component the learner builds (e.g., VPS, adapter, capability). Canonical list: `scripts/data/blocks/state.json`.
- **State file** — `learner-state.json`, written at phase transitions and on final completion, used for resume on re-launch.

---

## 1. Purpose

The diagnostic skill is the entry point to POS-builder. It accomplishes three things:

1. **Explains what POS is** and gives the learner a mental model of what they're building and how (through a coding agent, in natural language).
2. **Maps the learner's current state** — what tools, systems, habits, and automations they already have, matched against the 24 use cases.
3. **Recommends a personalized path** — which blocks to build first, in what order, based on the learner's pains and what's quick to set up.

The diagnostic does NOT build anything (except optionally triggering `/pos-stt-setup`). Its output is information: state files and recommendations.

---

## 2. Output Artifacts

The diagnostic produces three files in the learner's project directory:

### 2.1 `my-architecture.md`

Structured, machine-readable. Other skills read this file to understand what the learner already has.

Sections:
- **Current state** — for each relevant tool/system: name, category (notes, calendar, messenger, tasks, health, automation, STT, etc.), status (exists / not present / partial).
- **Recommended route** — ordered list of next blocks with rationale.
- **Full use case mapping** — all 24 use cases with status (has solution / no solution / partial) and priority (1-N or "not prioritized").

This file is updated by every subsequent skill as the learner builds their system. The diagnostic creates the first version.

### 2.2 `my-system.md`

Human-readable report. Written in conversational Russian. The learner can re-read it a month later and understand the context: what they had, what was missing, what was recommended and why.

### 2.3 `learner-state.json`

Internal state for the skill. Enables resume after interruption. Contains:
- `current_phase` — which phase the learner is in (1-5)
- `last_completed_step` — granular position within the phase
- `answers` — all learner responses captured so far
- `inventory` — structured mapping of what the learner has (built incrementally during the interview, written at phase transitions)
- `inventory.vibecoding` — nested readiness capture for downstream vibe-coding: `has_github_account`, `github_handle`, `gh_cli_installed`, `ever_vibe_coded`, `git_familiarity` (all default `null` until answered)
- `inventory.task_trackers` — raw free-text capture of the learner's answer to the Phase 3 tasks-and-projects zone (checklist ID 8). Classification / tier / primary designation is deferred to `/pos-tasks`. Defaults to `null` until answered.
- `use_case_coverage` — which of the 24 use cases are covered, partially covered, or uncovered
- `pains` — identified pain points
- `stt_status` — installed / skipped / already_has (+ tool name)
- `coding_agent_experience` — yes / no
- `timestamps` — per-answer timestamps

**Write frequency:** At phase transitions and on final completion. During a phase, the LLM keeps working answers in memory and writes them once when crossing into the next phase.

---

## 3. Skill Flow

Five phases, executed in order. Phases 1-2 are scripted (Say/Check/Action). Phases 3-4 are free-form interview with internal checklists. Phase 5 is scripted output.

```
Phase 1: Intro (scripted)
  → Phase 2: STT proposal (scripted, optional exit to /pos-stt-setup)
    → Phase 3: Inventory (free-form interview)
      → Phase 4: Pains & priorities (free-form → structured output)
        → Phase 5: Results (scripted output)
```

### 3.1 Phase 1 — Intro

**Type:** Scripted (Say/Check)

**Step 1.1 — What is POS:**

Say: «Добро пожаловать в POS-builder, интерактивный курс по созданию твоей собственной персональной операционной системы. Personal OS — это твои данные + твои автоматизации + твои агенты = помощь в том, чтобы жить свою жизнь наилучшим образом.

Ты наверняка уже пользуешься большим количеством разных инструментов и программ, которые помогают тебе решать различные задачи. Это могут быть календари, email клиенты, программы для ведения заметок и отслеживания целей.
Проблема в том, что эти инструменты созданы для массового использования, и не один из них не заточен под тебя. 
Как массмаркет одежда vs индивидуальный пошив. Только раньше абсолютному большинству людей разработка инструментов индивидуально под них была не доступна. 

Мы привыкли к этому. 
Но ситуация изменилась.

Прямо сейчас уникальное время, где каждый может создать и развивать набор умных инструментов под самого себя. 
Тебе больше не нужны навыки программирования, чтобы делать это. Тебе достаточно сформировать ментальные модели того, как работают автоматизации и LLM агенты, и обрести успешный опыт самостоятельного создания.
Этот интерактивный курс поможет тебе в этом.

Мы начнем с интервью, на основании которого я сформирую рекомендации, как тебе двигаться дальше. Готов (а)?»

Check: «Понятно? Есть вопросы про концепцию?»

If the learner asks questions — answer, then continue.

**Step 1.2 — Coding agent mental model (conditional):**

Say: «Скажи, ты раньше работал с Claude Code, Codex или похожими кодинговыми агентами?»

Check: Wait for answer.

If **no** → Say: «Вот это окно, в котором ты сейчас находишься — это и есть твой главный инструмент. Ты пишешь (или говоришь) что хочешь на обычном языке, а агент строит: пишет код, настраивает сервисы, подключает API. Тебе не нужно уметь программировать. Весь курс — это разговор в этом окне, из которого рождается твоя система. Агент сам будет читать, создавать и менять файлы когда это потребуется, и сам будет разрабатывать тебе программы, предварительно обсудив что тебе нужно и исправляя ошибки. Агент может сам найти всю необходимую информацию на твоем компьютере или в интернете. Привыкай к тому, что это окно - это все что тебе нужно»

If **yes** → skip the explanation, proceed to Step 1.3.

**Step 1.3 — Vibe-coding readiness:**

Say: «Хочу быстро понять, какая база для вайб-кодинга у тебя уже есть.»

Check 1 — GitHub account:
- Ask a binary question: есть ли уже GitHub-аккаунт.
- If **yes**, immediately ask a second short question for the handle and store it.
- If **no**, keep the handle empty (`null`) and continue.

Check 2 — `gh` CLI:
- First explain in one plain-talk sentence what `gh` is: the official terminal client for GitHub.
- Then ask whether it is already installed.
- If the learner is unsure, accept that and store `gh_cli_installed = null`.
- Do not attempt any install here.

Check 3 — Ever vibe-coded:
- Ask whether the learner has already tried vibe-coding, with an optional one-line example.
- If `coding_agent_experience == false` from Step 1.2, skip this Check entirely and auto-store `ever_vibe_coded = false`.

Check 4 — Git familiarity:
- Use a numeric menu: `1 совсем новое, 2 немного понимаю, 3 свободно`.
- Accept the digit or equivalent words silently.
- Map the stored value to `none | some | fluent`.

State handling:
- Hold all four readiness answers in memory during Phase 1.
- Write them once at the Phase 1 → Phase 2 transition together with `coding_agent_experience`.

**LLM instruction:** If the learner goes off-topic at any point during scripted phases, answer briefly, then return to the script. Do not ignore the learner's question, but do not let them derail the flow.

### 3.2 Phase 2 — STT Proposal

**Type:** Scripted (Say/Check) with possible exit to separate skill

**Step 2.1 — Propose STT:**

Say: «Дальше у нас будет разговорная часть — я буду задавать вопросы про твою жизнь, инструменты, привычки. Это удобнее делать голосом — ты просто говоришь, а текст появляется сам. LLM вроде меня хорошо работают с таким неДля этого нужна программа для голосового ввода. Хочешь поставить сейчас? Это займёт до 10 минут.»

Check: Wait for answer. Three branches:

| Learner says | Action |
|---|---|
| "Yes, let's set it up" | Save state (stt_status: in_progress). Hand off to `/pos-stt-setup`. On return, resume from Phase 3. |
| "I already have STT" | Ask which tool. Save state (stt_status: already_has, stt_tool: <name>). Write to `my-architecture.md`: STT section. Proceed to Phase 3. |
| "No" / "Not now" | Save state (stt_status: skipped). Proceed to Phase 3. |

### 3.3 Phase 3 — Inventory

**Type:** Free-form interview with internal checklist

**Step 3.1 — Open question:**

Say: «Теперь давай разберёмся, что у тебя уже есть. Расскажи, какими инструментами ты пользуешься для организации своей жизни. Например: планируешь ли день в календаре? Записываешь куда-то мысли или заметки? Как общаешься с близкими — мессенджеры, звонки? Может, хранишь где-то медицинские данные или отслеживаешь здоровье? Просто расскажи как есть, не надо ничего структурировать.»

Check: Wait for an extended answer. Do not rush — the learner may talk for a while, especially with STT.

**Step 3.2 — Gap-filling (internal LLM instruction, not visible to learner):**

The LLM holds an internal checklist: all 24 use cases grouped by life zones. After the learner's open answer, the LLM identifies which zones were not covered and asks follow-up questions.

**Life zones and situation-trigger questions:**

The spec will contain, for each zone, concrete life situations the LLM uses to formulate questions. The learner never sees use case names — only recognizable life scenarios. Examples:

| Zone | Situation triggers for questions | Maps to use cases |
|---|---|---|
| Information capture | «Ты находишь интересную статью, слышишь про классную книгу или фильм, тебе приходит важная мысль, ты понимаешь что нужно что-то сделать — что ты делаешь в каждом из этих случаев? Как ты сохраняешь эту информацию?» | Inbox, Obsidian vault, Personal digest |
| Planning & reflection | «Как у тебя начинается утро — планируешь ли ты день? В конце дня или недели оглядываешься на то, что произошло? Ставишь ли цели и отслеживаешь их?» | Morning brief, Calendar, Goal tracking |
| Communication | «Когда тебе приходит важное сообщение в чат, а ты не можешь им заняться прямо сейчас — что с ним происходит? Есть ли у тебя система, или оно просто остаётся в чате?» | TG inbox, Personal TG agent |
| Knowledge & learning | «Когда ты исследуешь какую-то тему — как собираешь и организуешь найденное? Есть ли у тебя система для сохранения полезных знаний надолго?» | Research/wiki, YouTube knowledge |
| Health | «У тебя есть медицинские данные в электронном виде? Отслеживаешь какие-то показатели здоровья?» | Health management |
| Automation | «Есть ли у тебя уже какие-то автоматизации — что-то, что работает само? Где они живут, как устроены?» | Foundation, Agent memory, Orchestration |
| Content & presentations | «Ты создаёшь контент — посты, презентации, документы? Как устроен этот процесс?» | Content creation, Presentation tools |

**Note:** The table above covers 7 zones with example triggers. The full mapping of all 24 use cases to situation triggers is authored as part of the teaching script (the CLAUDE.md file), not this spec. This spec defines the pattern: every use case maps to at least one concrete life situation. The teaching script will contain the exhaustive list.

**LLM rules for gap-filling:**
- Ask through life situations, never through use case names or technical terms.
- No hard limit on follow-up questions — cover the checklist. But if the learner signals fatigue ("enough questions", "let's move on"), respect it immediately: transition to Phase 4 with what you have, or offer the menu.
- If the learner says "I don't do that" or "I don't have that" — record the gap, don't push.
- After each answer, internally map it to specific use cases and blocks. Write to state.
- Focus on personal life, not work.

**Step 3.3 — Early exit:**

At ANY point during the diagnostic, if the learner says something like "just show me what you have" or "I'm advanced, give me the menu" — the LLM may decide to skip remaining inventory questions and jump to presenting the full use case menu (Phase 4, menu path). The LLM should gently try to return to the script once, but if the learner insists, respect it. This is an intelligent fallback, not a failure.

### 3.4 Phase 4 — Pains & Priorities

**Type:** Free-form interview transitioning to structured output

**Step 4.1 — Pain question:**

Say: «Теперь давай поймём, где у тебя больше всего теряется время или энергия. Подумай: какие повторяющиеся задачи в твоей жизни отнимают больше всего сил? Может, что-то, где приходится делать одно и то же руками каждый раз. Или где ты привлекаешь другого человека для помощи. Или где ты регулярно что-то теряешь или забываешь.»

Check: Wait for answer.

**Step 4.2 — Matching (internal LLM instruction):**

The LLM takes:
- What the learner **has** (from Phase 3)
- What the learner's **pains** are (from Step 4.1)
- The **24 use cases** with scope and tier
- The **architecture block dependency graph**

And builds a prioritized list using this logic:
1. **Obsidian vault is always first** — hard constraint, regardless of pains
2. Then: `pain × speed / dependency` — high pain + fast to build goes to the top; high pain + slow to build is included but lower; architecture dependencies are hard constraints (can't build an adapter without VPS)
3. Weight toward quick wins when pain is comparable — a 30-min setup that solves a moderate pain beats a 3-day setup that solves a slightly larger pain
4. Recommend **top 3 next blocks** with explanation of why each one and in what order

**Step 4.3 — Delivery format choice:**

Say: «У меня есть два варианта для тебя. Первый — я предложу персональный маршрут: три следующих блока, которые закроют твои главные боли максимально быстро. Второй — я покажу полное меню всего, что мы можем построить, и ты сам выберешь. Что больше нравится?»

Check: Wait for answer.

| Choice | Action |
|---|---|
| "Route" / "Recommendations" | Show top 3 blocks with rationale per block. Explain the order. |
| "Menu" | Show all 24 use cases grouped by life zones, with status indicators (already have / don't have). No imposed order. |

### 3.5 Phase 5 — Results

**Type:** Scripted (Action + Say)

**Step 5.1 — Write output files:**

Action: Create/update:
- `my-architecture.md` — full structured state (see §2.1)
- `my-system.md` — human-readable report (see §2.2)
- `learner-state.json` — final state with phase=5, complete=true

**Step 5.2 — Summary to learner:**

Say: Краткая сводка — «Вот что я понял о твоей системе» (3-5 пунктов). Что уже есть, где главные пробелы, какой маршрут рекомендую. Не дублировать содержимое файлов — говорить коротко, файлы уже записаны и ученик может их прочитать.

**Step 5.3 — Next step:**

Say: «Первый блок — настройка Obsidian vault как фундамента твоей системы. Готов начать?»

If **yes** → direct to the next skill (e.g., `/pos-vault`).
If **no** → «Ок, всё сохранено. Когда будешь готов — запусти `/pos-diagnostic` и мы продолжим с того места, где остановились.»

---

## 4. Resume Behavior

On every `/pos` invocation, the skill reads `learner-state.json`. If it exists and `complete` is not true:

Say: «В прошлый раз мы остановились на [описание фазы]. Хочешь продолжить оттуда или начать заново?»

If **continue** → load state, resume from `last_completed_step`.
If **start over** → archive old state (rename with timestamp), begin from Phase 1.

If `complete` is true → show current recommendations and offer to proceed to the next block, or re-run the diagnostic.

---

## 5. LLM Behavioral Rules

These rules apply throughout all phases:

1. **Return to script:** If the learner goes off-topic, answer briefly, return to current phase. Never ignore a question, never let the conversation derail.
2. **No meta-commentary:** Never say "I'm reading the script", "According to my instructions", "Let me check what's next". Speak as a teacher, not as an AI following a script.
3. **Tone:** Supportive, curious, conversational. Not robotic, not overly enthusiastic. The learner is sharing personal information about their life — respect that.
4. **No pressure:** If the learner doesn't want to answer something, move on. If they want to stop, save and stop.
5. **Intelligent fallback:** If the learner insists on skipping to the menu at any point, the LLM may comply after one gentle attempt to continue. This is not a rigid rail.
6. **Language:** All learner-facing text in Russian. Internal instructions in English.
7. **Save after every answer:** After each meaningful learner response, update `learner-state.json` and relevant sections of `my-architecture.md` incrementally.

---

## 6. Dependencies

| Dependency | Type | Description |
|---|---|---|
| `/pos-stt-setup` | Separate skill | STT installation flow. Called from Phase 2, returns to Phase 3. Not yet built. |
| `scripts/data/use-cases/state.json` | Data | Canonical list of 24 use cases with scope, tier, descriptions. |
| `scripts/data/blocks/state.json` | Data | Canonical list of 22 architecture blocks with layers, dependencies. |
| `scripts/data/uc-blocks/state.json` | Data | Use case → block mapping. |

---

## 7. Out of Scope

- **Building anything** — the diagnostic only collects information and recommends. No code, no installations (except STT via a separate skill).
- **Work-related questions** — the inventory focuses on personal life. Work tools and workflows are explicitly excluded from this version.
- **Detailed block descriptions** — the diagnostic names blocks and explains why they're recommended, but does not teach block content. That's each block's own skill.
- **`/pos-stt-setup` design** — separate spec, not covered here.
