# Skill Contract

> Canonical reference for shipped POS-builder skills under the thin-frame + agentic-body approach. Every skill after `pos-diagnostic` and `pos-stt-setup` follows this contract.

## Thesis

A block has two layers:

1. **Fixed frame** — authored by a human (Stas / Alex). States the end state, the mental models the block must teach, the required gates, and the explicit forbiddens. This is the contract.
2. **Behavioral body** — authored by Claude from the frame. Phase-by-phase interview-driven flow, resume logic, state writeback, handoff. Reviewed by the human; simulated against difficult personas.

The frame is permissive on tools and exact wording, prescriptive on safety and course-wide conventions. The body is free to research tools at runtime, branch on provider/OS, and adapt — as long as it honors every gate and never crosses a forbidden.

## File layout

```
skills/<skill-name>/
├── SKILL.md              ← single source of truth: frontmatter + role + rules + frame + body
├── <references>/         ← optional: extracted reference data if SKILL.md gets unwieldy
└── phase-skeleton.md     ← optional: intermediate artifact, phase list + frame-coverage audit
```

Symlinked into the learner's active agent registry:
- `~/.claude/skills/<skill-name>/` for Claude Code
- `~/.agents/skills/<skill-name>/` for Codex

Distributed bundle root:
- `plugins/pos-builder/skills/<skill-name>/` — shipped copy used by plugin installs
- `plugins/pos-builder/catalog/skill-catalog.json` — bundled runtime catalog

Authoring-only reference data:
- pedagogical use-case catalog
- architecture-block catalog

## SKILL.md structure

```markdown
---
name: <skill-name>
description: >-
  <one-line description + when-to-invoke triggers, including Russian phrasings>
---

# <Human-readable skill name>

> **Script instructions:** Следуй этому скрипту... (runtime discipline for the agent)
>
> ⚠️ **NARRATION CONTRACT — strictly enforced.** `Action (silent, no learner output):` lines are executed with ZERO learner-visible output. Never re-wrap them under `Build:` or `Say:`. JSON field names, snake_case keys, dot-paths, and labels from `state-contract.md` MUST NOT appear in learner-visible text. If a phase needs visible confirmation, use one plain Russian outcome sentence or emit nothing.

## Your Role
<2–4 sentences: what the learner is building + tone>

## Behavioral rules
<Numbered list, ~10 items — see "Course-wide behavioral principles" below>

## Data dependencies
<Files this skill reads at runtime: POS_HOME/learner-state.json, bundled skill-catalog.json, external docs>

## State contract
<Short prose summary + path to `./state-contract.md`. Keep the JSON contract in the sibling file, not inline in SKILL.md.>

## Resume Logic
<Phase 0 entry probe: state read → continue / start over / done-but-revisit branches>

---

## Fixed frame
### End state
### Mental models taught
### Required gates
### Forbidden

---

## Behavioral body
### Phase 0 — Entry probe
### Phase 1 — ...
...
### Phase N — Track and handoff

Every pause or completion branch ends with the literal token `===END-OF-SKILL===` on its own line. Do not use an HTML comment sentinel.

---

## References
```

## The Fixed frame sections

### End state

What the learner **has** when the block completes. Every item must be:

- **Verifiable** — the agent (or learner) can check it exists.
- **Concrete** — a file at a path, a config value, a state-file field, an installed package — not "vault is well organized" but "`_meta/naming.md` exists".
- **Bounded** — scoped to this block; don't leak next-block work.

Include the **wow-moment artifact** here if applicable (or mark explicitly "N/A — plumbing block").

### Mental models taught

3–5 named principles the block must plant.

Every mental model has two identifiers:

- `slug` — the canonical course-wide identity. The registry and `mental_models_taught` state both use the slug.
- `MMn` — the file-local handle used by that skill's frame coverage and body references.

Format:

```
1. **`slug` (MMn, new|reused).** 1–2 sentences. *Reinforced across the rest of the course* — if cross-block.
```

Cross-block reinforcement matters: if the `adapter` MM is taught in calendar, the email block should **reuse** that slug rather than mint a synonym. Models are the shared vocabulary.

Additional rules:

- MM numbers are **not** a course-wide registry. They are local to a skill. Sparse numbering is allowed when a skill preserves inherited labels for reused MMs.
- `mental_models_taught` is **slug-only**: `mental_models_taught.<slug> = { "at": "<ISO8601>", "by_skill": "<skill-name>" }`.
- Every reused MM needs a one-line concept anchor that makes sense standalone and does **not** name the prior block.
- A later skill may deepen a reused MM, but it keeps the same slug and does not overwrite first-teach attribution. If the concept changes materially, mint a new slug.

### Canonical MM registry

The registry below is the current source of truth for the shipped MM surface in this bundle.

#### Foundation and build-surface

- `build-without-traditional-coding` — Persistent personal systems no longer require traditional coding; the bottleneck shifts to clear thinking and clear requests. First taught in `pos-intro`.
- `conversation-is-build-surface` — The conversation window is where the system gets assembled, not just where questions get answered. First taught in `pos-intro`.
- `understanding-through-action` — In POS, you do not study the whole system first and build later; you take the next concrete step, and understanding accumulates through action. First taught in `pos-intro`.
- `think-aloud-not-dictation` — With an LLM, voice input can be messy thought, not perfect dictation. First taught in `pos-stt-setup`.
- `system-wide-voice` — Voice input becomes much more valuable when it follows the learner across apps instead of living inside one window. First taught in `pos-stt-setup`.
- `data-ownership` — Knowledge should live in files the learner owns; SaaS note apps are tenancy, local Markdown is property. First taught in `pos-vault`.
- `md-is-world` — Markdown is the durable substrate; the app is a viewer. First taught in `pos-vault`.
- `write-or-not-exist` — If it is not written into an agent-readable surface, it does not exist for the agent. First taught in `pos-vault`. Reused by `pos-goals`, `pos-morning-brief`, and `pos-day-summary`.
- `naming-first` — Choose the naming convention before the content or import work starts. First taught in `pos-vault`.
- `ontology-3plus1` — The vault needs a deliberate on-disk shape the learner can navigate by name. First taught in `pos-vault`.

#### Adapters, trust, and data surfaces

- `adapter` — An adapter is the bridge that lets the agent reach cloud or external data. First taught in `pos-calendar`. Reused by `pos-email`, `pos-telegram`, and `pos-tasks`.
- `voice-agent-over-ui` — For natural-language work, voice plus agent is often faster than fighting the UI. First taught in `pos-calendar`.
- `agent-as-assistant` — What the learner used to tell a human assistant, they can now tell the agent. First taught in `pos-calendar`. Reused by `pos-email`, `pos-telegram`, and `pos-tasks`.
- `personal-vs-corporate` — Personal and work surfaces live in different trust zones, policies, and blast radii. First taught in `pos-calendar`. Reused by `pos-email`, `pos-telegram`, and `pos-tasks`.
- `scopes-risk` — Different permissions imply different levels of trust and different levels of possible damage. First taught in `pos-calendar`. Reused by `pos-email`, `pos-telegram`, and `pos-tasks`.
- `time-awareness` — Once the calendar is connected, the agent can reason about today, this afternoon, and this week. First taught in `pos-calendar`.
- `inbox-as-flow` — Inbox-like surfaces are flow, not storage; what matters must leave the stream. First taught in `pos-email`. Reused by `pos-telegram`.
- `sender-trust` — Trust the sender before trusting the message. First taught in `pos-email`.
- `transport-vs-memory` — Telegram is good for transport; long-lived memory belongs elsewhere. First taught in `pos-telegram`.
- `whitelist-as-boundary` — A named whitelist is a real boundary; "only the important ones" is not. First taught in `pos-telegram`.
- `chat-as-ontology` — A chat is a role-bearing object, not just a text stream. First taught in `pos-telegram`.

#### Goals, planning, and review loops

- `goals-anchor` — Written goals stabilize the agent's alignment instead of forcing it to rebuild intent from scraps. First taught in `pos-goals`.
- `recall-is-signal` — What surfaces first in recall is signal; silence is signal too. First taught in `pos-goals`.
- `review-rhythm-keeps-goals-alive` — Goals stay live through recurring review, not one-time capture. First taught in `pos-goals`.
- `learner-controls-content` — The agent proposes a starting composition; the learner chooses the actual content surface. First taught in `pos-morning-brief`. Reused by `pos-day-summary`.
- `more-context-more-delegation` — The more stable context the agent has, the more decision work it can responsibly take on. First taught in `pos-morning-brief`.
- `scheduled-not-delivered` — A scheduler firing is not the same as a human actually receiving the output. First taught in `pos-morning-brief`. Reused conditionally by `pos-day-summary`.
- `plan-review-loop` — Morning planning and evening review are one learning loop for both the learner and the agent. First taught in `pos-day-summary`.

#### Repo, tracker, and authoring work

- `git-vs-github` — Local change tracking and the hosted collaboration surface are different layers. First taught in `pos-github-setup`.
- `issues-are-tasks-and-memory` — A GitHub issue is both a task and a re-readable memory thread. First taught in `pos-github-setup`.
- `stakeholder-sandwich` — The human owns the why and the constraints; code is the implementation in between. First taught in `pos-basic-vibecoding`.
- `spec-is-contract` — The spec is the contract; no spec, no code. First taught in `pos-basic-vibecoding`.
- `spec-drift` — The spec is living and code drifts; the agent must update the spec as reality changes. First taught in `pos-basic-vibecoding`.
- `skills-as-programs` — Skills and prompts are programs written in natural language. First taught in `pos-basic-vibecoding`.
- `skill-packs` — Reusable procedures can be packaged and installed as skills. First taught in `pos-basic-vibecoding`.
- `tracker-as-shared-memory` — A tracker can hold shared working memory, not just a checklist. First taught in `pos-tasks`.
- `agent-memory-home` — The agent needs one explicit long-lived home for its state, even if many trackers are connected. First taught in `pos-tasks`.
- `attention-is-the-resource` — Attention is the real scarce resource triage is trying to protect. First taught in `pos-triage`.
- `ranking-needs-priorities` — Ranking without explicit priorities is guesswork. First taught in `pos-triage`.
- `read-only-agency` — The agent can read and summarize, but the learner remains the actor. First taught in `pos-triage`.
- `draft-beats-ideal` — A rough first pass plus iteration beats waiting for a perfect first draft. First taught in `pos-triage`.

#### Advisors and grounded synthesis

- `llm-distillation` — LLMs can read very large corpora and extract the essence fast enough to remove "I don't have time to read all of this" as a blocker. First taught in `pos-advisors`.
- `persona-from-corpus` — A grounded public corpus is enough to build a usable advisor model of how someone thinks. First taught in `pos-advisors`.
- `advisor-not-verdict` — Advisors widen the set of angles; the final decision remains human. First taught in `pos-advisors`.
- `research-provider-saves-context` — Specialized research providers can spend context budget on the reading pass and leave the main conversation budget for synthesis and decisions. First taught in `pos-advisors`.

### Required gates

Hard checkpoints the unbounded body MUST honor. Each gate is a consent or safety checkpoint:

- Confirmations before irreversible actions (overwrites, sends, deletes, writes to learner files).
- Ordering constraints (pick naming before first import; backup before first write).
- Critical verifications (show OAuth scopes on a screen the learner sees; confirm credential location is NOT inside the vault).

Missing gates → blocks can fail destructively (cf. SSH-lockout scenario in #15).

Convention: name gates `G1`, `G2`, … so the body can reference them (`**Frame coverage:** G3, G9`).

### Forbidden

Explicit anti-patterns. Each forbidden is something the body MUST NOT do:

- Filesystem boundaries (never touch files outside the chosen dir).
- Data-shape violations (credentials never in Obsidian vault).
- Workflow shortcuts that defeat the block's point (auto-send AI replies; silent bulk actions).

Convention: name `F1`, `F2`, … for body cross-reference.

## Course-wide behavioral principles

Every skill's `## Behavioral rules` section restates these. They apply throughout ALL phases.

1. **Russian for learner / English for runtime instructions.** Section headers, `Build:` constraints, internal comments — English. Anything the learner sees — Russian.
2. **«ты», not «Вы».**
3. **Bite-sized.** Each `Say:` = 1–3 short phrases. Always a `Check:` between `Say:`s. Never a wall of text — split across multiple Say/Check pairs. **Narrow exception — mobilizing MM delivery:** when one mental model lands as a single coherent observation, an uninterrupted 3–5-phrase `Say:` is acceptable *only if* simulation runs (Phase 3) confirm the next learner turn shows engagement (emoji / «да») rather than confusion. Default remains bite-sized; the exception is per-MM, must be explicitly marked in the body, and reverts to default if simulation evidence does not support it.
4. **First principles.** Mental models derived from a concrete observation, not asserted as rules. The learner should feel "yes, obviously" — not "ok, I'll trust you".

   **Altitude test for mental models.** A line earns a slot in `### Mental models taught` only if it passes: *"Can a learner restate this MM one year later, without having seen this skill again?"* If no, it's not an MM — it's an implementation detail in disguise:

   - Implementation constraint or guard → demote to **Required gates**.
   - How-to / quick-win / product-specific tactic → demote to the body **Phase** that already teaches it.
   - Anti-pattern → demote to **Forbiddens**.
   - Skill-internal trivia with no portable lesson → delete.

   Concept altitude (good — passes the test):
   - "LLMs can read a lot of text and extract the essence." (pos-advisors)
   - "Markdown outlives the app; Obsidian is just a viewer." (pos-vault)
   - "Different permissions imply different levels of trust and potential damage." (pos-email / pos-calendar / pos-tasks)

   Rule altitude (bad — looks like an MM, isn't):
   - "Citation-gated distillation" → belongs in Gates (it's a check the body must pass before saving a persona card).
   - "Parallel isolated sub-agents" → belongs in Body / runtime logic (it's how the build phase executes, not what the learner internalizes).
   - "Claude Code voice mode is the first zero-install thing to try" → belongs in Body (it's a Phase 2 quick-win tactic, not a portable lesson).
5. **Plain talk for terminal things.** Before any new terminal concept (brew, keyring, OAuth, WSL, cron) — one sentence "что это", then the command.
6. **Ask before doing.** One-line Russian preview before any `Action (silent, no learner output):` or `Build:` — tell the learner what's about to happen.
7. **No meta-commentary.** Never «по скрипту», «сейчас фаза 4», «мне инструкция велит», «сейчас я объясню».
8. **Return to script.** Off-topic → brief answer → return to the current `Check:`.
9. **Never overwrite.** Agent-config files (`CLAUDE.md`, `AGENTS.md`) and other learner-owned files are append-only. If a section already exists, show a diff, ask before merging.
10. **Confirm before destructive ops.** Every irreversible action is gated. Never overwrite a file without an explicit `Check:`. Never touch files outside the scoped dir.
11. **Save state at phase transitions only.** Keep working data in memory during a phase; one write to `learner-state.json` when crossing into the next phase (~12 writes total per skill).
12. **Honest about cost.** If a tool is paid (Obsidian Sync, anything else) — say so upfront, don't hide it.

Adapter-specific additions (calendar, future email, future tg-inbox, future tasks):

13. **Never mix accounts.** Personal token never used for work calendar / email and vice versa. Separate auth per account.
14. **Never silent.** Don't connect all calendars / mailboxes / chats at once. Don't request `write`/`delete`/`send` scopes before the learner explicitly consents.
15. **Credentials never in Obsidian.** Tool-native dir (keyring / env-file) only — never in the learner's vault.

Simulation-discovered principles (2026-04-18, from pos-calendar simulation R12 — 4 persona×teacher runs):

16. **Skip pre-answered Checks.** If the learner has already answered a Check earlier in the same session (including Phase 1 pitch), acknowledge-and-confirm («Уже слышал — личный, так?»), do not re-ask the canonical question. Applies to provider, OS, write/delete preference, token storage location — every parametric Check.
17. **State writes are silent.** `Action (silent, no learner output):` is the required tag for state reads and writes. Never print `Build (narrated): записываю provider: "google", is_work_calendar: false` to the learner — no JSON keys, no field names, no key-value syntax in learner-visible output. Keep the canonical JSON schema in `state-contract.md`, not inline in SKILL.md. If you need to acknowledge a decision, translate to one plain Russian sentence. Infrastructure phases (Phase 0 resume probe, pure state reads) emit nothing to the learner.
18. **Pre-warn predictable anxiety.** If the next step shows the learner something likely to scare them (an "unverified app" warning, a scope list that looks broader than they agreed, a terminal command after promising "almost none"), warn one sentence before. Reactive reassurance costs trust reactive; proactive framing preserves it.
19. **One mental model per Say, with a Check.** Never stack two MMs in one turn. Every mental model gets its own derivation + Check before the next MM or phase transition. If the learner has asked a concrete how-to question, defer MM delivery — don't stuff theory in between their question and the answer.
20. **Track pending siblings.** If the learner names multiple sibling items during Phase 1 (multiple providers, multiple accounts, multiple calendars), record them as `pending_<domain>` in the state contract (e.g., `arch_blocks.calendar.pending_providers: ["outlook", "yandex"]`) and surface one sentence in the pitch: «начнём с X, остальных подключим по тому же шаблону». The next-block recommender in the final phase should queue them.
21. **Proof before choice — qualified by consequence.** When the learner must pick between tools / scopes / methods with **durable consequences** (money spent, account created, credential granted, scope elevated, external data written), surface the receipts first — research summary, criteria table, 1–2 concrete candidates with named tradeoffs — then ask. Never chain research + choice + next-phase build into one turn. The "Build (narrated)" pattern in simulation makes fake research indistinguishable from real; in live runs the research must be real, with links or tool output shown. **For low-consequence picks (cosmetic, reversible, default-acceptable — e.g., tag color, default text editor, default backup folder name), `default-with-opt-out` is acceptable:** a single `Say:` names the default plus an «если хочешь другое, скажи» invitation, no research ceremony.

22. **Error-visibility contract for external-dependency Builds.** Every `Build:` that depends on something outside the learner's machine (OAuth flow, package registry, provider API, scheduler unit start, network call) MUST define an explicit failure-path `Say:` in the body. The failure path states: (a) what happened, in plain Russian — one sentence, no stack trace dumped to the learner; (b) the offered next action — `retry` / `switch provider` / `manual escalate` / `skip and revisit`; (c) where the evidence lives (log path, command to re-run with verbose flag) so a debugger can pick it up later. **Forbidden:** silent failure, generic «попробуй ещё раз» without diagnosis, agent improvising recovery steps not in the frame. This formalizes principle 18 ("pre-warn predictable anxiety") for the failure case as well as the happy case.

23. **Numeric menus for ≥3-choice Checks.** Learner-facing Checks with three or more options must present a numeric menu with Russian labels, e.g. `1 macOS, 2 Linux, 3 Windows`. Learner types the digit. Slug/phrase synonyms accepted silently. 2-choice `да`/`нет` Checks exempt. Rationale: humans are token-limited (voice transcription, typos, cognitive load); typing `1` beats typing a slug.

24. **`mental_models_taught` tracking with two-branch teach/remind.** Every taught mental model has a canonical slug. First teach writes `mental_models_taught.<slug> = { "at": "<ISO8601>", "by_skill": "<skill-name>" }`. Later encounters split the teach block into two branches keyed on whether the slug is already populated — full teach if not, short reminder if yes. Reminder branches MUST NOT write state and MUST NOT reference the prior block by name.

## Two-phase teaching flow

Every block mixes two modes. The frame decides which mode each phase is in.

### GUIDED phase

Strict scripted structure. Used for: teaching concepts, making decisions, consent gates.

- `Say:` — output literally, word for word. Russian. 1–3 phrases.
- `Check:` — STOP and WAIT for the learner's answer. One question.
- `Action (silent, no learner output):` — silent execution (read state, write state, make a folder). Runtime only.
- Sidebar questions from the learner are allowed; answer briefly and return to the current `Check:`.

Voice and rhythm: bite-sized, first-principles, «ты», no meta-commentary, honest about cost.

### BUILDING phase

Unbounded execution within `Build:` constraints. Used for: installs, configs, verifications, creating files.

- `Build:` — freeform task. Agent researches tools at runtime, branches by OS/provider, iterates until the phase's completion gate passes.
- **Constraints** — English bullets inside the `Build:` block. These are inherited from the Fixed frame (forbiddens + gates) plus phase-specific constraints.
- **Completion gate** — concrete condition that ends the phase (file exists, API returns a value, learner confirms output). NOT "does this make sense?" — that's an understanding gate and belongs in GUIDED.

Gate types inside phases:

- **Understanding gate** — "Does this make sense?" Quick, conversational. GUIDED.
- **Completion gate** — "Has X finished?" Real wait, needs a fallback/troubleshooting path if it fails. BUILDING.

## Per-phase frame-coverage audit

After authoring, annotate each phase in the body with which frame items it satisfies:

```markdown
### Phase 5 — <name>

**Frame coverage:** **G2** (confirm provider), **G3** (personal-vs-work warning), **MM4** (personal vs corporate), **F5** (work account warn-only), **F10** (no account mix).
```

Read the annotations top-to-bottom after authoring — every gate and mental model from the frame must appear at least once. Gaps = frame items the body doesn't cover = broken skill.

## Body generation contract

When Claude authors the body from the frame:

1. **Phase 0 — Resume logic.** Read `learner-state.json` first. Branch: fresh start / continue from saved phase / done-but-revisit.
2. **Phase 1 — Read diagnostic + pitch.** Pull prior-skill state: provider hints from `inventory`, `stt_status`, already-done `arch_blocks.<sibling>`. One-line permissive pitch in Russian — what the learner gets at the end.
3. **Phase 2..N — Interview-driven progress.** Each phase teaches or decides or builds. Gate-respecting. Saves state on transition.
4. **Final phase — State writeback + handoff.** `arch_blocks.<block_id>` populated. Recommend the next block per the diagnostic route.

Voice: conversational, not robotic. Interviews where End state allows variation (which calendar, which accounts, which naming). Insists where End state is fixed (mandatory mental models, required artifacts).

## Author checklist (before handing frame to Claude for body generation)

- [ ] End state items are verifiable, not aspirational.
- [ ] At least one mental model the rest of the course can reference.
- [ ] Every irreversible action has a Required gate (`G<n>`).
- [ ] Prereq surface audited end-to-end. Core semantic inputs (required data, state, vault artifacts) belong in the frame prereq list. Branch-specific enablers belong in runtime gates. Do not declare the same foundation twice: if `/pos-goals` is already hard and `/pos-goals` itself hard-depends on `/pos-vault`, do not also hard-list `/pos-vault` here.
- [ ] Every forbidden has a concrete reason tied to a gate or end-state item.
- [ ] References name every external thing the body needs (docs, state files, other skills, 3rd-party tools).
- [ ] Wow-moment defined (or explicitly N/A for plumbing blocks).
- [ ] State contract block defined under `arch_blocks.<block_id>`.
- [ ] If this skill maps to a course use case, that use-case id exists in the authoring catalog.

## Review loop conventions

## Course-wide conventions

- **8 life domains** (used wherever a block touches goals / about-me / domain interviews): HEALTH · SELF · RELATIONSHIPS · CAREER · FINANCIAL · CREATIVITY · CONTRIBUTION · FUN/RECREATION
- **3+1 ontology** (default structure proposal for vault / knowledge blocks): raw-data / knowledge / engagement (3 epistemological) + working-memory (1 prescriptive)
- **Naming convention** (default): Pavalev's schema — `{project} {type} description – YYYY-MM-DD.md`. Canonical source: https://github.com/ai-mindset-org/pos-sprint/blob/main/practice/naming-convention-vault-structure.md
- **Supported agents** — course runtime officially supports Claude Code and Codex only.
- **Agent-config resolution** — `learner_profile.primary_agent == "claude-code"` → write to `CLAUDE.md`; `learner_profile.primary_agent == "codex"` → write to `AGENTS.md`. If `learner_profile.keep_agent_configs_in_sync == true`, keep both files aligned.
- **Rules-of-use template** — the learner's resolved agent-config file gets a `## <Block name>` section with English labels, Russian values. Append-only; diff-and-confirm if section exists.
- **Runtime home** — resolve `POS_HOME` first: environment override if present, otherwise `~/.pos-builder`.
- **Learner state files** — `learner-state.json`, `my-architecture.md`, and `my-system.md` live in `POS_HOME`. Skills read/write the shared state there, not in the learner's project CWD.
- **Bundled skill catalog** — runtime routing and next-block recommendations read from the bundled `skill-catalog.json`, not from authoring-only catalogs.

## Permissive framing principle

The calendar authoring loop discovered the right balance (R6 → R7 pass, 2026-04-17):

**Loosen** (hardcoding removed, agent researches at runtime):
- Package names (`@cocal/google-calendar-mcp` → "агент подбирает MCP-сервер")
- CLI commands (`gog calendar events --json` → "tool-specific")
- OAuth filenames, env var names, scope URLs
- OS × tool install matrices
- Scheduler choice (cron vs systemd timer)

**Keep prescriptive** (gate-preserving):
- "Credentials NOT in Obsidian vault" (F6/F7)
- Append-only agent-config file(s) with diff+confirm (F8)
- Concrete wow-moment test event (G11)
- Rules-of-use template lines (F3, F4, F11, F12)
- Backup format fixed by state contract (JSON)
- Every `G<n>` from the frame

Rule of thumb: if removing the hardcoding would let the agent make a better choice based on the learner's OS / provider / account type, loosen it. If removing it would let the agent skip a safety check, keep it.

## Related documents

- `docs/block-runtime-pattern.md` — soft-direction sketch of a 6-step learner-facing flow (pitch → context → propose → confirm → build → track). Predates the thin-frame pivot; the body shape that shipped skills actually run is defined here in the Body-generation contract section.
- `docs/blocks/diagnostic-spec.md` — diagnostic spec (the only skill NOT under this framework — it's the entry point with its own shape).
