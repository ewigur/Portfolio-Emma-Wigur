<a name="TOP"></a>

---
<p align="center">
Anchor's Lament is a multiplayer roguelite strategy game where team-building meets rhythm-based combat, where everything moves to the beat of an anchor.
</p>

---
## Contributions
### 🌐 Localization Work

One of the larger systems I worked on in this project was **localization**.

The main challenge was that the game had already been in development for months and relied heavily on text embedded directly in code—often through override functions that dynamically generated descriptive content.

**My contributions:**
- Refactored scripts containing hardcoded text  
- Introduced reusable localization functions  
- Replaced inline strings with localized entries  

This enabled scalable multi-language support without modifying core systems.

<p align="center">
  <strong><em>See examples below</em></strong>
</p>

<details>
<summary><strong>Damage Action (Before Localization)</strong></summary>
<br>

_Description logic before localization (hardcoded & English-only):_

```csharp
if (overrides != null)
{
    if (ovr.FieldName == "Damage")
    foreach (var ovr in overrides)
    {
        TieredInt ar = (TieredInt)ovr.GetValue();
        dmgNr = ar[(int)fish.Tier].ToString();
        if (ovr.FieldName == "Damage")
        {
            TieredInt ar = (TieredInt)ovr.GetValue();
            return ar[(int)fish.Tier].ToString();
        }
    }
    else { continue; }
}

return $"Deals {dmgNr} <color=#FF0000>damage</color> to the enemy player";
return Damage[(int)fish.Tier].ToString();
```
</details>

<details> <summary><strong>Localized Damage Action</strong></summary> <br>

Localized implementation using shared helpers:
```csharp
public override string GetDescription()
{
    return GetLocalizedDescription(Loc(("damage", "X")));
}

public override string GetDescription(Fish fish, List<ActionInstance.FieldOverride> fieldOverrides = null)
{
    string dmgNr = GetDamageValueForFish(fish, fieldOverrides);
    return GetLocalizedDescription(Loc(("damage", dmgNr)));
}
```
Parent class localization handler:
```csharp
protected string GetLocalizedDescription(params object[] args)
{
    if (!LocalizationSettings.InitializationOperation.IsDone ||
        LocalizationSettings.SelectedLocale == null)
    {
        Logger.LogWarning($"Localization not ready for {LocalizationKey}");
        return $"[{LocalizationKey}]";
    }

    string result = LocalizationSettings.StringDatabase.GetLocalizedString(
        LocalizationTable,
        LocalizationKey,
        args
    );
```
Debug fallback for missing entries:
```csharp
if (string.IsNullOrEmpty(result))
{
    Logger.LogWarning($"Localized string was empty for key '{LocalizationKey}'");
    return $"[{LocalizationKey}]";
}

return result;
}
```
</details>

<p align="center"> <img src="https://github.com/ewigur/Portfolio-Emma-Wigur/blob/main/Anchor/GIFs/LocShow.gif" width="500"/> </p>

---

### ♨️ Events

In between combat, a series of themed events appear. I worked on two of them - one of which I developed from scratch.

🍲 Shell's Kitchen

An existing event that required a UI overhaul and expanded gameplay.

My contributions:

- Created a consumable that restores HP and grants a bonus item
- Designed the bonus item to buff unit stats based on actions
- Collaborated with the lead artist to update visual assets
- Integrated new functionality into an existing system

  <p align="center">
  <strong><em>See example below</em></strong>
</p>

<details> <summary><strong>Food Consumable + adding bonus item</strong></summary> <br>
  
_The function that increases player health,\
  in addition to adding the bonusitem to player's inventory._
  ```csharp
    public override void Eat()
    {
        for (int i = 0; i < itemsToAdd; i++)
        {
            bonusItem.AddToInventory(bonusItem);
        }

        GameplayManager.Instance.GetTeam().MaxHealth += HealthToGain;
        GameplayManager.Instance.statBars.MaxHealth = GameplayManager.Instance.GetTeam().MaxHealth;
        GameplayManager.Instance.statBars.CurrentHealth = GameplayManager.Instance.GetTeam().MaxHealth;

        AudioManagement.Instance?.PlayEffectSound("EAT");
        GameplayManager.Instance.statBars.UpdateBars();
    }
  ```
  </details>
<p align="center"> <img src="https://github.com/ewigur/Portfolio-Emma-Wigur/blob/main/Anchor/GIFs/Shells.gif" width="500"/> </p>

🏷️ Tag Shop

A new event I developed as an extension of the core shop system.

Players can filter units by tags, allowing more strategic team building.

My contributions:

- Designed a new event flow with multiple shop paths
- Implemented randomized tag selection via UI
- Connected spawning logic to tag-based filtering
- Built layered UI to support the event flow

  <p align="center">
  <strong><em>See example below</em></strong>
</p>

<details> <summary><strong>Randomize Tag Options</strong></summary> <br>

_The function below is used to generate unique tag options while filtering out invalid ones._
```csharp
void RandomizeTags()
{
    chooseTagOverlay.SetActive(true);
    chooseShopOverlay.SetActive(false);

    EFishTag[] allTags = (EFishTag[])System.Enum.GetValues(typeof(EFishTag));

    // Filter out unwanted tags
    allTags = System.Array.FindAll(allTags, t =>
        t != EFishTag.NONE &&
        t != EFishTag.BIG &&
        t != EFishTag.SHARK &&
        t != EFishTag.COLD &&
        t != EFishTag.QUICK
    );

    // Pick 3 unique tags
    int indexA = Random.Range(0, allTags.Length);
    int indexB, indexC;

    do { indexB = Random.Range(0, allTags.Length); }
    while (indexB == indexA);

    do { indexC = Random.Range(0, allTags.Length); }
    while (indexC == indexA || indexC == indexB);

    EFishTag[] chosenTags = { allTags[indexA], allTags[indexB], allTags[indexC] };

    for (int i = 0; i < 3; i++)
    {
        int index = i;

        chooseTagText[i].text = L.Get("TagKeywords", chosenTags[i].ToString());
        chooseTagButton[i].interactable = true;

        chooseTagButton[i].onClick.RemoveAllListeners();
        chooseTagButton[i].onClick.AddListener(() =>
        {
            ActivateTagShop(chosenTags[index]);
        });
    }  
}
```
_The chosen tags is now used as active tags, and the shop will initialize accordningly_
```csharp
void InitializeShop(EFishTag tag)
{
        activeTags = tag;
        consumablesSpawned = 0;

        List<FishType> fishPool = new(ShopList.fishList);
        fishPool.RemoveAll(fish => fish == null);

        if (isTagShopActive)
        {
            fishPool.RemoveAll(fish => (fish.Tags & tag) == 0);
        }

        List<Consumable> consumablePool = new(ShopList.consumableList);

        int totalAvailable = fishPool.Count + consumablePool.Count;
        int itemsToSpawn = Mathf.Min(maxItems, totalAvailable);

        for (int i = 0; i < itemsToSpawn; i++)
        {
            SpawnRandomItem(fishPool, consumablePool);
        }

        UpdateSlotInteractivity(0, GameplayManager.Coins);
}
```
  </details>
<p align="center"> <img src="https://github.com/ewigur/Portfolio-Emma-Wigur/blob/main/Anchor/GIFs/TagShop.gif" width="400"/> </p>

_____________________________________________________________

### ⚔️ Combat Stats

I developed a system that tracks and aggregates damage output across combat encounters, both per round and across an entire run.

This allows players to analyze performance, identify their strongest units, and better understand the impact of different strategies over time.

My contributions:

- Built a system that gathers all active units at combat start
- Implemented real-time tracking of per-unit combat contributions
- Designed separation between round-based resets and persistent run data
- Created the first iteration of the UI to visualize combat statistics

  <p align="center">
  <strong><em>See examples below</em></strong>
</p>

<details> <summary><strong>Realtime event recordning</strong></summary> <br>
  
***The system is built around a clear separation between transient combat data and persistent run data,\
allowing stats to reset each round while still contributing to long-term analysis.***
  
_Separate dictionaries for player and enemy teams_
```csharp
private static Dictionary<FishInstance, FishStats> localRoundStats =
    new Dictionary<FishInstance, FishStats>();

private static Dictionary<FishInstance, FishStats> enemyRoundStats =
    new Dictionary<FishInstance, FishStats>();
```
_Clearing stats between rounds_
```csharp
public static void ClearStats()
{
    localRoundStats.Clear();
    enemyRoundStats.Clear();
}
```
_Recording stats in real time_
```csharp
public static void Record(FishInstance fish, EFishActionCallback type, float amount, TeamInstance team)
{
    if (fish == null || amount <= 0) return;

    var dict = GetDict(team);
    int intAmount = (int)amount;

    RecordToDict(dict, fish, type, intAmount);

    if (team == CombatManager.Instance.LocalTeam)
        PostGameStats.RecordTotalStats(fish.fromFish, type, intAmount);
}
```
_Flexible stat application system_
```csharp
internal static void ApplyStat(FishStats stats, EFishActionCallback type, int amount)
{
    switch (type)
    {
        case EFishActionCallback.ATTACK: stats.damage += amount; break;
        case EFishActionCallback.HEAL: stats.healing += amount; break;
        case EFishActionCallback.CRIT: stats.crit += 1; break;
        case EFishActionCallback.DEAL_POISON: stats.poison += amount; break;
        case EFishActionCallback.DEAL_INK: stats.ink += amount; break;
        case EFishActionCallback.ADD_ARMOR: stats.armor += amount; break;
        case EFishActionCallback.GIVE_IGNITE: stats.fire += amount; break;
        case EFishActionCallback.GAIN_GOLD: stats.gold += amount; break;
    }
}
```
_Persistent post-game stat tracking_
```csharp
public static void RecordTotalStats(Fish fish, EFishActionCallback type, int amount)
{
    if (!postGameStats.TryGetValue(fish, out FishStats stats))
    {
        stats = new FishStats();
        postGameStats.Add(fish, stats);
    }

    SwitchCases(stats, type, amount);
}
```
_Run-wide stat aggregation_
```csharp
public static FishStats CalculateTotalStats()
{
    FishStats totalStats = new();

    foreach (FishStats value in postGameStats.Values)
    {
        totalStats.healing += value.healing;
        totalStats.damage += value.damage;
        totalStats.crit += value.crit;
        totalStats.ink += value.ink;
        totalStats.poison += value.poison;
        totalStats.armor += value.armor;
        totalStats.fire += value.fire;
    }

    return totalStats;
}
```
_Populating stats in the UI_
```csharp
public void ReportStats()
{
    localStatsDisplay.PopulateStats(RoundStatManager.GetLocalStats());
    enemyStatsDisplay.PopulateStats(RoundStatManager.GetEnemyStats());
}
```
  </details>

  | First Iteration (Programmer UI) | Final Stat Display _(Visuals by UX designer)_ |
| ------------- | ------------- |
| ![](https://github.com/ewigur/Portfolio-Emma-Wigur/blob/main/Anchor/GIFs/OldStat.gif)  | ![](https://github.com/ewigur/Portfolio-Emma-Wigur/blob/main/Anchor/GIFs/StatDisplay.gif) |
_____________________________________________________________

👥 Developed by
<p align="center"> <img src="https://github.com/ewigur/Portfolio-Emma-Wigur/blob/main/ThumbNails/ImperialPlaygroundsWhiteLogoFramed.png" width="300"/> </p>
<p align="center"> <a href="#TOP"><strong>↑ Return to Top</strong></a> </p>
