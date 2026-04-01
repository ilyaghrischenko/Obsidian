date: 2026-04-02
tags: [ai, codex, context, architecture]
versions: [ASP.NET Core 9, Next.js 15]

## Что такое AGENTS.md?
`AGENTS.md` — это локальный файл конфигурации и контекста для ИИ-агента (например, Codex). Он работает как "законы стройплощадки" для конкретной директории или монорепозитория. Файл объясняет агенту специфику текущей кодовой базы, чтобы предотвратить галлюцинации и использование дефолтных паттернов из интернета.

## Зачем использовать?
- **Изоляция контекста:** Позволяет задать разные правила для бэкенда (`/backend/AGENTS.md`) и фронтенда (`/frontend/AGENTS.md`).
- **Экономия токенов:** Агенту не нужно угадывать стек или сканировать весь проект, базовые правила уже загружены в контекст.
- **Принуждение к код-стайлу:** Жестко фиксирует использование конкретных библиотек (FluentValidation) и подходов (отказ от `var`, использование `Result<T>`).

## ⚙️ Настройка
Файл должен лежать в корне модуля (например, рядом с `.slnx` или `.csproj`).

✅ Пример структуры для ASP.NET Core:
```md
# Project: LudaFit (Backend)

## Architecture & Patterns
- **Vertical Slice Architecture (VSA):** Each feature MUST be contained in a single static file.
- **Result Pattern:** Always return `Result<T>`.

## Critical Coding Rules (MUST FOLLOW)
- **Typing:** Use EXPLICIT types only.
- **Database:** Always use `.AsNoTracking()` for read-only EF Core queries.

## Workspace Commands
- **Build:** `dotnet build LudaFit.Server.slnx`
- **Run Tests:** `dotnet test LudaFit.Tests`
```

## Итог

`AGENTS.md` хранит специфику **КОНКРЕТНОГО** проекта. В нем прописываются используемые методы расширения, пути сборки и жесткие ограничения для конкретного слоя архитектуры.