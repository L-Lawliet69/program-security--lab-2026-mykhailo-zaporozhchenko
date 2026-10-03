# ====== LEADERBOARD MODULE BUSINESS LOGIC ======

> Bounded Context: `Leaderboard`. Відповідає за агрегацію результатів усіх зіграних матчів, ведення глобальної таблиці лідерів для всіх учасників та надання публічного доступу до статистики (кількість перемог, поразок, нічиїх, забитих та пропущених голів).

---

## 1. Purpose

Модуль `Leaderboard` володіє:
- Веденням та формуванням глобальної таблиці лідерів, доступної всім зареєстрованим користувачам.
- Збереженням агрегованих статистичних показників для кожного гравця:
  - Кількість зіграних матчів (`matchesPlayed`).
  - Кількість перемог (`wins`), поразок (`losses`), нічиїх (`draws`).
  - Кількість забитих голів (`goalsScored`).
  - Кількість пропущених голів (`goalsConceded`).
  - Різниця м'ячів (`goalDifference = goalsScored - goalsConceded`).
  - Відсоток перемог (`winRate`).
- Асинхронною обробкою івенту `MatchCompletedDE` від модуля `Gameplay`.
- Наданням сторінок лідерборду з пагінацією та пошуком за нікнеймом.

**NOT here:**
- Розрахунок голів окремих ударів у матчі (належить `Gameplay`).
- Автентифікація користувачів (належить `Identity`).

---

## 2. Entities

| Entity | Basic Fields | Description | Invariants |
|---|---|---|---|
| `LeaderboardEntry` *(Aggregate Root)* | `userId`, `nickname`, `activeCharacterName`, `matchesPlayed`, `wins`, `losses`, `draws`, `goalsScored`, `goalsConceded`, `points`, `updatedAt` | Запис конкретного гравця в рейтинговій таблиці. | - `userId` унікальний в таблиці лідерів.<br>- `matchesPlayed == wins + losses + draws`.<br>- Всі лічильники є цілими невід'ємними числами ($\ge 0$).<br>- `points = (wins * 3) + (draws * 1)`. |

---

## 3. Key Flows

### Флоу оновлення статистики після завершення матчу:
1. Модуль `Gameplay` емітить івент `MatchCompletedDE` із полями: `playerAId`, `playerBId`, `finalScoreA`, `finalScoreB`, `winnerId`.
2. Асинхронний слухач `UpdateLeaderboardOnMatchCompletedListener` отримує подію.
3. Виконується перевірка ідемпотентності (за `matchId`), щоб запобігти подвійному нарахуванню очок при збоях мережі.
4. В межах єдиної транзакції:
   - Для Гравця А:
     - `matchesPlayed += 1`
     - `goalsScored += finalScoreA`
     - `goalsConceded += finalScoreB`
     - Якщо `winnerId == playerAId` $\rightarrow$ `wins += 1`
     - Якщо `winnerId == null` (нічия) $\rightarrow$ `draws += 1`
     - Інакше $\rightarrow$ `losses += 1`
   - Для Гравця Б:
     - `matchesPlayed += 1`
     - `goalsScored += finalScoreB`
     - `goalsConceded += finalScoreA`
     - Якщо `winnerId == playerBId` $\rightarrow$ `wins += 1`
     - Якщо `winnerId == null` $\rightarrow$ `draws += 1`
     - Інакше $\rightarrow$ `losses += 1`
5. Перераховується рейтинг та позиція обох гравців.
6. Публікується подія `LeaderboardUpdatedDE`.

### Флоу перегляду таблиці лідерів:
1. Зареєстрований користувач відкриває розділ «Таблиця лідерів» (Leaderboard).
2. Клієнт надсилає `GetGlobalLeaderboardAQ` (з параметрами сторінки та сортування).
3. Сервер повертає відсортований список (за замовчуванням за `points DESC`, потім `goalDifference DESC`, потім `goalsScored DESC`).
4. Користувач бачить свій поточний ранг і показники всіх суперників.

---

## 4. Value Objects

| Value Object | Опис | Інваріанти |
|---|---|---|
| `RatingPointsVO` | Очки гравця в рейтингу | Обчислюється як $3 \times W + 1 \times D$ |
| `GoalRatioVO` | Різниця та статистика голів | `goalsScored >= 0`, `goalsConceded >= 0` |
| `WinRateVO` | Відсоток перемог | Число з рухомою комою в межах $[0.0, 100.0]\%$ |

---

## 5. Domain Policies

- `PositiveStatsDPolicy`: Статистичні лічильники матчів, голів та очок не можуть бути від'ємними за жодних умов.
- `LeaderboardRankingPolicy`: Правило ранжування: 1) Очки (`points`), 2) Різниця голів (`goalsScored - goalsConceded`), 3) Кількість забитих голів (`goalsScored`), 4) Кількість перемог (`wins`).

---

## 6. Domain Events

- `LeaderboardUpdatedDE` (userId, matchesPlayed, wins, goalsScored, goalsConceded, newRank)
