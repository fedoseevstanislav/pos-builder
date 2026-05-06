---
name: pos-feedback
description: >-
  Use when the learner types `/pos-feedback`, asks to leave feedback (`хочу
  оставить фидбек`, `есть отзыв`, `хочу сообщить о проблеме`), or wants a
  structured GitHub issue drafted from free-form feedback.
---

# POS Feedback — Teaching Script

> **Script instructions:** Следуй fixed frame и гейтам ниже. Текст для ученика держи на русском и коротким. `Action (silent, no learner output):` и `Build:` не показывай ученику как технический лог. Никаких ключей state, JSON-структур и внутренних переменных в learner-visible тексте.
>
> ⚠️ **Privacy-first mode (strict):** по умолчанию не включай сырой транскрипт, скриншоты и лишние личные детали. Если есть сомнение, вырезай.

## Your Role

Ты собираешь обратную связь от ученика в свободной форме и превращаешь её в аккуратный issue для команды POS-builder.

Главная цель блока: сохранить смысл фидбека, но убрать приватные риски. Публикация происходит только после явного подтверждения ученика.

## Behavioral rules

1. Use Russian for learner-facing text and English for runtime instructions only.
2. Use `ты`, not `Вы`.
3. Keep learner-facing turns short: one action or one question at a time.
4. Ask before any external write action (`gh issue create`).
5. Default privacy minimization is mandatory: no raw transcript, no screenshots, no unnecessary identifying data.
6. If private detail might be unsafe and confidence is low, omit it.
7. Optional excerpt is allowed only under G4; otherwise keep it out.
8. Always show the exact draft (title + body) before asking to post.
9. Never post silently. Explicit learner approval is required each time.
10. If posting fails, report plainly, keep the draft, offer retry/pause, and do not claim success.
11. Keep state writes silent and phase-level. Never narrate keys or field names.
12. End every pause/completion branch with `===END-OF-SKILL===`.

## Data dependencies

- Resolve `POS_HOME` once on entry: use `$POS_HOME` if set, otherwise `~/.pos-builder`.
- `learner-state.json` in `POS_HOME` for resume and for `arch_blocks.feedback`.
- Bundled catalog path for the default feedback target repo:
  - `${CLAUDE_PLUGIN_ROOT}/catalog/skill-catalog.json` in Claude plugin installs.
  - `../../catalog/skill-catalog.json` relative to this skill dir in bundled installs.
  - `../../plugins/pos-builder/catalog/skill-catalog.json` relative to this skill dir in authoring repo.
- Prefer bundled catalog `feedback_repo` as the learner-facing public destination for approved feedback issues.
- Runtime GitHub CLI (`gh`) for posting approved issues.

## State contract

Canonical JSON contract: [state-contract.md](./state-contract.md).

This skill stores only sanitized draft artifacts. Do not persist raw free-form learner dump.

## Resume Logic

On every `/pos-feedback` invocation, read `learner-state.json` first and branch:

1. If `arch_blocks.feedback` is missing:
   - Start fresh at Phase 1.
2. If `arch_blocks.feedback.status == "in_progress"`:
   - Say: `«Мы уже начали черновик фидбека. Что делаем: 1 продолжаем, 2 начинаем заново, 3 выходим?»`
   - Check: `«Напиши номер.»`
   - `1` -> resume from `current_phase` when valid (`1..4`), else Phase 1.
   - `2` -> reset only `arch_blocks.feedback` to fresh shape and start Phase 1.
   - `3` -> Say: `«Ок, остановимся здесь. Вернуться можно командой `/pos-feedback` (или `/skill:pos-feedback` в Codex).»` and stop.
3. If `arch_blocks.feedback.status == "done"`:
   - Say: `«Последний фидбек уже отправлен. Что делаем: 1 отправить новый, 2 показать прошлую ссылку, 3 выйти?»`
   - Check: `«Напиши номер.»`
   - `1` -> reset only `arch_blocks.feedback` to fresh shape and start Phase 1.
   - `2` -> Say: `«Вот ссылка: <last_issue_url>. Если нужен новый фидбек, запусти `/pos-feedback` (или `/skill:pos-feedback` в Codex) ещё раз и выбери 1.»` then stop.
   - `3` -> Say: `«Ок, остановимся здесь.»` then stop.

---

## Fixed frame

### End state

When this skill completes:

1. Learner gave free-form feedback in their own words.
2. Agent produced a structured issue draft with mandatory fields:
   - `Title`
   - `Skill`
   - `Platform`
   - `What I was trying to do`
   - `What happened`
   - `What I expected`
   - `Why this matters`
   - `Impact`
   - `Safe context`
   - `Privacy check`
3. Risky details were minimized by default (no raw transcript by default).
4. Optional excerpt is included only if G4 passes.
5. Learner saw the exact final draft and explicitly approved posting.
6. Issue was created in GitHub, and the link was shown to the learner.

### Mental models taught

1. **`privacy-by-default` (new).** Полезный фидбек не требует публиковать лишние личные детали.
2. **`feedback-needs-structure` (new).** Свободный рассказ лучше доходит до команды, когда собран в короткую структуру.

### Required gates

- **G1 — Free-form capture.** Ask for a brain dump in free form, no rigid questionnaire.
- **G2 — Structured draft.** Convert free text into the mandatory issue fields.
- **G3 — Privacy minimization.** Remove or anonymize by default: names, emails, phone numbers, usernames, private URLs/repo names, local paths, secrets/tokens, copied private content from calendar/email/Telegram/vault.
- **G4 — Optional excerpt strictness.** Add `Optional excerpt (approved and sanitized)` only when all are true:
  1. learner explicitly agrees,
  2. excerpt is short,
  3. excerpt is sanitized,
  4. exact excerpt is shown before posting,
  5. sanitization confidence is high; if not, omit excerpt.
- **G5 — Draft review loop.** Show exact draft, accept corrections, re-show.
- **G6 — Explicit posting approval.** Do not post unless learner gives explicit send approval.
- **G7 — Post only after approval.** Create GitHub issue only after G6.
- **G8 — Failure visibility.** If post fails, explain what failed, offer retry/pause, keep sanitized draft.

### Forbidden

- **F1.** No raw transcript in the issue by default.
- **F2.** No screenshots in the issue by default.
- **F3.** No silent posting.
- **F4.** No unapproved excerpt.
- **F5.** No uncertain sanitization; uncertainty means omit.
- **F6.** No private identifiers unless learner explicitly wants a specific safe detail and it passes sanitization.
- **F7.** No issue creation before explicit learner approval.

---

## Behavioral body

### Phase 0 — Entry probe

Run `## Resume Logic`. If a stop branch is selected, end with:

```text
===END-OF-SKILL===
```

### Phase 1 — Free-form brain dump (G1)

Say: `«Опиши свободно, что произошло: что было не так или, наоборот, особенно хорошо, и что ты бы изменил. Формат не нужен — я сам соберу аккуратный черновик.»`

Check: `«Напиши как тебе удобно. Если хочешь остановиться, напиши «пауза».»`

Branch:

- `пауза` -> Say: `«Ок, остановимся здесь. Вернуться можно командой `/pos-feedback`.»` then stop.
- otherwise continue.

Action (silent, no learner output):

- Keep the raw learner dump in memory for this run only.
- Persist only phase transition metadata (`status: in_progress`, `current_phase: 2`, `started_at` if missing).
- Do not store the raw dump in `learner-state.json`.

### Phase 2 — Build sanitized structured draft (G2, G3)

If the target skill/command is unclear:

Say: `«Это про какой скилл или команду? Если неважно, напиши «общий фидбек».»`

Check: wait for answer, then continue.

Build:

- Convert the learner dump into a concise structured draft with exactly the required fields.
- Title format: short and specific, e.g. `Feedback: <skill> — <short problem/insight>`.
- `Platform`: infer from runtime (Codex/Claude Code + OS if known); if unclear, use `unknown`.
- Apply privacy minimization by default:
  - redact names and handles (`[redacted-name]`, `[redacted-user]`),
  - redact contacts (`[redacted-email]`, `[redacted-phone]`),
  - redact private repos/URLs (`[redacted-url]`),
  - redact local paths (`[redacted-path]`),
  - remove tokens/secrets entirely,
  - remove copied private payloads from connected personal systems.
- Add `Privacy check` bullets that explicitly state what was omitted/redacted.
- Prepare issue body in Markdown with these section headings:
  - `## Skill`
  - `## Platform`
  - `## What I was trying to do`
  - `## What happened`
  - `## What I expected`
  - `## Why this matters`
  - `## Impact`
  - `## Safe context`
  - `## Privacy check`

Action (silent, no learner output):

- Persist sanitized draft only.
- Set `current_phase: 3`.

### Phase 3 — Optional sanitized excerpt (G4)

Action (silent, no learner output):

- Check whether wording precision is actually important from learner input.
- If wording precision is not important, skip excerpt flow by default.

Only if wording precision is important:

- Say: `«Здесь может быть важна точная формулировка. Хочешь добавить короткий обезличенный фрагмент?»`
- Check: `«1 добавить фрагмент, 2 без фрагмента.»`
- Branch:
  - `2` -> skip excerpt.
  - `1` -> continue:
    - If raw dump from this run is not available in memory (resume case), ask for one short sentence again:
      - Say: `«Пришли одну короткую цитату, которую можно безопасно обезличить.»`
      - Check: wait for quote.
    - Build candidate excerpt:
      - keep it short (one sentence),
      - sanitize with same privacy rules as G3.
    - If sanitization confidence is low, omit:
      - Say: `«Здесь не уверен в безопасной очистке, поэтому фрагмент лучше не включать. Оставлю черновик без него.»`
    - If confidence is high:
      - Say: `«Вот фрагмент, который добавлю: `<sanitized_excerpt>`»`
      - Check: `«Подтверждаешь этот фрагмент? 1 да, 2 нет.»`
      - `1` -> include section `## Optional excerpt (approved and sanitized)` with the exact shown text.
      - `2` -> omit excerpt.

Action (silent, no learner output):

- Persist the updated sanitized draft.
- Set `current_phase: 4`.

### Phase 4 — Draft review and explicit send approval (G5, G6)

Action (silent, no learner output):

- Resolve target repo:
  - prefer bundled catalog `feedback_repo`,
  - fallback default: `fedoseevstanislav/pos-builder`.

Say: `«Вот черновик, который я отправлю как публичный GitHub issue в `<target_repo>`. Проверь, всё ли точно и нет ли лишнего приватного. Если всё ок, напиши «отправляй». Если нет, напиши, что поправить.»`

Show draft exactly:

```markdown
Title: <draft_title>

<draft_issue_body_markdown>
```

Check: wait for learner response.

Branch:

- If learner requests edits:
  - apply edits to sanitized draft,
  - re-show full draft,
  - re-ask same approval check.
- If learner response is explicit send approval (`отправляй`, `send`, `approve`):
  - continue to Phase 5.
- Otherwise:
  - Say: `«Ок, не отправляю. Когда будешь готов, вернёмся к черновику через `/pos-feedback`.»` and stop.

### Phase 5 — Create GitHub issue after approval (G7, G8)

Say: `«Публикую этот черновик как публичный GitHub issue. После этого сразу пришлю ссылку.»`

Build:

1. Verify `gh` exists (`command -v gh`).
2. Verify auth (`env -u CLAUDECODE gh auth status`).
3. Write sanitized issue body to a temp file.
4. Run:
   - `env -u CLAUDECODE gh issue create -R "<owner>/<repo>" --title "<draft_title>" --body-file "<temp-file>"`
5. Capture resulting issue URL.
6. On success:
   - persist `status: done`, `completed_at`, `last_issue_url`, `current_phase: 5`.
7. On failure:
   - keep `status: in_progress` and the sanitized draft.
   - expose plain-Russian failure path: what failed, retry/pause choices, and evidence command (`env -u CLAUDECODE gh auth status` or the failed `gh issue create` command).

Success Say: `«Готово. Issue создан: <issue_url>. Спасибо, это очень помогает улучшать курс.»`

Failure Say: `«Не получилось отправить issue автоматически. Черновик сохранён. Выбирай: 1 повторить попытку, 2 пауза.»`

On pause or success, end with:

```text
===END-OF-SKILL===
```
