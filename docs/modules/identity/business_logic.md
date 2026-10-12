# Бізнес-логіка модуля Identity (Облікові записи та автентифікація)

> Модуль: `Identity`. Відповідає за реєстрацію, авторизацію користувачів, забезпечення унікальності нікнеймів, криптографічне хешування паролів та роботу віджета профілю у правому верхньому куті вікна гри **Unity (C#)**. Побудований на **простому C# ООП без використання Domain-Driven Design (DDD)**.

---

## 1. Призначення модуля

Модуль `Identity` забезпечує:
1. **Реєстрацію користувача:** введення пошти, нікнейму та пароля.
2. **Перевірку унікальності нікнейму:** перевірка імені без урахування регістру (`Alex` та `alex` вважаються однаковими, створення дублікатів блокується).
3. **Криптографічний захист паролів:** паролі хешуються за допомогою безпечного алгоритму (Argon2id або PBKDF2) з індивідуальною сіллю. Збереження паролів у відкритому вигляді заборонено.
4. **Авторизацію (Вхід):** перевірка введеного пароля за хешем та вхід у систему.
5. **Профіль гравця:** підтримка даних користувача (ID, нікнейм, пошта, ID обраного аніме-героя) для відображення у віджеті у правому верхньому куті вікна гри.

---

## 2. Моделі даних (C#)

```csharp
public class PlayerProfile
{
    public string Id { get; set; } = Guid.NewGuid().ToString();
    public string Email { get; set; } = string.Empty;
    public string Nickname { get; set; } = string.Empty;
    public string PasswordHash { get; set; } = string.Empty;
    public string PasswordSalt { get; set; } = string.Empty;
    public string ActiveCharacterId { get; set; } = "default_hero_01";
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
}
```

---

## 3. Сервіс автентифікації (`AuthService.cs`)

Звичайний C# клас із простими методами валідації та збереження:

```csharp
public class AuthService
{
    private readonly ISaveStorage _storage;

    public AuthService(ISaveStorage storage)
    {
        _storage = storage;
    }

    // Реєстрація нового користувача
    public bool Register(string email, string nickname, string password, out string errorMessage)
    {
        errorMessage = string.Empty;

        // 1. Валідація нікнейму
        if (string.IsNullOrWhiteSpace(nickname) || nickname.Length < 3 || nickname.Length > 20)
        {
            errorMessage = "Нікнейм має містити від 3 до 20 символів.";
            return false;
        }

        // 2. Перевірка унікальності без урахування регістру
        if (_storage.ExistsNickname(nickname.ToLowerInvariant()))
        {
            errorMessage = "Цей нікнейм уже зайнятий.";
            return false;
        }

        // 3. Валідація пароля
        if (string.IsNullOrEmpty(password) || password.Length < 8)
        {
            errorMessage = "Пароль має містити щонайменше 8 символів.";
            return false;
        }

        // 4. Генерація солі та хешування пароля (Argon2id / PBKDF2)
        string salt = PasswordHasher.GenerateSalt();
        string hash = PasswordHasher.HashPassword(password, salt);

        // 5. Створення профілю
        var profile = new PlayerProfile
        {
            Email = email.Trim(),
            Nickname = nickname.Trim(),
            PasswordHash = hash,
            PasswordSalt = salt
        };

        _storage.SaveProfile(profile);
        return true;
    }

    // Вхід у гру
    public PlayerProfile? Login(string login, string password, out string errorMessage)
    {
        errorMessage = string.Empty;

        var profile = _storage.FindByLogin(login.Trim().ToLowerInvariant());
        if (profile == null)
        {
            errorMessage = "Користувача не знайдено.";
            return null;
        }

        if (!PasswordHasher.VerifyPassword(password, profile.PasswordHash, profile.PasswordSalt))
        {
            errorMessage = "Невірний пароль.";
            return null;
        }

        return profile;
    }
}
```

---

## 4. Віджет профілю у правому верхньому куті

- При успішному вході `PlayerProfile` передається у компонент інтерфейсу `UserProfileWidget.cs` (Unity `MonoBehaviour`).
- У правому верхньому куті відображаються:
  - Нікнейм поточного гравця.
  - Аватарка обраного аніме-героя (завантажується за `ActiveCharacterId`).
  - Кнопка відкриття каталогу персонажів для зміни героя.
