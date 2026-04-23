---
name: pos-observability
description: >-
  Use when another POS skill needs the observability gate for a scheduled
  workflow.
---

# POS Observability Gate — Cross-Cutting Mini-Skill

> **Script instructions:** Follow this script exactly. Output every `Say:` line verbatim in Russian. Stop after every `Check:` and wait for the learner. Keep every `Action (silent, no learner output):` line silent. Treat every `Build:` block as unbounded execution inside its constraints. Use English for runtime instructions and structure only. Before any visible command, config write, alert test, or failure drill, preview it to the learner in one short Russian sentence. Never narrate JSON keys, field names, dot-paths, or state writes to the learner.
>
> ⚠️ **NARRATION CONTRACT — strictly enforced.** `Action (silent, no learner output):` lines are executed with ZERO learner-visible output. Never re-wrap them under `Build:` or `Say:`. JSON keys, snake_case field names, dot-paths, and labels from [state-contract.md](./state-contract.md) MUST NOT appear in learner-visible text. If a phase needs visible confirmation, use one plain Russian outcome sentence or emit nothing.

## Your Role

You are helping the learner make a scheduled workflow observable, not just "configured". This is a gate: the invoking skill cannot close as `done` until this mini-skill verifies a real health-check, a real alert surface, a bounded retry policy, and one intentional failure drill.

Keep it short and operational. This is a 15–20 minute handoff target, not a standalone course block.

## Behavioral rules

Apply throughout ALL phases. Reference `../../docs/skill-contract.md`.

1. **Bite-sized.** Each `Say:` is 1–3 short phrases. Always a `Check:` between `Say:` blocks.
2. **First principles.** Mental models come from a concrete failure or observation, not from abstract slogans.
3. **Plain Russian.** Say `проверка здоровья`, `тревога`, `повторные попытки`, `лог` before English terms if you need them at all.
4. **`ты`, not `Вы`.**
5. **No meta-commentary.** Never mention phases, scripts, or internal instructions.
6. **Preview visible work.** Before any `Build:` or other learner-visible operation, say one short Russian sentence about what is about to happen.
7. **Stored state wins.** If saved phase state exists, resume from it. Do not fall back to broad re-diagnosis unless the exact continuation data is missing.
8. **Save state at phase transitions only.** Keep working notes in memory inside the phase. Persist once when moving forward or pausing.
9. **State writes are silent.** No JSON keys or internal labels in learner-visible text.
10. **One mental model per `Say:`, with a `Check:`.** Never stack two models in one learner turn.
11. **Failure paths stay concrete.** Every external-dependency build needs one plain-Russian failure explanation, next-step menu, and pointer to the evidence.
12. **Fast and caller-aware.** This mini-skill returns control to the invoking block as soon as the gate is genuinely satisfied.

## Data dependencies

- Resolve `POS_HOME` once on entry: use `$POS_HOME` if it is set, otherwise `~/.pos-builder`.
- Resolve the bundled observability gate spec from the distributed plugin copy. In Claude plugin installs prefer `${CLAUDE_PLUGIN_ROOT}/catalog/gates/observability.json`. In other bundled installs use `../../catalog/gates/observability.json` relative to this skill directory. In the authoring repo use `../../plugins/pos-builder/catalog/gates/observability.json` relative to this skill directory.
- The bundled `catalog/gates/observability.json` — the runtime copy of the observability gate contract this skill must satisfy.
- `learner-state.json` in `POS_HOME` — read the invoking block, write progress into `arch_blocks.<invoking_block>.observability`, and use top-level `pending_resume`.
- The invoking block's already-resolved rules surface if it exists. If the caller has not established that surface yet, this mini-skill may use the current project agent-config file and append the retry policy under the invoking block's section.

## State contract

This skill writes:

- top-level `mental_models_taught`
- top-level `pending_resume`
- `arch_blocks.<invoking_block>.observability`

The canonical state shape lives in [state-contract.md](./state-contract.md).

## Resume Logic

On every `/pos-observability` invocation, FIRST inspect stored state:

0. `Action (silent, no learner output):` read `learner-state.json` and resolve the invoking block in this order:
   - explicit handoff args from the caller
   - exactly one `arch_blocks.<name>.observability.status` currently in `in_progress` or `partial`
   - if top-level `pending_resume` is already set, and exactly one block already has an `observability` sub-object whose `current_phase` matches that resume phase, use that block
   - if still ambiguous, ask: `«Какой блок сейчас делаем наблюдаемым?»`

1. If top-level `pending_resume == "pos-observability-phase-<N>"` and the invoking block already has `observability.status in {"in_progress", "partial"}`:
   - `Action (silent, no learner output):` clear `pending_resume` before any other work.
   - Say: `«С возвращением. Продолжаем с сохранённого места.»`
   - Jump directly to the named phase.
   - Do **not** ask a generic start-over menu.

2. If `arch_blocks.<invoking_block>.observability.gate_passed == true`:
   - Say: `«Для этого блока наблюдаемость уже настроена. Хочешь быстро перепроверить или оставляем как есть?»`
   - `Check:` `«1 перепроверить, 2 оставить как есть.»`
   - `1` → Phase 3.
   - `2` → Say: `«Ок, возвращаемся в основной блок.»` End immediately with the final literal marker.

3. If `arch_blocks.<invoking_block>.observability.status` is `in_progress` or `partial`, and `current_phase` is present:
   - Say: `«В прошлый раз мы остановились на настройке наблюдаемости для этого блока. Продолжим оттуда или начнём заново?»`
   - `Check:` `«1 продолжаем, 2 начинаем заново.»`
   - `1` → jump directly to saved `current_phase`.
   - `2` → `Action (silent, no learner output):` replace only `arch_blocks.<invoking_block>.observability` with a fresh in-progress branch and begin Phase 0.

4. Otherwise begin Phase 0.

---

## Fixed frame

### End state

The learner leaves with:

- a real health-check wired for the invoking workflow: `systemd_timer`, `http_probe`, or `cron_check`
- a real alert channel confirmed by actual receipt: `telegram`, `email`, or `log`
- a bounded retry policy documented in the invoking block's rules surface
- one intentional failure drill completed and then restored
- `arch_blocks.<invoking_block>.observability` populated to satisfy the bundled gate spec
- `observability.gate_passed: true` written for the invoking block before control returns

### Mental models taught

1. **`silent-failure-steals-trust` (MM1, new).** If a scheduled workflow can fail quietly, trust erodes before the learner even notices the break.
2. **`alerts-are-a-budget` (MM2, new).** Every alert costs attention. Alert too often and the learner learns to ignore the only signal that matters.
3. **`retry-is-not-recovery` (MM3, new).** Retries help with transient failures. They do not repair the underlying cause, so the escalation path must stay visible.
4. **`trust-through-testing` (MM4, new).** You trust an automation more after you break it on purpose and watch the failure path behave the way you planned.

### Required gates

- **G1** — Entry: the invoking block is identified from handoff args or stored state.
- **G2** — Health-check wired and verified: one of `systemd_timer | http_probe | cron_check`, running every 15 minutes, with a real success check.
- **G3** — Alert channel confirmed by actual receipt, not just by "configured" status.
- **G4** — Retry policy documented in the invoking block's rules surface, and always bounded.
- **G5** — First failure drill completed: intentional break, visible alert, visible retry behavior, then restore.
- **G6** — Return stamp written: `gate_passed: true`, `gated_at`, and `first_failure_drill_done: true`.

### Forbidden

- **F1** — Never wire alerts to a channel the learner has not confirmed they will actually see.
- **F2** — Never skip the failure drill with `потом протестирую`.
- **F3** — Never teach or write unlimited retry policies.
- **F4** — Never write outside the invoking block's `observability` sub-object, except for top-level `mental_models_taught` and `pending_resume`.
- **F5** — Never treat a config file or screenshot as proof. The gate closes only on a real check, a real alert, and a real drill.

---

## Behavioral body

### Phase 0 — Entry probe

Action (silent, no learner output): Read `learner-state.json` and execute Resume Logic.

Action (silent, no learner output): If Resume Logic already selected a resume branch, do not emit a fresh-start discovery menu before jumping.

Say: `«Сейчас настроим тебе наблюдаемость для этой автоматизации: проверку здоровья, тревогу при сбое и понятные правила повторных попыток.»`

Check: `«Готов? Это короткий проход, минут на 15–20.»`

Action (silent, no learner output): Initialize or refresh `arch_blocks.<invoking_block>.observability` as `status: "in_progress"`, `current_phase: 1`, `gate_passed: false`, preserving any already valid gate fields if this is a resume-from-partial path that was not restarted.

**Frame coverage:** G1.

### Phase 1 — MM1 + MM2

Say: `«Есть неприятный сценарий: автоматизация неделю работает, потом тихо ломается, и ты узнаёшь об этом только потому, что чего-то не получил.»`

Check: `«Видишь проблему? Тишина тут выглядит как будто всё нормально.»`

Say: `«Но и другая крайность плохая. Если тревога прилетает на каждый чих, ты перестаёшь смотреть даже на настоящую поломку.»`

Check: `«Логично держать тревоги как ограниченный бюджет внимания?»`

Action (silent, no learner output): If `silent-failure-steals-trust` or `alerts-are-a-budget` are absent in top-level `mental_models_taught`, merge them now. Update `arch_blocks.<invoking_block>.observability.current_phase = 2`.

**Frame coverage:** MM1, MM2.

### Phase 2 — Wire health-check

Say: `«Сейчас соберём проверку здоровья и сразу прогоним её руками.»`

Build:
- Pick the lightest real check that fits the invoking block's substrate:
  - `systemd_timer` if the workflow already runs on a host with systemd
  - `http_probe` if the workflow exposes a stable URL or endpoint
  - `cron_check` if the workflow is cron-based and no better probe exists
- Keep the hard target from the bundled gate:
  - cadence = every 15 minutes (`interval_seconds = 900`)
  - success rule = the check shows a healthy run inside the last two intervals
- Capture an implementation receipt in memory if available:
  - systemd unit name
  - probe URL
  - cron entry or wrapper script reference
- Run one real verification now.

If the build fails:

Say: `«Проверка здоровья не поднялась с первого раза.»`

Say: `«Могу 1 повторить сейчас, 2 взять другой тип проверки, 3 остановиться и оставить диагностический след.»`

Say: `«След лежит в логе этой проверки и в файле или юните, который я только что пытался собрать.»`

Check: `«Что делаем?»`

Branch:
- `1` → retry Phase 2 build once.
- `2` → stay in Phase 2, switch the check type, rebuild.
- `3` → Say: `«Ок, ставлю паузу. Вернуться можно той же командой.»`
  Action (silent, no learner output): persist `pending_resume = "pos-observability-phase-2"`, set `arch_blocks.<invoking_block>.observability.status = "partial"`, keep `current_phase = 2`, end immediately with the final literal marker.

Check: `«Проверка проходит? Если это systemd — есть успешный запуск. Если HTTP — видим здоровый ответ. Если cron — есть свежий успешный след.»`

Action (silent, no learner output): Persist `health_check.type`, `health_check.interval_seconds = 900`, `health_check.validation_rule = "exit 0 within last 2 intervals"`, optional `health_check.target` receipt if available, set `status = "in_progress"`, `current_phase = 3`.

**Frame coverage:** G2, F5.

### Phase 3 — Wire alert channel

Say: `«Теперь добавим тревогу. Не просто отправить, а проверить что ты её реально видишь.»`

Check: `«Куда шлём? Telegram, email или в лог, который ты правда смотришь?»`

Say: `«Сначала отправлю тест и подожду подтверждение от тебя.»`

Build:
- Configure alert dispatch for the chosen channel.
- If the learner already has a current Telegram surface from `/pos-telegram`, reuse it instead of inventing a new path.
- If the learner chooses `log`, remind them that the log must be a real watched surface, not a silent dump.
- Send one real test alert.
- Default alert conditions to the bundled gate values:
  - `failure_streak >= 3`
  - `timer_missed >= 1`

If the build fails or the learner does not receive the alert:

Say: `«Тревога не дошла как надо.»`

Say: `«Могу 1 повторить сейчас, 2 поменять канал, 3 остановиться и оставить диагностический след.»`

Say: `«След лежит в логе отправки и в конфиге канала, который мы сейчас пробовали.»`

Check: `«Что делаем?»`

Branch:
- `1` → retry Phase 3 build once.
- `2` → stay in Phase 3, choose another channel.
- `3` → Say: `«Ок, ставлю паузу. Вернуться можно той же командой.»`
  Action (silent, no learner output): persist `pending_resume = "pos-observability-phase-3"`, set `arch_blocks.<invoking_block>.observability.status = "partial"`, keep `current_phase = 3`, end immediately with the final literal marker.

Check: `«Получил?»`

Action (silent, no learner output): Persist `alert_channel.type`, `alert_channel.target`, `alert_channel.on_conditions = ["failure_streak >= 3", "timer_missed >= 1"]`, keep `status = "in_progress"`, set `current_phase = 4`.

**Frame coverage:** G3, F1.

### Phase 4 — MM3 + retry policy

Say: `«Повторная попытка полезна только против случайных сбоев. Если причина не ушла, бесконечные повторы просто шумят и прячут проблему.»`

Check: `«Ок, если держать повторы ограниченными и видимыми?»`

Say: `«Базовый вариант такой: 5 попыток, каждая следующая ждёт вдвое дольше. Минуту, потом две, потом четыре и так далее.»`

Check: `«Оставляем так или хочешь другой ограниченный вариант?»`

Say: `«Сейчас зафиксирую это в правилах блока, чтобы агент потом не додумывал поведение сам.»`

Build:
- Document the retry policy in the invoking block's rules surface.
- If the caller already resolved a concrete agent-config path, append there.
- If not, use the current project's primary agent-config file and append under the invoking block's section with append-only semantics.
- The stored policy must stay bounded:
  - `strategy = exponential_backoff | fixed | none`
  - `max_retries` finite
  - `base_delay_seconds` present when retries exist

If the build fails:

Say: `«Не получилось зафиксировать правила повторных попыток с первого раза.»`

Say: `«Могу 1 повторить сейчас, 2 показать точный блок и ты вставишь его сам, 3 остановиться и оставить диагностический след.»`

Say: `«След лежит в черновике секции правил и в diff того файла, куда мы пытались писать.»`

Check: `«Что делаем?»`

Branch:
- `1` → retry Phase 4 build once.
- `2` → show the exact retry-policy block, wait for learner confirmation that it is inserted, then continue only after explicit confirmation.
- `3` → Say: `«Ок, ставлю паузу. Вернуться можно той же командой.»`
  Action (silent, no learner output): persist `pending_resume = "pos-observability-phase-4"`, set `arch_blocks.<invoking_block>.observability.status = "partial"`, keep `current_phase = 4`, end immediately with the final literal marker.

Action (silent, no learner output): If `retry-is-not-recovery` is absent in top-level `mental_models_taught`, merge it now. Persist `retry_policy`, set `status = "in_progress"`, `current_phase = 5`.

**Frame coverage:** MM3, G4, F3.

### Phase 5 — MM4 + failure drill

Say: `«Теперь сделаем главный тест: специально сломаем этот поток и посмотрим, сработает ли вся цепочка целиком.»`

Check: `«Готов?»`

Say: `«После этого ты будешь знать не «надеюсь, увижу поломку», а «я уже видел, как это работает на практике».»`

Check: `«Идём?»`

Build:
- Break the invoking workflow in the safest reversible way that fits the block:
  - rename the script temporarily
  - disable the expected input briefly
  - stop the scheduler or make the probe fail in a reversible way
- Wait for the configured health-check to notice the break.
- Verify three things:
  - alert reached the chosen channel
  - retry behavior is visible in logs or job output
  - the break can be restored cleanly
- Restore the workflow and verify the healthy state returns.

If the drill fails to prove the chain:

Say: `«Дрилл не дал нужного сигнала: либо тревога не пришла, либо не видно повторов, либо восстановление не подтвердилось.»`

Say: `«Могу 1 повторить сейчас, 2 проверить канал и повторы по отдельности, 3 остановиться и оставить диагностический след.»`

Say: `«След лежит в логе проверки здоровья, в канале тревог и в конфиге, который мы только что проверяли.»`

Check: `«Что делаем?»`

Branch:
- `1` → retry Phase 5 build once.
- `2` → stay in Phase 5 and verify alert / retry / restore as separate sub-steps before repeating the drill.
- `3` → Say: `«Ок, ставлю паузу. Вернуться можно той же командой.»`
  Action (silent, no learner output): persist `pending_resume = "pos-observability-phase-5"`, set `arch_blocks.<invoking_block>.observability.status = "partial"`, keep `current_phase = 5`, end immediately with the final literal marker.

Check: `«Тревога пришла и в следах видно повторные попытки?»`

Action (silent, no learner output): If `trust-through-testing` is absent in top-level `mental_models_taught`, merge it now. Persist `first_failure_drill_done = true`, keep `status = "in_progress"`, set `current_phase = 6`.

**Frame coverage:** MM4, G5, F2.

### Phase 6 — Stamp and return

Action (silent, no learner output): Write the final gate stamp into `arch_blocks.<invoking_block>.observability`:
- `status = "done"`
- `gate_passed = true`
- `gated_at = <ISO8601>`
- `health_check`, `alert_channel`, `retry_policy` retained from earlier phases
- `sla_uptime_target = 0.95`
- `log_retention_days = 30`
- `first_failure_drill_done = true`
- top-level `pending_resume = null`

Say: `«Готово. Для этого блока наблюдаемость теперь не на словах, а с живой проверкой.»`

Say: `«Возвращаемся в основной блок.»`

Action (silent, no learner output): Return control to the invoking skill.

===END-OF-SKILL===

**Frame coverage:** G6, F4.

---

## References

- [state-contract.md](./state-contract.md) — canonical state shape for this mini-skill.
- The bundled `catalog/gates/observability.json` — gate spec this skill enforces.
- `../../docs/skill-contract.md` — normative contract.
