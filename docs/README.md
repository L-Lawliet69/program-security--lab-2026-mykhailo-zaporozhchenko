# Документація проекту Freekick (Anime Penalty Shootout Online)

Ласкаво просимо до документації багатокористувацької онлайн-гри **«Freekick»** — серії післяматчевих футбольних пенальті 1 на 1 з аніме-персонажами під ПК (Windows).

Проект розробляється як **любительська гра для особистого користування з друзями** в рамках лабораторної роботи з навчальної дисципліни **«Безпека програм та даних» (Program Security Lab 2026)**.

---

## 🧭 Навігація по документації

| 📜 [**Політика проекту (docs/project_policy.md)**](file:///d:/freekick/docs/project_policy.md) | Стандарти проекту | Організація Unity-проекту (C#), стандарти кодування, безпека даних збережень (HMAC-SHA256), криптографічний захист паролів, Git workflow. |
| 🛡️ [**Політика безпеки (docs/security_policy.md)**](file:///d:/freekick/docs/security_policy.md) | Стандарти безпеки ПЗ | Обов'язкові правила безпечної розробки, захист паролів, Commit-Reveal, HMAC-контроль цілісності, безпечне середовище. |
| 📊 [**Модель загроз STRIDE (docs/threat_model.md)**](file:///d:/freekick/docs/threat_model.md) | Моделювання загроз | Аналіз компонентів системи за STRIDE, діаграма DFD, загрози мережевого сніфінгу, контрзаходи. |
| 🔒 [**Політика конфіденційності (docs/privacy_policy.md)**](file:///d:/freekick/docs/privacy_policy.md) | Захист персональних даних | Правила обробки нікнейму, email та статистики дуелей, локальне зберігання даних на ПК гравця. |
| ⚽ [**Загальна бізнес-логіка (docs/business_logic.md)**](file:///d:/freekick/docs/business_logic.md) | Ігрова логіка | Зрозуміла легка логіка дуелі: 5 обов'язкових раундів (10 ударів), 3 напрямки (`LEFT`, `CENTER`, `RIGHT`), розрахунок голів/сейвів, жеребкування, сценарій нічиєї. |
| 📋 [**Реєстр модулів (docs/modules_registry.md)**](file:///d:/freekick/docs/modules_registry.md) | Структура коду | Опис підсистем: `Identity`, `Character`, `Gameplay`, `Leaderboard`, `Network`, `Security`. |
| 📐 [**Специфікація вимог до ПЗ (docs/srs.md)**](file:///d:/freekick/docs/srs.md) | Спрощений **ISO/IEC/IEEE 29148:2018** | Функціональні вимоги до гри, вимоги безпеки для лаби, критерії приймального тестування. |
| 🏛️ [**Опис архітектури (docs/architecture.md)**](file:///d:/freekick/docs/architecture.md) | Спрощений **ISO/IEC/IEEE 42010:2022** | Архітектурні схеми Unity-додатка, взаємодія хоста та клієнта, захист від сніфінгу ходів (Commit-Reveal), збереження даних у SQLite/JSON. |

---

## 📦 Специфікації за модулями (підсистемами)

Документація окремих функціональних підсистем проекту:

1. **Ігровий процес 1v1 (Gameplay):**
   - [docs/modules/gameplay/business_logic.md](file:///d:/freekick/docs/modules/gameplay/business_logic.md) — перебіг матчу, 5 обов'язкових раундів, 10 ударів, вибір 3 напрямків, розрахунок результату та нічия.
   - [docs/modules/gameplay/changelog.md](file:///d:/freekick/docs/modules/gameplay/changelog.md) — історія змін ігрового модуля.
   - [docs/modules/gameplay/tech_notes.md](file:///d:/freekick/docs/modules/gameplay/tech_notes.md) — технічна реалізація прихованого вводу (Anti-Sniffing) та сценарій раптової смерті (Sudden Death).

2. **Облікові записи та безпека (Identity):**
   - [docs/modules/identity/business_logic.md](file:///d:/freekick/docs/modules/identity/business_logic.md) — проста реєстрація/вхід, унікальність нікнеймів без урахування регістру, хешування паролів.
   - [docs/modules/identity/changelog.md](file:///d:/freekick/docs/modules/identity/changelog.md) — історія змін модуля авторизації.
   - [docs/modules/identity/tech_notes.md](file:///d:/freekick/docs/modules/identity/tech_notes.md) — параметри хешування паролів (Argon2id/PBKDF2) та робота з локальною БД/JSON.

3. **Каталог аніме-персонажів (Character):**
   - [docs/modules/character/business_logic.md](file:///d:/freekick/docs/modules/character/business_logic.md) — список франшиз («Blue Lock», «Captain Tsubasa» тощо), розгортання списку героїв, вибір активного героя у профілі.
   - [docs/modules/character/changelog.md](file:///d:/freekick/docs/modules/character/changelog.md) — історія змін модуля персонажів.
   - [docs/modules/character/tech_notes.md](file:///d:/freekick/docs/modules/character/tech_notes.md) — зберігання ассетів у Unity (`Resources` / `ScriptableObjects`).

4. **Рейтинг та статистика друзів (Leaderboard):**
   - [docs/modules/leaderboard/business_logic.md](file:///d:/freekick/docs/modules/leaderboard/business_logic.md) — таблиця результатів: перемоги, поразки, нічиї, забиті/пропущені голи, очкова система ($3 \times W + 1 \times D$).
   - [docs/modules/leaderboard/changelog.md](file:///d:/freekick/docs/modules/leaderboard/changelog.md) — історія змін модуля статистики.
   - [docs/modules/leaderboard/tech_notes.md](file:///d:/freekick/docs/modules/leaderboard/tech_notes.md) — захист цілісності сейву/таблиці рейтингу від редагування блокнотом (HMAC-SHA256).

---

## 🎯 Ключові принципи та акценти розробки

1. **Любительський формат для особистого вжитку:** Гра створюється для комфортного проведення матчів з друзями на ПК. Складні хмарні сервери, оркестрація контейнерів та важкі мікросервіси замінені легкою моделлю на Unity C# (локальний хост або простий C# бекенд).
2. **Типовий ігровий рушій (Unity / C#):** Вся логіка пишеться зрозумілим об'єктно-орієнтованим C# кодом, розділеним на логічні класи (`GameManager`, `MatchController`, `AuthService`, `CharacterCatalog`).
3. **Збереження концепції безпеки для лабораторної:**
   - Криптографічне хешування паролів із сіллю.
   - Захист від мережевого підглядання ходів суперника (Commit-Reveal).
   - Захист файлів збережень і статистики від накрутки значень (HMAC-SHA256 хеш-підпис).
   - Безпечне жеребкування черги ударів через CSPRNG.
