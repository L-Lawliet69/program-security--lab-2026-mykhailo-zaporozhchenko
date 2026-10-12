# Журнал змін (Changelog) — Gameplay Module

## [2026-10-12] [OOP-001] Спрощення архітектури та перехід на C# ООП для Unity
- Повністю вилучено Domain-Driven Design (DDD): прибрано Aggregate Roots, Value Objects, Domain Policies, Outsource Contracts.
- Запроваджено просту об'єктно-орієнтовану структуру на C#: класи `MatchSession`, `PenaltyEngine`, `ShotRecord`, переліки `Direction`, `ShotResult`, `MatchOutcome`.
- Переведено механізм Commit-Reveal на легку модель пам'яті хоста Unity (без зовнішнього Redis).
- Збережено всі правила гри: 5 обов'язкових раундів (10 ударів), 3 напрямки (`Left`, `Center`, `Right`), жеребкування CSPRNG та сценарій нічиєї (`Draw`).

## [2026-10-03] [DOCS-001] Початкова специфікація ігрового процесу
- Описано правила 5 раундів пенальті 1v1 між гравцями.
- Визначено механіку розрахунку голів і сейвів та сценарій нічиєї.
