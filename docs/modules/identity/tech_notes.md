# Технічні нотатки — Identity Module (Unity C#)

## Технічні деталі реалізації у C#

### 1. Криптографічний захист паролів
Для лабораторної роботи використовується надійне хешування паролів із сіллю:
- **Алгоритм:** **Argon2id** (через бібліотеку `Konscious.Security.Cryptography` для .NET) або вбудований стандартний **PBKDF2** (`Rfc2898DeriveBytes` з 100,000+ ітерацій та SHA-256).
- **Сіль:** 16-байтний випадковий масив, згенерований через `System.Security.Cryptography.RandomNumberGenerator`.
- **Захист від атак за часом (Timing Attacks):** порівняння хешів здійснюється через `CryptographicOperations.FixedTimeEquals()`.

### 2. Забезпечення унікальності нікнеймів
- Для виключення колізій і шахрайства регістр нікнеймів нормалізується за допомогою `string.ToLowerInvariant()`.
- У пам'яті використовується кеш `Dictionary<string, PlayerProfile>(StringComparer.OrdinalIgnoreCase)`.
- У базі даних SQLite поле зберігається з обмеженням `COLLATE NOCASE` або створюється індекс `CREATE UNIQUE INDEX idx_users_nick_lower ON users (LOWER(nickname))`.

### 3. Збереження облікових записів
- Профілі зберігаються у локальній базі `SQLite` або файлі `accounts.json` у каталозі `Application.persistentDataPath`.
- Файл підписується HMAC-SHA256 для запобігання несанкціонованій модифікації.
