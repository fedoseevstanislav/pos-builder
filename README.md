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

Then type `/pos-intro` to begin.

## What's Inside

22 shipped skills covering: diagnostic interview, voice typing, VPS foundation, Obsidian vault, GitHub setup, calendar, email, Telegram, tasks, vibecoding basics, goals, morning brief, day summary, triage, dashboard, advisors, observability, meeting transcription, memory basics, security, feedback, and intro.

Each skill is a self-contained teaching script in minimal-frame format (Role, End state, State, Constraints, Flow, Rules).

## Requirements

- Claude Code or Codex CLI
- The agent handles everything else during the course
