---
name: pos-vps
description: >-
  Use when the user types `/pos-vps`, asks to set up a server, or needs the
  infrastructure foundation for always-on POS automations.
---

Read this file on entry. Constraints and end state define the boundaries; flow guides the session.

**Role:** Teach a non-developer to provision and harden a personal VPS, install AI agent runtimes, and wire a first live Telegram bot as proof of life.

## End state

When done, the user has:

1. A provisioned VPS in a non-RU region, reachable via `ssh pos` with key-only login (ed25519, password auth disabled, root login disabled)
2. Base hardening: UFW (default deny incoming, 22/tcp allowed), fail2ban, unattended-upgrades, 4 GB swap
3. Runtimes: Node 20 LTS (via NodeSource, not system apt), Python 3.11+, git, tmux, systemd
4. Primary agent runtime installed and authenticated (per `learner_profile.primary_agent`); verified: `<agent> --version` succeeds and a test prompt gets a response
5. If the learner opted in: second agent runtime installed and authenticated; verified: `<agent> --version` succeeds and a test prompt gets a response
6. OpenClaw installed, one Telegram bot configured, gateway process running and healthy
7. `~/.ssh/config` entry with `Host pos` alias on the user's local machine
8. A wow moment: user messages their TG bot from phone, gets a response from the VPS-hosted agent
9. `learner-state.json` updated (see State section)
10. Architecture doc updated with VPS section

## State

Fields this skill reads and writes in `learner-state.json`. `last_completed_step` = last finished step; resume starts at the NEXT step. Short field names below refer to their full path under `arch_blocks.vps_foundation.*`.

- `arch_blocks.vps_foundation.status` (string: in_progress|done|incomplete) — written Step 1, Step 12
- `arch_blocks.vps_foundation.last_completed_step` (number) — written at each step milestone
- `arch_blocks.vps_foundation.completed_at` (ISO8601|null) — written Step 12
- `arch_blocks.vps_foundation.provider` (string) — written Step 2
- `arch_blocks.vps_foundation.region` (string) — written Step 2
- `arch_blocks.vps_foundation.monthly_cost` (string) — written Step 2
- `arch_blocks.vps_foundation.host` (string) — written Step 4
- `arch_blocks.vps_foundation.os` (string) — written Step 4
- `arch_blocks.vps_foundation.ssh_key_path` (string) — written Step 3
- `arch_blocks.vps_foundation.primary_agent_authenticated` (boolean) — written Step 8
- `arch_blocks.vps_foundation.secondary_agent_authenticated` (boolean|null) — written Step 8 if opted in, null otherwise
- `arch_blocks.vps_foundation.bot_telegram_handle` (string) — written Step 9

On entry: read `learner-state.json`. Check `last_completed_step` and resume from the next step. If `status == "incomplete"`, check which end-state items are missing and resume at the relevant step. Save state as the flow progresses.

Read-only dependencies: `learner_profile` (diagnostic must be complete). Soft: `arch_blocks.obsidian_vault.path` — if present, automation results land in the vault; if absent, VPS setup proceeds normally (vault is not required for infrastructure).

## Constraints

1. **Non-RU region required.** Claude Code requires unrestricted access to `api.anthropic.com`. Connections from Russian IPs are blocked. The VPS must be outside Russia. Warn RU-based users explicitly.
2. **SSH lockout prevention.** Before disabling root login or password auth, the three-step verification sequence in Step 5 is mandatory. Never skip or compress it.
3. **UFW order is critical.** `ufw allow 22/tcp` MUST execute before `ufw enable` / default deny. Wrong order = lockout. After enabling UFW, verify SSH still works from a new terminal before proceeding.
4. **Secrets stay secure.** SSH keys, API tokens, and bot tokens never inside the Obsidian vault, never inside any git-tracked directory. The agent must never echo, log, or display tokens or secret values in chat. When showing proposed config files that contain secrets, mask the secret values.
5. **No root after bootstrap.** Switch to the `pos` user as soon as it exists. Never run interactively as root after initial setup.
6. **User navigates provider UI.** The user clicks through provider dashboards themselves. Do not automate provider UI. Guide step by step, but the user does the clicking.
7. **Honest about cost.** State exact monthly prices in the user's currency. Never hide behind "small fee" or "affordable."
8. **Payment-accessible providers only.** Never recommend a provider the user cannot realistically pay from their country.
9. **Research must be live-verified.** When presenting provider options, verify current pricing and availability at runtime. Use `references/providers-matrix.md` as baseline, but check for changes.
10. **Anthropic access warning.** Before Claude Code auth, confirm the user has safe access to `anthropic.com` (VPN or non-RU location). Do not proceed without confirmation.
11. **Three separate auth systems.** Claude Code (Anthropic), Codex (OpenAI), and OpenClaw bot (Telegram Bot API) are independent. Never mix or confuse them.
12. **Node 20 via NodeSource.** Ubuntu 24.04 ships Node 18 via apt, which is insufficient for Claude Code. Always install Node 20 via NodeSource. Verify `node --version` shows 20.x.

## Flow

### Step 1 — Prerequisites and entry

Check prerequisites:
- **Hard:** `learner_profile` must exist. If absent, tell the user to run `/pos-diagnostic` first. Stop.
- **Soft:** `arch_blocks.obsidian_vault.path`. If present, note it for later (architecture doc, automation output paths). If absent, proceed -- VPS does not depend on vault.
- **Resume:** If `status == "in_progress"`, read `last_completed_step`, tell the user where they left off, pick up from the next step. If `status == "incomplete"`, check which end-state items are missing and resume at the relevant step.
- **Host-reachability guard:** If `host` is populated but does not respond to SSH, ask the user: (a) retry — maybe the server is temporarily unreachable, (b) mark current VPS state incomplete and start fresh from provider selection, or (c) exit now and re-run `/pos-vps` (or `/skill:pos-vps` in Codex) when ready.

Write (fresh start only): `status = "in_progress"`, `last_completed_step = 1`.

### Step 2 — Intro and provider selection

Deliver verbatim in Russian:

> Сейчас поднимем тебе личный сервер в интернете. На нём будет жить то, что должно работать 24/7 — диалог с агентом в Telegram, обработка транскриптов и подобные фоновые задачи.
>
> Ноутбук уходит в сон и теряет Wi-Fi. Сервер в дата-центре остаётся включённым, поэтому автоматизация не ждёт, пока ты не откроешь крышку.
>
> VPS — это просто компьютер в дата-центре. Ты арендуешь его помесячно, и он работает 24/7. На всё уйдёт час-полтора.

Then: confirm the user's location and payment method (determines provider tier). Read `references/providers-matrix.md`, filter to the user's constraints, present at most 3 options with exact prices and tradeoffs. Get explicit confirmation of provider + region + spec before proceeding. Default disk recommendation: 80 GB (40 GB is tight for real daily use).

Write: `provider`, `region`, `monthly_cost`, `last_completed_step = 2`.

### Step 3 — SSH key generation

Detect the user's OS (macOS/Linux/Windows). Generate SSH key pair (ed25519, `~/.ssh/pos_vps_ed25519`). Adapt commands to the user's OS. For Windows: try WSL first; if WSL fails within 15 minutes, fall back to Git Bash. Ask about passphrase explicitly -- do not default. Show how to copy the public key for the provider.

Write: `ssh_key_path`, `last_completed_step = 3`.

### Step 4 — VPS provisioning

Guide the user through creating the server in the provider's web UI, covering: account access, server creation, region, spec, OS selection (Ubuntu 24.04), SSH key, price confirmation. Surface provider-specific gotchas only when that provider is selected (e.g., on 4VPS the user must top up the balance before creating the server). Wait for the IP address.

If the provider gives root access, proceed. If non-root, adapt accordingly.

Write: `host`, `os`, `last_completed_step = 4`.

### Step 5 — First SSH, user setup, and root lockdown

Set up `~/.ssh/config` with `Host pos` alias pointing to the server IP. Verify SSH works. Create a `pos` user with sudo. Copy the SSH key to the `pos` user.

**Three-step lockout prevention sequence (mandatory, never skip):**

Tell the user:

> Сейчас критический момент. Мы заблокируем вход под root'ом — останется только пользователь pos. Но прежде — ТРОЙНАЯ проверка, чтобы не потерять доступ.

**Step A:** User opens a NEW terminal window (keep the current one open) and connects as `pos` with the SSH key. Must get explicit confirmation before proceeding. If it fails — DO NOT PROCEED. Debug first.

**Step B:** User runs `sudo whoami` in the new window. Must show `root`. If it fails — DO NOT PROCEED. Debug sudo first.

**Step C:** User confirms BOTH terminal windows are still open. Only then: set `PermitRootLogin no` and `PasswordAuthentication no` in `/etc/ssh/sshd_config`. Run `sudo sshd -t` to validate config syntax. If it fails — DO NOT restart sshd. Fix the config first. Then restart sshd.

**Post-lockdown verification:** User opens a THIRD terminal window and connects via `ssh pos`. If it works, safe to close old sessions. If it fails, use the existing `pos` session to revert sshd_config.

Update the local `~/.ssh/config` to change `User root` to `User pos`.

Windows note: if the user is on WSL or Git Bash, `ssh pos` works from that terminal. If they also want to connect from VS Code or PowerShell, the SSH config needs to be copied to `C:\Users\<name>\.ssh\config` -- but this is optional, not a blocker.

Write: `last_completed_step = 5`.

### Step 6 — Hardening

Teach the mental model: firewall, fail2ban, and auto-updates are the minimum routine that keeps a small personal server stable. Not security for security's sake -- stability.

**UFW (critical order):**
1. `sudo ufw allow 22/tcp` -- FIRST, before anything else
2. Verify the rule: `sudo ufw status numbered` must show 22/tcp ALLOW
3. `sudo ufw default deny incoming` -- explicitly set default policy to deny all incoming traffic
4. `sudo ufw enable` -- user confirms with `y`
5. Verify final state: `sudo ufw status verbose` must show `Default: deny (incoming)`, `Default: allow (outgoing)`, and `22/tcp ALLOW`. If any is missing, do not proceed -- debug.
6. Verify SSH still works from a NEW terminal window. If it fails, use the existing session to `sudo ufw disable` and debug.

**fail2ban:** Install, enable, verify `sudo fail2ban-client status sshd` shows the jail.

**unattended-upgrades:** Install, configure, verify active.

**Swap (4 GB):** Create swapfile, add to fstab, verify with `free -h`.

Write: `last_completed_step = 6`.

### Step 7 — Runtime install

Install Node 20 LTS via NodeSource (not system apt -- Ubuntu 24.04's default Node 18 is insufficient for Claude Code). Install Python 3.11+, git, tmux. If Ubuntu 22.04, add deadsnakes PPA for Python 3.11.

Verify versions: `node --version` must show 20.x, `python3 --version` must show 3.11+. If node shows 18.x, the NodeSource step failed -- debug before proceeding.

Write: `last_completed_step = 7`.

### Step 8 — Agent install

Read `learner_profile.primary_agent` to determine which runtime to install first. The primary agent is the required install; the second is optional.

Teach the mental model: Claude Code and Codex are two separate systems from two separate companies (Anthropic and OpenAI) with separate accounts and separate auth flows. Treating them as one thing creates avoidable confusion.

**Primary agent (required):**

If primary is Claude Code:
- **Before auth:** Confirm the user has safe access to `anthropic.com`. If in Russia, they need VPN or must auth through the VPS SSH tunnel. Do not proceed without confirmed safe access path.
- Install via npm, verify `claude --version`. Auth via `claude auth login` -- the user opens the browser link on their local machine, not the server. Verify with a test prompt.

If primary is Codex:
- Install (research current method at runtime), verify `codex --version`. Separate OpenAI account and auth. Verify with a test task.

**Auth failure handling.** If auth fails:
1. Tell the user what happened (exact error).
2. Check list: API key valid? Network connectivity OK? If Claude Code -- is `api.anthropic.com` reachable from the VPS (geo-block)?
3. One retry after the user confirms the fix.
4. If retry fails: do not mark the step complete. Tell the user the VPS is usable but agent auth needs to be resolved. Re-running `/pos-vps` (or `/skill:pos-vps` in Codex) resumes here.

**Second agent (optional):**

After the primary is authenticated, offer: "If you want both runtimes, we can set up [the other one] now. Or we can skip it and move on." If the user opts in, install and auth using the same flow as above (including failure handling). If they skip, set `secondary_agent_authenticated = null`. If the user opted in but auth failed, set `secondary_agent_authenticated = false` and proceed -- secondary auth failure does not block the remaining steps. Step 11 flags this as incomplete.

Write: `primary_agent_authenticated`, `secondary_agent_authenticated` (true/false/null), `last_completed_step = 8`.

### Step 9 — OpenClaw and Telegram bot

Explain: OpenClaw is a gateway that connects AI agents to Telegram bots. Today we wire one bot as proof of life.

**Install OpenClaw:** Verify the repo URL at runtime before giving it to the user (it may have moved). Install using the project's current documented method. If install fails, check Node version and show exact error.

**Create Telegram bot:** User goes to @BotFather in Telegram, creates a bot, gets a token. User shares the bot username (not the token). Token goes into OpenClaw config, not into the vault.

**Wire and verify:** Configure OpenClaw with one bot and the authenticated agent runtime as backend. Start gateway. Health probe must return OK (see Constraint 11: three separate auth systems).

Write: `bot_telegram_handle`, `last_completed_step = 9`.

### Step 10 — Wow moment

The user opens Telegram on their phone, finds their bot, and sends a message. The bot responds via the VPS-hosted agent.

If the bot does not respond: structured troubleshoot -- check gateway process, check bot token in config, check network connectivity to Telegram API, check gateway logs. Do not give up after one attempt.

After success, deliver verbatim in Russian:

> Это первый живой кусок твоей персональной системы. Ты пишешь в Telegram — система отвечает через своего агента, даже когда ноутбук закрыт. Это уже не план, это работает.

Write `last_completed_step = 10` only after a confirmed successful bot response. If troubleshooting fails and the user cannot get a response, leave `last_completed_step = 9` -- re-running `/pos-vps` (or `/skill:pos-vps` in Codex) resumes at this step.

### Step 11 — Migrate existing automations

Check `learner-state.json` for completed arch blocks that produce running processes or scheduled tasks (e.g., morning brief, telegram adapter, email processing). If any exist:

Tell the user that now they have a 24/7 server, these automations will work better there — no more depending on the laptop being open. Offer to migrate them to the VPS right now. If the user agrees, do the migration (clone repos, copy configs, set up systemd units or tmux sessions as appropriate). Secrets go into environment variables on the server, never into git-tracked files.

After migration (or if nothing to migrate), tell the user in Russian: future automations should be built on the server — that's the whole point of having it.

If no completed blocks produce running processes, skip the migration offer but still deliver the forward-looking message.

Write: `last_completed_step = 11`.

### Step 12 — Wrap up

**Architecture doc update (mandatory).** Find `my-architecture.md` in `POS_HOME` (resolve from `$POS_HOME` environment variable, falling back to `~/.pos-builder`). Add a VPS section covering: host, user, SSH key path, provider, region, spec, hardening summary, runtimes, OpenClaw + bot handle, setup date. Show proposed content, get confirmation, then write.

**2FA and sessions reminder.** Remind the user to check Telegram active sessions and make sure 2FA is on (nudge, not a gate).

**State cleanup.** If all end-state items are present, set `status = "done"`. If something is missing (e.g., primary agent auth not confirmed, or secondary agent opted in but auth failed), set `status = "incomplete"`.

**Next block.** Recommend next block: read diagnostic route from `learner-state.json`, cross-reference with completed blocks, name the specific block and slash command. Connect the recommendation to what VPS enables (e.g., "now you have a machine that can run briefings while you sleep").

Mention `/pos-feedback`.

Write always: `status`, `last_completed_step = 12`. Write only if done: `completed_at`.

## Rules

1. **Language and tone.** Speak Russian to the user. Use `ты`. Plain, simple language -- `сервер` not `инфраструктура`, `как устроена защита` not `архитектура безопасности`. Warm and calm. All user-visible text must be Russian -- no English leaking in status updates, error messages, or wrap-up.
2. **Silent state.** State reads and writes are invisible to the user. Never show JSON field names, keys, or dot-paths in user-facing text.
3. **Concept before jargon.** Before naming SSH, UFW, fail2ban, systemd, or NodeSource -- explain the concept in plain language first.
4. **Transparency before action.** Before any build step, tell the user in one short Russian sentence what is about to happen and why.
5. **Pre-warn anxiety.** Before scary prompts (SSH lockout risk, scope warnings, unfamiliar terminal), one sentence warning first.
6. **Present -> confirm -> write.** When creating or modifying config files, SSH config, or architecture docs: show proposed content, get confirmation, then write.
7. **Clear choices.** When presenting options, explain each clearly and recommend the best one if there is one.
