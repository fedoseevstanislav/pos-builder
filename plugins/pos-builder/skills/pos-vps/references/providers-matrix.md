# VPS Provider Matrix — POS-builder Reference

> Runtime reference for the pos-vps skill. Agent reads this at Phase 3 (provider selection).
> The agent MAY research current pricing at runtime — this file provides the baseline.
> Last verified: 2026-04-22. Source: POS Workshop 21.03.2026 VTT transcript + Exa research April 2026.

## Critical constraint

Claude Code requires unrestricted access to `api.anthropic.com`. As of April 2026, connections from Russian IP addresses are blocked by Anthropic. Therefore:

- **VPS located in Russia will NOT work** for Claude Code.
- The learner needs a VPS in a non-RU region.
- RU-based learners must use a provider that (a) accepts RU payment methods AND (b) offers non-RU server locations.

## Tier 1 — RU-card friendly, non-RU servers

Best fit for learners paying with Russian bank cards (МИР, RU-issued Visa/MC).

**Primary recommendation: 4VPS.su** — проверен на POS-воркшопе 21.03.2026 (Alex провёл 30+ учеников через регистрацию live). Alex использует 4VPS 8+ лет, 4 production VPS на нём.

| Provider | Non-RU regions | Payment | Min price (2 vCPU / 4 GB / 80 GB) | Signup | Notes |
|----------|---------------|---------|-------------------------------------|--------|-------|
| **4VPS.su** ★ | 36 DCs (EU, US, Asia) | RU cards, crypto | ~900–1400 ₽/mo | Email | **Рекомендация курса.** 8+ лет в production, проверен на воркшопе. |
| **Timeweb Cloud** | DE, NL, KZ | МИР, RU cards, SBP | ~800–1200 ₽/mo | Email, phone | 10-day trial. SLA 99.98%. Хороший UI. |
| **ISHosting** | EU, US, Asia | RU cards, crypto, USDT | ~$8–12/mo | Email | Flexible configs. Easy DC migration. |
| **RoboVPS** | FI, NL, DE, US | RU cards | ~800–1200 ₽/mo | Email | Hi-CPU Finland option. |
| **Beget** | KZ, LV, EU | МИР, RU cards | ~1000–1500 ₽/mo | Email | 30-day trial. Strong support. |
| **IHC.host** | EU (varies) | МИР, RU cards | ~1890 ₽/mo (4 GB) | Email | NVMe. Higher price tier. |

Referral link (для агента, НЕ показывать в scripted flow): `https://4vps.su/r/M88oXU43y9K9`

## Tier 2 — International (non-RU card required)

For learners with foreign bank cards or crypto.

| Provider | Regions | Payment | Min price (2 vCPU / 4 GB / 80 GB) | Signup | Notes |
|----------|---------|---------|-------------------------------------|--------|-------|
| **Hetzner Cloud** | DE (FSN, NBG), FI (HEL), US (ASH, HIL), SG | Non-RU Visa/MC | ~€7–9/mo (CPX21) | Email | Price increased 30-35% April 2026. Blocks RU users. Best $/perf for non-RU. |
| **DigitalOcean** | FRA, AMS, LON, NYC, SFO, SGP, BLR, SYD | Non-RU Visa/MC | ~$12/mo | Email, GitHub | Simple UI. Good docs. Dubai DC for ME region. |
| **Vultr** | 32 locations | Non-RU Visa/MC, crypto | ~$12/mo | Email | Wide geo. Hourly billing. |
| **Linode (Akamai)** | US, EU, APAC | Non-RU Visa/MC | ~$12/mo | Email | $100 free credit (60 days). |

## Tier 3 — RU-only servers (NOT suitable)

| Provider | Why unsuitable |
|----------|---------------|
| Aeza | Moscow/SPb only. No non-RU DCs. |
| Selectel (standard) | Russia-only by default. |

## Decision flow for the agent

```
1. Where is the learner?
   ├─ Russia → Tier 1 providers (non-RU server location!)
   ├─ CIS (KZ, AM, GE) → Tier 1 or Tier 2 depending on card
   ├─ EU → Tier 2 (Hetzner default if non-RU card)
   ├─ UAE/ME → Tier 2 (DigitalOcean BLR or Hetzner)
   └─ Other → Tier 2
2. What card does the learner have?
   ├─ RU-issued (МИР, RU Visa/MC) → Tier 1 only
   ├─ Foreign Visa/MC → Tier 1 or Tier 2
   └─ Crypto available → ISHosting, Vultr, or 4VPS
3. Budget?
   ├─ Minimum (~$5-8/mo) → Timeweb Cloud NL/DE, RoboVPS FI
   ├─ Standard (~$8-15/mo) → Any Tier 1 or Tier 2
   └─ Comfortable (~$15+/mo) → Hetzner CPX31, DO Premium
```

## Recommended defaults

- **RU learner, first VPS ever:** **4VPS.su**, US server, 2 vCPU / 4 GB / 80 GB.
- **Non-RU learner, first VPS:** Hetzner Cloud CPX21, Frankfurt DC.
- **Learner with existing VPS:** verify path — check region, OS, disk, Claude Code reachability.

## Disk size recommendation

Per Stas's live-test feedback (2026-04-17): 40 GB is tight for real daily use. Default recommendation is **80 GB**:

| Tier | Disk | Who | Price delta |
|------|------|-----|-------------|
| Course-only | 40 GB | Pass the course, then decide | ~$4-5/mo |
| **Daily use (default)** | **80 GB** | **Plan to use POS daily** | **~$7-10/mo** |
| Power user | 160 GB | Extraction pipelines, large vault, research dumps | ~$12-15/mo |

---

## Troubleshooting appendix (conditional — agent shows only when relevant)

> These problems are from real learners on the POS Workshop 21.03.2026 (30+ participants, 7 hours, live VPS setup on 4VPS). Agent should NOT front-load these warnings. Show each item ONLY at the phase step where the learner hits the actual problem.

### During provider signup (Phase 5)
- **Ошибка при создании VPS без пополнения баланса.** На 4VPS нужно сначала пополнить баланс, потом создавать сервер. Иначе — ошибка.
- **Пароль сервера** приходит во вкладку "Сообщения" в личном кабинете 4VPS, не на почту. Если ученик не нашёл пароль — направить туда.

### During SSH / first login (Phase 6)
- **РКН может заблокировать IP сервера.** ТСПУ блокирует пачками. Решение: у 4VPS можно сменить IP в панели управления (Настройки сервера → Сменить IP).

### During Claude Code auth (Phase 9)
- **Anthropic блокирует при обнаружении RU-IP.** НЕ подключаться к api.anthropic.com напрямую из России. VPS в US/EU снимает проблему (SSH = туннель). Если ученик случайно подключился без VPN с домашнего IP — риск блокировки аккаунта.

### During OpenClaw / TG bot (Phase 10-11)
- **Telegram-боты из RU не работают из-за блокировок.** Бот на VPS в US/EU работает без проблем — это одна из причин зачем VPS.

### Payment fallback (if RU card doesn't work at chosen provider)
- **PeoplBot** — сервис виртуальных карт. Alex использует ~1 год. Показывать ТОЛЬКО если карта не прошла, не заранее.
- **Антигравити VPN** — рекомендация участника воркшопа для обхода блокировок. Рисковая тема (аккаунты блокируют). Упоминать только если ученик сам спросит про VPN.
