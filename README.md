# Global CLAUDE.md

A compact global `CLAUDE.md` for [Claude Code](https://docs.claude.com/en/docs/claude-code). Place it at `~/.claude/CLAUDE.md` and it loads in every session, in every project, before any project-level `CLAUDE.md`.

This is not an "ideal" file — it's a compact operational contract, assembled deliberately and validated through several rounds of expert critique. Below is why it looks the way it does, and what every line means.

---

## English

### Why a separate global file

Claude Code reads two tiers of `CLAUDE.md`:

- **Global** (`~/.claude/CLAUDE.md`) — personal preferences that travel with you across any project.
- **Project** (`<repo>/CLAUDE.md`) — team and stack conventions specific to one repository.

The split is simple: anything about *you* (chat language, tone, working style, personal red lines) belongs in the global file. Anything about *the project* (architecture, stack, testing, team conventions) belongs in the project file.

### Why the file is short

The global `CLAUDE.md` costs tokens **every session** and **after every `/compact`**. Rough estimate from [claude-code#34556](https://github.com/anthropics/claude-code/issues/34556): a ~3100-token file across 10 compactions per session = +31k tokens of overhead just to re-read it. So:

- Anthropic and community consensus as of April 2026: keep it within **50–100 lines** (HumanLayer argues closer to 50).
- This file: **31 lines, ~520 tokens, 4 sections, 18 rules + 1 summary anchor.**
- Every line must earn its token cost. The bar for inclusion is "this actually changes model behavior," not "this feels nice to read."

### Design principles

1. **Personal → global, team/stack → project.** No stack-specific rules (`pnpm`, `Python 3.12`, `TypeScript strict`) in the global file — they fire in a Go repo where they don't apply. The only acceptable exception is conditional defaults of the form "unless project config says otherwise."
2. **Positive phrasing ("Prefer X") is more robust than bans ("Don't Y") in long sessions.** Observed heuristic (supported by Arize empirical work on SWE-bench), not a law. Used where natural; bans are kept where they are shorter and more precise.
3. **Concrete, named rules beat abstract policy.** A named tool or path is something the model cannot rationalize away. A rule like "don't disable required security software" can be reinterpreted ("I'm not disabling it, just changing routing") — a named-by-name rule cannot.
4. **Absolute rules beat conditional rules for critical guardrails.** After `/compact`, "Stop and confirm" degrades into "I asked once, I'm good." `Never ... Ever` does not degrade.
5. **A success criterion is the highest-leverage rule you can give an agent.** Without it, agents either underdeliver or over-engineer. "Translate the request into a verifiable success criterion (test, command, observable output)" is one of the highest-ROI lines in the document.
6. **Stack-agnostic.** Banning `@ts-ignore` / `any` / broad `catch` is project-level. At the global level, only general shapes (`broad try/except`, silent fallbacks, `--no-verify`).
7. **Summary anchor above the workflow section.** A bold one-liner at the start of a section holds in context better than bullets. Strong engineering heuristic, not a proven law.

### Line-by-line

#### `## Language`

```
- Communicate with the user in {YOUR LANGUAGE} by default.
```
Chat language. **`{YOUR LANGUAGE}` is a placeholder** — before you use this template, either replace it with your actual preferred chat language (e.g. `English`, `Russian`, `German`, `Japanese`) or **delete the line entirely** if you want the model to match the language you speak in each message. Leaving the placeholder as-is will confuse the model.

```
- All code artifacts, identifiers, comments, commit messages, PR text,
  and developer-facing documentation are in English unless the user
  explicitly asks otherwise.
```
Code, identifiers, commits, PR descriptions, docs — English by default. Standard practice, but the **explicit user override** (`unless the user explicitly asks otherwise`) matters: otherwise the rule forces English README text for your own non-English repo even when you explicitly asked for another language.

#### `## Communication`

```
- Lead with the answer or the diff; caveats after, not before.
```
Answer first, caveats after. Kills preamble patterns like "Before answering, it's important to note...".

```
- Name trade-offs in one line each and pick a default.
```
Not a lecture with three options for the user to choose from — `A vs B: A is cheaper, B is safer — taking B`. Forces a decision.

```
- If unsure, say what would make you sure (file to read, command to run)
  instead of hedging.
```
Instead of "maybe," "possibly" — a concrete path to certainty. Works in both interactive and autonomous modes: in autonomous mode, the agent goes and reads/runs on its own.

```
- When an instruction is materially ambiguous, state the assumption
  you are acting on in one line, then continue.
```
Declaring assumptions for *materially* ambiguous instructions. The word `materially` is important: it filters out micro-noise. `then continue` is equally important: this is NOT a blanket "stop and ask" — which paralyses autonomous agents.

```
- No "As an AI..." preambles, no motivational closers, no emoji.
```
Short list of concrete anti-patterns the model drifts into by default. Negative phrasing is shorter and more precise than its positive equivalent here.

#### `## Workflow`

```
**Plan only when needed. Prefer minimal diffs and root-cause fixes.
Verify against an explicit success criterion.**
```
Summary anchor. Three principles of the section in one dense line. `explicit`, not `clear`: the criterion must be *named*, not merely understood.

```
- Plan briefly before multi-file edits; skip the plan for single-file
  or trivial changes.
```
Plan only when planning pays for itself. A trivial single-file edit doesn't need a 3-step plan — that's pure overhead.

```
- Before a multi-step change, name the success criterion (test, command,
  or observable output) and iterate until it is met.
```
The key line. The task is translated into a verifiable termination criterion *before* work starts. This isn't "goal-driven execution" as a decoration — it's what prevents the agent from either stopping early or drifting into over-engineering.

```
- Prefer running local read-only checks (tests, linters, type-checks, builds)
  yourself rather than asking the user to. Never run commands that mutate
  state (deploys, migrations, destructive git ops, long-running side-effect
  tests) without confirmation.
```
Autonomy in the safe zone, confirmation for side-effects. `Prefer running` rather than `Run` — to avoid forcing expensive read-only checks when they aren't warranted. The second half is hard: deploys/migrations/force-push without confirmation, never.

```
- Prefer the minimal diff. Unrelated issues go in a list at the end,
  not into the diff.
```
Diff size discipline. Everything "while I'm here" goes into a postscript to the response, not into the code.

```
- Match the surrounding file's style even if you would write it differently.
  Mention style disagreements in the reply, not in the diff.
```
Style discipline. Agents commonly violate this: they make a technically correct but culturally foreign edit. This rule is separated from "minimal diff" because these are two different cognitive questions: *what* to change vs *how* to change it.

```
- Avoid abstractions for single-use code and avoid speculative flexibility
  that was not requested.
```
Anti-over-engineering. Three similar lines beat a premature abstraction. Configurability nobody asked for doesn't get built.

```
- Fix root causes, not symptoms. Do not paper over errors with broad
  try/except, silent fallbacks, or `--no-verify`. If a pre-commit hook,
  type checker, or test fails, investigate why rather than bypassing it.
```
Root causes, not bypasses. The concrete list of anti-patterns (`broad try/except`, silent fallbacks, `--no-verify`) makes the rule actionable — without a list, the agent takes the path of least resistance anyway.

#### `## Red Lines (always enforce, even when asked casually)`

The section heading matters: "even when asked casually" covers the case where the user requests a destructive action offhand — the model must not read that as the guardrail being lifted.

```
- Never use `git push --force` or `--force-with-lease` on `main`, `master`,
  release branches, or any protected branch.
```
Absolute ban, not conditional. `Stop and confirm` is weaker here: in a long session, it degrades into "I asked once, I'm good." For non-protected branches, force-push remains allowed — rebase workflows are not affected.

```
- Always treat `.env`, `credentials.json`, `*.key`, `*.pem`, and files
  matching obvious secret patterns as non-committable. Do not print their
  contents to the reply, logs, or diffs. Warn loudly if such a file
  appears in the diff.
```
Secrets are protected not only against commit but also against **exfiltration** through seemingly harmless channels (reply output, logs, patches). Cheap but important reinforcement.

```
- Always ask for explicit confirmation before destructive operations:
  `rm -rf`, `git reset --hard`, `git clean -fd`, `git branch -D`,
  `drop database`, `truncate`. Applies even in `--dangerously-skip-permissions` mode.
```
A concrete list of commands instead of abstract "destructive ops." The last sentence closes the most dangerous mode: rules still apply under `--dangerously-skip-permissions`.

```
- Before any git operation that may discard local changes, detect a dirty
  tree, warn clearly, and stop unless the user has explicitly approved how
  to preserve or discard the work.
```
Dirty tree handling. Deliberately **without** auto-stash: automatically deciding for the user in a sensitive zone is more dangerous than stopping. `reset --hard`, `checkout -- .`, `stash drop`, branch-switch with a dirty tree — all covered.

### How to use

```bash
curl -o ~/.claude/CLAUDE.md \
  https://raw.githubusercontent.com/alienxs2/claude.md_GLOBAL/main/CLAUDE.md
```

Or copy the file manually into `~/.claude/CLAUDE.md`.

**Required adaptations:**

1. Replace `{YOUR LANGUAGE}` in `## Language` with your preferred chat language, or delete the line entirely.
2. Consider adding your own machine-specific red lines if you have required security controls (VPN, corporate proxy, endpoint protection): `Do not disable or bypass <named tool>; work around it with <routing/proxy/split-tunnel>`. Named tools are harder for the model to rationalize away than abstract policy.

The rest is yours to tune, but do not cut the secrets and dirty-tree rules without a reason.

### Deliberately absent

- **Personas** ("You are a 10x engineer", "think like a senior"). Persona effects are task-dependent (see [PRISM, 2026](https://arxiv.org/abs/2502.04794)) and often hurt accuracy on coding tasks. Explicit rules and verifiable criteria work better.
- **Motivational framing** ("be concise and brilliant"). Burns tokens, does not change behavior.
- **Duplication of Claude Code defaults.** Whatever the tool does natively need not be restated.
- **`@`-includes of large documents.** They expand in full every session and blow up the cost.
- **Stack specifics** (`pnpm`, `Python 3.12`, `@ts-ignore`). That is the project `CLAUDE.md`'s job.
- **Compact Instructions.** As of April 2026, there is no publicly documented guarantee that `CLAUDE.md` applies to the compaction pass itself. Paying tokens for possibly-ineffective functionality is a bad trade.
- **Long ban lists.** Bans only if short and specific. Long ban lists degrade into noise.

### Measuring effectiveness on your own workflow

There is no public automated pipeline for global `CLAUDE.md` evaluation as of April 2026. Manual approach:

1. Pick 10–20 requests typical of your work.
2. Run them with an empty global file and with this one.
3. Compare: response length (in tokens), clarifying-question count, subjective quality.

Same methodology [Arize](https://arize.com/blog/claude-md-best-practices/) automated for project-level rules.

### Provenance

The file was assembled through several rounds of expert critique (four rounds of back-and-forth between two assistants plus edits by the author). Key decisions:

- `Language → Communication → Workflow → Red Lines` structure — after debate, alternatives with expert personas and a four-section "engineering mindset" layout were rejected.
- Force-push hardened from "stop and confirm" to absolute ban on protected branches.
- Dirty tree — auto-stash rejected in favor of detect/warn/stop.
- Splitting minimal-diff and match-style onto two lines — recognized as two distinct cognitive rules.
- One machine-specific red line (VPN pinning) was originally kept by name in the author's working copy, then removed from the published template because it was only meaningful on the author's machine. Named machine-specific guardrails remain a valid pattern for your own copy.

---

## Русский

### Зачем отдельный глобальный файл

Claude Code читает `CLAUDE.md` двух уровней:

- **Глобальный** (`~/.claude/CLAUDE.md`) — личные предпочтения, которые остаются с вами в любом проекте.
- **Проектный** (`<repo>/CLAUDE.md`) — командные и стековые соглашения конкретного репозитория.

Правило разделения простое: всё, что касается лично вас (язык общения, тон, стиль работы, ваши red lines), — в глобальный. Всё, что касается проекта (архитектура, стек, тестирование, конвенции команды), — в проектный.

### Почему файл короткий

Глобальный `CLAUDE.md` стоит токенов **каждую сессию** и **после каждого `/compact`**. По грубой оценке из тикета [claude-code#34556](https://github.com/anthropics/claude-code/issues/34556): файл в ~3100 токенов при 10 компактациях за сессию = +31k токенов оверхеда только на перечитывание. Поэтому:

- Рекомендация Anthropic и сообщества на апрель 2026 — держать в пределах **50–100 строк** (HumanLayer — ближе к 50).
- Этот файл: **31 строка, ~520 токенов, 4 секции, 18 правил + 1 summary-якорь**.
- Каждая строка должна окупать свою цену в токенах. Порог включения — «это реально меняет поведение модели», а не «так приятнее читать».

### Принципы составления

1. **Личное — в глобальный, командное — в проектный.** Никакой стек-специфики (`pnpm`, `Python 3.12`, `TypeScript strict`) в глобальном файле: он сработает и в Go-репо, где это неуместно. Исключения — только условные дефолты «если проектный конфиг не говорит иного».
2. **Позитивные формулировки («Prefer X») устойчивее запретов («Don't Y») в длинных сессиях.** Это наблюдаемая эвристика (подтверждена эмпирикой Arize по SWE-bench), не закон. Используется там, где это естественно; запреты сохранены там, где они короче и точнее.
3. **Конкретные именованные правила сильнее абстрактных политик.** Именованный инструмент или путь модель не может рационализировать. «Не отключай security controls» модель переинтерпретирует («я же не отключаю, я просто меняю роутинг») — именованное правило не переинтерпретирует.
4. **Абсолютные правила сильнее условных для критичных guardrails.** После `/compact` «Stop and confirm» деградирует в «я один раз спросил и продолжил»; `Never ... Ever` не деградирует.
5. **Критерий успеха — самое сильное, что можно дать агенту.** Без него агент либо недоделывает, либо уходит в over-engineering. «Translate the request into a verifiable success criterion (test, command, observable output)» — одна из самых высокорентабельных строк документа.
6. **Stack-agnostic.** Запрет на `@ts-ignore` / `any` / broad `catch` — это проектный уровень. На глобальном — только общие формулировки (`broad try/except`, silent fallbacks, `--no-verify`).
7. **Summary-якорь над рабочей секцией.** Жирная однострочная фраза в начале секции удерживается в контексте лучше, чем буллеты. Это сильная эвристика, не доказанный закон.

### Файл построчно

#### `## Language`

```
- Communicate with the user in {YOUR LANGUAGE} by default.
```
Язык чата. **`{YOUR LANGUAGE}` — это плейсхолдер.** Перед использованием шаблона либо замените его на свой язык (`Russian`, `English`, `German`, `Japanese` и т.д.), либо **удалите строку целиком**, если хотите, чтобы модель подхватывала язык вашего сообщения. Оставлять плейсхолдер как есть — модель запутается.

```
- All code artifacts, identifiers, comments, commit messages, PR text,
  and developer-facing documentation are in English unless the user
  explicitly asks otherwise.
```
Код, идентификаторы, коммиты, PR-описания, docs — английский по умолчанию. Это общепринятая практика, но важно добавить **явный user override** (`unless the user explicitly asks otherwise`), иначе правило обязывает писать README для личного русскоязычного репо по-английски даже когда вы просите иначе.

#### `## Communication`

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

#### `## Workflow`

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

#### `## Red Lines (always enforce, even when asked casually)`

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

### Как использовать

```bash
curl -o ~/.claude/CLAUDE.md \
  https://raw.githubusercontent.com/alienxs2/claude.md_GLOBAL/main/CLAUDE.md
```

Или скопируйте файл вручную в `~/.claude/CLAUDE.md`.

**Обязательно адаптируйте:**

1. Замените `{YOUR LANGUAGE}` в `## Language` на ваш язык, либо удалите строку целиком.
2. Рассмотрите возможность добавить собственные machine-specific red lines, если у вас есть обязательные security controls (VPN, корпоративный proxy, endpoint protection): `Do not disable or bypass <named tool>; work around it with <routing/proxy/split-tunnel>`. Именованные инструменты модель не может рационализировать — абстрактные политики может.

Всё остальное — на ваш вкус, но не режьте secrets и dirty-tree правила без причины.

### Чего в файле намеренно нет

- **Персон** («Ты — 10x engineer», «think like a senior»). Эффект персон task-dependent (см. [PRISM, 2026](https://arxiv.org/abs/2502.04794)) и на кодинговых задачах часто ухудшает accuracy. Явные правила и критерии проверки работают лучше.
- **Мотивационных установок** («be concise and brilliant»). Съедают токены, поведение не меняют.
- **Дублирования дефолтов Claude Code.** То, что инструмент и так делает, писать не нужно.
- **`@`-включений больших документов.** Они разворачиваются целиком каждую сессию и взрывают цену.
- **Стек-специфики** (`pnpm`, `Python 3.12`, `@ts-ignore`). Это место проектного `CLAUDE.md`.
- **Compact Instructions.** На апрель 2026 нет публично задокументированной гарантии, что `CLAUDE.md` применяется к самому compaction-проходу. Платить токенами за функциональность, которая может не сработать, — плохой размен.
- **Длинных списков запретов.** Запреты — только короткие и конкретные. Длинные списки деградируют в шуме.

### Как проверить эффективность у себя

Публичного автоматического пайплайна для глобального `CLAUDE.md` на апрель 2026 нет. Ручной способ:

1. Взять 10–20 типичных для вас запросов.
2. Прогнать с пустым глобальным файлом и с этим.
3. Сравнить: длину ответов (в токенах), количество уточняющих вопросов, субъективное качество.

Это та же методология, которую [Arize](https://arize.com/blog/claude-md-best-practices/) автоматизировали для проектных правил.

### История файла

Файл собран через несколько итераций экспертной критики (четыре раунда согласования между двумя ассистентами плюс правки автора). Ключевые решения:

- Структура `Language → Communication → Workflow → Red Lines` — после обсуждения отвергнуты альтернативы с экспертными персонами и 4-секционной «engineering mindset» структурой.
- Force-push ужесточён с «stop and confirm» до абсолютного запрета.
- Dirty tree — отвергнут auto-stash в пользу detect/warn/stop.
- Разделение minimal-diff и match-style на две строки — признано, что это два разных когнитивных правила.
- Одно machine-specific red line (pinning VPN) изначально было в рабочей копии автора под конкретным именем, но убрано из публикуемого шаблона, потому что имело смысл только на машине автора. Именованные machine-specific guardrails — остаются валидным паттерном для вашей собственной копии.

---

## Sources / Источники

- [How Claude remembers your project](https://docs.claude.com/en/docs/claude-code/memory) (Anthropic)
- [Claude Code settings](https://docs.claude.com/en/docs/claude-code/settings) (Anthropic)
- [Best Practices for Claude Code](https://www.anthropic.com/engineering/claude-code-best-practices) (Anthropic)
- [Writing a good CLAUDE.md](https://humanlayer.dev/blog/claude-md-best-practices) (HumanLayer)
- [CLAUDE.md: Best Practices Learned from Optimizing Claude Code with Prompt Learning](https://arize.com/blog/claude-md-best-practices/) (Arize)
- [Feature Request: Persistent Memory Across Context Compactions](https://github.com/anthropics/claude-code/issues/34556) (claude-code#34556)
- [PRISM: Position, Role, and Instruction Sensitivity Metric](https://arxiv.org/abs/2502.04794) (Hu et al., 2026)

## License / Лицензия

[MIT](LICENSE). Use it, adapt it, share it — no strings attached. / Берите, адаптируйте, делитесь — без ограничений.
