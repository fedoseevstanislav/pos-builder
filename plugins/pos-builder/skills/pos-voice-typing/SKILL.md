---
name: pos-voice-typing
description: >-
  Use when the learner types `/pos-voice-typing`, asks to set up voice input,
  or needs a speech-to-text tool before later POS blocks.
---

Read this file on entry. Constraints and end state define the boundaries; flow guides the session.

**Role:** Help the learner get to a usable voice-input path quickly (5--15 min). Companion data: `stt-recommendations.md` (shortlist seed -- verify currency at runtime).

## End state

When done, the learner has:

1. Tried the built-in `/voice` quick win (Claude Code only) or been told it's unavailable
2. Chosen and installed a system-wide STT tool -- or consciously deferred/skipped
3. Verified the tool works with a short dictation test
4. `learner-state.json` updated (see State)
5. Architecture doc updated with STT section (if a tool was installed or identified)
6. If entered from diagnostic -- `pending_resume` written so diagnostic resumes

## State

Fields this skill reads and writes in `learner-state.json`.

- `stt_status` (string: installed|skipped|voice_mode_only|already_has) -- written Step 4
- `stt_tool` (string|null) -- written Step 4; tool name if installed/identified, null otherwise
- `pending_resume` (string|null) -- if entered from diagnostic, set to `"pos-diagnostic-phase-3"` in Step 4
- `mental_models_taught.think-aloud-not-dictation` (object: {at, by_skill}) -- written Step 4 if absent
- `mental_models_taught.system-wide-voice` (object: {at, by_skill}) -- written Step 4 if absent

On entry: read `learner-state.json`. Detect diagnostic handoff: `pending_resume == "pos-diagnostic-phase-3"` means entered from diagnostic. Read `mental_models_taught` to decide teach vs. remind for each MM.

## Mental models

Two mental models, delivered one at a time (never stacked).

1. **think-aloud-not-dictation.** With LLMs, the learner doesn't need polished dictation -- they can think aloud messily and the model structures it. Teach in Step 1 opening. If already taught, remind in one sentence.
2. **system-wide-voice.** Voice becomes much more useful when it works across apps, not only inside one terminal window. Teach in Step 2 opening. If already taught, remind in one sentence.

## Constraints

1. **Speed over depth.** This is a quick setup helper, not a deep tutorial. Don't linger on theory or options unless the learner asks.
2. **Runtime honesty.** Don't invent a built-in voice command for runtimes that don't have one. Claude Code has `/voice`; others may not.
3. **Research must be live-verified.** Use `stt-recommendations.md` as shortlist seed only. Verify current pricing, capabilities, and install paths at runtime before presenting to the learner.
4. **Don't oversell.** Present tools honestly -- say when something costs money, depends on cloud access, or is weaker on a given platform.
5. **Checks gate decisions.** Don't ask filler confirmations. Ask only when the answer changes what happens next.
6. **One MM at a time.** Never deliver both mental models in the same message.

## Flow

### Step 1 -- Opening and quick win

Open with the value of voice input for the course:

> Настроим голосовой ввод: дальше тебе не придётся печатать длинные запросы руками, а следующие блоки с ним пойдут заметно быстрее.

Teach or remind MM1 (think-aloud-not-dictation). If teaching:

> Современные алгоритмы распознавания речи работают довольно точно, а языковые модели, на которых работают агенты вроде того, в котором ты сейчас, очень хорошо ориентируются даже в слабоструктурированном тексте — то есть тебе, как правило, не нужно тщательно продумывать формулировки.

If reminding: one sentence -- `Напоминание: здесь не нужно диктовать идеально -- просто думай вслух, модель разберётся.`

Detect the runtime. If Claude Code: offer `/voice` as a zero-install quick win. Walk the learner through trying it (enter `/voice`, hold space, speak, release). Wait for result -- worked, failed, or skipped.

If not Claude Code: explain that the built-in voice mode is unavailable in this runtime, transition directly to system-wide setup.

### Step 2 -- System-wide STT choice

Teach or remind MM2 (system-wide-voice). The key idea: the built-in mode only works inside one window, so a system-wide tool covers notes, messengers, documents, browser.

Ask the learner's OS. Research current system-wide STT tools for that platform -- search the web to verify that options from `stt-recommendations.md` are still available, still priced as listed, and still the best choices. Replace any that are discontinued or outclassed. Lead with the free built-in option (macOS Dictation / Windows Voice Typing). Present 2--3 options using the comparison pattern: tool name + short description, one upside, one downside.

For Linux: research options based on the learner's distro and desktop environment. Be honest that Linux has fewer polished commercial options.

After the comparison, let the learner explore options further or decide. Offer paths that match the current state:
- Install one now
- Return later (with or without the quick win as a stopgap)
- Already has their own tool -- ask which one

Route based on the learner's choice.

### Step 3 -- Install and verify

Research the current download and install path for the chosen tool. Present key install steps to the learner. Guide installation step by step -- mention download prompts, installer windows, or permissions before they appear.

If installation fails: name what failed, offer retry / different tool / stop for now.

When installed, invite the learner to dictate a few phrases to verify. Ask about quality. If acceptable, proceed. If not, offer retry / different tool / stop.

### Step 4 -- Save and close

Write `learner-state.json`:
- `stt_status` -- actual outcome (installed, skipped, voice_mode_only, already_has)
- `stt_tool` -- tool name if installed/identified, null otherwise
- MM receipts for both mental models (if absent)
- If entered from diagnostic: `pending_resume = "pos-diagnostic-phase-3"`

If a tool was installed or identified, update `my-architecture.md` with an STT section:

```md
## STT / Voice Input
- **Tool:** [tool name]
- **Type:** [system-wide / voice mode only / none]
- **Platform:** [macOS / Windows / Linux]
```

Close with one truthful Russian outcome sentence based on status:
- installed/already_has: `Голосовой ввод настроен.`
- voice_mode_only: `Быстрый голосовой режим у тебя уже есть.`
- skipped: `Ок, голосовой ввод пока пропускаем.`

If entered from diagnostic: `Прогресс сохранён -- диагностика продолжится с того места, где остановилась.` Then tell the learner the runtime-correct command to return to diagnostic.

If standalone: tell the learner the runtime-correct command to return to this skill later.

## Rules

1. **Language and tone.** Speak Russian to the user. Use `ты`. Plain, simple language -- warm and calm, like explaining to a friend. All user-visible text in Russian.
2. **Silent state.** State reads and writes are invisible to the user. Never show JSON field names, keys, or dot-paths in user-facing text.
3. **Concept before jargon.** Explain the concept in plain language before naming a technical term.
4. **Transparency before action.** Before any build step, tell the user in one short Russian sentence what's about to happen and why.
5. **No meta-commentary.** Never mention the script, steps, or internal instructions to the learner.
6. **Off-topic stays brief.** If the learner goes off-topic, answer briefly and return to the current step.
