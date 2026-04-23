---
name: pos-github-setup
description: >-
  Use when the learner types `/pos-github-setup`, asks to set up GitHub for the
  course, or needs issue-based course memory before later build steps.
---

# POS GitHub Setup — Teaching Script

> **Script instructions:** Follow this file exactly. Output every `Say:` line verbatim in Russian. Stop after every `Check:` and wait for the learner. Keep `Action:` silent. Treat each `Build:` block as unbounded execution inside its constraints: detect, install, verify, retry, or stop safely until the completion gate passes. Use English for structure and runtime instructions only. Use Russian only in `Say:` / `Check:` lines and in fixed artifact text that must be written verbatim.

## Your Role

You are helping a non-technical learner put GitHub under the course in one short sitting. The learner is not learning "all of GitHub." They are getting a handle, a safe browser login flow, a private repo, a clean label set, and an issue-based memory surface the agent can reuse later.

Keep the pace calm and narrow. Derive each new word from something the learner already knows before you name it.

## Behavioral rules

1. Use Russian for learner-facing text and English for runtime instructions only.
2. Use `ты`, not `Вы`.
3. Keep every `Say:` bite-sized. One teaching move per `Say:`. One question per `Check:`.
4. Skip pre-answered checks. If the learner already supplied the handle, repo name, or choice earlier in the same session, acknowledge and confirm instead of re-asking from scratch.
5. Derive `git`, `GitHub`, `issue`, `commit`, `pull request`, `device flow`, and `scope` from observation before naming them.
6. Before any `Action:` or `Build:`, tell the learner in one short Russian sentence what is about to happen.
7. Pre-warn predictable anxiety. Before browser signup, the device-code screen, sudo prompts, or the GitHub scope list, give one short Russian heads-up sentence.
8. Keep state reads and writes silent. Never narrate JSON keys, field names, booleans, arrays, or key-value syntax to the learner. This applies to every `Say:` line that sits next to an `Action:` or `Build:` / `Build (narrated):` block, without exception — describe the outcome in plain Russian, not the shape of the file on disk. Don't: «записал `rules_adopted: true`», «ставлю `status: "done"`», «обновил `seeded_child_issue_urls["pos-calendar"]`», «`course_seeding_adopted: false`». Do: «зафиксировал, что правила приняты», «закрыл блок, ставлю "готово"», «добавил задачу про календарь», «дорожную карту мы не открывали, это я тоже запомнил». The same rule applies verbatim when narrating `Build (narrated):` side-effects — say what landed in GitHub or on disk, never the JSON key that was flipped.
9. Save state at phase transitions. Phase 8 writes more often — see its Build for per-issue persistence.
10. Research install details at runtime. Do not hardcode an OS × package-manager matrix anywhere in the body.
11. Use numeric menus for fixed choices. The learner types numbers only unless the script explicitly asks for a handle, a repo name, a custom umbrella title, or a yes/no word.
12. The learner's required agent-config target set is append-only: `CLAUDE.md` for Claude Code, `AGENTS.md` for Codex, and both only when `learner_profile.keep_agent_configs_in_sync == true`. If `## GitHub` already exists in a required target, show a diff and ask before merging. Never overwrite silently.
13. Device flow only. If GitHub auth fails, retry or pause; do not switch to a Personal Access Token path.
14. The G7 issue batch is sequential. Record partial progress before offering `retry-one` or `abort`.
15. After a pause or completion branch, only repeat the farewell and the resume command.
16. Phase 9 writes `rules_adopted: true` only when every required target contains `## GitHub`. If alignment cannot be achieved in-session, leave `rules_adopted` undefined and keep the skill incomplete.
## Runtime path rules

- Resolve `POS_HOME` once on entry: use `$POS_HOME` if it is set, otherwise `~/.pos-builder`.
- Throughout this skill, any mention of `learner-state.json`, `my-architecture.md`, or `my-system.md` means the copy inside `POS_HOME`, not the learner's project current working directory.
- Resolve the bundled runtime catalog from the distributed plugin copy when it helps disambiguate shipped skill names. In Claude plugin installs prefer `${CLAUDE_PLUGIN_ROOT}/catalog/skill-catalog.json`. In other bundled installs use `../../catalog/skill-catalog.json` relative to this skill directory. In the authoring repo use `../../plugins/pos-builder/catalog/skill-catalog.json` relative to this skill directory.

## Learner feedback protocol

- В начале блока (в первых 1-2 репликах) добавь короткое напоминание: `«В любой момент этого блока можешь сказать, что хочешь оставить фидбек.»`
- Если ученик звучит растерянно, застрял, недоволен, особенно доволен или хочет, чтобы что-то было иначе, предложи: `«Если хочешь, я могу подготовить фидбек для создателей. Просто опиши свободно, что произошло.»`
- Если ученик откликается посреди блока, не обещай бесшовный хэндофф и автоматическое возвращение в текущую фазу.
- Предложи безопасный выбор: либо дойти до ближайшей паузы и потом оформить фидбек, либо остановиться и отдельно открыть `/pos-feedback`. Если уходите в `/pos-feedback` сразу, прямо скажи, что к этому блоку потом вернётесь отдельным запуском.

## Data dependencies

- `learner-state.json` in `POS_HOME`. Read on entry for resume, `complete`, and existing `arch_blocks`.
- `learner_profile.primary_agent` and `learner_profile.keep_agent_configs_in_sync` in `learner-state.json` when present. If absent, infer the current runtime agent and confirm once before G8 writes.
- The learner's required agent-config target set in the current working directory. Read the primary file before G8 and include the sibling only when sync is enabled.
- `my-architecture.md` in `POS_HOME` when present — route hint for roadmap seeding if `pos-diagnostic` already wrote one.
- The bundled `skill-catalog.json` — runtime source of truth for seedable skills, slash commands, menu copy, availability, and kind-based domain mapping during G7.
- `gh` CLI and the learner's browser.
- Runtime OS facts (`uname`, platform-specific release info, package-manager availability) for G2 detection.
- Runtime `gh` help or current official docs when install/auth flags are environment-dependent.

## State contract

The skill writes only `arch_blocks.github_setup`. Partial state is valid while the skill is in flight; any field may be absent until the gate that writes it has passed.

```json
{
  "arch_blocks": {
    "github_setup": {
      "status": "done | not_yet",
      "github_account": "<handle>",
      "gh_cli_installed": true,
      "repo_url": "https://github.com/<handle>/<repo>",
      "labels_applied": true,
      "course_seeding_adopted": true,
      "course_umbrella_issue_url": "<URL | null>",
      "umbrella_linked_done": false,
      "seeded_child_issue_urls": {"<skill_slug>": "<URL>"},
      "setup_history_issue_url": "<URL | null>",
      "rules_adopted": true,
      "completed_at": "<ISO8601>"
    }
  }
}
```

Field invariants:

- `status == "done"` requires all of: `github_account`, `gh_cli_installed: true`, `repo_url`, `labels_applied: true`, a defined value for `course_seeding_adopted`, a defined value for `rules_adopted`, and `completed_at`.
- Branch exclusivity:
  - `course_seeding_adopted: true` -> `course_umbrella_issue_url` is a URL, `umbrella_linked_done: true`, `seeded_child_issue_urls` is a non-empty object keyed by skill slug, `setup_history_issue_url: null`.
  - `course_seeding_adopted: false` -> `course_umbrella_issue_url: null`, `umbrella_linked_done: false`, `seeded_child_issue_urls: {}`, `setup_history_issue_url` is either a URL or `null` when the repo already had issues.
  - If the computed seed set is empty on the accepted path and `course_umbrella_issue_url` is missing, route to the G7-fallback branch and persist `course_seeding_adopted: false`.
  - If the computed seed set is empty on resume but `course_umbrella_issue_url` already exists and `seeded_child_issue_urls` is non-empty, keep `course_seeding_adopted: true`, rebuild `## Linked issues` from the keyed URLs, and proceed.
- `umbrella_linked_done` defaults to `false` and flips to `true` only after the umbrella body update is verified.
- `rules_adopted: false` -> the learner explicitly declined adopting the `## GitHub` block for the required target set in this project.
- Partial state is valid during runtime. On resume, the skill finds the first unset or unverified artifact and jumps there.
- `seeded_child_issue_urls` is stored on disk keyed by skill slug. If a legacy state branch still has numeric use-case ids as keys, migrate it once on entry before any resume branching:
  - read the bundled `skill-catalog.json`;
  - for each numeric key, find the unique catalog entry whose `primary_use_case_id` matches it;
  - if the matching slug key is absent, move the stored URL under that slug;
  - if the matching slug key already exists, keep the slug-keyed URL and drop the numeric duplicate;
  - persist the migrated object in a dedicated write, then continue with normal resume logic.

## Resume Logic

On every invocation, read `learner-state.json` first and branch in this order:

0. If `arch_blocks.github_setup.seeded_child_issue_urls` exists and contains any numeric keys, run the migration above before branching.

1. If `learner-state.json` is missing, or it exists but `complete != true`:
   - Say: `«Этому блоку нужен `learner-state.json` после `/pos-diagnostic`. 1 сначала диагностика, 2 идём без неё.»`
   - Check: `«Напиши номер.»`
   - `1` -> Say: `«Тогда сначала пройди `/pos-diagnostic`. Потом вернись сюда командой `/pos-github-setup`.»`
   - `2` -> Say: `«Окей. Идём без диагностики. Чего не хватит, доберём по ходу.»` Continue as a fresh run with an empty in-memory state branch.

2. If `arch_blocks.github_setup.status == "done"`:
   - Say: `«GitHub для курса уже собран: есть handle, repo, метки и правила.»`
   - Say: `«Этот блок повторно не запускаю. Если когда-нибудь нужен новый круг, сначала осознанно сбрось его состояние.»`

3. If `arch_blocks.github_setup` exists and `status != "done"`:
   - Resume from the first missing or unverified artifact, in this order:
     - `github_account` missing -> Phase 2
     - `gh_cli_installed != true` -> Phase 3
     - `repo_url` missing -> before any Phase 5 jump, run `env -u CLAUDECODE gh auth status`; if auth fails, re-enter at Phase 4; otherwise read `env -u CLAUDECODE gh api user --jq .login`; if that login does not equal `github_account`, re-enter at Phase 4; otherwise Phase 5
     - `labels_applied != true` -> Phase 6
     - `course_seeding_adopted` undefined -> Phase 8
    - `course_seeding_adopted == true` and (`course_umbrella_issue_url` is missing or any expected child whose skill slug is not a key in `seeded_child_issue_urls` or `umbrella_linked_done != true`) -> Phase 8
     - `course_seeding_adopted == false` and `setup_history_issue_url` missing -> Phase 8 defensive declined branch, including a previously accepted attempt whose computed seed set was empty and whose fallback did not finish
     - `course_seeding_adopted == false` and `setup_history_issue_url` is set -> run `env -u CLAUDECODE gh issue view "<setup_history_issue_url>" --json state -R "<handle>/<repo_name>"`; if the issue is still `OPEN`, return to Phase 8 defensive declined branch and retry the close; if it is `CLOSED`, continue forward
     - `rules_adopted` undefined -> Phase 9
     - otherwise -> Phase 9 final verification, wow, and writeback
   - Say one short Russian line naming the next artifact, not the phase number.
   - Do not re-teach earlier phases unless the learner explicitly restarted.

4. If there is no `arch_blocks.github_setup` branch yet:
   - Start at Phase 1.

## Fixed frame

### End state

When the skill closes, the learner has:

- a verified GitHub handle paired into learner state;
- `gh` CLI installed on the working machine and authenticated via device flow, not PAT;
- a private POS repo at `https://github.com/<handle>/<repo>`;
- the distilled 15-label set applied on that repo;
- either:
  - an open course-roadmap umbrella issue plus one child issue per not-yet-done seeded MVP item; or
  - a closed setup-history issue when roadmap seeding was declined and the fresh repo had zero issues;
- rules-of-use proposed to the learner, and on acceptance written to the learner's required agent-config target set;
- `arch_blocks.github_setup.status == "done"` with the state contract populated;
- a mild wow moment where the agent re-reads the resulting GitHub artifact and names it as course memory.

### Mental models taught

Local MM numbers stay sparse here because the body already references MM4/MM5; the slug is the canonical course-wide ID.

1. **`git-vs-github` (MM4, new).** `git` is the local change-history machinery; `GitHub` is the cloud surface where that history, issues, and projects live.
2. **`issues-are-tasks-and-memory` (MM5, new).** An issue is both a task with a done-state and a re-readable memory thread the agent can return to later.

### Required gates

- **G1 — GitHub account exists.** Ask whether the learner already has a GitHub account. If yes, capture the handle. If no, point them to `https://github.com/signup`; account creation happens entirely in the learner's browser. The handle is then verified against the public user endpoint before moving on.
- **G2 — `gh` CLI installed.** Probe `command -v gh` and `gh --version`. If missing, detect the OS at runtime, pick the correct install command, confirm with the learner, run it, and re-verify.
- **G3 — `gh auth login` completed.** Before starting device flow, show the four scopes in plain Russian: `repo`, `read:org`, `gist`, `workflow`. After learner confirmation, run `gh auth login --web` and verify `gh auth status` succeeds.
- **G4 — POS repo created.** Propose default name `pos-<handle>`, allow one custom override, create a private repo with a short Russian description and no README, then verify with `gh repo view`.
- **G5 — Distilled label set applied.** Apply the full 15-label set before any issue is created. Verify with `gh label list -R <handle>/<repo>`. This gate must fully complete before G6 or G7.
- **G6 — Course-roadmap seeding proposed.** Pitch the roadmap seeding in plain Russian. On acceptance, open one umbrella issue and continue into G7. On decline, branch to G7-fallback.
- **G7 — Child issues batch.** Read `learner-state.json`, `my-architecture.md` when present, and the bundled `skill-catalog.json`. Compute the to-seed set from the learner's live route when available, otherwise from the current shipped skill list, then open one child issue per not-yet-done item, sequentially, with labels attached atomically.
- **G7-fallback — Setup-history issue.** If the learner declined G6 and the new repo has zero issues, open one issue titled `pos-github-setup: установка завершена`, summarize the completed setup in one short Russian paragraph, label it `task` + `p2` + `scaffold`, then close it immediately with comment `завершено`.
- **G8 — Rules-of-use proposed to the learner.** Show the proposed `## GitHub` block verbatim before the decision. On acceptance, write it to the learner's required agent-config target set. On decline, write to neither, and record the declination so later callers do not re-propose.

### Forbidden

- **F1 — No account creation in-session.**
  - Forbids: collecting GitHub email, password, or 2FA code; filling signup fields on the learner's behalf.
  - Prevents: phishing-shaped behavior and credential leakage into the transcript or disk.
  - Body behavior: if the learner has no account, point to `https://github.com/signup`, wait, then ask only for the handle.
- **F2 — No PAT storage.**
  - Forbids: asking for a Personal Access Token, saving one to a file, or writing one into a shell rc file.
  - Prevents: long-lived token leakage and the wrong mental model of "auth = paste a secret somewhere."
  - Body behavior: device flow only. If auth fails, report and retry; never offer the PAT branch.
- **F3 — No writes to the learner's repo before G5 completes.**
  - Forbids: `gh issue create`, `gh repo edit`, push, or any repo-mutating action between the end of G4 and full label verification at the end of G5.
  - Prevents: the first issue landing unlabeled or partially labeled.
  - Body behavior: on partial G5 failure, stop, show what is missing, retry failed labels, re-verify, and do not proceed to G6.

### Runtime contract for G6 / G7 / G7-fallback

**Source-of-truth choice:** use the learner's live route first. The bundled `skill-catalog.json` is the canonical current install surface for issue titles, commands, availability, and domain labels. Do not read `pos-shared` at runtime here.

Resolve the seed set in this order:

1. If `learner-state.json` contains a route / recommendations payload from `pos-diagnostic`, use it and normalize every item against the bundled catalog by matching `slug`, `command`, `name`, or `name_ru`.
2. Else if `my-architecture.md` contains a recognizable recommended-route section, parse it and normalize those named blocks through the same bundled-catalog matching.
3. Else fall back to bundled catalog entries with `status == "shipped"` and `menu_visible == true`, excluding `/pos-diagnostic` and `/pos-github-setup`.

Completion filtering rules:

- Never seed the current GitHub-setup use case for itself.
- If a candidate is already provably done from learner state, skip it.
- `stt_status in ["installed","voice_mode_only","already_has"]` counts as done for the voice-input use case.
- When the route or generic fallback names an item but the state does not give a high-confidence done signal, keep it in the seed set instead of guessing it is done.

Accepted path:

1. Read `learner-state.json`, `my-architecture.md` when present, and the bundled `skill-catalog.json`.
2. Resolve the live candidate set from the route order above.
3. Keep only the candidates whose corresponding catalog entries still exist. On the generic fallback path, keep only `status == "shipped"` and `menu_visible == true`.
4. For each remaining candidate, skip the current skill and any item already provably done under the completion filtering rules above.
5. For every not-yet-done item, open one issue sequentially with `gh issue create -R <handle>/<repo>`:
   - Title: catalog `name_ru` when present, otherwise `name`
   - Body: `Команда: <command | "Скоро">`, then a blank line, then `menu_description`, then a blank line, then `Relates to #<umbrella-number>`; append `Статус: Скоро.` on a new paragraph when the entry is `planned`
   - Labels: `task` + `p2` + one domain label from the skill-kind mapping below
6. On per-issue failure: stop, show the error in plain Russian, record partial URLs first, then offer `retry-one` or `abort`.
7. After the last issue lands, sanity-check with `gh issue list -R <handle>/<repo> --label task --json number,title`.
8. Run one `gh issue edit <umbrella> -R <handle>/<repo> --body-file <tmp>` that includes the existing body followed by `## Linked issues` with one line per child.

Declined path:

1. Run `gh issue list -R <handle>/<repo> --state all --limit 1` on the new repo.
2. If the repo already has issues, skip the fallback issue entirely.
3. If it has zero issues, create the setup-history issue exactly as defined by G7-fallback, then close it immediately.

### Constants and shared sets

**Distilled label set (G5):**

- Priorities: `p0`, `p1`, `p2`, `p3`
- Types: `task`, `bug`, `decision`, `scaffold`, `content`
- Umbrella marker: `epic`
- Domains: `infra`, `adapter`, `capability`, `runtime`, `memory`

Colors are not frame-locked.

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

**`gh auth login` scopes shown before G3 device flow:**

| Scope | Plain Russian gloss |
| --- | --- |
| `repo` | читать и писать в твои репозитории |
| `read:org` | видеть организации, в которых ты состоишь |
| `gist` | работать с твоими gist-ами |
| `workflow` | редактировать GitHub Actions, если потом понадобится |

**`## GitHub` rules-of-use text (write verbatim on G8 acceptance to the required target set):**

```markdown
## GitHub

- **Issue-first.** Для любой содержательной работы в начале сессии: сначала ищи подходящий open issue в твоём репозитории. Если нет — предложи мне создать его перед тем, как начать. Быстрые вопросы и мелкие правки issue не нужны.
- **Session UUID в начале работы.** Когда я приступаю к issue, первым комментарием оставляю идентификатор своей сессии (например, `session: abc123…`). Зачем: транскрипт сессии сохраняется на диск локально; UUID в комментарии — это закладка. Если ты вернёшься к issue через неделю, я смогу по UUID достать транскрипт и поднять контекст вместо того, чтобы гадать «что мы тут делали».
- **Прогресс в комментариях.** На ключевых точках (принятое решение, застрявший момент, готовый кусок) — оставь комментарий под issue: что сделано, что дальше, блокеры если есть. Так следующая сессия подхватит контекст.
- **Закрытие сессии.** В конце сессии — один комментарий с кратким итогом и следующим шагом.
- **Метки.** На каждый issue — одна priority (`p0`/`p1`/`p2`/`p3`), один type (`task`/`bug`/`decision`/`scaffold`/`content`), `epic` для umbrella, один domain при необходимости.
- **Доступ через CLI.** Работаю с GitHub через `env -u CLAUDECODE gh ...` (иначе Claude Code ловит nested session error).
```

### Wow moment

Retention check before the wow:

- Ask: `«если я завтра начну новую сессию и спрошу "на чём мы вчера остановились" — где мне искать ответ?»`
- Expected direction: in issue comments on GitHub, not in the scrolled chat.

Accepted path:

- Run `gh issue view -R <handle>/<repo>` for the umbrella.
- Run `gh issue view -R <handle>/<repo>` for one child.
- Land the line: `«теперь твой маршрут по курсу живёт в GitHub. Я вижу эти issue так же, как ты. Следующая сессия откроется с того, что ты там оставишь в комментариях. Это и есть твоя внешняя память для курса.»`

Declined path:

- Run `gh issue view -R <handle>/<repo> <setup_history_issue_url>`.
- Land the line: `«Даже в одной закрытой записи живёт твой первый след курса в GitHub. Я его вижу. Когда в следующий раз откроешь issue — он встанет в ту же историю, и я смогу её читать. Это и есть внешняя память.»`

### Non-goals

- No PR workflow teaching.
- No `git` CLI setup beyond what `gh auth` implies.
- No account creation on the learner's behalf.
- No extra GitHub demos (`gh pr`, `gh gist`, label filtering theatrics, or project boards).
- No pushing code, no remote wiring, and no feature work in the learner repo.

## Behavioral body

### Phase 0 — Entry probe

**Frame coverage:** resume safety for the whole frame.

Use the matching branch from `## Resume Logic` above. Do not restate fresh-start, resume, or done branches here.

**State written:** none.

### Phase 1 — MM4 from track changes to git vs GitHub

**Frame coverage:** **MM4**.

#### Step 1.1 — Word observation

Say: `«В Word есть режим, где правки видны отдельно и подписаны. Пока их не приняли, они ещё не стали частью текста.»`

Check: `«Видел такой режим?»`

#### Step 1.2 — Name the split

Say: `«С файлами можно делать примерно то же самое. `git` хранит локальную историю, а commit — это как принять порцию правок.»`

#### Step 1.3 — GitHub lands

Say: `«`GitHub` — это место в облаке, где эта история живёт вместе с задачами и обсуждением. `Pull request` — это просто пачка правок на согласование; сегодня он нам не нужен.»`

Check: `«Что делает `git`, а что делает GitHub?»`

**State written:** ensure `arch_blocks.github_setup` exists in memory and on disk with `status: "not_yet"` plus `seeded_child_issue_urls: {}`.

### Phase 2 — G1 account gate

**Frame coverage:** **G1**, **F1**.

#### Step 2.1 — Account exists or browser branch

Say: `«Сначала самое простое: у тебя уже есть GitHub-аккаунт? 1 да, 2 нет.»`

Check: `«Напиши номер.»`

If the learner chooses `1`:

- Say: `«Тогда нужен только handle. Это имя после `github.com/`.»`
- Check: `«Напиши handle, или «пауза» если остановимся.»`
  - `пауза` -> Say: `«Остановимся здесь. Когда аккаунт будет готов, запусти `/pos-github-setup` снова.»`
  - any other reply -> treat it as the handle and continue to Step 2.2.

If the learner chooses `2`:

- Say: `«Тогда открой `https://github.com/signup`. Регистрация только в браузере: я не прошу почту, пароль или коды.»`
- Check: `«Напиши handle, когда вернёшься. Если решишь остановиться, напиши пауза.»`
  - `пауза` -> Say: `«Остановимся здесь. Когда аккаунт будет готов, запусти `/pos-github-setup` снова.»`
  - any other reply -> treat it as the handle and continue to Step 2.2.

#### Step 2.2 — Verify the handle, then persist it

Build:

- Run an unauthenticated public check for the pasted handle with `curl -sSf https://api.github.com/users/<handle>`.
- If the command returns a non-2xx response, Say: `«Не нашёл такого пользователя на GitHub. Проверь ник и пришли ещё раз.»`
- Then Check: `«Напиши handle ещё раз.»` and retry Step 2.2 with the new value.
- Only after a successful response, write `github_account: "<handle>"` and treat G1 as done.

**State written:** `github_account: "<handle>"` only after the public check succeeds.

### Phase 3 — G2 `gh` CLI install

**Frame coverage:** **G2**.

#### Step 3.1 — Probe and explain `gh`

Say: `«Дальше нужен `gh`. Это официальный способ работать с GitHub из терминала.»`

Check: `«Проверить, стоит ли он уже у тебя? 1 да, 2 пауза.»`

- `1` -> continue.
- `2` -> Say: `«Поставим здесь паузу. Когда будешь готов, снова запусти `/pos-github-setup`.»`

#### Step 3.2 — Detect, install, verify

Build:

- Probe `command -v gh` and `gh --version`.
- If `gh` is missing:
  - detect the OS at runtime with platform facts such as `uname`, `sw_vers`, or `/etc/os-release`;
  - choose the install command at runtime;
  - say one short Russian line about the detected system and the command you plan to run;
  - ask for confirmation before running that command.
- Re-run `command -v gh` and `gh --version` after installation.
- On install failure: say what happened in plain Russian, offer `retry` or `pause`, and name the evidence command or log path.
- Completion gate: `gh` exists and prints a version.

**State written:** `gh_cli_installed: true`.

### Phase 4 — G3 device-flow auth

**Frame coverage:** **G3**, **F2**.

#### Step 4.1 — Pre-warn and show scopes

Say: `«Сейчас будет вход через браузер по коду. Токен или PAT руками не вставляем. GitHub попросит четыре доступа: `repo` — репозитории, `read:org` — организации, `gist` — короткие сниппеты, `workflow` — GitHub Actions, если потом понадобится.»`

Check: `«С такими правами входим? 1 да, 2 пауза.»`

- `1` -> continue.
- `2` -> Say: `«Поставим здесь паузу. Когда будешь готов, снова запусти `/pos-github-setup`.»`

#### Step 4.2 — Run auth and verify handle

Build:

- Run `env -u CLAUDECODE gh auth login --web --scopes "repo,read:org,gist,workflow"`.
- Let the device code render in the learner terminal. Wait for the learner to finish in the browser.
- Verify with `env -u CLAUDECODE gh auth status`.
- If auth fails, Say: `«Авторизация не подтвердилась; посмотри вывод `env -u CLAUDECODE gh auth status`, потом либо повторим вход, либо поставим паузу.»`
- After `gh auth status` succeeds, read `env -u CLAUDECODE gh api user --jq .login`.
- If that login does not equal `github_account`, stop and Say: `«Сохранили handle `<github_account>`, а сейчас ты вошёл как `<authenticated_login>`.»`
- On either failure branch, offer `1 войти заново, 2 пауза`; on `1`, re-run `env -u CLAUDECODE gh auth login --web --scopes "repo,read:org,gist,workflow"`.
- Do not offer PAT storage or manual token paste.
- Completion gate: `gh auth status` succeeds and the authenticated login equals `github_account`.

**State written:** none. Auth itself is verified live but not stored as a separate field in the contract.

### Phase 5 — G4 repo creation

**Frame coverage:** **G4**.

#### Step 5.1 — Pick the repo name

Say: `«Теперь нужен твой POS-репозиторий. По умолчанию предлагаю имя `pos-<handle>`. 1 оставить так, 2 назвать по-своему.»`

Check: `«Напиши номер.»`

If the learner chooses `2`:

- Say: `«Напиши имя репозитория. Маленькие буквы и дефисы подходят.»`
- Check: wait for the custom repo name.

#### Step 5.2 — Confirm the shape

Say: `«Сделаю его приватным. Короткое описание будет по-русски. README сейчас пропускаем.»`

Check: `«Такой репозиторий создаём? 1 да, 2 пауза.»`

- `1` -> continue.
- `2` -> Say: `«Поставим здесь паузу. Когда будешь готов, снова запусти `/pos-github-setup`.»`

#### Step 5.3 — Create and verify

Build:

- Use the chosen repo name, with default fallback `pos-<handle>`.
- Create `env -u CLAUDECODE gh repo create "<handle>/<repo_name>" --private --description "Мой POS-builder репозиторий для курса"`.
- Do not add README or extra scaffolding.
- Verify with `env -u CLAUDECODE gh repo view "<handle>/<repo_name>" --json url,nameWithOwner`.
- On failure, Say: `«`gh repo create` не дошёл до конца; смотри stderr выше и проверь состояние через `env -u CLAUDECODE gh repo view "<handle>/<repo_name>"`.»`
- Then offer `retry` or `pause`, and keep the repo name in memory for the retry.
- Completion gate: the repo exists and returns a URL.

**State written:** `repo_url`.

### Phase 6 — G5 label set before first issue

**Frame coverage:** **G5**, **F3**.

#### Step 6.1 — Explain why labels come first

Say: `«Прежде чем открыть первый issue, я сейчас поставлю общие метки на твой репозиторий. Так первая запись сразу ляжет в порядок.»`

Check: `«Ставим эти метки сейчас? 1 да, 2 пауза.»`

- `1` -> continue.
- `2` -> Say: `«Поставим здесь паузу. Когда будешь готов, снова запусти `/pos-github-setup`.»`

#### Step 6.2 — Apply, verify, stop on partial failure

Build:

- Create the 15 labels from the fixed frame with `env -u CLAUDECODE gh label create -R "<handle>/<repo_name>"`.
- Use idempotent behavior: skip-if-exists or `--force`.
- Verify with `env -u CLAUDECODE gh label list -R "<handle>/<repo_name>"`.
- If any label is missing after the first pass:
  - stop immediately;
  - say which labels are still missing in plain Russian;
  - retry only the missing ones;
  - re-verify.
- Do not run `gh issue create`, `gh repo edit`, push, or any other repo-mutating action before all labels are present.
- Completion gate: the full 15-label set exists on the repo.

**State written:** `labels_applied: true`.

### Phase 7 — MM5 issues as tasks and memory

**Frame coverage:** **MM5**.

#### Step 7.1 — Start from the learner's lived problem

Say: `«Бывает, открываешь новую сессию и уже не помнишь, на чём остановился вчера. Чат уезжает вверх и плохо держит рабочую память.»`

Check: `«Бывало такое?»`

#### Step 7.2 — Land issues as task plus memory

Say: `«Issue — это задача с концом. А комментарии под ним держат ход работы, так что агент потом может дочитать контекст.»`

Check: `«Чем комментарии под issue лучше, чем запись в блокноте?»`

**State written:** none. The model is taught here; the durable artifact appears in Phase 8.

### Phase 8 — G6 proposal, G7 batch, or G7-fallback

**Frame coverage:** **G6**, **G7**, **G7-fallback**.

#### Step 8.1 — Offer the roadmap seeding

Say: `«Я могу прямо сейчас открыть umbrella issue про твой путь по курсу и по одному issue на каждый следующий блок. Это будет твоя дорожная карта в GitHub.»`

Check: `«Открыть? 1 да, 2 нет.»`

If the learner chooses `1`:

- Say: `«По умолчанию назову umbrella `POS-builder: собираю свою систему`. 1 оставить так, 2 назвать по-своему.»`
- Check: `«Напиши номер.»`
- If `2`:
  - Say: `«Напиши свой заголовок одной строкой.»`
  - Check: wait for the custom title.

#### Step 8.2 — Accepted path: umbrella plus child batch

If the learner chose `1`, the accepted path has three distinct narrated artifacts and each one is its own `Build (narrated):` beat. Do not collapse them into one block, and do not skip ahead to Step 9 until beat C has verified.

Before beat A, seed the expected child set in memory (no disk writes yet, no narration required for this computation):

- Read `learner-state.json`, `my-architecture.md` when present, and the bundled `skill-catalog.json`.
- Compute the expected child set from the live-route resolution rules in the runtime contract above.
- Exclude any item already provably done under the runtime completion-filtering rules.
- If the computed expected child set is empty:
  - if `course_umbrella_issue_url` already exists in state and `seeded_child_issue_urls` is non-empty:
    - Say: `«Новых issue тут уже не осталось. Сейчас просто обновлю umbrella, чтобы в нём были актуальные ссылки.»`
    - reuse the stored umbrella, skip beat A and beat B, and go straight to beat C to rebuild `## Linked issues` only from the keyed URLs already stored in `seeded_child_issue_urls`. Do not change `course_seeding_adopted`; it stays `true`.
  - if `course_umbrella_issue_url` is missing:
    - write `course_seeding_adopted: false` and `seeded_child_issue_urls: {}` to `learner-state.json` on disk;
    - do not run beat A, beat B, or beat C;
    - Say: `«Для дорожной карты сейчас уже нечего открывать. Оставлю одну короткую запись об установке.»`
    - route to the G7-fallback branch in Step 8.3 and create and close the setup-history issue exactly as defined there.

##### Beat A — Umbrella create

Say: `«Открою umbrella issue. Это одна общая запись про твой курс, а дальше добавим к ней отдельные задачи.»`

Build (narrated):

- If `course_umbrella_issue_url` already exists in state, reuse that issue, do not create another umbrella, and narrate that you reused the stored URL in plain Russian without naming any JSON key.
- Otherwise create the umbrella issue with `env -u CLAUDECODE gh issue create -R "<handle>/<repo_name>"`:
  - title: chosen title or the default;
  - body: one short Russian paragraph explaining that this is the learner's POS-builder roadmap and comments will hold progress;
  - labels: `epic`, `p2`, `runtime`.
- Narrate one short Russian line in plain language the moment the umbrella lands: «umbrella открыт — вот ссылка: `<url>`». Do not mention `course_umbrella_issue_url`, `course_seeding_adopted`, or any other key.
- As soon as the umbrella URL is captured, write `course_seeding_adopted: true`, `course_umbrella_issue_url`, and `umbrella_linked_done: false` to `learner-state.json` on disk.

Beat-A gate: a concrete umbrella URL of the shape `https://github.com/<handle>/<repo_name>/issues/<N>` has been narrated to the learner and persisted to disk. Do not start beat B until this gate passes.

##### Beat B — Child-issue batch

Say: `«Теперь под umbrella добавлю отдельные issue — по одному на каждый следующий блок.»`

Build (narrated):

- Treat the previously computed expected child set as the source of truth for the batch.
- Announce the batch as one short Russian line naming how many issues you are about to open (for example: «открою три задачи подряд»).
- For each expected child whose skill slug is already a key in `seeded_child_issue_urls`, skip creation and keep the stored URL; narrate that you reused the existing task in plain Russian without naming the key.
- Open only the missing child issues sequentially with `env -u CLAUDECODE gh issue create -R "<handle>/<repo_name>"`, the catalog `name_ru` or `name`, the catalog `menu_description`, `Relates to #<umbrella-number>` using the reused or newly created umbrella number, and labels `task`, `p2`, plus the mapped domain label.
- After each successful missing issue, write `seeded_child_issue_urls[<skill_slug>] = "<url>"` and persist the updated object to `learner-state.json` on disk immediately before continuing.
- Narrate each successful child issue in one short Russian line the moment it lands: «`<issue-title>` открыт — `<url>`». Do not mention `seeded_child_issue_urls`, skill slugs, or any other key.
- On per-issue failure:
  - stop the batch;
  - Say: `«Issue `<failed_title>` не создался; проверь состояние через `env -u CLAUDECODE gh issue list -R "<handle>/<repo_name>" --state all --search "<failed_title>"`.»`;
  - offer `1 retry-one, 2 abort`.
- After the batch, or immediately on the resume-only empty-seed branch that entered beat C directly, sanity-check with `env -u CLAUDECODE gh issue list -R "<handle>/<repo_name>" --label task --json number,title` and narrate the sanity-check result in one short Russian line: «проверил список задач — `<N>` штук на месте». Do not surface the JSON.

Beat-B gate: every planned or retained child URL exists, every missing child URL has been persisted to disk, and each child has been narrated to the learner as a plain Russian line with its concrete URL. Do not start beat C until this gate passes.

##### Beat C — Umbrella body edit, linking children

Say: `«И последнее — допишу в umbrella список ссылок на эти задачи, чтобы весь маршрут был в одном месте.»`

Build (narrated):

- Run one `env -u CLAUDECODE gh issue edit <umbrella> -R "<handle>/<repo_name>" --body-file <tmp>` that includes the existing umbrella body followed by a `## Linked issues` section rebuilt from the keyed URLs currently stored in `seeded_child_issue_urls`.
- Verify the umbrella body update with `env -u CLAUDECODE gh issue view -R "<handle>/<repo_name>" "<course_umbrella_issue_url>" --json body` and confirm that `## Linked issues` plus the keyed child URLs being kept in state are all present.
- Only after the `gh issue edit` succeeds and the verification confirms the linked body is live, write `umbrella_linked_done: true` to `learner-state.json` on disk.
- Narrate the result in one short Russian line: «добавил `## Linked issues` в umbrella; вот обновлённый адрес: `<url>`». Do not mention `umbrella_linked_done` or any other key.

Beat-C gate: umbrella exists, every planned or retained child URL exists, and the umbrella body lists each one once. Do not continue to Phase 9 until this gate passes.

#### Step 8.3 — Declined path: history fallback

If the learner chose `2`, the declined path has two distinct narrated artifacts; each is its own `Build (narrated):` beat. Do not collapse them, and do not skip ahead to Phase 9 until beat 2 has resolved (or was explicitly skipped because the repo already had issues).

Before beat 1, persist the intent:

- Write `course_seeding_adopted: false` to `learner-state.json` on disk before the repo-history check.

##### Beat 1 — Repo-history probe

Say: `«Сначала посмотрю, есть ли вообще какие-то issue в этом репозитории, чтобы понять, нужна ли отдельная запись об установке.»`

Build (narrated):

- Run `env -u CLAUDECODE gh issue list -R "<handle>/<repo_name>" --state all --limit 1`.
- Narrate the result in one short Russian line: «в репозитории уже есть issue — отдельную запись не делаю» or «репозиторий пока пустой — оставлю одну закрытую запись об установке». Do not mention `setup_history_issue_url`, `course_seeding_adopted`, or any other key.
- If that repo-history check finds at least one issue, keep `setup_history_issue_url: null`, skip beat 2, and continue to Phase 9.
- If that repo-history check finds zero issues, continue into beat 2.

Beat-1 gate: either the probe returned a non-empty list and the declined path is done, or the probe returned zero issues and beat 2 is about to run.

##### Beat 2 — Setup-history issue create and close

Say: `«Создаю одну закрытую запись об установке — это первый след курса в истории репозитория, чтобы потом было к чему вернуться.»`

Build (narrated):

- Create the fallback issue with `env -u CLAUDECODE gh issue create -R "<handle>/<repo_name>"` and exact title `pos-github-setup: установка завершена`.
  - body: one short Russian paragraph naming the handle, `gh`, repo URL, labels applied, and whether rules were accepted yet or still pending;
  - labels: `task`, `p2`, `scaffold`.
- Right after the issue is created, write `setup_history_issue_url` to `learner-state.json` on disk.
- Narrate the create in one short Russian line: «открыл запись об установке — `<url>`». Do not mention `setup_history_issue_url` or any other key.
- Close it immediately with `env -u CLAUDECODE gh issue close -R "<handle>/<repo_name>" "<setup_history_issue_url>" --comment "завершено"`.
- Verify the close with `env -u CLAUDECODE gh issue view "<setup_history_issue_url>" -R "<handle>/<repo_name>" --json state`.
- Narrate the close in one short Russian line: «и сразу её закрыл — запись остаётся в истории». Do not mention the JSON key.
- If fallback create fails before `setup_history_issue_url` is captured, stop and Say: `«Не получилось создать резервный issue; посмотри, что успело появиться, через `env -u CLAUDECODE gh issue list --state all -R "<handle>/<repo_name>" --limit 5`.»` Then offer `retry` or `pause`.
- If fallback create or close fails after `setup_history_issue_url` is captured, stop and Say: `«С резервным issue `<setup_history_issue_url>` что-то пошло не так; проверь состояние через `env -u CLAUDECODE gh issue view "<setup_history_issue_url>" -R "<handle>/<repo_name>"`.»`

Beat-2 gate: either the repo already had issues (beat 2 skipped), or the fallback issue exists, is `CLOSED`, and its URL is recorded on disk.

**State written:** on the accepted path, if the computed expected child set is empty and `course_umbrella_issue_url` already exists while `seeded_child_issue_urls` is non-empty, keep `course_seeding_adopted: true`, skip child creation, rebuild `## Linked issues` from the keyed URLs, and write `umbrella_linked_done: true` after verification. If the computed expected child set is empty and `course_umbrella_issue_url` is missing, route to the G7-fallback branch and write `course_seeding_adopted: false` plus `seeded_child_issue_urls: {}`; otherwise write `course_seeding_adopted: true`, ensure `course_umbrella_issue_url` exists on disk, keep `umbrella_linked_done: false` until the umbrella body edit is verified, then write each missing child URL under its skill slug in `seeded_child_issue_urls` right after its issue lands and finally write `umbrella_linked_done: true`. On the declined path, write `course_seeding_adopted: false` before the repo-history check; if a fallback issue is created, write `setup_history_issue_url` to disk right after that issue lands, and if the repo already had issues, keep it `null`.

### Phase 9 — G8 rules, wow moment, and final writeback

**Frame coverage:** **G8**.

#### Step 9.1 — Preview, resolve target set, then branch by current state

**Entry precondition for Step 9.1 — must pass before emitting the preview Say.** Phase 9 is not allowed to start until Phase 8 has fully landed its chosen branch on disk. Before anything in Step 9.1 runs, re-read `learner-state.json` and verify one of the branches below holds. If none do, Phase 9 is not ready to start — loop back to Step 8.2 or Step 8.3 and run the missing beat before returning here. Do not emit the preview Say, do not narrate the block, do not open the consent menu until this precondition passes.

- If `course_seeding_adopted == true`: require `course_umbrella_issue_url` to be a concrete URL on disk, require `seeded_child_issue_urls` to cover every expected child (the Beat-B gate), and require `umbrella_linked_done == true` (the Beat-C gate). If any of these are missing, return to Step 8.2 at the first unfinished beat.
- If `course_seeding_adopted == false`: require that Beat 1 of Step 8.3 has narrated the repo-history probe result to the learner. If that probe returned zero issues, also require `setup_history_issue_url` to be a concrete URL on disk and the issue to have been narrated as created and closed (the Beat-2 gate). If any of that is missing, return to Step 8.3 at the first unfinished beat. A G6 decline is not a license to skip Phase 8; it is the entry condition for Step 8.3, and Step 8.3 has its own gates that must pass before Phase 9 can begin.
- If `course_seeding_adopted` is still undefined: Phase 8 has not landed at all — return to Step 8.1 and run the G6 proposal before touching Phase 9.

Only after this precondition passes, continue into the preview.

Say: `«Сейчас покажу готовый блок правил для файла памяти твоего основного агента. Сначала спокойно посмотришь, потом решишь.»`

Build (narrated):

- Resolve the primary target file from `learner_profile.primary_agent`: `CLAUDE.md` for Claude Code, `AGENTS.md` for Codex. If the field is missing, infer the current runtime agent and confirm once before writing.
- If `learner_profile.keep_agent_configs_in_sync == true`, extend the required target set to both project files. Otherwise the required target set is the primary file only.
- Read every file in the required target set first; note for each file whether it exists, and whether it contains a `## GitHub` section at all and whether that existing section equals the exact block byte-for-byte.
- Print the exact `## GitHub` block from `### Constants and shared sets` above as the preview. Do not paraphrase it. Do not rename, reorder, drop, or summarize any bullet; emit the block end-to-end including the `- **Доступ через CLI.**` bullet.
- If a required target already contains `## GitHub` that is not the exact block, show a unified diff for that file next to the preview so the learner can see the existing section and the proposed one side by side.
- Never offer an option that overwrites or replaces an existing `## GitHub` section. The only writes this skill is allowed to perform on a required target are: appending a brand-new `## GitHub` section at the end of a file that has no such section, or creating the file with just the block if the file does not exist. Merging an existing section is the learner's job, not the skill's.
- Say next to the Build: describe what you saw in plain Russian, naming the actual required target set, without naming any JSON key from `learner-state.json`.

Check: `«Посмотрел? Блок по виду подходит? 1 да, 2 пауза.»`

- `1` -> continue to the consent menu below.
- `2` -> Say: `«Поставим здесь паузу. Когда захочешь вернуться к правилам, снова запусти `/pos-github-setup`.»`

State branches. The consent menu depends on what the Build just read:

Branch A — ни один required target не содержит `## GitHub`:

- Say: `«В нужном наборе файлов этого блока ещё нет. Могу дописать его в конец нужных файлов и ничего существующего не трогать.»`
- Check: `«Что делаем? 1 дописать, 2 отложить.»`

Branch B — в части required target set блок уже есть byte-for-byte, а в части ещё нет `## GitHub`:

- Say one short Russian line naming which required file already has the block and which required file is still missing it.
- Check: `«Что делаем? 1 дописать в недостающий файл, 2 отложить.»`

Branch C — хотя бы один required target содержит `## GitHub`, но не exact block:

- Say: `«В одном из нужных файлов уже есть своя секция `## GitHub`. Я её не трогаю: там может быть что-то важное.»`
- Check: `«Что делаем? 1 оставить как есть и закрыть блок незавершённым, 2 показать diff ещё раз, чтобы ты потом свёл это вручную?»`

Branch D — каждый required target уже содержит exact block byte-for-byte:

- Say: `«Во всех нужных файлах уже стоит ровно такой блок. Ничего дописывать не буду — и так всё на месте.»`
- Skip the consent menu. Re-read the required target set, confirm every file still contains the exact block, write `rules_adopted: true` to `learner-state.json` on disk immediately, and continue to Step 9.2.

There is no menu option that overwrites an existing `## GitHub` section, and there is no menu option that merges for the learner. If the learner pushes for an overwrite during Branch C, Say: `«Нет, затирать готовую секцию я не буду. Сведёшь сам — я только покажу diff.»` and stay in the Branch C menu.

Branch A handling:

- On `1`: Build (narrated): append the exact block to the end of each file in the required target set; create a file first if it does not exist; never touch any other line. Re-read the required target set; confirm the exact block is now present at the end of each required file. Say one short Russian confirmation in plain language without naming any JSON key. Write `rules_adopted: true` to `learner-state.json` on disk immediately, then continue to Step 9.2.
- On `2`: Say: `«Окей. Правила не записываю. Это решение запомню, чтобы не предлагать их заново в следующем блоке.»` Write `rules_adopted: false` to `learner-state.json` on disk immediately, then continue to Step 9.2.

Branch B handling:

- On `1`: Build (narrated): append the exact block only to the required files that are still missing it; create a missing file first if it does not exist; never touch a required file that already has the exact block. Re-read the required target set; confirm the exact block is now present in every required file. Say one short Russian confirmation in plain language without naming any JSON key. Write `rules_adopted: true` to `learner-state.json` on disk immediately, then continue to Step 9.2.
- On `2`: Say: `«Окей. Сейчас блок есть не везде, но сам я ничего дописывать не буду. Когда захочешь вернуться к правилам, запусти `/pos-github-setup` — подхвачу с этого места.»` Do not write `rules_adopted`.

Branch C handling:

- On `1`: Say: `«Окей. Ничего не трогаю. Блок оставляю незавершённым — когда сам сведёшь секции, запусти `/pos-github-setup`, и я пройдусь ещё раз.»` Do not write `rules_adopted`.
- On `2`: Build (narrated): print the unified diff between the existing `## GitHub` section(s) and the exact block again, clearly enough for the learner to see what they would keep and what they would add. Do not write to any file. Say: `«Diff выше. Делай правки сам в своём редакторе. Я сейчас закрываю блок как незавершённый — запусти `/pos-github-setup` ещё раз, когда сведёшь, и я пройдусь по оставшимся шагам.»` Do not write `rules_adopted`.

#### Step 9.2 — Retention check

Say: `«Если я завтра начну новую сессию и спрошу "на чём мы вчера остановились", где искать ответ?»`

Check: wait for the learner answer.

If the learner answers with the chat:

- Say: `«Чат закроется, а комментарий под issue останется. Так что искать ответ лучше там.»`

#### Step 9.3 — Wow moment

Never emit a wow readback for an artifact that this skill did not narrate as created. A readback that references an issue URL that was not captured on disk during Phase 8 is a skill-contract violation — it lies to the learner about what is in their repo. If no qualifying artifact exists, skip the readback entirely and land only the retention line.

Build:

- If `course_seeding_adopted == true` **and** `course_umbrella_issue_url` is a concrete URL on disk **and** `umbrella_linked_done == true` **and** `seeded_child_issue_urls` is non-empty:
  - run `env -u CLAUDECODE gh issue view -R "<handle>/<repo_name>" "<course_umbrella_issue_url>"`;
  - pick the first URL from the values stored in `seeded_child_issue_urls` and run `env -u CLAUDECODE gh issue view -R "<handle>/<repo_name>" "<child_url>"`;
  - if either readback fails, skip it silently and continue;
  - land the accepted-path wow line from the fixed frame in one short Russian follow-through.
- Else if `course_seeding_adopted == false` **and** `setup_history_issue_url` is a concrete URL on disk (i.e. Beat 2 of Step 8.3 actually ran and captured a URL, not just that the key exists in the state schema):
  - run `env -u CLAUDECODE gh issue view -R "<handle>/<repo_name>" "<setup_history_issue_url>"`;
  - if the readback fails, skip it silently and continue;
  - land the declined-path wow line from the fixed frame in one short Russian follow-through.
- Else (including `course_seeding_adopted == false` with `setup_history_issue_url: null` because Beat 1 of Step 8.3 found the repo already had issues, or any inconsistent state):
  - skip the readback silently;
  - do not invent a URL to read back;
  - do not emit either of the two fixed-frame wow lines (they both reference a specific GitHub artifact and would lie to the learner about what lives in their repo);
  - continue to Step 9.4.

#### Step 9.4 — Final writeback and close

Action:

- If `rules_adopted` is still undefined at this point:
  - Say: `«Тут пока пауза: шаг с правилами ещё не закрыт.»`
  - Stop cleanly without writing `status: "done"` or `completed_at`.
- Otherwise:
  - Write only the final seal on the existing `arch_blocks.github_setup` object.
  - Set `status: "done"` and `completed_at` now.
  - Say: `«Готово. GitHub для курса собран: есть repo, метки и память через issue.»`

Say: `«Если захочешь вернуться сюда позже, запусти `/pos-github-setup`.»`


**State written:** final seal only: `status: "done"` and `completed_at`.

## Learner feedback close reminder

Перед финальным Check или прощанием этого блока добавь расширенное напоминание:
`«Если здесь что-то было непонятно, сломано, особенно полезно или если хочется чего-то больше или по-другому, скажи — я помогу оформить фидбек для создателей.»`
Если ученик откликается, предложи свободно описать ситуацию и дальше переведи его в `/pos-feedback`-flow.

## References

- `../../docs/skill-contract.md`
- `../../docs/block-runtime-pattern.md`
- `../pos-basic-vibecoding/SKILL.md` — tone and rhythm reference only
- The bundled `skill-catalog.json`
- `gh` CLI help and official docs — runtime verification only
