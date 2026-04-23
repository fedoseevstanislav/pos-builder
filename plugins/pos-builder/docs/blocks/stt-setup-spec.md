# STT Setup Skill Spec

**Skill command:** `/pos-stt-setup`
**Type:** Teaching script (CLAUDE.md) for LLM — scripted with conditional branches
**Language:** Russian (learner-facing), English (internal LLM instructions)
**Duration:** 5-15 min (depends on whether learner installs a tool)
**Prerequisites:** Claude Code running, called from `/pos-diagnostic` Phase 2
**Returns to:** `/pos-diagnostic` Phase 3

---

## 1. Purpose

The STT setup skill helps the learner enable voice input. It is called from the diagnostic when the learner agrees to set up speech-to-text. The skill accomplishes three things:

1. **Motivates voice input** — explains how LLMs handle spoken text and why voice input is a fundamental skill for working with AI, not just a convenience feature.
2. **Quick win with Claude Code Voice** — lets the learner try voice input immediately with zero installation.
3. **Recommends a universal STT tool** — explains why a system-wide tool is better than voice mode in one app, and gives platform-specific recommendations.

The skill does NOT install anything automatically. It recommends and helps if the learner asks.

---

## 2. Output

No separate output files. The skill updates the diagnostic state:

- `learner-state.json` → `stt_status`: one of `"installed"`, `"voice_mode_only"`, `"skipped"`
- `learner-state.json` → `stt_tool`: tool name if installed
- `my-architecture.md` → STT section updated (tool name, type)

---

## 3. Skill Flow

Four phases, executed in order. All phases are scripted (Say/Check/Action).

```
Phase 1: LLM + Voice pitch
  → Phase 2: Claude Code Voice Mode (test)
    → Phase 3: Universal STT recommendation
      → Phase 4: Track & return to diagnostic
```

### 3.1 Phase 1 — LLM + Voice Pitch

**Type:** Scripted (Say)

**Step 1.1 — Why voice + LLM is powerful:**

The key insight to convey: LLMs are fundamentally different from traditional software in how they handle spoken text. You don't need to speak in complete sentences or worry about structure. Just talk — the LLM restructures, cleans up fillers, and extracts meaning. This makes voice input not just "convenient dictation" but a qualitatively different way to interact with AI.

Points to cover:
- LLMs understand messy, stream-of-consciousness speech
- No need to speak perfectly — fillers, corrections, incomplete thoughts are all fine
- "Just talk and then LLM structures" is faster than typing for most people
- This is a skill you'll use throughout the entire course and beyond

### 3.2 Phase 2 — Claude Code Voice Mode

**Type:** Scripted (Say/Check) with test

**Step 2.1 — Explain voice mode:**

Tell the learner that Claude Code has built-in voice input. Explain activation:
1. Type `/voice` and press Enter to enable voice mode
2. Hold **Space** to talk (push-to-talk), release to send
3. Text appears as you speak

**Step 2.2 — Test:**

Propose the learner try it right now: "Enable voice mode and say something — ask me a question or tell me something about yourself."

Wait for the learner to try. If it works — celebrate briefly and move on. If it doesn't work (microphone issues, permissions, SSH session) — acknowledge, skip, and move to Phase 3.

### 3.3 Phase 3 — Universal STT Recommendation

**Type:** Scripted (Say/Check) with conditional branches

**Step 3.1 — Why external tool is better than voice mode:**

Three points:
1. **Claude Code Voice works only in this window.** Close the terminal — voice input is gone.
2. **External STT = one skill, every app.** Obsidian, Telegram, browser, documents — you dictate the same way everywhere.
3. **You're learning to work with voice as a habit**, not as a feature of one program. This is a foundational skill for working with AI.

**Step 3.2 — Ask platform:**

Ask: "What operating system do you use? Mac or Windows?"

**Step 3.3 — Recommendations:**

Present top 3 tools for the learner's platform. Read tool details from `stt-recommendations.md`.

**Important framing note for Windows:** Windows ecosystem is weaker for Russian-language STT. The built-in option (Win+H) has known issues with Russian. Be honest about this — it's not a dealbreaker, but the learner should know.

After presenting recommendations:

| Learner says | Action |
|---|---|
| "Help me install [tool]" | Guide through installation step by step. Be patient, answer questions. |
| "I'll install later" | Fine. Note which tool they're interested in. |
| "I'll just use voice mode" | Fine. That's a valid choice. |
| "I already have STT" | Ask which tool, record it. |

### 3.4 Phase 4 — Track & Return

**Step 4.1 — Update state:**

Update `learner-state.json`:
- If learner installed a tool: `stt_status: "installed"`, `stt_tool: "<name>"`
- If learner chose voice mode only: `stt_status: "voice_mode_only"`
- If learner deferred: `stt_status: "skipped"`

Update `my-architecture.md` STT section if a tool was installed or identified.

**Step 4.2 — Return:**

Say: "Ready, let's continue the diagnostic." Return to `/pos-diagnostic` Phase 3.

---

## 4. LLM Behavioral Rules

1. **Same rules as diagnostic:** Return to script, no meta-commentary, supportive tone, no pressure.
2. **Don't oversell:** Present tools honestly. Don't hide costs or limitations.
3. **Installation depth:** Recommend, don't install automatically. If the learner asks for help — guide step by step with patience. If they don't ask — don't push.
4. **Platform honesty:** If Windows Russian STT is weak, say so. Don't pretend all platforms are equal.
5. **Speed:** This skill should feel quick (5-15 min). Don't turn it into a lecture. The pitch is 2-3 messages max, then action.

---

## 5. Dependencies

| Dependency | Type | Description |
|---|---|---|
| `/pos-diagnostic` | Calling skill | STT setup is called from Phase 2, returns to Phase 3 |
| `learner-state.json` | State file | Read on entry, updated on exit |
| `my-architecture.md` | Output file | STT section updated if tool installed |
| `stt-recommendations.md` | Reference | Tool details for recommendations |

---

## 6. Out of Scope

- **Custom STT pipelines** — no Whisper server setup, no API integration. That's for advanced blocks later.
- **Troubleshooting hardware** — if the microphone doesn't work, suggest the learner fix it outside the skill and come back.
- **Comparing all STT tools** — we recommend 3 per platform, not a comprehensive market review.
- **Linux STT** — Linux users are expected to be technical enough to find their own solution. Mention that options are limited and offer to help research if asked.
