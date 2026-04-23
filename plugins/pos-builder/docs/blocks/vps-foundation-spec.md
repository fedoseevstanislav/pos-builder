---
status: draft
author: Alex Vasiliev (via JARVIS)
date: 2026-04-16
intended_for: review by Stas before Sun 19.04 meeting
---

# VPS Foundation — Architecture Block Spec

**Block ID:** #1 (arch blocks state.json)
**Layer:** Infrastructure
**Depends on:** `/pos-diagnostic` (soft — seeds route context and `my-architecture.md`, but the block can still be invoked directly)
**Maps to use case:** #1 «Фундамент: VPS, стек, безопасность, первый запуск» (scope=mvp, tier=basic)
**Skill command:** `/pos-vps-foundation` (proposed — single name for all prereq steps including security hardening)
**Type:** Teaching script with build steps
**Language:** Russian (learner-facing), English (internal)
**Duration:** 60-90 min (includes waiting for DNS, some provider variance)
**Mode:** Vibe-code (per block-runtime-pattern §2.6) — fast iterative build with agent executing, learner approving

---

## 0. Why this is a spec draft

This is my (Alex) proposal for how architecture blocks should be specced and implemented. Format follows Stas's `diagnostic-spec.md` as a reference. Sharing before Sun 19.04 meeting to calibrate: is this the level of detail we want? Too much / too little? Missing sections?

---

## 1. Purpose

VPS Foundation is usually the first architecture block a learner assembles after the Diagnostic. It produces a personal always-on Linux host running Claude Code (or Codex), properly secured, ready to be the target for every subsequent automation block.

**What the block accomplishes:**
1. **Provisions** a VPS appropriate to the learner's region and budget
2. **Secures** it with baseline hardening (SSH keys only, no password auth, firewall, auto-updates, fail2ban)
3. **Installs** Claude Code + required runtimes (Node.js LTS, Python 3.11+, git, tmux)
4. **Verifies** reachability and agent-reachability from the learner's laptop
5. **Writes** the VPS artefact into `my-architecture.md` so future blocks can reference `{vps_host}`, `{vps_user}`, `{ssh_key_path}`

The block does NOT deploy anything beyond the bare agent runtime. Every subsequent block (morning-brief pipeline, systemd timers, adapters) consumes the VPS as an already-present resource.

---

## 2. Output Artifacts

### 2.1 On the learner's laptop

- `~/.ssh/pos_vps_ed25519` + `.pub` — dedicated SSH key pair (new, not reusing other keys)
- `~/.ssh/config` updated with a `Host pos` alias pointing to the VPS
- Shell alias / function `pos` = `ssh pos` (learner-confirmed preference)

### 2.2 On the VPS

- Ubuntu 22.04 LTS or 24.04 LTS, kernel current
- User `pos` with sudo, SSH key installed, `PermitRootLogin no`, `PasswordAuthentication no`
- `ufw` active: default deny in, allow 22/tcp in (or custom SSH port), allow out all
- `unattended-upgrades` enabled for security patches
- `fail2ban` active with sshd jail
- Claude Code installed via npm (`npm install -g @anthropic-ai/claude-code` or whatever the current distribution is at course-author time)
- Node.js LTS (v20+), Python 3.11+, `git`, `tmux`, `htop`, `curl`, `jq`, `rsync` installed
- `/home/pos/pos/` directory created as the learner's working root for future blocks

### 2.3 In `my-architecture.md`

Section added/updated:

```markdown
## VPS Foundation
- **Host:** pos.<learner-domain-or-ip>
- **User:** pos
- **SSH key:** ~/.ssh/pos_vps_ed25519
- **Provider:** <hetzner/digitalocean/timeweb/...>
- **Region:** <fra1/nbg1/ru-1/...>
- **Spec:** 2 vCPU / 4GB RAM / 40GB SSD (or whatever the learner chose)
- **Hardening:** UFW, fail2ban, unattended-upgrades, SSH-key-only
- **Runtime:** Node v20.x, Python 3.11.x, Claude Code vX.Y.Z
- **Setup date:** YYYY-MM-DD
```

### 2.4 In the learner's Obsidian vault

- `00-Inbox/{pos-foundation} {journal} VPS Setup — <date>.md` — dated note capturing the learner's choices (provider, region, spec) and why, plus any issues they hit. Used for later reflection.

---

## 3. Block Runtime (per block-runtime-pattern.md)

Six steps, in order.

### 3.1 Gate

This block runs only when the Diagnostic recommends it in the route, or when the learner explicitly invokes `/pos-vps-foundation`. If the learner already has a VPS (captured in `my-architecture.md` from Diagnostic Phase 3), the block branches: skip provision, run only the "verify & bring to POS standard" path.

### 3.2 Pitch (3 short messages)

**Message 1:** «Сейчас поднимем тебе личный сервер в интернете — он станет домом для всего что ты будешь строить: утренние брифинги, обработка транскриптов, агенты которые работают пока ты спишь. Это не офисная тема — любой может арендовать VPS за $5-10 в месяц.»

**Message 2:** «Зачем нужен VPS, а не ноутбук — ноутбук закрывается, VPS работает 24/7. Когда мы перейдём к автоматизациям, которые должны срабатывать по расписанию или в ответ на события — без 24/7 они не работают.»

**Message 3:** «Время на этот блок — около часа. Пятнадцать минут выбор провайдера, оплата. Десять минут — базовая защита. Остальное — установка Claude Code и проверка что всё работает. Поехали?»

### 3.3 Architecture Context

Show from `my-architecture.md` what's already set up (from Diagnostic or previous blocks):
- Coding agent: ✅ installed locally (from Diagnostic Phase 1)
- Obsidian vault: {status}
- Current VPS: {none / existing — if existing, branch to "verify path"}

Explain what this block adds: «VPS — Infrastructure layer. После этого блока мы добавим сюда Storage (Obsidian sync) и Agent (CLAUDE.md для твоего VPS). Это фундамент под всё остальное.»

### 3.4 Propose Tools & Approaches

Region-aware provider recommendation. LLM asks:

Say: «Где ты физически находишься? Это нужно чтобы выбрать сервер поближе — задержка меньше, и некоторые провайдеры работают только в определённых регионах.»

Check: Wait for answer. Branch:

| Region | Recommended providers (in order) | Notes |
|--------|----------------------------------|-------|
| Россия | Timeweb Cloud, RuVDS, Selectel | Оплата картой Мир. Бонус: нет риска блокировки. Минус: хуже международная связность. |
| ЕС (включая UK) | Hetzner Cloud, DigitalOcean FRA1/NBG1 | Лучший $/perf у Hetzner. DO — проще UI. |
| ОАЭ / Ближний Восток | DigitalOcean Dubai, AWS me-central-1 | Dubai DC — лучшая latency для региона. |
| США / Канада | DigitalOcean, Vultr, Linode | Стандартный выбор. |
| Другое | Hetzner (работает из многих стран) | Fallback. |

Say: «Минимальная конфигурация: 2 vCPU, 4GB RAM, 40GB SSD — $5-10/мес. Этого хватит на все блоки курса. Если не уверен в оплате — начни с $5, апгрейд всегда можно сделать.»

**Alternate paths:**
- **«Не хочу платить сейчас»** → предложи локальный альтернативный путь (Docker на ноутбуке для обучения pattern'ов, с переходом на VPS когда учебные автоматизации станут критическими). Не блокер курса.
- **«У меня уже есть сервер»** → branch to verify/upgrade path.
- **«Не понимаю что выбрать»** → LLM предлагает Hetzner CPX11 (€3-4/мес, FRA datacenter) как default-safe для любого региона кроме РФ.

### 3.5 Learner Confirms

Learner picks provider + spec. LLM reads back: «Значит: Hetzner Cloud, CPX11 ($4.50/мес), регион Frankfurt, Ubuntu 22.04 LTS, ssh-ключ новый создадим. Всё верно?»

Learner confirms or corrects. No building before explicit yes.

### 3.6 Build

Step by step, each with a **Check:** gate before moving on. Agent does the execution, learner provides credentials when needed, approves each destructive / irreversible action.

**Build 1 — SSH key:**
- Agent generates `~/.ssh/pos_vps_ed25519` + `.pub` (no passphrase for now — teach secure passphrase in a later security block)
- Learner is told the public key path. They'll paste it into provider UI.

**Build 2 — VPS provisioning:**
- Agent walks the learner through the provider UI (Hetzner console, DO droplet create, etc.) — with screenshots/descriptions, not automated (provider UIs change; manual gives the learner signal they're in control).
- Learner pastes SSH public key into provider UI.
- Learner confirms region, spec, image (Ubuntu 22.04 LTS).
- Learner clicks "Create" — agent waits, asks for the assigned IP.

**Build 3 — First SSH + baseline hardening:**
- Agent generates `~/.ssh/config` entry, suggests learner adds a `pos` alias.
- Learner runs `ssh pos` — verifies first login (root or ubuntu user depending on provider).
- Agent generates a hardening script. Learner reads it, agent explains each line. Learner runs it. Script:
  - Creates `pos` user with sudo
  - Installs + configures ufw, fail2ban, unattended-upgrades
  - Disables root SSH, disables password auth
  - Forces SSH reconnect as `pos` — at this point root is locked out. **This is the critical step.** Agent does a final check: `ssh pos` works as `pos@vps` before terminating root session.

**Build 4 — Runtime install:**
- Agent runs (via SSH session or gives learner copy-paste):
  ```bash
  sudo apt update && sudo apt install -y nodejs npm python3.11 python3.11-venv git tmux htop jq rsync curl
  # Or use nvm for Node LTS matching Claude Code requirement
  npm install -g @anthropic-ai/claude-code
  claude --version
  ```
- Learner sees version string → first confirm that something ran correctly on their server.

**Build 5 — Working directory:**
- Agent creates `~/pos/` as the learner's working root
- `mkdir -p ~/pos/{scripts,logs,data}`

**Wow moment checkpoint:** «Теперь набери `ssh pos` и запусти `claude`. Ты в Claude Code на своём сервере. Попроси его что-нибудь — создать файл, проверить диск, что угодно. Попробуй и возвращайся.»

### 3.7 Track

Agent updates `my-architecture.md` VPS section (§2.3). Appends an entry to `00-Inbox/{pos-foundation} {journal} VPS Setup — <date>.md` with the learner's choices + any problems hit + resolution.

Sets `arch_blocks.vps_foundation.status = "done"` in a future state-tracking file (not yet specified — tracked in a separate architecture-tracking issue).

---

## 4. Resume Behavior

On re-invocation, the block checks `my-architecture.md`:

- **No VPS section** → start from Gate
- **VPS section present, status=partial** → show the learner their current state, ask which step they're on: «Вижу что ты начал — VPS поднят в {provider}, но harden не проходил. Продолжим с harden?»
- **VPS section present, status=done** → offer verify-path: SSH check, Claude Code version check, ufw status check, report any drift

---

## 5. LLM Behavioral Rules

Same as diagnostic-spec §5 plus:

8. **Destructive action gate:** Before any command that locks the learner out of their server (disable root SSH, ufw default deny) — the LLM MUST state explicitly: «Сейчас я запущу команду которая закроет root-доступ. Если что-то пойдёт не так — пересоздадим VPS. Готов?» and wait for explicit «да» or «поехали».

9. **Cost transparency:** Every time the block recommends a paid service, state the monthly cost in USD/EUR/RUB explicitly. Never hide cost behind "small fee" or "affordable".

10. **Region sensitivity:** Never recommend a provider the learner can't realistically pay from their country. RU learner → don't push DO or AWS without mentioning payment complications; EU learner → don't push Timeweb without explaining why.

11. **Don't automate provider UI:** Explicitly. The learner clicks through provider dashboards themselves. This is slow, but it's learnable, transferrable, and makes the learner feel in control (not that a magic agent did arcane things to their credit card).

---

## 6. Dependencies

| Dependency | Type | Description |
|---|---|---|
| `/pos-diagnostic` | Soft prior skill | Preferred because it seeds `my-architecture.md` with at least a Current state section. If diagnostic was skipped, block offers to run a 5-min abbreviated version first. |
| Learner's credit card / payment method | External | Cannot be automated. Block halts at Build 2 if learner can't pay. |
| `my-architecture.md` | State file | Read at Gate (branch detection) and updated at Track. |
| Provider API tokens | Optional, v1 | For advanced learners: provider CLI/API for fully scripted provisioning. Out of scope for MVP — manual UI path is the teachable default. |
| Next block: `/pos-foundation` | Outbound | This block's Wow moment ends with "your agent is on your server" — `/pos-foundation` is the natural next call to start populating CLAUDE.md + Obsidian. |

---

## 7. Common Pitfalls (for author reference)

These are things I (Alex, 18+ years ops) know will bite beginners. Block content should anticipate:

- **DNS / propagation time** — if learner uses a domain, propagation can be 5-60 min. Don't wait synchronously; teach them to use IP directly first.
- **Provider UI drift** — screenshots go stale monthly. Descriptions in the block should be tool-agnostic («найди кнопку Create Droplet / Create Server / Создать сервер»).
- **SSH key permissions** — on macOS/Linux, `chmod 600` for private key, or SSH refuses. Explicit step.
- **Windows learners** — WSL2 vs PowerShell vs PuTTY. Default: WSL2, with explicit note "если ты на Windows и нет WSL, давай сначала поставим — займёт 10 минут".
- **Corporate laptops** — some block outbound SSH to port 22. Teach how to switch SSH to 443 if this bites (rare, but real for enterprise-issued laptops).
- **Firewall rule ordering** — ufw default deny + allow 22/tcp — if the allow isn't applied before the deny, learner is locked out. Script must sequence correctly.
- **DNS AAAA records / IPv6** — if VPS has IPv6 and learner's ISP doesn't, weird connection flaps. Teach `ssh -4` as a backup.

---

## 8. Out of Scope

- **Advanced security hardening** — AIDE, auditd, SELinux/AppArmor tuning, key rotation policies. Those belong to a later security-hardening block (not in MVP).
- **Backups** — VPS provider snapshots mentioned briefly; proper backup strategy is a separate block.
- **DNS / domain purchase** — optional appendix. Most MVP learners use IP addresses. Domain is a nice-to-have, not prerequisite.
- **Container / Docker / Kubernetes** — introduced later when needed (e.g., running isolated services). MVP uses bare-metal apt + systemd.
- **Multi-region / HA** — absurd for a personal VPS at this tier.

---

## 9. Cross-checks (before author-complete)

- [ ] Pair with an actual beginner through this block end-to-end; time each step; note where they stall.
- [ ] Verify hardening script on fresh Ubuntu 22.04 + 24.04 LTS images across 2 providers (Hetzner, DO).
- [ ] Verify Claude Code install path current against Anthropic's latest release notes.
- [ ] Confirm block-runtime-pattern §3 "friction reduction on iteration" — first draft vs. pass-2 wording.

---

## 10. Open questions for Stas

1. **Slash command name:** `/pos-vps-foundation` or keep original `/pos-foundation` (which absorbs the VPS as a sub-step)? I prefer explicit `/pos-vps-foundation` because the block is substantial on its own and the learner earns a completion marker.

2. **Region branching depth:** I listed 5 regions × 2-3 providers each. Is this too opinionated? Should we defer provider choice to the learner's own research (with just principles)?

3. **"No VPS budget" fallback:** I propose a local-Docker path as non-blocker. You ok with this, or do you want the block to insist on VPS (teaching "invest in your infrastructure" as a principle)?

4. **Format feedback:** Is this spec level-of-detail appropriate? For my other 15 blocks I'd use the same template. If you want shorter / longer / different sections, tell me now.

5. **Security ownership:** Use case #1 description says "Про безопасность надо будет подумать отдельно как научить пользователя постоянно о ней думать, это важно". Is security the responsibility of this block (baseline hardening only) or a separate transversal concern? My position: baseline here, deep security as a later dedicated block per the v1 architecture's "Secrets / Logging / Provenance / Watchdog" cross-cutting principles.
