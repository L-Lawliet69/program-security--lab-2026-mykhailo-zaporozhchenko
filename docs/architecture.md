# Опис архітектури системи (Architecture Description) — Freekick

**Відповідно до положень міжнародного стандарту ISO/IEC/IEEE 42010:2022 (у спрощеному академічному форматі)**

| | |
| --- | --- |
| **Документ** | Architecture Description (AD) |
| **Система** | Freekick — Любительська онлайн-гра "Пенальті" для ПК (Unity C#) |
| **Версія** | 2.0.0 |
| **Дата оновлення** | 2026-10-12 |
| **Статус** | Approved / Baseline |
| **Автор** | Mykhailo Zaporozhchenko |

---

## 1. Ідентифікація архітектури та контекст

Система **«Freekick»** — це любительська комп'ютерна онлайн-гра (дуель 1v1 серії післяматчевих пенальті з аніме-персонажами) для платформи **PC (Windows)**, створена на ігровому рушії **Unity (C#)**.

Гра призначена для особистого користування та спільних матчів з друзями по мережі, а також виконується як практичне завдання до лабораторної роботи з дисципліни **«Безпека програм та даних» (Program Security Lab 2026)**.

Замість надмірно складних корпоративних підходів архітектура базується на **простій та прозорій модульній об'єктно-орієнтованій моделі (ООП на C#)** у середовищі Unity, що ідеально підходить для студентського проекту та навчальних завдань із захисту даних.

---

## 2. Зацікавлені сторони (Stakeholders & Concerns)

| Зацікавлена сторона | Роль | Ключові інтереси (Concerns) |
|---|---|---|
| **Гравці (Друзі / Користувачі)** | Кінцеві гравці | Зручність гри на ПК, простий запуск матчу з другом по мережі, вибір улюблених аніме-персонажів, чесний підрахунок очок. |
| **Викладач / Студент (Безпека ПЗ)** | Лабораторна робота | Дотримання вимог безпеки: надійне хешування паролів, захист від сніфінгу ходів (Anti-Sniffing), перевірка цілісності збережень (HMAC-SHA256), відсутність SQL-ін'єкцій. |
| **Розробник (Unity C#)** | Підтримка проекту | Простота коду, чиста ООП структура без громіздких патернів, розділення логіки та графіки (UI), легке тестування. |

---

## 3. Архітектурні представлення (Views)

### 3.1. Мережевий контекст (Network & System Topology)

Для гри з друзями використовується легка клієнт-серверна топологія з моделлю **Host-Authoritative** (один гравець запускає хост, інший підключається як клієнт через локальну мережу, прямий IP або код кімнати):

```mermaid
flowchart LR
    subgraph HostPC ["ПК Гравця А (Хост / Сервер)"]
        UnityHost["Unity Клієнт + Хост-контролер"]
        LocalSaveA["Локальні збереження (SQLite / JSON)\n+ HMAC-SHA256"]
        UnityHost --- LocalSaveA
    end

    subgraph ClientPC ["ПК Гравця Б (Клієнт)"]
        UnityClient["Unity Клієнт"]
        LocalSaveB["Локальні збереження (SQLite / JSON)\n+ HMAC-SHA256"]
        UnityClient --- LocalSaveB
    end

    UnityClient <== "Мережевий сокетний зв'язок (LAN / Direct IP / Код кімнати)" ==> UnityHost
```

- **Хост-авторитет (Host-Authoritative):** гравець-хост або легкий локальний сокетний сервер тримає стан кімнати, синхронізує таймер та розраховує голи/сейви.
- **Збереження даних:** облікові записи та статистика зберігаються локально на кожному ПК у файлі `save.json` або базі `SQLite` з контрольною сумою HMAC-SHA256 для захисту від підробки значень.

---

### 3.2. Компонентна архітектура клієнта Unity (C#)

Архітектура коду організована за принципом простого поділу обов'язків:

```mermaid
flowchart TB
    subgraph UI ["Шар інтерфейсу (Unity MonoBehaviour & UI Canvas)"]
        AuthUI["Вікно входу / реєстрації"]
        ProfileWidget["Віджет профілю (правий верхній кут)"]
        CharacterCatalogUI["Каталог тайтлів і героїв"]
        MatchUI["Ігровий екран (ворота, кнопки напрямку, рахунок)"]
        LeaderboardUI["Таблиця результатів"]
    end

    subgraph Managers ["Шар ігрових менеджерів (MonoBehaviour)"]
        GameManager["GameManager (Життєвий цикл гри)"]
        MatchController["MatchController (Керування 10 ударами)"]
        NetworkManager["NetworkManager (Обмін пакетами між друзями)"]
    end

    subgraph Core ["Шар чистої C# логіки (Plain C# Classes)"]
        PenaltyEngine["PenaltyEngine (Розрахунок голів, жеребкування)"]
        MatchSession["MatchSession (Дані матчу, 5 раундів)"]
        PlayerData["PlayerData & Enums"]
    end

    subgraph Services ["Шар сервісів та безпеки (Data & Security)"]
        AuthService["AuthService (Перевірка нікнеймів, вхід)"]
        SaveManager["SaveManager (Запис / читання JSON та SQLite)"]
        SecurityHelper["SecurityHelper (Argon2id/PBKDF2, HMAC-SHA256, CSPRNG)"]
    end

    UI --> Managers
    Managers --> Core
    Managers --> Services
    Core --> Services
```

---

### 3.3. Представлення безпеки: Захист від сніфінгу (Commit-Reveal)

Щоб суперник не міг перехопити напрямок удару через мережевий аналізатор трафіку (Wireshark), вибори напрямків надсилаються приховано і розкриваються одночасно:

```mermaid
sequenceDiagram
    autonumber
    actor PlayerA as Стрілець (Гравець А)
    participant Host as Хост матчу (Unity Host)
    actor PlayerB as Воротар (Гравець Б)

    Note over PlayerA,PlayerB: Початок удару (таймер 10 секунд на вибір)
    PlayerA->>Host: Пакет: SubmitDirection(Left) [Приховано]
    Note over Host: Хост зберігає вибір Стрільця, НЕ передаючи його Воротарю!
    Host-->>PlayerA: Підтвердження: Вибір прийнято
    PlayerB->>Host: Пакет: SubmitDirection(Center) [Приховано]
    Note over Host: Отримано вибори від ОБОХ гравців!
    Note over Host: PenaltyEngine.ResolveShot(Left, Center) => GOAL
    Host->>PlayerA: Розкриття результату: (Стрілець = Left, Воротар = Center, Результат = GOAL)
    Host->>PlayerB: Розкриття результату: (Стрілець = Left, Воротар = Center, Результат = GOAL)
    Note over PlayerA,PlayerB: Обидва клієнти одночасно програють анімацію удару та оновлюють табло
```

---

### 3.4. Модель даних (Прості класи та зв'язки)

```mermaid
classDiagram
    class PlayerProfile {
        +string Id
        +string Nickname
        +string Email
        +string PasswordHash
        +string ActiveCharacterId
    }

    class AnimeTitle {
        +string Id
        +string Name
        +List~AnimeCharacter~ Characters
    }

    class AnimeCharacter {
        +string Id
        +string Name
        +string TitleId
        +string SpritePath
    }

    class MatchSession {
        +string MatchId
        +string PlayerAId
        +string PlayerBId
        +string FirstShooterId
        +int CurrentRound
        +int TotalShots
        +int ScoreA
        +int ScoreB
        +MatchOutcome Outcome
        +List~ShotRecord~ History
    }

    class ShotRecord {
        +int RoundNumber
        +string ShooterId
        +string KeeperId
        +Direction ShotDirection
        +Direction DiveDirection
        +ShotResult Result
    }

    class LeaderboardEntry {
        +string Nickname
        +int MatchesPlayed
        +int Wins
        +int Losses
        +int Draws
        +int GoalsScored
        +int GoalsConceded
        +int Points
    }

    PlayerProfile "1" --> "1" AnimeCharacter : обирає
    AnimeTitle "1" --> "*" AnimeCharacter : містить
    MatchSession "1" --> "*" ShotRecord : записує
    PlayerProfile "1" --> "1" LeaderboardEntry : має статистику
```

---

## 4. Журнал архітектурних рішень (ADR)

### ADR-001: Вибір Unity (C#) та легкого мережевого коду для ПК
- **Контекст:** Проект є любительською грою для особистого вжитку з друзями та завданням до лабораторної роботи з безпеки ПЗ. Розгортати важкі мікросервіси в хмарі є недоцільним та ресурсомістким.
- **Рішення:** Розробка на рушії Unity з мовою C# для платформи Windows. Мережева взаємодія реалізується через легкий C# код (Unity Netcode / Mirror / сокети) з підключенням за кодом кімнати або прямим IP.
- **Наслідки:** Швидка розробка, легкий запуск на ПК без необхідності орендувати сервери.

### ADR-002: Використання чистої та зрозумілої модульної ООП архітектури на C#
- **Контекст:** Проект створюється як любительська гра для гри з друзями та лабораторна робота. Надмірна кількість складних абстракцій і шаблонів корпоративного рівня ускладнювала розробку та розуміння простої гри в пенальті.
- **Рішення:** Використання класичного об'єктно-орієнтованого проектування (C# класи, моделі, менеджери `GameManager`, `MatchController`, `AuthService`, звичайні переліки).
- **Наслідки:** Код є лаконічним, легким для сприйняття, швидко модифікується та легко покривається юніт-тестами.

### ADR-003: Обов'язкове завершення всіх 5 раундів (10 ударів) та фіксація нічиєї
- **Контекст:** Для справедливого обліку статистики забитих та пропущених м'ячів між друзями необхідно зберігати повну історію серії.
- **Рішення:** Гра завжди триває рівно 5 раундів (по 5 ударів у кожного гравця, сумарно 10 ударів) незалежно від поточного відриву. При рівності голів після 10 ударів фіксується нічия (`Draw`), і обом гравцям нараховується по +1 очку в таблицю.
- **Наслідки:** Чесна та повна статистика матчів.

### ADR-004: Схема захисту від сніфінгу ходів та контроль цілісності сейвів
- **Контекст:** Лабораторна робота вимагає демонстрації практичних навичок захисту даних.
- **Рішення:**
  1. Приховане зведення ходів на хості (Commit-Reveal) для захисту від мережевого перехоплення пакетів у Wireshark.
  2. Додавання підпису HMAC-SHA256 до файлу збереження `save.json` для виявлення ручного редагування очок у блокноті.
  3. Хешування паролів із сіллю (Argon2id або PBKDF2).
- **Наслідки:** Повне виконання навчальних вимог дисципліни «Безпека програм та даних».
