# Technical Notes — Identity Module

## Криптографічні вимоги та оптимізація

### 1. Конфігурація Argon2id
- **Рекомендація NIST / OWASP:**
  ```php
  password_hash($rawPassword, PASSWORD_ARGON2ID, [
      'memory_cost' => 65536, // 64 MB
      'time_cost'   => 3,     // 3 iterations
      'threads'     => 1,     // 1 thread
  ]);
  ```
- **Захист від атак за часом (Timing Attacks):** Верифікація пароля здійснюється функцією `password_verify()`, яка виконує порівняння за постійний час, унеможливлюючи витік інформації через аналіз затримок відповіді.

### 2. Запобігання колізіям нікнеймів
- У реляційній базі створюється унікальний функціональний індекс:
  ```sql
  CREATE UNIQUE INDEX idx_users_nickname_lower ON users (LOWER(nickname));
  ```
- Це гарантує, що `Naruto`, `naruto` та `NARUTO` не можуть бути зареєстровані як різні користувачі, виключаючи імперсонацію та плутанину в таблиці лідерів.
