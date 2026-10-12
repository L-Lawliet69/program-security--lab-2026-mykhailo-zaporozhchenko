# Бізнес-логіка модуля Character (Каталог аніме-персонажів)

> Модуль: `Character`. Відповідає за зберігання каталогу аніме-тайтлів, перелік доступних аніме-персонажів та надання інтерфейсу вибору/зміни персонажа користувачем у вікні профілю в **Unity (C#)**. Побудований на **простому C# ООП без використання Domain-Driven Design (DDD)**.

---

## 1. Призначення модуля

Модуль `Character` забезпечує:
1. **Зберігання аніме-тайтлів:** список франшиз («Blue Lock», «Captain Tsubasa», «Inazuma Eleven», «Kuroko no Basket» тощо).
2. **Зберігання персонажів:** перелік героїв конкретного аніме з іменами, іконками для UI та спрайтами для відображення на полі.
3. **Ієрархічний вибір:** гравець відкриває каталог у налаштуваннях профілю, обирає тайтл, розгортає персонажів і обирає свого героя.
4. **Прив'язку до профілю:** обраний герой закріплюється за поточним користувачем, відображається у віджеті у правому верхньому куті та використовується в матчах 1v1.

---

## 2. Моделі даних (C#)

```csharp
public class AnimeTitle
{
    public string Id { get; set; } = string.Empty;
    public string Name { get; set; } = string.Empty;
    public string Description { get; set; } = string.Empty;
    public List<AnimeCharacter> Characters { get; set; } = new();
}

public class AnimeCharacter
{
    public string Id { get; set; } = string.Empty;
    public string TitleId { get; set; } = string.Empty;
    public string Name { get; set; } = string.Empty;
    public string SpriteResourcePath { get; set; } = string.Empty;
    public bool IsDefault { get; set; } = false;
}
```

Моделі персонажів у Unity можуть також зберігатися як звичайні **ScriptableObjects** (`[CreateAssetMenu]`) або завантажуватися з JSON/SQLite конфігурації.

---

## 3. Каталог персонажів (`CharacterCatalog.cs`)

Звичайний C# сервіс для роботи з персонажами:

```csharp
public class CharacterCatalog
{
    private readonly List<AnimeTitle> _titles = new();

    public CharacterCatalog()
    {
        InitializeCatalog();
    }

    private void InitializeCatalog()
    {
        var blueLock = new AnimeTitle { Id = "bluelock", Name = "Blue Lock" };
        blueLock.Characters.Add(new AnimeCharacter 
        { 
            Id = "isagi_yoichi", 
            TitleId = "bluelock", 
            Name = "Isagi Yoichi", 
            SpriteResourcePath = "Sprites/Characters/isagi",
            IsDefault = true 
        });
        blueLock.Characters.Add(new AnimeCharacter 
        { 
            Id = "bachira_meguru", 
            TitleId = "bluelock", 
            Name = "Bachira Meguru", 
            SpriteResourcePath = "Sprites/Characters/bachira" 
        });

        var tsubasa = new AnimeTitle { Id = "tsubasa", Name = "Captain Tsubasa" };
        tsubasa.Characters.Add(new AnimeCharacter 
        { 
            Id = "ozora_tsubasa", 
            TitleId = "tsubasa", 
            Name = "Tsubasa Ozora", 
            SpriteResourcePath = "Sprites/Characters/tsubasa" 
        });

        _titles.Add(blueLock);
        _titles.Add(tsubasa);
    }

    public List<AnimeTitle> GetAllTitles() => _titles;

    public AnimeCharacter? GetCharacterById(string id)
    {
        foreach (var title in _titles)
        {
            var character = title.Characters.Find(c => c.Id == id);
            if (character != null) return character;
        }
        return null;
    }

    public AnimeCharacter GetDefaultCharacter()
    {
        return _titles[0].Characters[0];
    }
}
```

---

## 4. Сценарій вибору героя у грі

1. Гравець натискає на свій аватар у правому верхньому куті вікна гри.
2. Відкривається вікно вибору героїв (UI Canvas).
3. Відображаються плашки аніме-тайтлів.
4. Гравець клікає на тайтл (наприклад, «Blue Lock») $\rightarrow$ розгортається сітка з картками персонажів даного тайтлу.
5. Гравець обирає персонажа та натискає «Вибрати».
6. У поточному `PlayerProfile` оновлюється `ActiveCharacterId`.
7. У правому верхньому куті оновлюється аватарка, і вибір зберігається у файл сейву.
