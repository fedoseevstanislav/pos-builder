# STT Tool Recommendations — Reference Data

Internal reference for the LLM. NOT shown to the learner.
The LLM reads this to give accurate, specific recommendations.

Last updated: 2026-04-13.

---

## macOS

### 1. Wispr Flow (best paid)

- **Price:** $12-15/month. Free tier: 2,000 words/week.
- **How it works:** Cloud-based. Audio sent to servers (SOC 2 Type II certified). Ensemble of models (Whisper, Gemini, Scribe).
- **System-wide:** Yes, works in any text field in any app.
- **Russian:** Excellent. 100+ languages with auto-detection.
- **Key strengths:** AI cleanup — removes filler words ("ну", "это", "как бы"), handles self-corrections (if you restart a sentence, keeps only the final version), auto-punctuation, adapts formatting style per app. Command Mode for voice-driven editing. Custom dictionary.
- **Key weaknesses:** Requires internet. No offline mode. $12-15/month is premium pricing.
- **Installation:** Download from wispr.com. Drag to Applications. Grant Accessibility and Microphone permissions. Activate with global hotkey (configurable).
- **Why recommend first:** Best accuracy for Russian, AI cleanup saves real editing time, cross-platform (also works on Windows, iOS, Android). Free tier is enough to test and decide.

### 2. Superwhisper (best for privacy / offline)

- **Price:** $8.49/month, $85/year, or $250 lifetime (one-time).
- **How it works:** Local processing. Runs Whisper models on-device. No data leaves the Mac.
- **System-wide:** Yes, menu bar app, works in any text field.
- **Russian:** Good with large Whisper model (large-v3). Quality depends on model size — large = good, small = mediocre.
- **Key strengths:** 100% offline and private. No audio sent anywhere. Lifetime purchase option = no subscription. 100+ languages.
- **Key weaknesses:** Mac-only. Slower with large models on older Macs (M1 is fine, Intel is slow). Less "smart" formatting than Wispr — closer to raw transcription. No filler removal.
- **Installation:** Download from superwhisper.com. Install. Download preferred Whisper model (large-v3 recommended for Russian, ~3GB). Grant Accessibility and Microphone permissions.
- **Why recommend second:** Best option if the learner cares about privacy or has unreliable internet. Lifetime purchase is attractive vs. monthly subscription.

### 3. macOS Dictation (free built-in)

- **Price:** Free. Built into macOS.
- **How it works:** On-device on Apple Silicon (M1+). Falls back to cloud on Intel Macs.
- **System-wide:** Yes, works in any text field.
- **Russian:** Acceptable. Below Whisper-based tools in accuracy, but usable for casual input.
- **Key strengths:** Zero cost, zero setup. Already installed. Works offline on Apple Silicon.
- **Key weaknesses:** No AI cleanup — raw transcription only. No filler removal. Lower accuracy than Wispr/Superwhisper. Session time limits. Less reliable with complex or fast speech.
- **Installation:** System Settings → Keyboard → Dictation → Enable. Set language to Russian. Shortcut: double-press Fn key (configurable).
- **Why recommend third:** Zero-commitment option. Good enough to test whether voice input works for the learner before investing in a paid tool.

---

## Windows

**Important note:** The Windows ecosystem is weaker for Russian-language STT than macOS. The built-in option (Windows Voice Typing, Win+H) has known issues with Russian recognition on Windows 11 — it frequently fails even with the Russian speech pack installed. Be honest about this when presenting recommendations.

### 1. Wispr Flow (best paid, same as Mac)

- **Price:** $12-15/month. Free tier: 2,000 words/week.
- **How it works:** Cloud-based. Same technology as Mac version.
- **System-wide:** Yes, works in any text field across 20,000+ apps.
- **Russian:** Excellent. Same model ensemble as Mac.
- **Key strengths:** Cross-platform — if the learner later gets a Mac or uses both, same tool. Best accuracy. AI cleanup. Free tier to test.
- **Key weaknesses:** Requires internet. Subscription pricing.
- **Installation:** Download from wispr.com. Run installer. Grant microphone permissions. Activate with global hotkey.
- **Why recommend first:** On Windows, this is the safest choice for Russian. Proven quality, same experience as Mac.

### 2. TypeWhisper (best free, open-source)

- **Price:** Free (GPL v3). Open-source on GitHub.
- **How it works:** Local engines — Parakeet TDT 0.6B (English-focused) or Canary 180M Flash (multilingual, includes Russian) — run on CPU (no GPU needed). Also supports cloud backends (Groq Whisper, OpenAI, Deepgram) as optional alternatives.
- **System-wide:** Yes, global hotkey with auto-paste into any app.
- **Russian:** Works via Canary 180M Flash engine (NVIDIA NeMo, multilingual). Learner must switch engine in settings — default is Parakeet TDT, which is English-focused.
- **Key strengths:** Free, open-source, local by default (private), no subscription, plugin architecture, per-app profiles, HTTP API.
- **Key weaknesses:** Young project (released February 2026) — may have rough edges. Needs engine switch to Canary for Russian (not default). Requires some technical comfort to install from GitHub.
- **Installation:** Download from GitHub releases. Run installer or extract. First launch downloads the selected engine (~300-600MB). Configure hotkey in settings, select Canary engine for Russian.
- **Why recommend second:** Best free option with local processing. Worth trying once Canary engine is selected. If local quality is not good enough, switch to cloud backend (Groq Whisper) for better results.

### 3. WhisperTyping (free alternative)

- **Price:** Free.
- **How it works:** Cloud-based. Uses OpenAI Whisper Large model. Virtual keyboard types into any app.
- **System-wide:** Yes, via virtual keyboard injection.
- **Russian:** Whisper Large handles Russian well. No specific complaints in reviews.
- **Key strengths:** Free, uses the proven Whisper Large model, praised for accuracy on Reddit.
- **Key weaknesses:** Cloud-only (audio sent to OpenAI). Unsigned binary — Windows SmartScreen shows a security warning on first run (safe to dismiss, but can scare non-technical users). Less polished UI.
- **Installation:** Download from website. Windows will show SmartScreen warning — click "More info" → "Run anyway". Grant microphone permissions. Configure hotkey.
- **Why recommend third:** Free, proven Whisper model, but unsigned binary and cloud dependency make it less clean than TypeWhisper.

---

## Linux

No specific recommendations. Linux users are expected to be technical enough to find their own solution. Options are limited for system-wide dictation. If the learner asks, offer to help research — starting points: nerd-dictation (local Vosk-based, English-focused), Whisper-based custom scripts, or browser extensions as a workaround.

---

## Recommendation presentation guidelines

When presenting recommendations to the learner:

1. **Lead with the free tier / free option** framing — even Wispr Flow has a free tier (2K words/week). The learner shouldn't feel pressured to pay immediately.
2. **Be concrete about installation** — don't just say "download it", walk through the actual steps if asked.
3. **Be honest about tradeoffs** — cloud = better quality but sends audio; local = private but potentially lower quality.
4. **Match to the learner's signals** — if they mentioned privacy concerns in the diagnostic, emphasize Superwhisper/TypeWhisper. If they want "just works", emphasize Wispr Flow.
5. **Don't overwhelm** — present the top recommendation first, then mention the other two briefly. Only expand on alternatives if the learner asks.
