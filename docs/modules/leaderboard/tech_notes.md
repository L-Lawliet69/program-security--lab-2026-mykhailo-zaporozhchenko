# Technical Notes — Leaderboard Module

## Оптимізація та забезпечення цілісності

1. **Ідемпотентність і захист від дублювання:**
   - Для запобігання повторному нарахуванню результатів при ретраях повідомлень створюється таблиця оброблених івентів `processed_match_events(match_id UUID PRIMARY KEY, processed_at TIMESTAMP)`.
   - Запис у таблицю та оновлення статистики виконуються в одній транзакції PostgreSQL.

2. **Швидкісні вибірки (High Read Throughput):**
   - Для перших 100 позицій глобальної таблиці лідерів використовується Redis Sorted Set (`ZREVRANGEBYSCORE leaderboard:points`), що дозволяє повертати топ гравців за $O(\log(N) + M)$ практично миттєво.
   - Повна таблиця у реляційній БД індексується:
     ```sql
     CREATE INDEX idx_leaderboard_rank ON leaderboard_entries (points DESC, (goals_scored - goals_conceded) DESC, goals_scored DESC);
     ```
