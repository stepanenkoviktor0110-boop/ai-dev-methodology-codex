# Lessons Learned: feature-execution

### 2026-03-25 design-pipeline: Параллельные агенты превышали лимит системы

**Problem:** Wave execution порождал 6-8 агентов одновременно (tasks + reviewers), вызывая OOM и index.lock конфликты.
**Cause:** Лимит параллельных агентов не был определён; feature-execution полагался на max_threads из конфига, который не учитывал реальные ограничения системы.
**Solution:** 4 итерации фиксов: batching → hard limit 5 → снижение до 4 → дедупликация правила в AGENTS.md.
**Rule:** Перед spawn wave проверяй количество агентов. Жёсткий лимит 4 — batch если больше. Закрывай агентов после сбора результатов перед следующим batch.

### 2026-03-25 design-pipeline: Cherry-pick между framework и project репозиториями

**Problem:** Агенты cherry-pick'или коммиты из framework repo (~/.agents/) в project repo, создавая merge-конфликты в несуществующих файлах.
**Cause:** Агенты не различали два repo с разными историями; git подсказывал cherry-pick как способ "перенести изменения".
**Solution:** 3 итерации: disambig docs → NEVER cherry-pick → промоушен до RULE #1 в AGENTS.md.
**Rule:** Framework обновляется ТОЛЬКО через `git pull` в ~/.agents/. НИКОГДА не cherry-pick между repos. Если агент предлагает cherry-pick — это ошибка.

### 2026-03-25 design-pipeline: Model fallback маскировал использование неправильной модели

**Problem:** Агенты молча переключались на более дешёвую модель когда целевая была недоступна, без уведомления.
**Cause:** Fallback-логика давала агентам "отмазку" не проверять какую модель они реально используют.
**Solution:** Убраны все fallbacks. Одна модель на tier. Если недоступна — STOP и сообщить пользователю.
**Rule:** Никаких model fallbacks. Один tier = одна модель. Недоступна → остановиться, не молча заменять.

### 2026-03-26 shift-confirmation: Одна и та же ошибка типов повторяется через волны

**Problem:** В Task 1 ревьюер нашёл `confirmationStatus: string` вместо enum. Исправили. В Task 4 — ровно та же ошибка в report types. Агент Task 4 не знал о находке Task 1.
**Cause:** Каждый teammate запускается с чистым контекстом. decisions.md фиксирует "что сделано", но не "какие ошибки ревьюеры нашли и как их избежать".
**Rule:** Когда ревью находит *паттерн ошибки* (не разовый баг), lead добавляет предупреждение в промт следующего teammate: "В предыдущих задачах ревьюеры находили [X] — убедись, что новый код не повторяет эту ошибку."

### 2026-03-26 shift-confirmation: Агенты систематически пишут implementation-тесты вместо behavioral

**Problem:** В каждой задаче (1, 2, 4) test-reviewer находил тесты, которые проверяют форму вызова mock (`toHaveBeenCalledWith`) вместо результата функции. В Task 1 — compile-time type checks. В Task 2 — flaky setTimeout. В Task 4 — where-clause-only assertions.
**Cause:** Query-shape тесты проще написать чем настроить mock data для поведенческих assertions. Агенты оптимизируют на "тест проходит", не на "тест ловит баг".
**Rule:** В промт teammate добавить: "Каждый тест ОБЯЗАН иметь assertion на результат функции (return value / output shape), а не только на аргументы вызова mock. `toHaveBeenCalledWith` допускается как дополнение, не как единственная проверка."

### 2026-03-26 shift-confirmation: Pre-existing проблемы всплывают в каждом ревью

**Problem:** Security auditor нашёл IDOR в `markEvent()` при ревью Task 2. Та же находка повторилась в audit wave. Корректная работа аудитора, но тратит время на уже известные проблемы.
**Cause:** Нет реестра известных проблем, которые не нужно повторно репортить.
**Rule:** Завести `known-issues.md` на уровне проекта. Перед ревью агент читает его и пропускает задокументированные проблемы. При нахождении pre-existing issue — добавить в known-issues, а не повторно репортить.
