# pos-goals — Locked Frame

> Frame locked 2026-04-21 during brainstorming session with Stas. References:
> - Skill contract: `docs/skill-contract.md` (normative — do not restate here)
> - Sibling (tone/rhythm only, anti-anchored at Phase 1): `skills/pos-morning-brief/SKILL.md` — latest shipped

## Overview

`pos-goals` is the **mandatory prerequisite** for the alignment layer of the course. It captures the learner's life goals (8-domain Wheel-of-Life by default, learner-overridable) into `<vault>/Goals/goals.md`, optionally decomposes them into a yearly/monthly/weekly structure in `<vault>/Goals/horizons.md`, and schedules recurring review events through the learner's already-connected calendar. The resulting `arch_blocks.goals.status = "done"` is the hard-gate surface that `pos-morning-brief`, `pos-triage`, `pos-day-summary`, and `pos-meeting-sync` depend on.

The skill is agentic on elicitation (four runtime modes, scaffold phrases combat blank-page, soft per-horizon gap-fill) and prescriptive on safety, consent, credential location, and downstream contract. «Что всплыло под щедрым вопросом — то и есть реальный приоритет» is the central stance; the skill never forces content, never pathologizes silence.

## Prerequisites

**Hard-gate at Phase 0** — missing → route to corresponding `/pos-<skill>` with one-line Russian reason, pause with standard resume pattern:
- `arch_blocks.vault.status = "done"` (need vault for any goals file write — G1).
- `arch_blocks.calendar.status = "done"` (at the scheduling phase — G5; routed later, not at Phase 0, because elicitation and writing `goals.md` don't require calendar).

**Write-scope conditional (at scheduling phase)** — pos-calendar must have write scope granted; if only read scope → scope-elevation handoff via `/pos-calendar` with `pending_resume = "pos-goals-after-calendar-write"`. Exact field name for write-scope state is runtime-discovered from pos-calendar's schema (not hardcoded in this frame).

**No other prereqs.** The skill runs independently of all other blocks (`pos-tasks`, `pos-email`, `pos-telegram`, `pos-basic-vibecoding` — all irrelevant here).

## End State (outcome-shaped, soft on content)

1. **`<vault>/Goals/goals.md`** (default path; learner may redirect) — H2 sections matching `taxonomy_labels`. In default Wheel-of-Life taxonomy, 8 domains in fixed order (HEALTH · SELF · RELATIONSHIPS · CAREER · FINANCIAL · CREATIVITY · CONTRIBUTION · FUN/RECREATION). Every domain has either (a) 1+ line of learner content or (b) explicit «не в фокусе сейчас» tag. In custom taxonomy («свои» mode), the learner's structure wins — no per-domain floor, only a sanity check. Frontmatter: `updated: <ISO date>`.

2. **`<vault>/Goals/horizons.md`** (default path; learner may redirect) — exists if learner opted into at least one horizon **or** `scheduling_mode = "file"` (horizons.md also holds the file-fallback cadence top-matter, even when horizon sections are empty). Three H2 sections in fixed order: «Годовые» · «Месячные» · «Недельные». Short focus-style lists, loosely domain-tagged. Empty sections allowed. Frontmatter: `updated: <ISO date>` plus `weekly_review` / `monthly_review` / `yearly_review` cadence lines when `scheduling_mode = "file"`.

3. **Review scheduling — primary path: calendar events.** Three recurring events created via the tool `pos-calendar` set up (MCP / `gog` / CLI — runtime-discovered from `arch_blocks.calendar`):
   - **«Мои цели на неделю»** — learner-chosen time + recurrence (default: Sunday 18:00 weekly).
   - **«Мои цели на месяц»** — default: last Sunday of month.
   - **«Мои цели на год»** — default: late December or first week of January (**not** «через год от сегодня» — anchor to natural year-boundary).
   
   Notes field points to `goals.md` (+ `horizons.md` if present). Events contain no auto-invocations (F7). Event metadata (provider-specific id, title, recurrence, first-fire) stored in `calendar_event_refs[]`. **All intended event writes must succeed before `status: "done"`**; partial-write failure invokes the error-visibility contract (principle 22) — concrete failure surfacing, learner choice between retry / switch provider / manual / re-scope-with-consent. Weekly-event visibility (G8) remains the learner-facing verification anchor.

4. **Fallback path: file-based cadence intent** — only if learner explicitly refuses calendar scheduling (G7). `horizons.md` top-matter lines `weekly_review: <cadence>` / `monthly_review: <cadence>` / `yearly_review: <cadence>` as plain text. Fallback is explicit-opt-only, never silent.

5. **`arch_blocks.goals`** populated in `learner-state.json` per state contract below. The hard-gate surface for morning-brief / triage / day-summary / meeting-sync.

6. **Verification artifact** — learner has opened their calendar and visually confirmed the **weekly** review event appears in the correct slot (G8). Monthly + yearly created in the same pass but not the verification focus.

7. **Agent-rules section** — short Russian `## Цели` block in the learner's POS project agent-rules target set, resolved from `learner_profile.primary_agent`: `CLAUDE.md` for Claude Code, `AGENTS.md` for Codex, and both only if `learner_profile.keep_agent_configs_in_sync == true`. If the learner profile is missing, infer the current runtime agent and confirm once before the write. Section content: canonical paths (`goals_path`, `horizons_path` if present) + review cadence. Append-only; diff-and-confirm if section exists (G9).

**Target duration:** 60–90 min (structured mode with deep per-domain engagement can run longer). Braindump mode: 45–60 min typical. «Свои» mode: 20–30 min (no elicitation, just confirm + schedule). Update mode: 15–30 min. The skill supports mid-phase pause/resume via the standard Phase 0 pattern; state writes at phase transitions only (principle 11).

## Mental Models taught (4)

1. **Цели = якорь для агента.** Без записанных целей агент каждый ход выводит заново из обрывков контекста — последние сообщения, открытые файлы. Дорого и нестабильно. С записанными целями поверхность alignment стабильна — агент не гадает, а сверяется. *(Spine of the alignment layer across the course — reinforced in `pos-morning-brief` daily brief, `pos-triage` priority ranking, `pos-day-summary` retrospective, `pos-meeting-sync` meeting prep.)*

2. **Что всплыло первым — то и есть реальный приоритет.** Под щедрым вопросом fast-recall = живое; scraping-to-answer = aspirational / performance. Тишина — не провал, а сигнал. «Не в фокусе сейчас» — валидное состояние. *(Drives the skill's stance — we don't force content in silent domains, we tag them. Applies to any mode.)*

3. **Если не записано — агент не увидит.** У агентной памяти нет telepathy-слоя. Записанный файл — единственная общая поверхность между учеником и агентом. *(Surfaced lightly in `pos-vault`; lands in full here. Also alive across adapters — `pos-calendar` (events = written intent), `pos-telegram`, `pos-email`, `pos-tasks`.)*

4. **Цели живут через ритм ревью.** Цель, записанная 6 месяцев назад и ни разу не пересмотренная — окаменелость. Ревью не бюрократия, а акт, который держит компас живым. Без ритма downstream-скиллы читают устаревший файл, alignment деградирует. *(Grounds the scheduling phase — `arch_blocks.goals.updated_at` + morning-brief's `life_goals: 60 days` staleness threshold depend on this MM.)*

## Required Gates

- **G1. Vault prereq at Phase 0.** Read `arch_blocks.vault.status`. If `!= "done"` → hard-route to `/pos-vault` with `pending_resume = "pos-goals-after-vault"` and one-line Russian reason. No goals file can exist without a vault root.

- **G2. Existing-file diff-and-confirm.** Probe for pre-existing goals artifact: `<vault>/Goals/goals.md`, `<vault>/_meta/goals.md` (transitional — pre-retrofit pos-vault learners, see #115), or the path stored in `arch_blocks.goals.goals_path` if a prior run was interrupted. If anything detected, show current content, explicit Check before reading/reformatting/migrating. Never silent overwrite, never silent `_meta/` → `Goals/` migration.

- **G3. Tagged-pass floor verified before `goals.md` write** (applies when `taxonomy == "wheel_of_life"`). All 8 domains carry either learner content or explicit «не в фокусе сейчас» tag. No advance to write with any domain empty or absent. When `taxonomy == "custom"` («свои» mode), G3 is replaced by a sanity check (file parses as Markdown, non-empty, has at least one H2).

- **G4. Explicit write consent for Goals-folder files.** Applies to any create/update of `goals.md` AND `horizons.md` — first-run, update mode, file-fallback cadence writes, all equally gated. Show the fully rendered file (sections + frontmatter with `updated: <ISO>`, plus `weekly_review` / `monthly_review` / `yearly_review` top-matter lines when horizons.md is used for file-fallback cadence) back as a preview block. Explicit Check «записываем?» before disk write. Principle 16 applies (skip Check if learner confirmed the exact render in the immediately prior turn).

- **G5. Scheduling-mode check at scheduling phase.** Read `arch_blocks.calendar`. Four branches:
  - `status == "done"` + write scope available (exact field name runtime-discovered from `arch_blocks.calendar`'s schema) → default path: `scheduling_mode = "calendar"`, proceed to G6.
  - `status == "done"` but write scope not granted → offer two options: (i) scope elevation via `/pos-calendar`, pause with `pending_resume = "pos-goals-after-calendar-write"`, return to G5; (ii) affirmative file-fallback → `scheduling_mode = "file"`, skip to G7.
  - `status != "done"` → offer two options: (i) hard-route to `/pos-calendar`, pause with `pending_resume = "pos-goals-after-calendar"`, return to G5; (ii) affirmative file-fallback → `scheduling_mode = "file"`, skip to G7.
  - Learner proactively opts out of calendar scheduling at the scheduling-phase open (before any probe) → `scheduling_mode = "file"`, skip to G7 directly.
  
  Silent downgrade from calendar to file on prereq failure is forbidden (F5); every file-fallback path here requires the learner to affirmatively pick it.

- **G6. Review-event preview before write.** Show all events to be created (weekly + monthly + yearly — or the subset the learner opted into) as a preview block: title, time, recurrence, notes pointer. Explicit Check «создаём?» before any calendar write. All events in the approved preview must be written successfully; partial-write failure invokes principle 22's error-visibility contract (concrete failure line, learner choice of retry / switch / manual / re-scope-with-consent) — `status: "done"` not reached until the preview's full scope resolves.

- **G7. File-fallback preview and write consent.** Reachable only when `scheduling_mode = "file"` was set by an affirmative learner choice at G5 (prereq-missing branch, scope-missing branch, or up-front opt-out). Show the cadence lines to be written as `horizons.md` top-matter (`weekly_review: <cadence>` / `monthly_review: <cadence>` / `yearly_review: <cadence>`) as a preview block. Explicit Check «записываем ритм в файл?» before disk write. File-fallback write flows through G4 consent for the `horizons.md` file itself.

- **G8. Verification: weekly event visible.** Before closing the scheduling phase, learner opens their calendar client and confirms the weekly review event is visible in the correct slot. One-step retry / edit-time / skip branch if not visible. Monthly + yearly created in the same pass but not the verification focus — weekly = soonest visible.

- **G9. Agent-rules file write — resolve + confirm.** Before writing the `## Цели` section:
  - Resolve the primary target from `learner_profile.primary_agent`: `CLAUDE.md` for Claude Code, `AGENTS.md` for Codex.
  - If the learner profile is missing, infer the current runtime agent and confirm once.
  - If `learner_profile.keep_agent_configs_in_sync == true`, include the sibling file in the required target set.
  - Show the proposed section as a diff. Append-only; diff-and-confirm if a required target already has the section.

## Skill-specific runtime logic

- **R1 — Four-mode elicitation converging to one output.**
Phase 1 offers three first-run modes and one re-entry mode. Labels in learner-visible text always come with a one-line shape explanation, never bare names.
  - **структурно** — sequential walk through the 8 domains (or custom taxonomy if learner chooses override), scaffold phrase opens each beat.
  - **брейндамп** — free dump, agent structures live, soft gap-fill after.
  - **свои** — learner brings existing goals file, skill accepts as-is, no restructuring.
  - **обновить** — re-entry update interview, auto-offered on `status == "done"` re-entry only.
  
  Mode recorded in `arch_blocks.goals.mode`. No mode switching mid-run (F8); reset is explicit.

- **R2 — Dual-axis structuring during braindump parse.**
When mode = брейндамп, the agent parses free text and tags each fragment along **both** axes simultaneously: domain (8 Wheel-of-Life or custom) and horizon (long-term / yearly / monthly / weekly / unspecified). A single fragment can appear in both outputs (e.g., «в этом году хочу начать бегать» = HEALTH + yearly). Structured mode doesn't need this — per-domain prompts naturally ask long-term by default, with horizon prompts layered after.

- **R3 — Scaffold-phrase pattern as first-class blank-page combat.**
The agent proposes concrete fill-in-the-blank shapes, not generic examples. Default shape: «сейчас в [domain] у меня [X], хочу к [timeframe] — [Y]». Learner adapts / replaces / rejects. Structured mode opens **each** domain beat with a scaffold; braindump mode uses scaffold only if learner visibly freezes. Shape is domain-appropriate, not one-template-fits-all.

- **R4 — Soft horizon elicitation after `goals.md` lands.**
After goals.md is written, agent:
  1. Reviews horizon-tagged content already surfaced (from R2 parse, or from structured-mode long-term answers that hint at timing).
  2. Shows it back in natural language: «вот что уже просвечивает по горизонтам: на этот год — …, на этот месяц — …».
  3. Softly asks to fill gaps, one horizon at a time in natural order (yearly → monthly → weekly): «а на этот год ещё что-то добавим?».
  4. Per-horizon decline allowed inline («на неделю не хочу сейчас — ок, пропускаем»); skill advances.
  
  Never a single upfront «хочешь расписать каскад да/нет?» gate. Never the word «cascade» / «каскад» in learner-visible text (F11).

- **R5 — Runtime-discovered calendar write surface.**
Applies only when G5 resolves to `scheduling_mode = "calendar"` (not the file-fallback branches). The body reads `arch_blocks.calendar` to discover what `pos-calendar` set up (MCP server name, CLI binary, env vars, whatever `pos-calendar` wrote). Frame doesn't hardcode the tool or command — body researches at runtime per the permissive-framing principle. Three event writes share the discovered interface. Partial-write failure and full-failure both hit the error-visibility contract (principle 22); `status: "done"` is not written until the G6 preview scope fully resolves (retried, re-scoped with learner consent, or explicitly converted to file-fallback via a fresh G5 → G7 consent round).

- **R6 — Migration from pre-retrofit `_meta/goals.md`** (transitional, first ~6 months after ship).
If G2's probe finds `_meta/goals.md` on disk (pre-retrofit pos-vault learner, see #115) but not `Goals/goals.md`:
  1. Show content, confirm it's the file we're inheriting.
  2. Confirm new location (default `Goals/goals.md`, soft — learner may redirect).
  3. Read → reformat to the 8-domain tagged-pass structure (or custom taxonomy if learner prefers) → write to new path → set `arch_blocks.goals.goals_path`.
  4. One-line Check: «оставить старый `_meta/goals.md` как есть или удалить?». Never silent-delete.
  
  Runtime branch retires once #115 is shipped and no new learners produce `_meta/goals.md`.

- **R7 — Update mode (re-entry after `status == "done"`).**
Phase 0 resume probe detects `arch_blocks.goals.status == "done"` AND `goals.md` exists → offer update mode as default re-entry path. Short interview, not 8 × Say/Check:
  1. Read current state (`goals.md` + `horizons.md` if present), render back to learner as concise preview.
  2. One open Check: «что изменилось с последнего раза?». Learner names deltas.
  3. Delta-capture in memory. Content-to-tag and tag-to-content flips allowed per domain.
  4. Horizon refresh — soft, opt-in per horizon (R4 shape, shorter).
  5. Review-cadence check. «Ритм ревью оставляем или двигаем?» — if moved and mode is calendar, update events via G5+G6 (read `calendar_event_refs`, update in place or delete-and-recreate — runtime-discovered). If mode is file, update `horizons.md` top-matter.
  6. G4 diff-and-confirm on the updated files.
  7. Writeback: `arch_blocks.goals.updated_at` refreshed. `schema_version` unchanged.
  
  Full-redo escape: «хочу переделать с нуля» → runs structured / braindump from scratch, overwrites on G4 confirm.
  
  Phase 0 branch table:
  | State detected | Default path |
  |---|---|
  | `arch_blocks.goals` missing or `status != "done"` | First-run (Phase 1 mode pick: структурно / брейндамп / свои) |
  | `status == "done"` + goals.md exists | Update mode (with full-redo escape) |
  | `status == "done"` + goals.md missing / unreadable | Repair mode — confirm file lost, route to first-run with learner-approved overwrite |

- **R8 — «Свои» mode (bring your own goals file).**
Learner has existing goals in their own structure. Flow:
  1. Agent confirms path (default `<vault>/Goals/goals.md` or learner-named).
  2. Reads file, shows content back as preview: «вот что прочитал — всё верно, с этим работаем?».
  3. Extracts section names → `taxonomy_labels` in state; sets `taxonomy = "custom"`.
  4. Sanity check (G3 custom variant): file is Markdown, has ≥1 H2 + non-empty. If not, one-line issue report + offer switch to структурно / брейндамп.
  5. If `horizons.md` exists at a known path, read it. Otherwise, offer R4 soft elicitation or skip.
  6. Proceed to scheduling (G5 → G6 → G7) as usual.
  7. State write: `mode = "bring_your_own"`, `taxonomy = "custom"`.
  
  No restructuring, no forced tagging, no 8-domain imposition. Learner's structure wins.

## Constants and shared sets

### C1 — Life-domain set (default, overridable)

**Default (proposed as opening anchor):** 8 Wheel-of-Life domains, fixed order: HEALTH · SELF · RELATIONSHIPS · CAREER · FINANCIAL · CREATIVITY · CONTRIBUTION · FUN/RECREATION.

**Override path:** learner uses «свои» mode (brings file) or redirects at the opening mode-pick («у меня своё деление»). Recorded as:
- `arch_blocks.goals.taxonomy ∈ {"wheel_of_life", "custom"}`
- `arch_blocks.goals.taxonomy_labels: string[]` — actual H2 names in file order

Downstream consumers read `taxonomy_labels` to parse — they don't assume Wheel-of-Life specifically.

### C2 — Horizon set (fixed)

| Internal key | Learner-facing label | Storage |
|---|---|---|
| `long_term` | «долгосрочные» | captured in `goals.md` (the domain content itself) |
| `yearly` | «на год» | `horizons.md` → `## Годовые` |
| `monthly` | «на месяц» | `horizons.md` → `## Месячные` |
| `weekly` | «на неделю» | `horizons.md` → `## Недельные` |

Three H2s in `horizons.md`, fixed order. Downstream consumers parse positionally.

### C3 — Silent-domain tag phrase (fixed, Wheel-of-Life only)

**«не в фокусе сейчас»** — exact learner-visible string for any default-taxonomy domain with no content. Downstream consumers detect tag by exact string match or empty-content-with-tag heuristic. In custom taxonomy, no equivalent floor — learner's structure wins.

### C4 — Mode labels (learner-facing, always with shape explanation)

| Label | Shape explanation (Phase 1 writer tunes) | When reachable |
|---|---|---|
| **структурно** | «пройду по 8 сферам жизни по очереди, помогу формулировать» | First-run |
| **брейндамп** | «наливай всё подряд как придёт, я разложу после» | First-run |
| **свои** | «у тебя уже есть цели в своём виде — заберём как есть» | First-run OR entry-probe detects existing non-default file |
| **обновить** | «у тебя уже записаны цели — пройдёмся коротко, что изменилось» | Auto-offered on `status == "done"` re-entry only |

Stored: `arch_blocks.goals.mode ∈ {"structured", "braindump", "bring_your_own", "update"}` (English keys in state, Russian in `Say:` — principle 1).

### C5 — Default calendar-event titles (soft — learner may rename)

- **«Мои цели на неделю»**
- **«Мои цели на месяц»**
- **«Мои цели на год»**

Learner-ownership phrasing (not «review» loanword). Stored as-written in `calendar_event_refs[].title`.

### C6 — Default paths (soft — learner may redirect)

- `<vault>/Goals/goals.md`
- `<vault>/Goals/horizons.md`

Resolved paths stored in `arch_blocks.goals.goals_path` and `.horizons_path`. In «свои» mode, resolved path is wherever the learner's existing file lives — no move unless requested.

### C7 — Review-event default timing windows (soft)

Body proposes these when the learner doesn't have a preferred cadence:
- **Weekly**: Sunday 18:00–19:00 (any day / time OK).
- **Monthly**: last Sunday of the month (any day OK).
- **Yearly**: late December or first week of January — **not «через год от сегодня»**. Anchors to the natural year-boundary reflection moment regardless of when pos-goals is run.

Body offers, learner picks; picked times stored in `calendar_event_refs[].recurrence`.

### C8 — Downstream staleness threshold (informational)

`pos-morning-brief` hardcodes `life_goals: 60 days`. `pos-goals` only records `updated_at`; does not enforce staleness itself.

## State Contract — `arch_blocks.goals`

```json
{
  "arch_blocks": {
    "goals": {
      "schema_version": 1,
      "status": "in_progress | done",
      "started_at": "<ISO-8601>",
      "completed_at": "<ISO-8601> | null",
      "current_phase": 0,

      "mode": "structured | braindump | bring_your_own | update | null",
      "taxonomy": "wheel_of_life | custom | null",
      "taxonomy_labels": ["HEALTH","SELF","RELATIONSHIPS","CAREER","FINANCIAL","CREATIVITY","CONTRIBUTION","FUN/RECREATION"],

      "goals_path": "Goals/goals.md",
      "horizons_path": "Goals/horizons.md | null",

      "scheduling_mode": "calendar | file | null",
      "calendar_event_refs": [
        {
          "horizon": "weekly",
          "id": "<provider-specific event id>",
          "title": "Мои цели на неделю",
          "recurrence": "<RRULE string or cadence description>",
          "first_fire": "<ISO-8601>"
        },
        { "horizon": "monthly", "id": "…", "title": "Мои цели на месяц", "recurrence": "…", "first_fire": "…" },
        { "horizon": "yearly",  "id": "…", "title": "Мои цели на год",    "recurrence": "…", "first_fire": "…" }
      ],
      "review_cadence": {
        "weekly":  "<cadence string | null>",
        "monthly": "<cadence string | null>",
        "yearly":  "<cadence string | null>"
      },

      "pending_resume": "pos-goals-after-vault | pos-goals-after-calendar | pos-goals-after-calendar-write | null",
      "updated_at": "<ISO-8601>"
    }
  }
}
```

Notes:
- `schema_version = 1` per v0.1 DoD cross-cutting rule.
- Fields default to `null` before Phase 1; populated at phase transitions (principle 11).
- `calendar_event_refs` populated only when `scheduling_mode == "calendar"`; empty list when `scheduling_mode == "file"`.
- `review_cadence` populated regardless of mode (intent-level semantic "what rhythm did we agree on"). In file-fallback mode, these strings are also what's written to `horizons.md` top-matter verbatim.
- `pending_resume` is set only when paused for a prereq route-out; cleared on resume.
- `completed_at` written on status → `"done"` transition; persists across update-mode re-entries (updated_at bumps, completed_at stays as original completion anchor).

**Cross-skill state reads:**
- `arch_blocks.vault.status` — G1 prereq, read-only.
- `arch_blocks.calendar.status` + write-scope field (runtime-discovered) — G5 prereq, read-only.
- Top-level `mental_models_taught: [{skill, mm_key}, ...]` — append MM1–MM4 entries on completion (course-wide cross-skill MM tracking per #23).

**Migration notes:**
- From pre-retrofit pos-vault (#115): `arch_blocks.vault.goals_status` silently dropped when #115 ships (vault-side schema_version bump handles that). `pos-goals` itself does not read or migrate `goals_status`. File-level migration of `_meta/goals.md` content handled by R6.
- Within pos-goals: schema_version 1 is the initial shape. No internal migrations yet.

## Landing moment

> Internal-vocabulary rule: frame prose follows the learner-visible restraint — «landing moment», not «wow moment» (F11, F12). Matches the skill's broader stance: goals are a working anchor, not an achievement.

**The moment** — two beats, both observable, neither forced:

1. **Beat A — Readback.** At the end of the writing pass, agent shows the learner `goals.md` rendered (8 domain headings or custom taxonomy with learner's own words under each) + `horizons.md` rendered if exists. One screen. The «вот как это выглядит одним экраном» image is what lands «я держал это в голове, а теперь оно снаружи».

2. **Beat C — First weekly event visible.** After G6 writes review events and G8 verification asks the learner to open their calendar, they see «Мои цели на неделю» sitting on next Sunday at 18:00 (or whatever cadence they picked). Weekly event is the emotional anchor (soonest visible); monthly and yearly are created in the same pass but not the verification focus.

**The trigger:** G8 passes — learner confirms the weekly event is visible in the correct slot.

**The landing line** (Phase 1 author tunes; one sentence, restrained):

> «Теперь цели записаны, и встреча с собой стоит в календаре. Утренний бриф, триаж и итоги дня будут на них опираться, а ты — пересматривать по расписанию.»

**Degraded-path variants:**

| Path | What lands |
|---|---|
| Default (Wheel-of-Life + calendar scheduling) | Full A + C — primary case. |
| «Свои» mode + calendar scheduling | A softer (learner's own structure read back); C unchanged. |
| Any mode + file-fallback (no calendar) | A is the full beat; C replaced by «в горизонтах записан ритм ревью, агент о нём знает». |
| Update mode | A is a diff readback; C fires only if scheduling changed — otherwise continuity, no new moment. |

**What the landing moment is NOT:**
- Not a simulation of what morning-brief will say tomorrow.
- Not multiple moments strung together — one beat, A then C, then one sentence, then close.
- Not a victory lap — no «поздравляю, ты молодец», no emoji-heavy farewell (F12).

## Forbidden

- **F1. Never auto-infer goals from other data.** No generation from tasks / calendar / TG / email / vault contents. The skill reads and reformats only what the learner has written as goals (R6 / «свои» mode) or directly says in Phase 3. *Prevents:* MM3 integrity erosion, fabricated alignment surface.

- **F2. Never force content in a silent domain.** In Wheel-of-Life taxonomy, empty domain gets the «не в фокусе сейчас» tag (G3) or stays untagged for learner to address. Agent never fills with fabricated aspirations. *Prevents:* MM2 violation, fake-alignment feeding downstream.

- **F3. Never silent-overwrite, silent-delete, or silent-migrate.** Every write to `goals.md` / `horizons.md` / the resolved agent-rules target set is diff-and-confirm. R6 migration requires explicit consent for read/reformat/move; deletion of old file requires separate Check. *Prevents:* data loss, trust break, G2/G4/G9 bypass.

- **F4. Never write outside the scoped set.** Allowed write targets: content files (`goals_path`, `horizons_path`), the learner's resolved POS project agent-rules target set, calendar events via pos-calendar's tool, and `learner-state.json` for state contract writes per principle 11 (specifically `arch_blocks.goals.*` updates and the top-level `mental_models_taught` append on completion). No other filesystem or network writes. *Prevents:* filesystem scope creep, cross-skill interference.

- **F5. Never silently fall back calendar → file.** G5 failure routes via `pending_resume`; does NOT silently degrade to `horizons.md` lines. File-fallback (G7) fires only on affirmative refusal. *Prevents:* the most dangerous failure mode — learner thinks reviews are scheduled when they're just text.

- **F6. Never touch credentials or initiate OAuth.** Scheduling uses pos-calendar's already-configured auth. Skill never prompts for tokens, never touches keyring / env, never reads auth state. Broken auth → route to `/pos-calendar`. *Prevents:* credential sprawl, principle-15 violation.

- **F7. Never embed auto-invocations in calendar events.** Event notes contain plain-text path pointer and nothing else. No `/pos-weekly-review` slash-commands, no shell commands, no agent-hints. *Prevents:* scope creep, learner confusion, future-skill name coupling.

- **F8. Never switch modes mid-run.** Once learner picked, they stay on that branch. Reset is explicit — «начнём заново» → re-enter Phase 1. Update mode reachable only via re-entry, not mid-run. *Prevents:* state confusion, partial-data inconsistency, R1/R7 intent violation.

- **F9. Never prescribe content.** Scaffold phrases provide shape, never prescription. No «ты должен хотеть X», no pathologizing («а почему пусто в CREATIVITY?»). Neutral, non-judgmental, domain-respectful. *Prevents:* MM2 violation, learner-agency erosion.

- **F10. Never validate custom taxonomy beyond sanity check.** «Свои» mode sanity: file parses, non-empty, ≥1 H2. No minimum section count, no naming rules, no structural correctness enforcement, no Wheel-of-Life equivalence check. *Prevents:* R8 intent violation, agent-knows-better stance.

- **F11. Never use internal vocabulary in learner-visible text.** Banned from `Say:`: «cascade», «каскад», «wow», «landing moment», «tagged-pass floor», «taxonomy», any `arch_blocks.*` field name, any JSON key. Frame / body comments / state may use them; learner prose never. *Prevents:* principle-17 violation, jargon leak.

- **F12. Never dramatize the landing moment.** One sentence after G8 confirmation. No «поздравляю, ты молодец», no multi-sentence closer, no emoji-heavy farewell, no victory-lap framing. *Prevents:* tone mismatch with MM1/MM4, Block 7 intent violation.

## Course-wide principles applied

Body inherits all 22 principles from `docs/skill-contract.md`. Especially load-bearing for this skill:
- **1** (Russian for learner / English for runtime).
- **3** (bite-sized `Say`/`Check` rhythm — the 8-domain walk is long; pacing matters).
- **9** (never overwrite — universal; G2/G4/G9 operationalize it).
- **10** (confirm before destructive ops — every file write and calendar event creation).
- **11** (state writes at phase transitions only — ~12 writes across the skill).
- **16** (skip pre-answered Checks — braindump mode surfaces content that obviates later Checks).
- **17** (state writes are silent — no JSON keys in learner-visible text, F11 operationalizes).
- **18** (pre-warn predictable anxiety — mostly for scope-elevation handoff to `/pos-calendar`).
- **19** (one mental model per Say — MM1–MM4 each get their own beat; don't stack).
- **21** (proof before choice — qualified by consequence — cascade framework picks, if the body researches one, need receipts).
- **22** (error-visibility contract — calendar event creation, file writes, scope elevation all need explicit failure paths).

## Body-authoring notes (for Phase 1 writer, not part of the frame contract itself)

- Body runs a resume probe at Phase 0 per the branch table (R7). First-run branches on mode pick at Phase 1.
- The 8-domain walk in structured mode is the longest single phase. Consider pacing: every few domains may deserve a brief meta-Check («как идёт? устал — можем паузу») without re-asking mode choice.
- Braindump mode's structuring step happens in memory (principle 11). The learner should see a coherent mid-conversation recap, not a silent internal reorganization followed by a wall-of-text readback.
- Scaffold phrases (R3) are the skill's signature move. Phase 1 writer should produce concrete per-domain examples — HEALTH scaffold ≠ FINANCIAL scaffold ≠ CREATIVITY scaffold. One template doesn't fit all domains.
- The soft horizon gap-fill (R4) is where the skill's restraint stance shows most. Don't re-litigate declined horizons; accept and move on.
- Calendar write at G6 is the skill's only external side-effect. Wrap it in the error-visibility contract (principle 22) — concrete failure modes: write scope revoked mid-phase, network blip, event conflict (existing event at the same slot).
- The update mode (R7) is short. Phase 1 writer should resist the temptation to make it a full re-run — it's explicitly a delta capture, not a re-elicitation.
- Permissive-framing (see `feedback_soft_framing`): default taxonomy, default paths, default titles, default cadences are offers the learner can redirect. Hard only on safety (gates + forbiddens).

## References

- `docs/skill-contract.md` — normative 22 principles; do not restate in `SKILL.md`.
- `docs/block-runtime-pattern.md` — runtime flow reference.
- `docs/blocks/morning-brief-spec.md` — sibling frame (composite skill; reads pos-goals as hard-gate dependency).
- `skills/pos-morning-brief/SKILL.md` — latest shipped sibling; Phase 1 writer anchors tone/rhythm only, anti-anchoring guard active on structure/content.
- `skills/pos-vault/SKILL.md` — related prerequisite artifact home.
- `skills/pos-calendar/SKILL.md` — G5 prereq; write-scope state field runtime-discovered from its schema.
