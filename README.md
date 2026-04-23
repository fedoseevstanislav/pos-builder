# POS Builder

POS Builder — это практический курс внутри Claude Code и Codex, в котором мы шаг за шагом собираем твою персональную рабочую систему (Personal OS): данные, автоматизации и агентов.

## Что здесь происходит

В каждом блоке мы:

- разбираем, что собираем и зачем;
- даём ключевые понятия и логику решений;
- выбираем следующий шаг и собираем нужную часть твоей Personal OS.

## С чего начать

Начнём с двух скиллов:

- `POS Intro` — чтобы понять, как устроен курс и чего от него ожидать;
- `POS Diagnostic` — чтобы пройти короткую диагностику и получить маршрут под себя.

Если ты впервые используешь агентов в Claude Code или Codex, не пытайся сразу понять всё. Начни с простого: с интро и диагностики. Дальше курс проведёт тебя.

## Установка в Claude Code

1. Открой Claude Code.
2. Добавь marketplace:

```text
/plugin marketplace add fedoseevstanislav/pos-builder
```

3. Установи плагин:

```text
/plugin install pos-builder@pos-builder
```

4. Обнови плагины:

```text
/reload-plugins
```

5. Начни новый тред и открой `POS Intro` из плагина `POS Builder`.

Примечание: в Claude плагины могут показывать свои навыки как namespaced-команды внутри плагина. Если не видишь прямую команду, просто открой список плагина и выбери `POS Intro`.

## Установка в Codex CLI

1. Добавь marketplace:

```bash
codex plugin marketplace add fedoseevstanislav/pos-builder
```

2. Запусти Codex и открой список плагинов:

```text
/plugins
```

3. Найди `POS Builder` и установи его.
4. Начни новый тред и открой `POS Intro` из установленного плагина.

## Какие скиллы сейчас входят

### Старт

- `POS Intro` — коротко объясняем, что это за курс и как по нему идти.
- `POS Diagnostic` — собираем твой контекст и определяем, с чего лучше начать именно тебе.
- `POS STT Setup` — с агентами удобно работать голосом. Настраиваем преобразование голоса в текст.

### Фундамент

- `POS VPS` — поднимаем сервер, чтобы агенты могли работать в режиме 24/7.
- `POS Vault` — настраиваем Obsidian Vault как единое хранилище для всей информации твоей Personal OS.
- `POS GitHub Setup` — создаём GitHub-аккаунт, настраиваем хранение кода и автоматизаций и рабочую память агентов через GitHub Issues (задачи).

### Подключение каналов и систем

- `POS Calendar` — даём агенту доступ к календарю.
- `POS Email` — даём агенту доступ к почте.
- `POS Telegram` — даём агенту доступ к Telegram.
- `POS Tasks` — даём агенту доступ к системе учёта задач.

### Навыки и рабочие ритуалы

- `POS Basic Vibecoding` — обучаем базовым принципам и практикам вайбкодинга и сразу применяем их, чтобы создать свой первый скилл.
- `POS Goals` — выявляем и сохраняем цели в разных областях жизни, чтобы агенты понимали, к чему тебе важно двигаться.
- `POS Morning Brief` — настраиваем автоматический утренний бриф: план на день на основе подключённых данных.
- `POS Day Summary` — настраиваем автоматическую процедуру закрытия дня: короткую ретроспективу по итогам дня.
- `POS Triage` — создаём скилл, который помогает беречь внимание и быстро понимать, что сейчас важнее всего.

### Дополнительные блоки

- `POS Dashboard` — собираем личную панель управления нашим вниманием.
- `POS Advisors` — создаём персональный совет экспертов, которые помогают нам принимать решения.
- `POS Observability` — настраиваем контроль за автоматизациями, чтобы вовремя замечать сбои и проблемы.

### Обратная связь

- `POS Feedback` — помогаем оформить отзыв, проблему или идею в аккуратный GitHub issue для создателей курса.

## Для кого это

Этот курс — для тех, кто раньше не работал с кодинговыми агентами, но хочет быстро научиться делать для себя полезные автоматизации, понять, как работать с агентами в жизни, и постепенно разобраться, как всё это устроено.

Мы идём через практику: сначала коротко объясняем идею, а потом сразу применяем её в своей Personal OS.

## English

POS Builder is a hands-on course inside Claude Code and Codex where we build your personal operating system (Personal OS) step by step: data, automations, and agents.

## What Happens Here

In each block we:

- explain what we are building and why;
- give the key concepts and decision logic;
- choose the next step and build the next part of your Personal OS.

## Where to Start

Start with two skills:

- `POS Intro` — to understand how the course works and what to expect from it;
- `POS Diagnostic` — to go through a short diagnostic and get a path that fits you.

If this is your first time using agents in Claude Code or Codex, do not try to understand everything at once. Start simple: intro first, then diagnostic. The course will guide you from there.

## Install in Claude Code

```text
/plugin marketplace add fedoseevstanislav/pos-builder
/plugin install pos-builder@pos-builder
/reload-plugins
```

Then start a new thread and open `POS Intro` from the `POS Builder` plugin.

Note: in Claude, plugin skills may appear as namespaced commands inside the plugin. If you do not see a direct command, open the plugin list and choose `POS Intro`.

## Install in Codex CLI

```bash
codex plugin marketplace add fedoseevstanislav/pos-builder
```

Then open `/plugins`, install `POS Builder`, start a new thread, and open `POS Intro`.

## Current Skills

### Getting Started

- `POS Intro` — we briefly explain what this course is and how to move through it.
- `POS Diagnostic` — we gather your context and decide what makes the most sense for you to do first.
- `POS STT Setup` — working with agents by voice is often easier. We set up speech-to-text.

### Foundation

- `POS VPS` — we set up a server so agents can work 24/7.
- `POS Vault` — we set up Obsidian Vault as the single place for all information in your Personal OS.
- `POS GitHub Setup` — we create a GitHub account, set up code and automation storage, and give agents working memory through GitHub Issues.

### Connecting Channels and Systems

- `POS Calendar` — we give the agent access to your calendar.
- `POS Email` — we give the agent access to your email.
- `POS Telegram` — we give the agent access to Telegram.
- `POS Tasks` — we give the agent access to your task system.

### Skills and Routines

- `POS Basic Vibecoding` — we teach the basic principles and practices of vibecoding and apply them right away to create your first skill.
- `POS Goals` — we identify and save goals across life areas so agents understand what matters to you.
- `POS Morning Brief` — we set up an automatic morning brief: a plan for the day based on connected data.
- `POS Day Summary` — we set up an automatic end-of-day routine: a short daily retrospective.
- `POS Triage` — we create a skill that helps protect your attention and quickly understand what matters most right now.

### Additional Blocks

- `POS Dashboard` — we build a personal dashboard for managing our attention.
- `POS Advisors` — we create a personal council of experts to help us make decisions.
- `POS Observability` — we set up control over automations so we notice failures and problems in time.

### Feedback

- `POS Feedback` — we help you turn feedback, a problem, or an idea into a clean GitHub issue for the course creators.

## Who This Is For

This course is for people who have not worked with coding agents before, but want to quickly learn how to build useful automations for themselves, learn how to work with agents in real life, and gradually understand how all of this works.

We learn through practice: first we explain the idea briefly, then we apply it right away in your own Personal OS.
