# ====== CHARACTER MODULE BUSINESS LOGIC ======

> Bounded Context: `Character`. Відповідає за каталог аніме-франшиз (тайтлів), перелік доступних аніме-персонажів та надання інтерфейсу вибору/зміни персонажа користувачем у вікні профілю.

---

## 1. Purpose

Модуль `Character` володіє:
- Зберіганням та структуризацією аніме-тайтлів (`AnimeTitle`) — наприклад, «Blue Lock», «Captain Tsubasa», «Inazuma Eleven», «Kuroko no Basket», «Naruto».
- Зберіганням каталогу персонажів (`Character`), що належать конкретному тайтлу (ім'я, візуальні ассети, аватарки, звукові репліки).
- Наданням ієрархічного вибору персонажа: користувач спочатку обирає тайтл зі списку, звідти розгортається список персонажів, і користувач обирає потрібного героя.
- Механізмом зміни активного персонажа в налаштуваннях профілю гравця у будь-який момент поза матчем.

**NOT here:**
- Збереження профілю акаунта користувача (належить `Identity`).
- Ігрова анімація та розрахунок ударів (належить `Gameplay`).
- Статистика гравця (належить `Leaderboard`).

---

## 2. Entities

| Entity | Basic Fields | Description | Invariants |
|---|---|---|---|
| `AnimeTitle` *(Aggregate Root)* | `uuid`, `name`, `slug`, `description`, `charactersCount`, `isActive` | Франшиза/тайтл аніме. | - `uuid` іммутабельний.<br>- `name` та `slug` унікальні.<br>- Не може бути активованим без жодного персонажа. |
| `Character` *(Entity, owned by `AnimeTitle`)* | `uuid`, `titleId`, `name`, `avatarUrl`, `fullArtUrl`, `isDefault`, `isAvailable` | Конкретний аніме-персонаж. | - `titleId` вказує на валідний тайтл.<br>- `name` не порожнє.<br>- Хоча б один персонаж у системі помічений як `isDefault == true` (для нових реєстрацій). |

---

## 3. Status Lifecycle

Для `AnimeTitle`: `DRAFT` $\rightarrow$ `ACTIVE` $\rightarrow$ `ARCHIVED`.
Для `Character`: `AVAILABLE` $\rightarrow$ `LOCKED` (якщо розблоковується досягненнями) $\rightarrow$ `DEPRECATED`.

---

## 4. Key Flows

### Флоу вибору аніме-персонажа:
1. Гравець клікає на аватар профілю у правому верхньому куті вікна програми.
2. Клієнт надсилає запит `GetAnimeCatalogAQ`.
3. Бекенд повертає список активних тайтлів з попереднім переглядом.
4. Гравець клікає на бажаний тайтл (наприклад, «Blue Lock»).
5. Інтерфейс викачує/розгортає список персонажів даного тайтлу (Ісагі, Бачіра, Рін, Нагі).
6. Гравець клікає на персонажа та натискає «Обрати персонажа» $\rightarrow$ відправляється `SelectActiveCharacterAC`.
7. `CharacterAvailabilityDPolicy` підтверджує валідність та доступність персонажа.
8. Профіль гравця оновлюється, новий персонаж одразу відображається у віджеті профілю у правому верхньому куті та буде використовуватися у наступних 1v1 матчах.

---

## 5. Value Objects

| Value Object | Опис | Інваріанти |
|---|---|---|
| `TitleIdVO` | Ідентифікатор тайтлу | UUID v4 |
| `CharacterIdVO` | Ідентифікатор персонажа | UUID v4 |
| `CharacterNameVO` | Ім'я персонажа | Рядок 1..50 символів, очищений від HTML |
| `AssetUrlVO` | Посилання на зображення персонажа | Валідний HTTPS URL або шлях до локального ассету |

---

## 6. Domain Policies

- `CharacterAvailabilityDPolicy`: Перевіряє, що персонаж доступний у каталозі, не заблокований і належить активному тайтлу.
- `DefaultCharacterExistsDPolicy`: Гарантує, що в системі завжди існує щонайменше один загальнодоступний базовий персонаж за замовчуванням.

---

## 7. Domain Events

- `ActiveCharacterChangedDE` (userId, previousCharacterId, newCharacterId, selectedAt)
- `NewCharacterAddedDE` (characterId, titleId, name)
