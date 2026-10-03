# Документація проекту Freekick (Anime Penalty Shootout Online)

Ласкаво просимо до офіційного репозиторію проектної документації та архітектурних специфікацій багатокористувацької онлайн-гри **«Freekick»**.

Проект розробляється в рамках лабораторної роботи з дисципліни **«Безпека програм та даних» (Program Security Lab 2026)**.

---

## 🧭 Навігація по документації

| Документ | Стандарт / Шаблон | Опис |
|---|---|---|
| 📜 [**Політика проекту (docs/project_policy.md)**](file:///d:/freekick/docs/project_policy.md) | На основі `project-policy-example.md` | Обов'язкові вимоги до коду, архітектури (DDD), криптографічного захисту (Argon2id), системи винятків `FK.*`, тестування та Git Workflow. |
| ⚽ [**Загальна бізнес-логіка (docs/business_logic.md)**](file:///d:/freekick/docs/business_logic.md) | На основі `business_logic.md` | Концептуальний зміст доменної моделі: агрегати `Match`, `UserAccount`, раунди, обов'язкові 5 ударів, жеребкування та розрахунок голів. |
| 📋 [**Реєстр модулів (docs/modules_registry.md)**](file:///d:/freekick/docs/modules_registry.md) | Рекомендації з організації `docs/` | Опис меж Bounded Contexts: Identity, Character, Gameplay, Leaderboard, SharedKernel та зв'язків між ними. |
| 📐 [**Специфікація вимог до ПЗ (docs/srs.md)**](file:///d:/freekick/docs/srs.md) | **ISO/IEC/IEEE 29148:2018** | Функціональні (`FR-*`) та нефункціональні (`NFR-*`) вимоги до системи, захист від атак, криптографія, матриця верифікації та приймання. |
| 🏛️ [**Опис архітектури (docs/architecture.md)**](file:///d:/freekick/docs/architecture.md) | **ISO/IEC/IEEE 42010:2022** | Архітектурні точки зору (Context, Functional, Security Anti-Sniffing, Information ERD), журнал архітектурних рішень (ADR-001..004). |

---

## 📦 Специфікації за модулями (Bounded Contexts)

Кожен функціональний контекст системи задокументований згідно з проектною політикою у відповідних підпапках:

1. **Ігровий процес 1v1 (Gameplay):**
   - [docs/modules/gameplay/business_logic.md](file:///d:/freekick/docs/modules/gameplay/business_logic.md) — логіка матчу, жеребкування CSPRNG, серія з 5 обов'язкових раундів (10 ударів), вибір 3 напрямків.
   - [docs/modules/gameplay/changelog.md](file:///d:/freekick/docs/modules/gameplay/changelog.md) — історія змін модуля.
   - [docs/modules/gameplay/tech_notes.md](file:///d:/freekick/docs/modules/gameplay/tech_notes.md) — технічні нотатки щодо Commit-Reveal захисту від сніфінгу.

2. **Автентифікація та облікові записи (Identity):**
   - [docs/modules/identity/business_logic.md](file:///d:/freekick/docs/modules/identity/business_logic.md) — реєстрація, унікальність нікнеймів без урахування регістру, Argon2id хешування.
   - [docs/modules/identity/changelog.md](file:///d:/freekick/docs/modules/identity/changelog.md) — історія змін модуля.
   - [docs/modules/identity/tech_notes.md](file:///d:/freekick/docs/modules/identity/tech_notes.md) — технічні параметри Argon2id та індексація БД.

3. **Каталог аніме-персонажів (Character):**
   - [docs/modules/character/business_logic.md](file:///d:/freekick/docs/modules/character/business_logic.md) — список тайтлів («Blue Lock», «Captain Tsubasa» тощо), викачування списку персонажів, вибір героя у профілі.
   - [docs/modules/character/changelog.md](file:///d:/freekick/docs/modules/character/changelog.md) — історія змін модуля.
   - [docs/modules/character/tech_notes.md](file:///d:/freekick/docs/modules/character/tech_notes.md) — кешування ассетів та дефолтні значення.

4. **Глобальний рейтинг та статистика (Leaderboard):**
   - [docs/modules/leaderboard/business_logic.md](file:///d:/freekick/docs/modules/leaderboard/business_logic.md) — публічна таблиця лідерів, лічильники перемог, поразок, забитих і пропущених голів.
   - [docs/modules/leaderboard/changelog.md](file:///d:/freekick/docs/modules/leaderboard/changelog.md) — історія змін модуля.
   - [docs/modules/leaderboard/tech_notes.md](file:///d:/freekick/docs/modules/leaderboard/tech_notes.md) — ідемпотентна обробка подій та Redis Sorted Sets.

---

## 🎯 Ключові особливості гри та вимоги безпеки

1. **Автентифікація та профілі:** Реєстрація за електронною поштою, строго унікальним нікнеймом (case-insensitive) та надійним паролем. Паролі захищені за допомогою **Argon2id**.
2. **Аніме-ростер у профілі:** У правому верхньому куті вікна програми гравець бачить свій профіль, де може відкрити каталог тайтлів, розгорнути персонажів будь-якого аніме та обрати свого активного героя.
3. **Онлайн-матч 1 на 1:** Жеребкування черги першого удару (CSPRNG), по черзі по 1 удару кожним гравцем, рівно 5 раундів (10 ударів сумарно).
4. **Механіка 3 напрямків:** Стрілець та Воротар обирають `LEFT`, `CENTER` або `RIGHT`. Якщо вибори збіглися — сейв, якщо різняться — гол.
5. **Обов'язкові 5 ударів:** Навіть якщо після 3-го раунду результат математично визначений (наприклад, 3:0), матч обов'язково продовжується до кінця всіх 10 ударів для повної фіксації статистики забитих та пропущених м'ячів.
6. **Захист від сніфінгу:** Сервер фіксує вибори ходів за моделлю одночасного прихованого вводу (Commit-Reveal), унеможливлюючи перехоплення вибору суперника через аналіз сокетного трафіку.
7. **Глобальний лідерборд:** Загальнодоступна таблиця лідерів з автоматичним оновленням статистики після кожної дуелі.
