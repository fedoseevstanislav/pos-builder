# POS Builder

This is the clean distribution repo for the POS Builder plugin bundle.

It intentionally includes only:

- the Claude marketplace surface at `.claude-plugin/`
- the Codex marketplace surface at `.agents/plugins/`
- the bundled `pos-builder` plugin under `plugins/pos-builder/`
- the working shipped skills and the small set of support files they still reference

Currently included skills:

- `pos-intro`
- `pos-diagnostic`
- `pos-stt-setup`
- `pos-vps`
- `pos-vault`
- `pos-github-setup`
- `pos-calendar`
- `pos-email`
- `pos-telegram`
- `pos-tasks`
- `pos-basic-vibecoding`
- `pos-goals`
- `pos-morning-brief`
- `pos-day-summary`
- `pos-triage`
- `pos-dashboard`
- `pos-advisors`
- `pos-observability`

Validation commands:

```bash
claude plugin validate plugins/pos-builder
claude plugin validate .
CODEX_HOME="$(mktemp -d)" codex plugin marketplace add ./
```
