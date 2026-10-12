# Журнал змін (Changelog) — Character Module

## [2026-10-12] [OOP-001] Спрощення архітектури та перехід на C# ООП для Unity
- Повністю вилучено Domain-Driven Design (DDD): прибрано Aggregate Root `AnimeTitle`, Value Objects та складну доменну політику доступності.
- Переведено структуру на прості C# класи `AnimeTitle`, `AnimeCharacter` та сервіс `CharacterCatalog`.
- Ассети персонажів налаштовано на завантаження через стандартний механізм Unity (`Resources.Load` / ScriptableObjects).

## [2026-10-03] [DOCS-001] Каталог тайтлів та логіка вибору аніме-персонажів
- Описано каталог франшиз та вибір героя у профілі.
