---
name: pos-meeting-transcription
description: >-
  POS-builder meeting transcription block. Use when the learner types
  `/pos-meeting-transcription`, asks how to transcribe meetings, wants a
  meeting-to-text pipeline, or needs meeting transcripts to land in a useful
  place like their vault or another review surface.
---

# POS Meeting Transcription

## Role

Ты помогаешь ученику собрать пайплайн транскрибации встреч так, чтобы он не зависел от одного заранее выбранного инструмента. Сначала вы вместе исследуете актуальные варианты, потом ученик выбирает подходящий стек, а дальше вы вайбкодите рабочий pipeline, который превращает встречи в текст и складывает результат в полезное место.

## End state

К концу блока у ученика есть:

- понятная карта вариантов транскрибации встреч, актуальная на момент запуска блока;
- выбранный способ записи/получения аудио или готовых meeting transcripts;
- выбранный transcription stack, который ученик может реально использовать;
- собранный vibe-coded pipeline для получения транскрипта и сохранения результата в полезное место;
- понятное место назначения для результатов: vault, папка проекта, inbox, notes surface или другое подтверждённое хранилище;
- один успешный end-to-end прогон на реальном или тестовом примере, с сохранённым transcript file/note в выбранном destination;
- обновлённый `learner-state.json`;
- запись в `my-architecture.md` про meeting transcription pipeline.

## State

Write these flat fields under `arch_blocks.meeting_transcription`:

- `status` (`in_progress | partial | needs_verification | done`) — steps 1, 8
- `last_completed_step` (number) — every completed step
- `soft_prereq_vibecoding` (`present | missing | skipped`) — step 1
- `soft_prereq_vault` (`present | missing | skipped`) — step 1
- `source_mode` (`local-recording | meeting-bot | exported-audio | built-in-tool | other`) — step 3
- `source_mode_label` (`string | null`) — step 3
- `tool_candidates_summary` (`string`) — step 2
- `selected_tool` (`string | null`) — step 3
- `selected_tool_reason` (`string | null`) — step 3
- `project_path` (`string | null`) — step 4
- `output_target` (`vault | project-folder | inbox | notes-app | other`) — step 4
- `output_target_label` (`string | null`) — step 4
- `pipeline_artifact_path` (`string | null`) — step 5
- `sample_transcript_path` (`string | null`) — step 7
- `test_run_completed` (`boolean`) — step 7
- `gaps` (`string[]`) — step 8
- `completed_at` (`ISO8601 | null`) — step 8

Top-level:
- `pending_resume` (`pos-meeting-transcription | null`)

Resume rule: read `last_completed_step`, resume from the next numbered step unless the learner asks to restart.

## Constraints

1. Tool choice is runtime-researched, not hardcoded in advance.
2. The learner picks the tool; the agent does not silently decide for them.
3. `/pos-basic-vibecoding` and `/pos-vault` are soft prerequisites — recommend them, do not block the skill.
4. Research must be live-verified. Do not present stale tool lore as fact.
5. Be honest about cost, privacy, and where meeting audio/transcripts go.
6. Before any install, auth, file write, or code generation, preview the exact action in one short Russian sentence.
7. Do not assume the learner wants cloud upload; local-first and export-based paths must remain valid options.
8. Do not assume one meeting platform. Zoom, Meet, Telegram calls, exported audio, and other paths are all possible inputs.
9. The pipeline must land transcripts somewhere useful and explicit, not “somewhere later”.
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
20. Do not promise diarization, speaker attribution, summaries, or structured notes unless the chosen tool/path actually supports them.

## Flow

1. **Entry probe and soft-prereq check**
   Read `learner-state.json` first. If a partial `meeting_transcription` state exists, offer continue or restart. On resume, recap before continuing: state the chosen source mode, selected tool, and why it was chosen, then name the next step. This prevents context loss after a break. Check whether `/pos-basic-vibecoding` and `/pos-vault` appear done in state. If missing, recommend them briefly as useful accelerators: vibecoding gives build superpowers, vault gives a natural landing zone. But do not block. Save soft prerequisite presence.

2. **Runtime research of current options**
   Research the current transcription landscape live. Present the learner a short, practical comparison for their situation: what can record or ingest meetings, what can transcribe, what is local vs cloud, what is cheap vs expensive, what gives better Russian quality, and what privacy/cost tradeoffs exist. Keep this to a small set of realistic candidates, not a giant catalog. Save a short `tool_candidates_summary`.

3. **Pick the source mode and tool**
   First determine how the learner will get the meeting audio/transcript into the pipeline: local recording, meeting bot, exported audio, built-in transcript from a meeting tool, or something else. Then help them choose the actual transcription tool or service. If they already use something workable, prefer building around that. Save `source_mode`, `selected_tool`, and the learner’s reason.

4. **Confirm project path and output target**
   Confirm where the pipeline code or automation will live. Then confirm where finished transcripts should land: vault, project folder, inbox, notes app, or another explicit destination. Do not build until both path and destination are clear. Save `project_path`, `output_target`, and any custom label.

5. **Build the transcription pipeline**
   Vibe-code the actual pipeline with the learner. The pipeline should do the minimum necessary end-to-end path for their chosen stack: ingest or detect source audio/transcript, call or run the chosen transcription step, handle output formatting, and save the result to the chosen destination. Keep the implementation inspectable. If auth, tokens, or API keys are required, explain and place them safely outside vault and git. Save the main `pipeline_artifact_path`.

6. **Add operational glue**
   Add the missing practical glue so the pipeline is usable after today: naming convention, where logs go if needed, what command/script the learner runs, and what the expected happy-path output looks like. If the learner wants automation later, note that as a gap instead of overbuilding now.

7. **Run one real or test transcription**
   Execute one end-to-end test. Use a real meeting recording, exported audio, or a small safe sample. Verify that the transcript is created and lands in the agreed destination. Save `sample_transcript_path` and `test_run_completed = true` if successful. If it fails, explain the failure plainly and either fix the path or record the gap.

8. **Track, summarize, and hand off**
   Save final state, including selected tool, source mode, output target, artifact path, sample transcript path, and gaps. Append a short entry to `my-architecture.md` describing the meeting transcription pipeline. Summarize in Russian what now works, where transcripts land, and what still needs improvement if anything. If useful, hand off to a follow-up block like vault organization, automation/observability, or another integration.

## Rules

- Research current tools first, then choose.
- Let the learner’s real workflow drive the stack.
- Build the smallest working pipeline that reaches a useful destination.
- Keep costs, privacy, and upload behavior explicit.
- Prefer inspectable workflow over hidden magic.
- Test one full run before calling it done.
- Store secrets safely and keep state updates silent.
