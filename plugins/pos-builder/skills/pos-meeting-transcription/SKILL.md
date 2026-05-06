---
name: pos-meeting-transcription
description: >-
  POS-builder meeting transcription block. Use when the learner types
  `/pos-meeting-transcription`, asks how to transcribe meetings, wants a
  meeting-to-text pipeline, or needs meeting transcripts to land in a useful
  place like their vault or another review surface.
---

Read this file on entry. Constraints and end state define the boundaries; flow guides the session.

# POS Meeting Transcription

## Role

Ты помогаешь ученику собрать пайплайн транскрипции встреч — без привязки к одному заранее выбранному инструменту. Сначала вместе исследуете актуальные варианты, потом ученик выбирает стек, а дальше вы вайбкодите рабочий пайплайн: встречи в текст, из текста — саммари, задачи, открытые вопросы, и всё это раскладывается по нужным местам.

## End state

К концу блока у ученика есть:

- понятная карта вариантов транскрипции встреч, актуальная на момент запуска блока;
- выбранный способ записи или получения аудио (или готовых транскриптов);
- выбранный стек транскрипции, который ученик может реально использовать;
- собранный vibe-coded пайплайн: получение транскрипта и сохранение результата в конкретное место;
- понятное место назначения: vault, папка проекта, inbox, заметки или другое подтверждённое хранилище;
- один успешный прогон на реальном или тестовом примере — транскрипт сохранён в выбранное место;
- извлечённые из транскрипта артефакты: саммари встречи, согласованные задачи, открытые вопросы — проверенные учеником;
- задачи и follow-up встречи отправлены в подключённые интеграции (трекер задач, календарь) или сохранены в файлы, если интеграции ещё не настроены;
- обновлённый `learner-state.json`;
- запись в `my-architecture.md` про пайплайн транскрипции встреч.

## State

Fields this skill reads and writes in `learner-state.json`. `last_completed_step` = last finished step number; resume starts at the NEXT step. Save state as the flow progresses — write relevant fields immediately at each milestone.

Write these flat fields under `arch_blocks.meeting_transcription`:

- `status` (`in_progress | done | incomplete`) — step 1, step 10
- `last_completed_step` (number) — every completed step
- `soft_prereq_vibecoding` (`present | missing | skipped`) — step 1
- `soft_prereq_vault` (`present | missing | skipped`) — step 1
- `tool_candidates_summary` (`string`) — step 2
- `source_mode` (`local-recording | meeting-bot | exported-audio | built-in-tool | other`) — step 3
- `source_mode_label` (`string | null`) — step 3
- `selected_tool` (`string | null`) — step 3
- `selected_tool_reason` (`string | null`) — step 3
- `project_path` (`string | null`) — step 4
- `output_target` (`vault | project-folder | inbox | notes-app | other`) — step 4
- `output_target_label` (`string | null`) — step 4
- `pipeline_artifact_path` (`string | null`) — step 5
- `sample_transcript_path` (`string | null`) — step 7
- `test_run_completed` (`boolean`) — step 7
- `extraction_validated` (`boolean`) — step 8
- `extracted_summary_path` (`string | null`) — step 8
- `tasks_routed_to` (`tracker | file | null`) — step 9
- `followups_routed_to` (`calendar | file | null`) — step 9
- `routing_gaps` (`string[]`) — step 9
- `gaps` (`string[]`) — step 10
- `completed_at` (`ISO8601 | null`) — step 10

Top-level:
- `pending_resume` (`pos-meeting-transcription | null`)

On entry: read `learner-state.json` and evaluate in this order:
1. `status == "done"` → show summary and offer to exit or start over.
2. `status == "incomplete"` → check which end-state items are missing and resume at the relevant step.
3. `last_completed_step` exists → resume from the next step.
4. No prior state → fresh start.

## Constraints

1. Tool choice is runtime-researched, not hardcoded in advance.
2. The learner picks the tool; the agent does not silently decide for them.
3. `/pos-basic-vibecoding` (или `/skill:pos-basic-vibecoding` в Codex) and `/pos-vault` (или `/skill:pos-vault` в Codex) are soft prerequisites — recommend them, do not block the skill.
4. Research must be live-verified. Do not present stale tool lore as fact.
5. Be honest about cost, privacy, and where meeting audio/transcripts go.
6. Before any install, auth, file write, or code generation, preview the exact action in one short Russian sentence.
7. Do not assume the learner wants cloud upload; local-first and export-based paths must remain valid options.
8. Do not assume one meeting platform. Zoom, Meet, Telegram calls, exported audio, and other paths are all possible inputs.
9. The pipeline must land transcripts in an explicit, useful destination confirmed before build.
10. Secrets stay secure — never in vault, never in git, never echoed back.
11. If a chosen tool needs auth or upload, explain that clearly before the learner commits.
12. Prefer small inspectable scripts/workflows over hidden automation magic.
13. The final pipeline must be end-to-end testable with one sample run.
14. If the learner already has a working transcription tool, build around it instead of forcing a replacement.
15. Keep teaching practical: explain the decision tradeoffs, then build.
16. If runtime research or install fails, explain plainly what failed, what the next action is, and where the evidence lives.
17. Update `my-architecture.md` append-only. Never overwrite prior notes silently.
18. Keep state writes silent.
19. If the learner’s requirement is underspecified, ask for the missing operational detail before building.
20. Never route extracted tasks or follow-ups to integrations without the learner validating the extracted items first.
21. Discover available integrations (tasks, calendar) from `learner-state.json` at runtime. Do not hardcode which integrations exist.
22. If an integration is not connected, save to files and tell the learner which skill to run to enable routing, then return here.
23. When the pipeline needs LLM processing, prefer the learner's primary agent (`claude -p` for Claude Code, `codex exec` for Codex) over requiring a separate API key. The learner already has a working subscription.

## Flow

1. **Entry probe and soft-prereq check**
   Read `learner-state.json`. If `meeting_transcription` state exists, follow the resume rule from the State section. On resume, recap before continuing: source mode, selected tool, reason, then name the next step. Check whether `/pos-basic-vibecoding` (или `/skill:pos-basic-vibecoding` в Codex) and `/pos-vault` (или `/skill:pos-vault` в Codex) appear done in state. If missing, recommend briefly: vibecoding gives build superpowers, vault gives a natural landing zone. Do not block. Save soft prerequisite presence. Write `status = "in_progress"` on fresh start.

2. **Runtime research of current options**
   Research the current transcription landscape live. Present the learner a short, practical comparison for their situation: what can record or ingest meetings, what can transcribe, what is local vs cloud, what is cheap vs expensive, what gives better Russian quality, and what privacy/cost tradeoffs exist. Keep this to a small set of realistic candidates, not a giant catalog. Save a short `tool_candidates_summary`.

3. **Pick the source mode and tool**
   First determine how the learner will get the meeting audio/transcript into the pipeline: local recording, meeting bot, exported audio, built-in transcript from a meeting tool, or something else. Then help them choose the actual transcription tool or service. If they already use something workable, prefer building around that. Save `source_mode`, `source_mode_label`, `selected_tool`, and `selected_tool_reason`.

4. **Confirm project path, output target, and preview full pipeline**
   Pipeline scripts and automation code should live in a git repo (check if the learner already has a repo for automations, e.g. `pos-automations`; if not, create one). Confirm the repo path. Then confirm where finished structured notes should land: vault, project folder, or another explicit destination. Do not build until both path and destination are clear. Before building, preview the full pipeline vision: transcribe → extract (summary, tasks, open questions) → learner validates → route to integrations (task tracker, calendar) or save to files. When confirming the output target, explicitly remind the learner: the structured note is just the base output — after we build it, we add routing that sends tasks to the task tracker and follow-up meetings to the calendar. This is not the final step. Save `project_path`, `output_target`, and any custom label.

5. **Build the transcription pipeline**
   Vibe-code the actual pipeline with the learner. The pipeline should do the minimum necessary end-to-end path for their chosen stack: ingest or detect source audio/transcript, call or run the chosen transcription step, handle output formatting, and save the result to the chosen destination. Keep the implementation inspectable. If auth, tokens, or API keys are required, explain and place them safely outside vault and git. Save the main `pipeline_artifact_path`.

6. **Add operational glue**
   Make the pipeline usable after today: naming convention for output files, where logs go if needed, what command or script the learner runs, and what the expected happy-path output looks like. If the learner wants automation later, note that as a gap instead of overbuilding now. No new state fields — save `last_completed_step` only.

7. **Run one real or test transcription**
   Execute one end-to-end test of the transcription stage. Use a real meeting recording, exported audio, or a small safe sample. Verify the transcript is created and lands in the agreed destination. Save `sample_transcript_path` and `test_run_completed = true` on success. If it fails, explain plainly and either fix or record the gap. Steps 8-9 will use this same transcript to build and test post-processing.

8. **Build the post-processing stage into the pipeline**
   Build in two layers: first create the extraction logic as a skill (a prompt file + invocation pattern using the learner's primary agent per Constraint 23). A skill is easy to test — the learner can invoke it on any transcript and see results immediately. Once the skill works, wrap it into the pipeline script so it runs as part of the automated flow.

   The extraction skill takes a transcript and extracts three categories:
   - meeting summary;
   - agreed-upon tasks (who, what, deadline if mentioned);
   - open questions or unresolved topics.

   Output: a structured meeting note saved to the output target. Before routing, the pipeline needs learner validation of extracted items. For the validation channel: check `learner-state.json` — if `arch_blocks.telegram.status == "done"`, propose sending the extracted results to Telegram for review (the learner can read and approve from their phone). Otherwise fall back to terminal or file-based review. Build this as a reusable part of the pipeline, not a one-time demo. Save `extracted_summary_path` and `extraction_validated = true` after the first successful validated run.

9. **Build routing into connected integrations**
   Check `learner-state.json` for connected integrations and wire up the corresponding routing as part of the pipeline:
   - If `arch_blocks.tasks.status == "done"`: add a routing step that creates validated tasks in the learner's task tracker. The pipeline presents proposed tasks and creates them on confirmation. Save `tasks_routed_to = "tracker"`.
   - If `arch_blocks.calendar.status == "done"`: add a routing step that creates follow-up meetings or calendar events from extracted items. Same confirm-then-create pattern. Save `followups_routed_to = "calendar"`.
   - If an integration is not connected: the pipeline saves the corresponding items to a local file (tasks as a checklist, follow-ups as a list with proposed dates). Tell the learner which skill to run to enable direct routing (`/pos-tasks` (или `/skill:pos-tasks` в Codex), `/pos-calendar` (или `/skill:pos-calendar` в Codex)), and that they can return to `/pos-meeting-transcription` (или `/skill:pos-meeting-transcription` в Codex) afterward to wire up the routing. Save `tasks_routed_to = "file"` or `followups_routed_to = "file"` accordingly. Record missing integrations in `routing_gaps`.
   - Summary and open questions always go to the output target (vault note or file) regardless of integrations.

10. **Track, summarize, and hand off**
    Check end-state items. If all critical items are met (pipeline built, test passed, extraction validated, routing wired or filed), write `status = "done"` and `completed_at`. If critical items are missing (e.g., test never passed, extraction not validated), write `status = "incomplete"` and record what is missing in `gaps`. `gaps` covers overall capability shortfalls; `routing_gaps` (from step 9) covers specifically which integrations are not connected. Append a short entry to `my-architecture.md` describing the full pipeline: transcription, extraction, validation, routing. Summarize in Russian what the learner now has and what each stage does. If routing gaps exist, name the skills that would close them. If useful, hand off to a follow-up block.

## Rules

- Research current tools first, then choose.
- Let the learner’s real workflow drive the stack.
- Build the smallest working pipeline that reaches a useful destination.
- Keep costs, privacy, and upload behavior explicit.
- Prefer inspectable workflow over hidden magic.
- Test one full run before calling it done.
- Extracted items are proposals until the learner validates them.
- Route to connected integrations; fall back to files and point to the missing skill.
- Store secrets safely and keep state updates silent.
