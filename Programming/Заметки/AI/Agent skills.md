date: 2026-04-02
tags: [ai, codex, skills, scaffolding]
versions: [Codex]

## Что такое Agent skills (SKILL.md)?
**Agent skill** (описывается в файле `SKILL.md`) — это универсальный, переиспользуемый алгоритм (чертеж) для выполнения конкретной инженерной задачи. Он хранится глобально (обычно в `~/.codex/skills/`) и не привязан к конкретному проекту.

## Зачем использовать?
- **Абстракция паттернов:** Описывает **КАК** писать код (например, структуру Vertical Slice), а не для какого проекта.
- **Автоматизация рутины:** Содержит встроенный цикл `Workflow & Execution` (планирование -> генерация -> проверка компилятором -> самоисправление).
- **Защита от антипаттернов:** Позволяет жестко запретить ИИ использовать нежелательные технологии или привычки (например, запрет на `MediatR` или `Repository pattern`).

## ⚙️ Настройка
Скилл состоит из папки с обязательным конфигурационным файлом `agents/openai.yaml` (дает права агенту на запуск CLI-команд) и самой инструкции `SKILL.md`.

✅ Пример ключевого блока `SKILL.md` для генерации VSA:
```md
## Logic Placement Rule (Endpoint vs. Handler)
- **Simple Logic (< 100 lines):** Write logic DIRECTLY inside the `Endpoint` class.
- **Complex Logic:** Extract logic into a nested `Handler` class and inject it.

## Workflow & Execution
1. **Plan:** Analyze the request and output a plan.
2. **Generate:** Write the feature in a single file.
3. **Verify:** Run `dotnet build`. Fix errors up to 3 times. Do NOT auto-commit.
```

## Итог

Скилл — это строгая архитектурная инструкция, очищенная от локальных неймспейсов и специфики конкретных БД. Он учит ИИ собирать каркас кода, полагаясь на ошибки компилятора (Compiler-Driven Development) для автокоррекции зависимостей.