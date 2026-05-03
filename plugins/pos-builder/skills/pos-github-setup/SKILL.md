---
name: pos-github-setup
description: >-
  Use when the learner types `/pos-github-setup`, asks to set up GitHub for the
  course, or needs issue-based course memory before later build steps.
---

Read this file on entry. Constraints and end state define the boundaries; flow guides the session.

**Role:** Help a non-technical learner set up GitHub for the course in one short sitting -- account, CLI, private repo, labels, issue-based roadmap, and rules-of-use.

## End state

When done, the learner has:

1. A verified GitHub handle paired into learner state
2. `gh` CLI installed and authenticated via device flow (not PAT)
3. A private POS repo at `https://github.com/<handle>/<repo>`
4. The distilled 15-label set applied on that repo
5. Either: an open course-roadmap umbrella issue plus one child issue per candidate skill (done items created and closed with summary, in-progress items annotated, not-started items open) -- or a closed setup-history issue when roadmap seeding was declined and the fresh repo had zero issues
6. Rules-of-use proposed; on acceptance written to the learner's agent-config target set
7. `arch_blocks.github_setup.status == "done"` with state contract populated
8. A wow moment where the agent re-reads the resulting GitHub artifact and names it as course memory

## State

Fields this skill reads and writes in `learner-state.json` under `arch_blocks.github_setup`. `last_completed_step` = last finished step; resume starts at the NEXT step.

- `status` (string: done|not_yet) -- written Steps 1, 9
- `last_completed_step` (number) -- written at each step
- `github_account` (string) -- written Step 3
- `gh_cli_installed` (boolean) -- written Step 4
- `repo_url` (string) -- written Step 6
- `labels_applied` (boolean) -- written Step 7
- `course_seeding_adopted` (boolean) -- written Step 8
- `course_umbrella_issue_url` (string|null) -- written Step 8
- `umbrella_linked_done` (boolean) -- written Step 8
- `seeded_child_issue_urls` (object keyed by skill slug) -- written Step 8
- `setup_history_issue_url` (string|null) -- written Step 8
- `rules_adopted` (boolean) -- written Step 9
- `completed_at` (ISO8601) -- written Step 9

On entry: read `learner-state.json`. Check `last_completed_step` and resume from the next step. Save state as the flow progresses -- write relevant fields immediately at each milestone.

Read-only dependencies: `learner_profile` (prereq check), `learner_profile.primary_agent`, `learner_profile.keep_agent_configs_in_sync`.

**Legacy migration:** if `seeded_child_issue_urls` contains any numeric keys, migrate once on entry before resume: read `skill-catalog.json`, match each numeric key to the catalog entry whose `primary_use_case_id` matches, move the URL under the slug key, drop numeric duplicates, persist, then continue.

**Branch-exclusive field normalization:** when `course_seeding_adopted == true`, `setup_history_issue_url` must be `null`. When `course_seeding_adopted == false`, `course_umbrella_issue_url` must be `null`, `umbrella_linked_done` must be `false`, `seeded_child_issue_urls` must be `{}`. Normalize on write.

## Mental models

1. **git-vs-github (MM4).** `git` is the local change-history machinery; `GitHub` is the cloud surface where that history, issues, and projects live. *Teach* in Step 2; *remind* if the learner confuses the two later.
2. **issues-are-tasks-and-memory (MM5).** An issue is both a task with a done-state and a re-readable memory thread the agent can return to later. *Teach* in Step 2; *remind* in Step 8 when creating issues.

## Constraints

1. **Device flow only.** `gh auth login --web` with scopes. If auth fails, retry or pause -- never offer a PAT path.
2. **No account creation in-session.** Never collect GitHub email, password, or 2FA code. Point to `https://github.com/signup`; ask only for the handle afterward.
3. **No repo writes before labels.** No `gh issue create`, `gh repo edit`, push, or any repo-mutating action between repo creation and full label verification.
4. **Labels before first issue.** The full 15-label set must be verified before any issue is created.
5. **Sequential issue batch with per-issue persistence.** Create child issues one at a time. Write each URL to `seeded_child_issue_urls[<slug>]` immediately after creation. On per-issue failure: stop, offer retry-one or abort.
6. **Append-only agent-config.** Never overwrite an existing `## GitHub` section. The only allowed writes: append a new `## GitHub` at the end of a file that has none, or create the file with the block if it does not exist. Merging is the learner's job.
7. **Present, confirm, write.** Show proposed content to the learner, get confirmation, then write -- for all artifacts (labels, issues, rules-of-use, state).
8. **Research install details at runtime.** Detect OS with platform facts; never hardcode an OS x package-manager matrix.
9. **One mental model at a time.** Teach the concept before naming the term.
10. **Seed set: live route first, catalog fallback.** Use learner's route from diagnostic if available; else `my-architecture.md` recommended-route; else shipped catalog entries with `status == "shipped"` and `menu_visible == true`, excluding `/pos-diagnostic` and `/pos-github-setup`.
11. **Honest about cost.** When something costs money, say so up front.
12. **Feedback on-demand.** Do not front-load feedback prompts. If the learner asks or shows clear frustration/enthusiasm, offer to route to the feedback skill.

## Constants

**Distilled label set (15 labels):**

- Priorities: `p0`, `p1`, `p2`, `p3`
- Types: `task`, `bug`, `decision`, `scaffold`, `content`
- Umbrella marker: `epic`
- Domains: `infra`, `adapter`, `capability`, `runtime`, `memory`

Colors are not locked.

**Skill kind -> domain label mapping:**

| Kind | Domain label |
| --- | --- |
| `foundation` | `infra` |
| `adapter` | `adapter` |
| `capability` | `capability` |
| `workflow`, `alignment`, `cross_cutting`, `runtime`, `diagnostic` | `runtime` |

**Default titles:**

- Umbrella issue: `POS-builder: собираю свою систему`
- Setup-history issue: `pos-github-setup: установка завершена`
- Default umbrella domain label: `runtime`

**`gh auth login` scopes:**

| Scope | Plain Russian gloss |
| --- | --- |
| `repo` | читать и писать в твои репозитории |
| `read:org` | видеть организации, в которых ты состоишь |
| `gist` | работать с gist |
| `workflow` | редактировать GitHub Actions, если потом понадобится |

**`## GitHub` rules-of-use text (write verbatim on acceptance):**

Use the variant matching the runtime. For generic, use the Claude Code variant.

Claude Code variant (`claude-code` or `generic`):

```markdown
## GitHub

- **Issue-first.** Для любой содержательной работы — сначала найди или создай open issue. Быстрые вопросы и мелкие правки issue не требуют.
- **Session UUID.** При начале работы над issue — первым комментарием оставь идентификатор сессии (`session: <id>`).
- **Прогресс в комментариях.** На ключевых точках (решение принято, блокер, готовый кусок) — комментарий под issue: что сделано, что дальше, блокеры.
- **Закрытие сессии.** В конце сессии — комментарий с итогом и следующим шагом.
- **Метки.** На каждый issue — одна priority (`p0`/`p1`/`p2`/`p3`), один type (`task`/`bug`/`decision`/`scaffold`/`content`), `epic` для umbrella, один domain при необходимости.
- **Доступ через CLI.** Работай с GitHub через `env -u CLAUDECODE gh ...`.
```

Codex variant (`codex`):

```markdown
## GitHub

- **Issue-first.** Для любой содержательной работы — сначала найди или создай open issue. Быстрые вопросы и мелкие правки issue не требуют.
- **Session UUID.** При начале работы над issue — первым комментарием оставь идентификатор сессии (`session: <id>`).
- **Прогресс в комментариях.** На ключевых точках (решение принято, блокер, готовый кусок) — комментарий под issue: что сделано, что дальше, блокеры.
- **Закрытие сессии.** В конце сессии — комментарий с итогом и следующим шагом.
- **Метки.** На каждый issue — одна priority (`p0`/`p1`/`p2`/`p3`), один type (`task`/`bug`/`decision`/`scaffold`/`content`), `epic` для umbrella, один domain при необходимости.
- **Доступ через CLI.** Работай с GitHub через `gh ...`.
```

## Flow

### Step 1 -- Entry and resume

Check prerequisites:
- **Hard:** `learner_profile` must exist (from `/pos-diagnostic`). If absent, tell the learner to run diagnostic first. Stop.
- **Incomplete diagnostic:** if `learner-state.json` exists and `complete != true` (diagnostic started but not finished): offer 1 run/continue diagnostic first, 2 continue without full diagnostic (explicit opt-in required).
- **Soft:** if `learner-state.json` exists but has no `arch_blocks.github_setup`, offer: 1 run diagnostic first, 2 continue without it.
- **Done:** if `status == "done"`, tell the learner GitHub setup is already complete. Stop.
- **Resume:** if `last_completed_step` exists, tell the learner where they left off and resume from the next step. On resume, verify artifacts from completed steps still exist: `github_account` is set, `gh auth status` succeeds, repo is accessible via `gh repo view`, labels are present. If a prior artifact is missing or invalid, route back to the step that creates it.

Write (fresh start): `status = "not_yet"`, `seeded_child_issue_urls = {}`, `last_completed_step = 1`.

### Step 2 -- Opening pitch (MM4, MM5)

Deliver verbatim in Russian as a single block:

> «В этом блоке мы подключим тебе GitHub аккаунт, а также создадим первые GitHub Issues и репозиторий.
>
> Если раньше доводилось работать в Microsoft Word, то возможно помнишь, что в нем есть режим отслеживания правок, где они видны в тексте документа и подписаны. Пока их не приняли, они не стали эталонной частью текста. Git – это специальный инструмент, который делает то же самое с любыми файлами и папками. Он ведет и хранит историю изменений. Когда их накопилось несколько и ты хочешь их зафиксировать — делаешь commit, это как нажать «принять».
>
> GitHub – по большому счёту то же самое, только для совместной работы. Git и GitHub – как локальный Word и Google Doc. Это не единственный такой инструмент, но наиболее распространенный на сегодня.
>
> GitHub Issues (буквально «проблемы», в русской версии GitHub «задачи») это встроенный в GitHub механизм управления задачами. Он простой, очень гибкий, и бесплатный – идеальный вариант для того, чтобы начать вести задачи для своих агентов. Если у тебя уже пройден /pos-vault, то ты уже знаешь, что LLM ничего, совсем ничего не запоминают и всё, что может пригодиться в работе потом необходимо записывать. GitHub Issues это простой способ сделать своего рода внешнюю память по тому, что сделано, с каким результатом, и что еще предстоит.
>
> Конечно, работать со всем этим ты будешь не своими руками, а через агента. Он сам будет делать коммиты, сам будет создавать, изменять и закрывать задачи.
>
> Вопросы, или идём дальше?»

Write: `last_completed_step = 2`.

### Step 3 -- GitHub account gate

Ask whether the learner already has a GitHub account. If yes, capture the handle. If no, point to `https://github.com/signup` -- account creation happens entirely in the browser. Wait for the handle.

Verify the handle against the public user endpoint (unauthenticated check). On failure, ask them to try again.

Write: `github_account`, `last_completed_step = 3`.

### Step 4 -- `gh` CLI install

Briefly explain what `gh` is (the official CLI for working with GitHub from the terminal). Probe `command -v gh` and `gh --version`. If missing: detect OS at runtime, pick the install command, confirm with the learner, run it, re-verify. On install failure: show what happened, offer retry or pause.

Write: `gh_cli_installed = true`, `last_completed_step = 4`.

### Step 5 -- Device-flow auth

Before starting, show the four scopes in plain Russian using the scope table from Constants. After confirmation, run `env -u CLAUDECODE gh auth login --web --scopes "repo,read:org,gist,workflow"`. Let the device code render in the terminal.

Deliver the scope pre-warning verbatim:

> «Сейчас будет вход через браузер — GitHub покажет одноразовый код, ты его подтвердишь. Токен вручную вставлять не нужно. GitHub запросит четыре разрешения: `repo` — читать и писать в твои репозитории; `read:org` — видеть организации, в которых ты состоишь; `gist` — работать с gist; `workflow` — редактировать GitHub Actions, если потом понадобится.»

Verify with `env -u CLAUDECODE gh auth status`. Then read `env -u CLAUDECODE gh api user --jq .login` and confirm it matches `github_account`. If mismatch: offer re-login, update handle, or pause. On 'update handle': clear `github_account`, route back to Step 3 for recapture before allowing repo creation.

Write: `last_completed_step = 5`.

### Step 6 -- Repo creation

Propose default name `pos-<handle>`, allow one custom override. Create a private repo with a short Russian description and no README. Verify with `env -u CLAUDECODE gh repo view`.

Write: `repo_url`, `last_completed_step = 6`.

### Step 7 -- Label set

Explain the label system in plain Russian — not just what labels are, but why they matter: agents can't scroll through a list of issues the way a human would; labels let the agent instantly filter by urgency, type of work, and area of the system. Without them, the agent has to read every issue title to decide what to work on. Three groups (priority — what's urgent; type — what kind of work; domain — which part of the system) plus `epic` for umbrella issues that group smaller tasks. Present the proposed 15-label set, then ask the learner whether this system works for them or if they'd like to adjust it (add, remove, rename). Adapt if they want changes — but note that downstream skills expect the distilled set, so warn if they remove a label that other skills use.

Once confirmed, apply all labels with `env -u CLAUDECODE gh label create -R "<handle>/<repo>"`. Use idempotent behavior (skip-if-exists or `--force`). Verify with `env -u CLAUDECODE gh label list`. If any missing after first pass: retry missing, re-verify. Do not proceed until all 15 are confirmed.

Write: `labels_applied = true`, `last_completed_step = 7`.

### Step 8 -- Course seeding

This is the most complex step. Two paths: accepted (umbrella + children) and declined (fallback).

**Proposal.** Pitch roadmap seeding in plain Russian: one umbrella issue for the course path, one child issue per upcoming block -- this becomes their roadmap in GitHub. Ask: 1 yes, 2 no. If yes, offer default umbrella title or custom.

**Accepted path.**

Compute the seed set: read `learner-state.json`, `my-architecture.md` if present, and `skill-catalog.json`. Resolve candidates per Constraint 10. Classify each candidate using its catalog `state_branch` field: look up `arch_blocks.<state_branch>.status`. For STT: check top-level `stt_status`. Classify as done / in-progress / not started:
- **done** -- `arch_blocks.<state_branch>.status == "done"` or equivalent high-confidence signal. For STT: `stt_status in ["installed","voice_mode_only","already_has"]`.
- **in-progress** -- partial `arch_blocks.<state_branch>` branch exists with `status != "done"`.
- **not started** -- no state signal.

*Empty seed set edge cases:*
- If umbrella already exists with children in state: keep `course_seeding_adopted = true`, skip creation, rebuild `## Linked issues` from stored URLs, write `umbrella_linked_done = true`. Continue.
- If no umbrella exists: set `course_seeding_adopted = false`, route to declined path.
- If umbrella exists but children empty: reuse the existing umbrella, attempt to populate children. If seed set is still empty, close the orphan umbrella with a comment, set `course_seeding_adopted = false`, route to declined path.

Before creating the umbrella issue, briefly remind the learner of MM5: each issue is both a task and a re-readable memory thread the agent can return to later.

*Umbrella creation:* Create with `env -u CLAUDECODE gh issue create -R "<handle>/<repo>"` -- title, body (one short Russian paragraph explaining this is the POS-builder roadmap), labels `epic` + `p2` + `runtime`. If umbrella already exists in state, reuse it. Write `course_seeding_adopted = true`, `course_umbrella_issue_url`, `umbrella_linked_done = false` immediately.

*Child issue batch:* For each candidate whose slug is already in `seeded_child_issue_urls`, skip and keep stored URL. For each missing candidate, create sequentially with `env -u CLAUDECODE gh issue create -R "<handle>/<repo>"`:
- Title: catalog `name_ru` (or `name`)
- Body: `Команда: <command | "Скоро">`, blank line, `menu_description`, blank line, `Relates to #<umbrella-number>`
- In-progress items: append `Статус: в процессе.` with state summary
- Planned (not yet shipped): append `Статус: Скоро.`
- Labels: `task` + `p2` + domain label from kind mapping
- Done items: immediately close with short Russian summary from learner state

Write each `seeded_child_issue_urls[<slug>]` immediately after creation. On per-issue failure: stop batch, offer retry-one or abort. After batch, sanity-check with `gh issue list -R <handle>/<repo> --label task --state all --json number,title,state`.

*Umbrella body edit:* Run `env -u CLAUDECODE gh issue edit <umbrella> -R "<handle>/<repo>" --body-file <tmp>` to replace or create the `## Linked issues` section (rebuild from stored URLs rather than appending) with one line per child (done items with ~~strikethrough~~). Verify with `gh issue view`. Write `umbrella_linked_done = true`, `setup_history_issue_url = null`.

**Declined path.**

Write `course_seeding_adopted = false` immediately. Check if repo has issues: `env -u CLAUDECODE gh issue list -R "<handle>/<repo>" --state all --limit 1`.
- If repo already has issues: write `setup_history_issue_url = null` (key present, value null). Skip fallback.
- If repo has zero issues: create one issue titled `pos-github-setup: установка завершена`, body summarizing completed setup, labels `task` + `p2` + `scaffold`. Write `setup_history_issue_url` immediately. Close with comment `завершено`. Verify close.

Normalize branch-exclusive fields after either path completes.

Write: `last_completed_step = 8`.

### Step 9 -- Rules-of-use, wow, and close

**Entry precondition.** Re-read `learner-state.json` and verify Step 8 fully landed before proceeding:
- If `course_seeding_adopted == true`: require `course_umbrella_issue_url` is a URL and `umbrella_linked_done == true`.
- If `course_seeding_adopted == false`: require `setup_history_issue_url` key exists in state (value may be null).
- If `course_seeding_adopted` is undefined: return to Step 8.

**Rules preview.** Explain in plain Russian: we're about to write a set of rules into the agent's instructions file. This is how the agent will know to always track work in GitHub Issues — without these rules, each new session starts blank and the agent won't follow the workflow on its own. These are directives for the agent, not documentation for you.

Resolve the primary target file from `learner_profile.primary_agent` (`CLAUDE.md` for Claude Code, `AGENTS.md` for Codex). If `primary_agent` is absent or unknown, infer the current runtime agent (Claude Code -> `claude-code`, Codex -> `codex`), confirm once with the learner, then resolve the target file. If `keep_agent_configs_in_sync == true`, extend to both files. Read each target; note whether `## GitHub` exists and whether it matches the exact block byte-for-byte.

Print the exact `## GitHub` block from Constants (runtime-correct variant). If a target has a different existing `## GitHub`, show a unified diff.

**Four branches:**

*Branch A -- no target contains `## GitHub`:* Offer to append to all targets. On accept: append, verify, write `rules_adopted = true`. On defer: write `rules_adopted = false`.

*Branch B -- some targets have exact block, some have no `## GitHub`:* Offer to append to missing files only. On accept: append to missing, verify, write `rules_adopted = true`. On defer: write `rules_adopted = false`. Stop.

*Branch C -- a target contains `## GitHub` but not the exact block:* Show diff. Never overwrite. Offer: 1 leave incomplete, 2 show diff again for manual merge. On defer: write `rules_adopted = false`. Stop.

*Branch D -- every target already has the exact block:* Skip consent. Write `rules_adopted = true`. Continue.

**What has been achieved.** If `rules_adopted == true`:

> «Теперь, когда правила прописаны, при работе в новой сессии можно говорить своему агенту продолжить работать с issue / задачей номер такой-то. Если агент сохранял результаты работы, то он сможет очень быстро продолжить с точки, на которой вы в прошлый раз остановились»

If `rules_adopted == false`:

> «Даже без прописанных правил, при работе в новой сессии можно говорить своему агенту продолжить работать с issue / задачей номер такой-то. Правда, без правил в файле агент может не делать это сам — придётся напоминать»

**Wow moment.** Direct the learner to look at GitHub themselves. Conditional on what was created:
- If `course_seeding_adopted == true` and umbrella URL exists and children non-empty: give the learner the umbrella URL and tell them to open it in a browser. Deliver: `«Открой эту ссылку в браузере — это твой маршрут по курсу. Каждая задача — это отдельный урок. Я вижу эти issue так же, как ты, и в следующий раз смогу подсказать, какой из уроков ещё не пройден.»`
- If `course_seeding_adopted == false` and `setup_history_issue_url` is a concrete URL: give the URL. Deliver: `«Открой ссылку — это твой первый след в GitHub. Я её вижу и смогу подхватить контекст даже в новой сессии.»`
- Otherwise: skip the wow text entirely.

**Final writeback.** If `rules_adopted` is undefined, stop without writing done. Otherwise: write `status = "done"`, `completed_at`, `last_completed_step = 9`. Tell the learner in one or two short Russian sentences that setup is complete. Mention they can leave feedback if they want.

## Rules

1. **Language and tone.** Speak Russian to the user. Use `ты`. Plain, simple language. Warm and calm, like explaining to a friend. All user-visible text must be Russian -- no English in status updates or messages.
2. **Silent state.** State reads and writes are invisible to the user. Never show JSON field names, keys, or dot-paths in user-facing text.
3. **Concept before jargon.** Derive `git`, `GitHub`, `issue`, `commit`, `device flow`, and `scope` from observation before naming them.
4. **Transparency before action.** Before any build step, tell the user in one short Russian sentence what is about to happen and why.
5. **Clear choices.** When presenting choices, explain each option and recommend the best one if there is one. Numeric menus for fixed choices; the learner types numbers unless the script asks for a handle, repo name, custom title, or yes/no word.
6. **Present -> confirm -> write.** Show proposed content, get confirmation, then write. For config files, issue content, rules-of-use, and architecture docs.
7. **Skip pre-answered checks.** If the learner already supplied the handle, repo name, or choice earlier, acknowledge and confirm instead of re-asking.
