# POS Builder

POS Builder — это практический курс внутри Claude Code и Codex.

Он нужен не для того, чтобы «изучать ИИ вообще», а для того, чтобы шаг за шагом собрать свою рабочую систему: данные, автоматизации и агентов.

## Что здесь происходит

Вместо длинной теории ты идёшь по одному блоку за раз:

1. короткая вводная,
2. диагностика,
3. рекомендованный следующий шаг,
4. дальше — сборка нужных тебе частей системы.

Идея простая: сначала ближайшее полезное действие, потом понимание.

## С чего начать

Начинай с двух скиллов:

- `POS Intro` — чтобы понять, как устроен курс и чего от него ожидать
- `POS Diagnostic` — чтобы пройти короткое интервью и получить маршрут под себя

Если ты устанавливаешь POS Builder впервые, не пытайся сразу понять все скиллы. Достаточно зайти во вводную и потом в диагностику.

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

- `POS Intro` — коротко объясняет, что это за курс и как по нему идти.
- `POS Diagnostic` — собирает твой контекст и предлагает, с чего лучше начать именно тебе.
- `POS STT Setup` — помогает настроить голосовой ввод, если хочешь работать голосом.

### Фундамент

- `POS VPS` — поднимает серверную базу для always-on автоматизаций.
- `POS Vault` — настраивает Obsidian vault как общий дом для знаний и артефактов.
- `POS GitHub Setup` — заводит GitHub-контур для памяти курса, issue-based работы и следующих билдов.

### Подключение каналов и систем

- `POS Calendar` — подключает календарь.
- `POS Email` — подключает почту с осторожной моделью прав.
- `POS Telegram` — подключает Telegram.
- `POS Tasks` — помогает выбрать и подключить одну систему задач.

### Навыки и рабочие ритуалы

- `POS Basic Vibecoding` — первый guided блок по вайбкодингу.
- `POS Goals` — фиксирует жизненные цели и опоры.
- `POS Morning Brief` — собирает утренний бриф из подключённых источников.
- `POS Day Summary` — помогает закрывать день и сверять план с реальностью.
- `POS Triage` — собирает короткий приоритетный обзор «что сейчас важнее».

### Дополнительные блоки

- `POS Dashboard` — собирает визуальный экран с ключевыми штуками твоей системы.
- `POS Advisors` — помогает выделить полезных «советников» из твоих материалов и опыта.
- `POS Observability` — добавляет наблюдаемость для scheduled-потоков, чтобы они не ломались тихо.

## Для кого это

Этот репозиторий — для ученика, который хочет собирать personal AI operating system постепенно, а не проектировать всё заранее.

Если тебе ближе формат «покажи следующий шаг и помоги сделать его», ты в правильном месте.

## English

POS Builder is the public distribution repo for a Claude Code + Codex plugin bundle that helps a learner assemble a personal AI operating system step by step.

Start with `POS Intro`, then `POS Diagnostic`.

Claude Code:

```text
/plugin marketplace add fedoseevstanislav/pos-builder
/plugin install pos-builder@pos-builder
/reload-plugins
```

Codex CLI:

```bash
codex plugin marketplace add fedoseevstanislav/pos-builder
```

Then open `/plugins`, install `POS Builder`, and start with `POS Intro`.
