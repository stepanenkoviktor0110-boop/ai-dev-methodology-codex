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
