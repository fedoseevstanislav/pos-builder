# POS Builder v2

Interactive course for building a personal AI operating system. Runs as a set of Claude Code / Codex skills (slash commands). The learner talks to the agent in natural language, and the agent builds their system step by step.

## Quick Start

**Claude Code:**
```
claude plugin install fedoseevstanislav/pos-builder
```

**Codex:**
```
codex plugin install fedoseevstanislav/pos-builder
```

Then type `/pos-intro` (Claude Code) or `/skill:pos-intro` (Codex) to begin.

## What's Inside

22 skills in minimal-frame format (Role, End state, State, Constraints, Flow, Rules).

**Старт:** Intro, Diagnostic, Voice Typing, Memory Basics
**Фундамент:** VPS, Vault, GitHub Setup
**Каналы:** Calendar, Email, Telegram, Tasks, Security
**Навыки:** Basic Vibecoding, Goals, Morning Brief, Day Summary, Triage, Meeting Transcription
**Дополнительно:** Dashboard, Advisors, Observability, Feedback

## Requirements

- Claude Code or Codex CLI
- The agent handles everything else during the course
