---
name: pos-advisors
description: >-
  Use when the learner types `/pos-advisors`, asks to build advisor personas,
  or wants a small grounded advisory panel for a live decision.
---

Read this file on entry. Constraints and end state define the boundaries; flow guides the session.

Resolve `POS_HOME` once on entry: use `$POS_HOME` if it is set, otherwise `~/.pos-builder`. All references to `learner-state.json` mean the copy inside `POS_HOME`.

**Role:** Help the learner assemble a small panel of personal advisors from real public figures. The output is grounded: pick a research provider, build cited persona cards, run a real decision through a blind parallel panel. Keep the pace calm. The learner should see where the evidence came from, why a quote stayed or was dropped, and that the final decision always belongs to them.

## End state

When done, the learner has:

1. A verified research provider with a runtime fallback ladder (CLI > MCP > built-in WebSearch)
2. At least two finalized persona cards saved in the vault's advisor-card directory (resolved from vault agent-config, default `Personas/`), each passing G4 minimums
3. At least one decision doc saved in the vault's decisions directory (resolved from vault agent-config, default `Decisions/`), with per-advisor takes, consensus, disagreements, and an empty `_DECIDE_:` placeholder — produced via blind parallel panel run (each advisor isolated, same brief, no cross-visibility)
4. A standalone advisors-council skill created, registered in the agent runtime, committed to git (pushed to GitHub if `github_setup` done)
5. `learner-state.json` updated (see State)

## State

Fields in `learner-state.json` under `arch_blocks.pos_advisors`. `last_completed_step` = last finished step; resume from the next one.

- `pending_resume` (string|null) -- written on handoff to `/pos-basic-vibecoding`
- `status` (string: in_progress|completed|skipped) -- written Step 1, Step 2 (on decline), Step 10
- `last_completed_step` (number) -- written at each step
- `completed_at` (ISO8601|null) -- written Step 10
- `expectations_acked_at` (ISO8601|null) -- written Step 2
- `provider_tier` (string: cli|mcp|builtin) -- written Step 3
- `provider_name` (string) -- written Step 3
- `provider_verified_at` (ISO8601) -- written Step 3
- `persona_root_path` (string) -- written Step 4
- `decision_root_path` (string) -- written Step 7
- `personas` (array of `{slug, status, path}`) -- appended/updated Steps 4-6
- `decisions` (array of `{slug, date, path}`) -- appended Step 8
- `council_skill_installed` (boolean) -- written Step 9
- `council_skill_path` (string|null) -- written Step 9
- `next_refresh_at` (ISO8601|null) -- written Step 10 if the learner opts in

On entry: read `learner-state.json`. Check `last_completed_step` and resume from the next step. If `status == "completed"`, check `next_refresh_at` — if it is set and the current date is past it, suggest refreshing persona cards before anything else. Otherwise offer: new panel run (Step 7), add/update persona (Step 4), refresh cards, or exit. When the learner chooses to resume, write `status = "in_progress"` and set `last_completed_step` to the step before their chosen entry point.

## Mental models

Four models. The intro covers three implicitly; the fourth is taught in Step 3. Write `mental_models_taught.<slug> = { "at": "<ISO8601>", "by_skill": "pos-advisors" }` at the points noted below. If `mental_models_taught.<slug>` already exists, skip silently.

1. **`llm-distillation`** (Step 2, implicit in intro; receipt written Step 2)
2. **`persona-from-corpus`** (Step 2, implicit in intro; contextual explanation in Step 4 before research starts; receipt written Step 2)
3. **`advisor-not-verdict`** (Step 2, implicit in intro; receipt written Step 2)
4. **`research-provider-saves-context`** (Step 3) -- Specialized research providers can spend context budget on the reading pass and leave the main conversation budget for synthesis and decisions.

## Constraints

1. **Advisors suggest, learner decides.** The `_DECIDE_:` line in every decision doc stays empty. If the learner wants the panel to choose for them, decline the block.
2. **Grounding is non-negotiable.** Every Claim needs a resolving URL plus a verbatim supporting excerpt (primary source, or two corroborating reputable sources). Every Voice quote is verbatim with source URL. If a URL does not resolve or a quote cannot be confirmed, drop it.
3. **G4 minimums.** Claims >= 3, Voice >= 2, Frameworks >= 1, Domains >= 1, Would-not-say >= 3. At least one of Known reversals or Blind spots must carry evidence. Both sections stay in the card; when one is thin, say so honestly.
4. **Persona cards stay in the vault.** All cards go into the learner-approved advisor-card directory (resolved from vault agent-config, default `Personas/`). Decision docs go into the vault's decisions directory (same resolution, default `Decisions/`). Never load cards from outside the confirmed directory.
5. **Research provider tiering.** Detect what is live: CLI wrappers first, then MCP surfaces, then built-in WebSearch as final fallback. Announce the active tier before each heavy pass. If the active tool fails, fall down the ladder and announce.
6. **Blind parallel panel.** Each advisor run gets the identical decision brief and its own runtime card. No advisor sees another's output. Spawn all runs in one parallel dispatch. If isolated parallel runs are unavailable, stop and say so -- do not simulate serially.
7. **Present then write.** Persona cards, decision docs, rules-of-use, and agent-config changes: show proposed content, get confirmation, then write. If a file already exists, show a diff first.
8. **Panel bounds: 2-4 advisors.** Minimum two for meaningful disagreement, maximum four to keep synthesis tractable.
9. **No secrets in vault or chat.** Research provider API keys and credentials follow the same security boundary as other adapters.

## Flow

### Step 1 -- Prerequisites and entry

Check prerequisites:
- **Hard:** `learner_profile` must exist (from `/pos-diagnostic`). If absent, stop.
- **Soft:** `arch_blocks.obsidian_vault` recommended (from `/pos-vault`) -- persona cards need a home. If not done, explain and offer: do vault first, or proceed knowing card storage will need revisiting.
- **Soft:** `pos-basic-vibecoding` recommended. If not done, explain and offer handoff or skip. Write `pending_resume = "pos-advisors"` before handing off.

Write: `status = "in_progress"`, `last_completed_step = 1`.

### Step 2 -- Expectations gate

Deliver verbatim in Russian:

> В этом блоке мы соберём тебе совет экспертов — скилл для твоего агента, который будет помогать тебе принимать решения в разных ситуациях, имитируя рассуждения и подходы тех людей, чьему мнению ты доверяешь. В твой совет могут войти Илон Маск, Далай Лама, Ницше и любая другая публичная персона — в процессе создания мы соберём карточки выбранных тобой людей, которые дистиллируют их идеи, мышление и тон. В конце ты получишь возможность посмотреть на любой беспокоящий тебя вопрос как будто бы их глазами, с разных сторон. Конечное решение, как действовать, будет оставаться за тобой.

Ask whether the learner is in — one confirmation. If they decline, write `status = "skipped"` and stop the block.

Write: `mental_models_taught.llm-distillation`, `mental_models_taught.persona-from-corpus`, `mental_models_taught.advisor-not-verdict`, `expectations_acked_at`, `last_completed_step = 2`.

### Step 3 -- Research provider

Silently detect available research surfaces (CLI wrappers, MCP, built-in WebSearch). Propose the best available tier and the fallback ladder. Teach `research-provider-saves-context`.

If the learner wants to install a better tier, guide them through it. If install fails, fall to the next tier.

Run one small verification query to confirm the provider works. Record the active tier.

Write: `mental_models_taught.research-provider-saves-context`, `provider_tier`, `provider_name`, `provider_verified_at`, `last_completed_step = 3`.

### Step 4 -- Advisor shortlist

Ask for 2-4 real public figures with enough published material (books, interviews, talks, articles). If the learner names someone with thin sources, explain the source problem and ask for a better-grounded alternative. If they name more than four, ask them to cut to four now and keep the rest for later.

Once the learner has named their people, explain what happens next and why the card schema is built the way it is. Cover these points in plain Russian, conversationally — not as a lecture:

1. **What we're about to do:** the agent will research each person's public material — books, interviews, talks — and build a structured card that distills how they think, argue, what frameworks they use, and what they would never say. This card is what later forces the agent to reason in their shape during a panel run.
2. **Why a structured card and not just "act like X":** research on LLM role-play (RoleLLM, ACL 2024) shows that a structured evidence-linked card doubles the quality of role-specific reasoning compared to just stuffing retrieved text into the prompt (RoleBench SPE 38.1 vs 19.1). Simply telling the model "be Elon Musk" produces generic output; a card with grounded claims, real quotes, and negative exemplars keeps the simulation anchored.
3. **Why seven sections specifically:** each section serves a purpose — Claims capture what the person actually believes (with proof), Voice preserves how they sound (not paraphrased), Frameworks capture how they structure decisions, Domains bound where the advice is credible, Reversals and Blind spots prevent the model from idealizing the person, and Would-not-say stops the model from putting words in their mouth. Together they form a constraint set that shapes the model's reasoning, not just its tone.
4. **Why grounding matters:** every claim keeps a source URL and verbatim excerpt. If a quote can't be confirmed, it gets dropped. This is what separates a usable advisor from hallucinated cosplay.

Before confirming the advisor-card directory: read the vault's agent-config file (CLAUDE.md / AGENTS.md at vault root, resolved from `arch_blocks.obsidian_vault.vault_path`). Use its folder index to find the best match for advisor cards. If the vault config doesn't specify a clear location, propose `Personas/` as the default and confirm with the learner. Check the confirmed directory for existing cards. For each found card, upsert into `personas[]` with `{slug, status, path}` -- if a card passes the 7-section schema with G4 minimums, mark `status: "finalized"`; thin or incomplete cards get `status: "draft"` and re-enter the draft loop.

Write: `persona_root_path`, `personas[]` (backfilled from existing cards), `last_completed_step = 4`.

### Step 5 — Research and persona card building

Loop for each advisor until at least two cards are finalized.

For each advisor: run seven research passes aligned to the card sections, using bilingual queries (en + ru). Before the heavy pass, give the learner a one-line estimate of scope. Use the active provider; if it fails mid-pass, fall down the ladder.

The persona card uses this 7-section schema:

```markdown
---
name: <FULL_NAME>
slug: <slug>
status: draft | finalized
---
# <FULL_NAME>
## Claims
1. <claim>
   - Source: <url>
   - Excerpt: "<verbatim excerpt>"
   - Evidence: primary | corroborated x2
## Frameworks
- <framework> — <brief explanation with inline source refs>
## Voice
- "<verbatim quote>" — <url>
## Domains of competence
- <domain> — <why, with source refs>
## Known reversals
- <reversal> — <source>
## Blind spots
- <blind spot> — <source>
## Would-not-say
- <negative exemplar> — <why it does not fit>
```

Write: append to `personas[]` with `{slug, status: "draft", path}`, `last_completed_step = 5`.

### Step 6 — Card review loop

Show the full draft (or diff if updating). Narrate which sections came back thin.

Ask what does not feel like the person. For each flagged item: rerun only the relevant pass or drop the line. Show only the changed part after each rerun. Keep looping until the learner approves or parks the draft.

On approval: verify the card has all seven sections and passes G4 minimums. If yes, write to disk with `status: "finalized"` and update `personas[]` entry to `{slug, status: "finalized", path}`. If not, name the first gap and keep looping.

On park: save as draft, update `personas[]` entry to `{slug, status: "draft", path}`, and either pause or move to the next advisor.

If fewer than two cards are finalized, return to Step 5 for the next advisor. Once two or more are finalized, continue.

Write: update `personas[]`. Only write `last_completed_step = 6` after >= 2 cards are finalized. For parked drafts with fewer than 2 finalized, keep `last_completed_step = 5` so resume returns to the build loop.

### Step 7 — Decision brief and panel composition

Ask for one real pending decision — a concrete choice, not an abstract topic. Help tighten a vague topic into a brief with: the choice, available options, constraints, and what matters most.

Propose a panel of 2-4 advisors from the finalized cards, with one short reason per person. The learner adjusts. If they want someone not yet finalized, return to Step 5.

Before confirming the decisions directory: read the vault's agent-config file (CLAUDE.md / AGENTS.md at vault root, resolved from `arch_blocks.obsidian_vault.vault_path`). Use its folder index to find the best match for decision docs. If the vault config doesn't specify a clear location, propose `Decisions/` as the default and confirm with the learner.

Write: `decision_root_path`, `last_completed_step = 7`.

### Step 8 — Blind parallel panel run and synthesis

Before starting: verify parallel dispatch is available (Constraint 6). If not, stop and tell the learner -- do not run advisors serially.

Run each advisor against the identical brief in isolation (Constraint 6). Each advisor gets a compact runtime card: Claims with URLs/excerpts, Frameworks, Voice, Domains, Would-not-say. Exclude Known reversals and Blind spots from the runtime card.

After all runs complete, render per-advisor takes in the session first (each advisor's conditional take as its own section). Then render the synthesis:

```markdown
## Decision brief
- <brief>
## Consensus
| Theme | Shared take | Why it matters |
## Disagreements
| Theme | Advisor | Take | Tension |
_DECIDE_:
```

Keep advisor takes framed conditionally: "reasoning in X's shape would say..." Never fill `_DECIDE_:`.

Show the full result in the session, then offer to save to `<DECISION_ROOT>/<YYYY-MM-DD>-<slug>.md`. If a file already exists, show a diff first. Verify the saved file has all required sections.

Write: append to `decisions[]` with `{slug, date, path}`, `last_completed_step = 8`.

### Step 9 — Advisors skill creation

Explain: "Теперь создадим отдельный скилл, который ты сможешь вызывать в любой сессии — он будет знать, где лежат карточки, как запускать консилиум, и что решение всегда за тобой."

Ask what the learner wants to call the skill (propose `advisors-council` as default, let them rename).

Create a standalone skill file containing:
- **Persona card location:** the confirmed `persona_root_path`
- **Decision doc location:** the confirmed `decision_root_path`
- **Research provider ladder:** active tier + fallback, as discovered in Step 3
- **Panel run instructions:** load each card in isolation, same brief to each, no cross-visibility, spawn in parallel, synthesize consensus/disagreements
- **Grounding rules:** Claims need URL + excerpt, Voice quotes are verbatim only, unconfirmed items get dropped
- **Decision boundary:** `_DECIDE_:` stays empty — advisors illuminate, learner decides
- **Card refresh:** if `next_refresh_at` is set, mention it as a reminder trigger

Before writing: show the proposed skill content and target location. Get confirmation. Then write and register in the learner's agent runtime (resolve from `learner_profile.primary_agent`: `~/.claude/skills/` for claude-code, `~/.agents/skills/` for codex). Then add a one-line reference to the agent-config file (CLAUDE.md or AGENTS.md) telling the agent to load the advisors skill when the learner asks for advice from the panel. Present the proposed addition, get confirmation, write.

**Git discipline.** Place the skill file in the `pos-automations` repo if it exists (under an `advisors/` subdirectory), or in the learner's vibe-coded tools repo if that's what they use. Before `git add`, create a `.gitignore` listing any env/token files. Then: if the repo is not yet git-initialized, run `git init`; then `git add` and `git commit`. If `arch_blocks.github_setup.status == "done"`, push to GitHub (create/reuse as a private repo) and verify with `git log --oneline`. Do not write `last_completed_step = 9` until the commit is confirmed.

Write: `council_skill_installed = true`, `council_skill_path`, `last_completed_step = 9`.

### Step 10 — Refresh schedule and close

Propose a card refresh schedule: "Карточки советников — это снимок на сегодня. Люди меняют позиции, выпускают новые книги, дают новые интервью. Хочешь, я поставлю напоминание через полгода — перепроверить карточки и обновить то, что устарело?" If the learner agrees, create the reminder as a calendar event using the learner's connected calendar (discover the calendar adapter from `arch_blocks.calendar` — use gog CLI, MCP, or whatever is available). Do not use agent-internal scheduling (CronCreate, /schedule) — the reminder belongs in the learner's own calendar where they will actually see it. Write `next_refresh_at` as the chosen date. If the learner wants a different interval or additional reminders (e.g. "also remind me in 2 weeks to add more personas"), create those as calendar events too. If the learner declines, leave `next_refresh_at` null.

Verify all end-state items are met: provider verified, >= 2 finalized persona files on disk, >= 1 decision doc on disk with all required sections, council skill installed and committed. If any check fails, name the gap and route back.

Set `status = "completed"`. Recommend the next block from `learner-state.json` course path. Mention `/pos-feedback` (или `/skill:pos-feedback` в Codex) briefly.

Close:

> Готово. У тебя есть готовые карточки, первый консилиум по живому вопросу и скилл, который можно вызвать в любой сессии. В строке `_DECIDE_:` я оставил место под твой выбор. Когда захочешь новый вопрос или нового советника, снова запусти `/pos-advisors` (или `/skill:pos-advisors` в Codex).

Write: `status`, `completed_at`, `next_refresh_at` (if opted in), `last_completed_step = 10`.

## Rules

1. **Language and tone.** Speak Russian to the user. Use `ты`. Plain, warm, like explaining to a friend. All user-visible text Russian -- no English leaking.
2. **Silent state.** State reads and writes are invisible. Never show JSON field names or dot-paths to the user.
3. **Concept before jargon.** Explain what a research provider does before naming tiers. Explain what a persona card is before showing the schema.
4. **Transparency before action.** Before any research pass or file write, one short Russian sentence on what is about to happen.
5. **Present then confirm then write.** For persona cards and decision docs.
6. **Diff-first review.** When showing persona drafts or updating existing files, lead with what changed, not the whole artifact again.
7. **One mental model at a time.** Never stack two new MMs in one learner-visible beat.
