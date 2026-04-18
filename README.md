# Global CLAUDE.md

Глобальный `CLAUDE.md` для [Claude Code](https://docs.claude.com/en/docs/claude-code). Файл кладётся в `~/.claude/CLAUDE.md` и подгружается в каждой сессии, в каждом проекте, до любого проектного `CLAUDE.md`.

Это не «идеальный» файл — это компактный operational contract, собранный осознанно и согласованный через несколько итераций экспертной критики. Ниже — почему он именно такой, и что означает каждая строка.

---

## Зачем отдельный глобальный файл

Claude Code читает `CLAUDE.md` двух уровней:

- **Глобальный** (`~/.claude/CLAUDE.md`) — личные предпочтения, которые остаются с вами в любом проекте.
- **Проектный** (`<repo>/CLAUDE.md`) — командные и стековые соглашения конкретного репозитория.

Правило разделения простое: всё, что касается лично вас (язык общения, тон, стиль работы, ваши red lines), — в глобальный. Всё, что касается проекта (архитектура, стек, тестирование, конвенции команды), — в проектный.

## Почему файл короткий

Глобальный `CLAUDE.md` стоит токенов **каждую сессию** и **после каждого `/compact`**. По грубой оценке из тикета [claude-code#34556](https://github.com/anthropics/claude-code/issues/34556): файл в ~3100 токенов при 10 компактациях за сессию = +31k токенов оверхеда только на перечитывание. Поэтому:

- Рекомендация Anthropic и сообщества на апрель 2026 — держать в пределах **50–100 строк** (HumanLayer — ближе к 50).
- Этот файл: **32 строки, ~560 токенов, 4 секции, 19 правил + 1 summary-якорь**.
- Каждая строка должна окупать свою цену в токенах. Порог включения — «это реально меняет поведение модели», а не «так приятнее читать».

## Принципы составления

1. **Личное — в глобальный, командное — в проектный.** Никакой стек-специфики (`pnpm`, `Python 3.12`, `TypeScript strict`) в глобальном файле: он сработает и в Go-репо, где это неуместно. Исключения — только условные дефолты «если проектный конфиг не говорит иного».
2. **Позитивные формулировки («Prefer X») устойчивее запретов («Don't Y») в длинных сессиях.** Это наблюдаемая эвристика (подтверждена эмпирикой Arize по SWE-bench), не закон. Используется там, где это естественно; запреты сохранены там, где они короче и точнее.
3. **Конкретные именованные правила сильнее абстрактных политик.** «Не трогай Hiddify VPN» модель не может рационализировать; «не отключай security controls» — может («это же не совсем disabling, я просто меняю роутинг»).
4. **Абсолютные правила сильнее условных для критичных guardrails.** После `/compact` «Stop and confirm» деградирует в «я один раз спросил и продолжил»; `Never ... Ever` не деградирует.
5. **Критерий успеха — самое сильное, что можно дать агенту.** Без него агент либо недоделывает, либо уходит в over-engineering. «Translate the request into a verifiable success criterion (test, command, observable output)» — одна из самых высокорентабельных строк документа.
6. **Stack-agnostic.** Запрет на `@ts-ignore` / `any` / broad `catch` — это проектный уровень. На глобальном — только общие формулировки (`broad try/except, silent fallbacks, --no-verify`).
7. **Summary-якорь над рабочей секцией.** Жирная однострочная фраза в начале секции удерживается в контексте лучше, чем буллеты. Это сильная эвристика, не доказанный закон.

## Файл построчно

### `## Language`

```
- Communicate with the user in Russian by default.
```
Язык чата. У вас — свой.

```
- All code artifacts, identifiers, comments, commit messages, PR text,
  and developer-facing documentation are in English unless the user
  explicitly asks otherwise.
```
Код, идентификаторы, коммиты, PR-описания, docs — английский по умолчанию. Это общепринятая практика, но важно добавить **явный user override** (`unless the user explicitly asks otherwise`), иначе правило обязывает писать README для личного русскоязычного репо по-английски даже когда вы просите иначе.

### `## Communication`

```
- Lead with the answer or the diff; caveats after, not before.
```
Сначала ответ, потом оговорки. Убивает преамбулы и хедж-паттерны вида «Прежде чем отвечать, важно отметить...».

```
- Name trade-offs in one line each and pick a default.
```
Не лекция с тремя вариантами на выбор пользователю, а «A vs B: A дешевле, B надёжнее — беру B». Форсирует принятие решения.

```
- If unsure, say what would make you sure (file to read, command to run)
  instead of hedging.
```
Вместо «возможно», «вероятно» — конкретный путь к уверенности. Работает и в интерактивных, и в автономных сценариях: в автоматическом режиме модель идёт и читает/запускает сама.

```
- When an instruction is materially ambiguous, state the assumption
  you are acting on in one line, then continue.
```
Декларирование допущений при **материально** неоднозначных инструкциях. Слово `materially` важно: отсекает шум на микро-мелочах. `then continue` — столь же важно: это НЕ blanket «stop and ask», которое парализует автономного агента.

```
- No "As an AI..." preambles, no motivational closers, no emoji.
```
Короткий список конкретных антипаттернов, которые модель любит по инерции. Запретная форма здесь короче и точнее позитивного эквивалента.

### `## Workflow`

```
**Plan only when needed. Prefer minimal diffs and root-cause fixes.
Verify against an explicit success criterion.**
```
Summary-якорь. Три принципа секции одной плотной строкой. `explicit`, а не `clear`: критерий должен быть **названным**, а не просто понятным.

```
- Plan briefly before multi-file edits; skip the plan for single-file
  or trivial changes.
```
План только когда он окупается. Тривиальная однофайловая правка не требует трёх-пунктового плана — это лишний overhead.

```
- Before a multi-step change, name the success criterion (test, command,
  or observable output) and iterate until it is met.
```
Ключевая строка. Задача переводится в проверяемый критерий завершения до того, как начата работа. Это не «goal-driven execution» как декорация — это то, без чего агент не знает, когда остановиться.

```
- Prefer running local read-only checks (tests, linters, type-checks, builds)
  yourself rather than asking the user to. Never run commands that mutate
  state (deploys, migrations, destructive git ops, long-running side-effect
  tests) without confirmation.
```
Автономия в безопасной зоне, подтверждение на side-effects. `Prefer running`, а не `Run`, — чтобы не заставлять агента прогонять дорогие проверки там, где это неуместно. Вторая половина — жёстко: deploys/migrations/force-push без подтверждения никогда.

```
- Prefer the minimal diff. Unrelated issues go in a list at the end,
  not into the diff.
```
Дисциплина объёма диффа. Всё, что «раз уж я тут», — в постscriptum к ответу, не в код.

```
- Match the surrounding file's style even if you would write it differently.
  Mention style disagreements in the reply, not in the diff.
```
Дисциплина стиля. Агенты массово нарушают это: делают технически правильную правку, но культурно инородную. Правило разнесено от «minimal diff» потому, что это два разных когнитивных вопроса: «что менять» и «как менять».

```
- Avoid abstractions for single-use code and avoid speculative flexibility
  that was not requested.
```
Защита от over-engineering. Три похожие строки лучше преждевременной абстракции. Конфигурируемость, которую никто не просил, не строим.

```
- Fix root causes, not symptoms. Do not paper over errors with broad
  try/except, silent fallbacks, or `--no-verify`. If a pre-commit hook,
  type checker, or test fails, investigate why rather than bypassing it.
```
Корневые причины, а не обход. Конкретный список анти-паттернов (`broad try/except`, silent fallbacks, `--no-verify`) делает правило исполнимым: без списка агент всё равно пойдёт по пути наименьшего сопротивления.

### `## Red Lines (always enforce, even when asked casually)`

Заголовок секции важен: «even when asked casually» закрывает случаи, когда пользователь просит destructive-действие мимоходом — модель не должна воспринимать это как снятие guardrail.

```
- Never use `git push --force` or `--force-with-lease` on `main`, `master`,
  release branches, or any protected branch.
```
Абсолютный запрет, не условный. `Stop and confirm` здесь слабее: в длинной сессии деградирует до «я один раз спросил». Для non-protected веток force-push остаётся разрешённым — rebase-флоу не страдает.

```
- Always treat `.env`, `credentials.json`, `*.key`, `*.pem`, and files
  matching obvious secret patterns as non-committable. Do not print their
  contents to the reply, logs, or diffs. Warn loudly if such a file
  appears in the diff.
```
Секреты защищены не только от коммита, но и от **эксфильтрации** через безобидные каналы (вывод в чат, логи, патчи). Это дешёвое, но важное усиление.

```
- Always ask for explicit confirmation before destructive operations:
  `rm -rf`, `git reset --hard`, `git clean -fd`, `git branch -D`,
  `drop database`, `truncate`. Applies even in `--dangerously-skip-permissions` mode.
```
Конкретный список команд вместо абстрактного «destructive». Последняя фраза закрывает самый опасный режим: в `--dangerously-skip-permissions` правила всё равно действуют.

```
- Before any git operation that may discard local changes, detect a dirty
  tree, warn clearly, and stop unless the user has explicitly approved how
  to preserve or discard the work.
```
Dirty tree. Намеренно **без** auto-stash: автоматическое принятие решения за пользователя в чувствительной зоне опаснее, чем остановка. `reset --hard`, `checkout -- .`, `stash drop`, branch-switch с грязным деревом — всё сюда.

```
- On this machine, Hiddify VPN is a required security control. Do not
  disable or bypass it. Work around it with routing, proxy settings, or
  split-tunnel configuration instead.
```
**Machine-specific правило**, сохранено намеренно. Конкретное именованное ограничение модель не может рационализировать; абстрактное «do not disable security controls» — может. Если адаптируете файл под себя — **эту строку нужно либо переписать под ваш реальный контроль, либо удалить**.

---

## Как использовать

```bash
curl -o ~/.claude/CLAUDE.md \
  https://raw.githubusercontent.com/alienxs2/claude.md_GLOBAL/main/CLAUDE.md
```

Или скопируйте файл вручную в `~/.claude/CLAUDE.md`.

**Обязательно адаптируйте:**

1. Язык чата в `## Language` (по умолчанию — русский).
2. Строку про Hiddify — под ваш реальный security control или удалите, если такого нет.
3. Всё остальное — на ваш вкус, но не режьте secrets и dirty-tree правила без причины.

## Чего в файле намеренно нет

- **Персон** («Ты — 10x engineer», «think like a senior»). Эффект персон task-dependent (см. [PRISM, 2026](https://arxiv.org/abs/2502.04794)) и на кодинговых задачах часто ухудшает accuracy. Явные правила и критерии проверки работают лучше.
- **Мотивационных установок** («be concise and brilliant»). Съедают токены, поведение не меняют.
- **Дублирования дефолтов Claude Code.** То, что инструмент и так делает, писать не нужно.
- **`@`-включений больших документов.** Они разворачиваются целиком каждую сессию и взрывают цену.
- **Стек-специфики** (`pnpm`, `Python 3.12`, `@ts-ignore`). Это место проектного `CLAUDE.md`.
- **Compact Instructions.** На апрель 2026 нет публично задокументированной гарантии, что `CLAUDE.md` применяется к самому compaction-проходу. Платить токенами за функциональность, которая может не сработать, — плохой размен.
- **Длинных списков запретов.** Запреты — только короткие и конкретные. Длинные списки деградируют в шуме.

## Как проверить эффективность у себя

Публичного автоматического пайплайна для глобального `CLAUDE.md` на апрель 2026 нет. Ручной способ:

1. Взять 10–20 типичных для вас запросов.
2. Прогнать с пустым глобальным файлом и с этим.
3. Сравнить: длину ответов (в токенах), количество уточняющих вопросов, субъективное качество.

Это та же методология, которую [Arize](https://arize.com/blog/claude-md-best-practices/) автоматизировали для проектных правил.

## История файла

Файл собран через несколько итераций экспертной критики (четыре раунда согласования между двумя ассистентами плюс правки автора). Ключевые решения:

- Структура `Language → Communication → Workflow → Red Lines` — после обсуждения отвергнуты альтернативы с экспертными персонами и 4-секционной «engineering mindset» структурой.
- Force-push ужесточён с «stop and confirm» до абсолютного запрета.
- Dirty tree — отвергнут auto-stash в пользу detect/warn/stop.
- Разделение minimal-diff и match-style на две строки — признано, что это два разных когнитивных правила.
- Hiddify оставлен именованно — признано, что конкретные правила сильнее абстрактных.

## Источники

- [How Claude remembers your project](https://docs.claude.com/en/docs/claude-code/memory) (Anthropic)
- [Claude Code settings](https://docs.claude.com/en/docs/claude-code/settings) (Anthropic)
- [Best Practices for Claude Code](https://www.anthropic.com/engineering/claude-code-best-practices) (Anthropic)
- [Writing a good CLAUDE.md](https://humanlayer.dev/blog/claude-md-best-practices) (HumanLayer)
- [CLAUDE.md: Best Practices Learned from Optimizing Claude Code with Prompt Learning](https://arize.com/blog/claude-md-best-practices/) (Arize)
- [Feature Request: Persistent Memory Across Context Compactions](https://github.com/anthropics/claude-code/issues/34556) (claude-code#34556)
- [PRISM: Position, Role, and Instruction Sensitivity Metric](https://arxiv.org/abs/2502.04794) (Hu et al., 2026)

## Лицензия

[MIT](LICENSE). Берите, адаптируйте, делитесь — без ограничений.
