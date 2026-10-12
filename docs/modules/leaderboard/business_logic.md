# Бізнес-логіка модуля Leaderboard (Таблиця результатів та статистика)

> Модуль: `Leaderboard`. Відповідає за підрахунок і збереження результатів матчів між друзями, формування турнірної таблиці та надання доступу до статистики (кількість перемог, поразок, нічиїх, забитих і пропущених голів) у **Unity (C#)**. Побудований на **простому C# ООП без використання Domain-Driven Design (DDD)**.

---

## 1. Призначення модуля

Модуль `Leaderboard` забезпечує:
1. **Агрегацію статистики:** накопичення показників кожного гравця після завершення серії пенальті:
   - Кількість зіграних матчів (`MatchesPlayed`).
   - Кількість перемог (`Wins`), поразок (`Losses`), нічиїх (`Draws`).
   - Кількість забитих голів (`GoalsScored`).
   - Кількість пропущених голів (`GoalsConceded`).
   - Різниця голів (`GoalDifference = GoalsScored - GoalsConceded`).
   - Рейтингові очки (`Points = Wins * 3 + Draws * 1`).
2. **Оновлення після матчу:** отримання результатів від `MatchController` після завершення 10 ударів.
3. **Відображення таблиці:** сортування записів у UI таблиці лідерів.
4. **Контроль цілісності (для лаби):** збереження статистики з перевірочним підписом HMAC-SHA256 для запобігання підробці очок у текстовому редакторі.

---

## 2. Модель даних (C#)

```csharp
public class LeaderboardEntry
{
    public string Nickname { get; set; } = string.Empty;
    public int MatchesPlayed { get; set; } = 0;
    public int Wins { get; set; } = 0;
    public int Losses { get; set; } = 0;
    public int Draws { get; set; } = 0;
    public int GoalsScored { get; set; } = 0;
    public int GoalsConceded { get; set; } = 0;

    // Властивості, що обчислюються автоматично
    public int GoalDifference => GoalsScored - GoalsConceded;
    public int Points => (Wins * 3) + (Draws * 1);
}
```

---

## 3. Сервіс таблиці лідерів (`LeaderboardService.cs`)

Звичайний C# клас для оновлення та сортування результатів:

```csharp
public class LeaderboardService
{
    private readonly List<LeaderboardEntry> _entries = new();
    private readonly ISaveStorage _storage;

    public LeaderboardService(ISaveStorage storage)
    {
        _storage = storage;
        _entries = _storage.LoadLeaderboard();
    }

    // Оновлення статистики після завершення матчу
    public void RecordMatchResult(string playerA, string playerB, MatchOutcome outcome, int scoreA, int scoreB)
    {
        var entryA = GetOrCreateEntry(playerA);
        var entryB = GetOrCreateEntry(playerB);

        // Оновлюємо ігри та голи
        entryA.MatchesPlayed += 1;
        entryA.GoalsScored += scoreA;
        entryA.GoalsConceded += scoreB;

        entryB.MatchesPlayed += 1;
        entryB.GoalsScored += scoreB;
        entryB.GoalsConceded += scoreA;

        // Оновлюємо результати матчу
        switch (outcome)
        {
            case MatchOutcome.PlayerAWon:
                entryA.Wins += 1;
                entryB.Losses += 1;
                break;
            case MatchOutcome.PlayerBWon:
                entryB.Wins += 1;
                entryA.Losses += 1;
                break;
            case MatchOutcome.Draw:
                entryA.Draws += 1;
                entryB.Draws += 1;
                break;
        }

        // Зберігаємо оновлені дані з HMAC-підписом
        _storage.SaveLeaderboard(_entries);
    }

    // Отримання відсортованого списку для UI
    public List<LeaderboardEntry> GetSortedLeaderboard()
    {
        return _entries
            .OrderByDescending(e => e.Points)
            .ThenByDescending(e => e.GoalDifference)
            .ThenByDescending(e => e.GoalsScored)
            .ToList();
    }

    private LeaderboardEntry GetOrCreateEntry(string nickname)
    {
        var entry = _entries.Find(e => e.Nickname.Equals(nickname, StringComparison.OrdinalIgnoreCase));
        if (entry == null)
        {
            entry = new LeaderboardEntry { Nickname = nickname };
            _entries.Add(entry);
        }
        return entry;
    }
}
```

---

## 4. Сценарій відображення у грі

1. Гравець у головному меню або після матчу відкриває вкладку «Таблиця лідерів».
2. `LeaderboardUI.cs` запитує у `LeaderboardService` відсортований список гравців.
3. Відображається таблиця з колонками: Ранг | Нікнейм | Ігри | В | Н | П | Забиті | Пропущені | Очки.
