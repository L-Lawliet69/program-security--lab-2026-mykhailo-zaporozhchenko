# Реєстр модулів та Bounded Contexts — Freekick

| | |
| --- | --- |
| **Проект** | Freekick — Онлайн-гра "Пенальті" з аніме-персонажами |
| **Документ** | Реєстр модулів та архітектурних меж (Bounded Contexts) |
| **Версія** | 1.0.0 |
| **Дата оновлення** | 2026-10-03 |

> Цей документ є обов'язковим реєстром усіх модулів та обмежених контекстів (Bounded Contexts) системи **Freekick**. Він визначає зону відповідальності кожного модуля, його агрегати, зовнішні залежності та рівень безпеки.

---

## 1. Загальна карта контекстів (Context Map)

```
+-------------------------------------------------------------------+
|                        Presentation Layer                         |
|   (HTTP Controllers, WebSocket Gateway, CLI Seeders/Admin)        |
+-------------------------------------------------------------------+
                                  │
                                  ▼
+-------------------------------------------------------------------+
|                        Application Layer                          |
|             (CQRS Use Cases, Commands, Queries, DTOs)             |
+-------------------------------------------------------------------+
                                  │
                                  ▼
+-------------------+     +--------------------+     +--------------+
|     Identity      |     |     Character      |     |   Gameplay   |
| (Accounts, Auth,  |     |  (Anime Titles &   |     |  (1v1 Match, |
|     Security)     |     |     Rosters)       |     |  5 Rounds)   |
+-------------------+     +--------------------+     +--------------+
          │                         │                        │
          └─────────────────────────┼────────────────────────┘
                                    ▼
                         +--------------------+
                         |    Leaderboard     |
                         | (Global Rankings,  |
                         |    Statistics)     |
                         +--------------------+
                                    │
                                    ▼
                         +--------------------+
                         |    SharedKernel    |
                         | (Base VO, Events,  |
                         | Exception Registry)|
                         +--------------------+
```

---

## 2. Перелік Bounded Contexts

### 2.1. `Identity` (Облікові записи, автентифікація та профілі)
- **Шлях у вихідному коді:** `src/Domain/Identity/`, `src/Application/Identity/`
- **Документація модуля:** [docs/modules/identity/business_logic.md](file:///d:/freekick/docs/modules/identity/business_logic.md)
- **Призначення:** Забезпечення безпечної реєстрації, авторизації, зберігання та перевірки паролів (Argon2id), видачі сесійних токенів, керування профілем користувача та гарантії унікальності нікнеймів.
- **Aggregate Root:** `UserAccount`
- **Ключові ентіті:** `Profile`
- **Сегмент помилок:** `FK.AUTH.x`
- **Зовнішні контракти (Outsource Contracts):** `IUserRepository`, `IPasswordHasher`, `ITokenProvider`
- **Залежності:** `SharedKernel`

### 2.2. `Character` (Каталог тайтлів та персонажів)
- **Шлях у вихідному коді:** `src/Domain/Character/`, `src/Application/Character/`
- **Документація модуля:** [docs/modules/character/business_logic.md](file:///d:/freekick/docs/modules/character/business_logic.md)
- **Призначення:** Зберігання структурованого каталогу аніме-франшиз (тайтлів) та відповідних персонажів. Надання інтерфейсу для вибору та зміни активного персонажа в налаштуваннях профілю.
- **Aggregate Root:** `AnimeTitle`
- **Ключові ентіті:** `Character`
- **Сегмент помилок:** `FK.CHAR.x`
- **Зовнішні контракти (Outsource Contracts):** `ICharacterRepository`, `ITitleRepository`
- **Залежності:** `SharedKernel`

### 2.3. `Gameplay` (Ігровий рушій 1v1 пенальті)
- **Шлях у вихідному коді:** `src/Domain/Gameplay/`, `src/Application/Gameplay/`
- **Документація модуля:** [docs/modules/gameplay/business_logic.md](file:///d:/freekick/docs/modules/gameplay/business_logic.md)
- **Призначення:** Керування життєвим циклом матчу 1v1: жеребкування першого удару (CSPRNG Coin Toss), покрокове пробиття пенальті (5 ударів у кожного гравця, сумарно 10), захист від сніфінгу вибору (Commit-Reveal / прихований ввід), розрахунок результатів сейвів/голів та фіксація підсумку гри.
- **Aggregate Root:** `Match`
- **Ключові ентіті:** `PenaltyRound`, `ShotAttempt`
- **Сегмент помилок:** `FK.GAME.x`
- **Зовнішні контракти (Outsource Contracts):** `IMatchRepository`, `IRandomNumberGenerator`, `IRealtimeGameNotifier`
- **Залежності:** `SharedKernel`, посилання на `UserIdVO` з `Identity` та `CharacterIdVO` з `Character`.

### 2.4. `Leaderboard` (Глобальний рейтинг та статистика)
- **Шлях у вихідному коді:** `src/Domain/Leaderboard/`, `src/Application/Leaderboard/`
- **Документація модуля:** [docs/modules/leaderboard/business_logic.md](file:///d:/freekick/docs/modules/leaderboard/business_logic.md)
- **Призначення:** Ведення загальної публічної таблиці лідерів для всіх учасників. Накопичення кількості матчів, перемог, поразок, нічиїх, забитих та пропущених голів, розрахунок коефіцієнтів та місця в топі.
- **Aggregate Root:** `LeaderboardEntry` (Read/Write Model)
- **Сегмент помилок:** `FK.LEAD.x`
- **Зовнішні контракти (Outsource Contracts):** `ILeaderboardRepository`
- **Залежності:** `SharedKernel`; асинхронно реагує на подію `MatchCompletedDE` від `Gameplay`.

### 2.5. `SharedKernel` (Спільне ядро)
- **Шлях у вихідному коді:** `src/Domain/SharedKernel/`
- **Призначення:** Фундаментальні абстракції, інтерфейси сутностей та об'єктів-значень, базові класи винятків (`AFKException`), контракти доменних подій.
- **Сегмент помилок:** `FK.CORE.x`
- **Залежності:** Нуль зовнішніх залежностей.

---

## 3. Матриця міжмодульної взаємодії

| Ініціатор | Одержувач | Спосіб взаємодії | Призначення |
|---|---|---|---|
| `Identity` | `Leaderboard` | Domain Event (`UserRegisteredDE`) | Створення початкового запису гравця в таблиці лідерів з нульовою статистикою |
| `Identity` | `Character` | Reference by ID (`CharacterIdVO`) | Прив'язка обраного персонажа до профілю користувача |
| `Gameplay` | `Identity` | Reference by ID (`UserIdVO`) | Ідентифікація учасників матчу |
| `Gameplay` | `Character` | Reference by ID (`CharacterIdVO`) | Візуалізація аватара та спеціальних анімацій гравця в матчі |
| `Gameplay` | `Leaderboard` | Domain Event (`MatchCompletedDE`) | Атомарне оновлення статистики зіграного матчу (перемога/поразка, забиті/пропущені голи) |
