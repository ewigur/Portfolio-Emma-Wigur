<a name="TOP"></a>

<p align="center"> <img src="https://github.com/ewigur/Portfolio-Emma-Wigur/blob/main/ThumbNails/Anchorslamentlogo%20(2).png" width="500"/> </p>

---
<p align="center">
Anchor's Lament is a multiplayer roguelite strategy game where team-building meets rhythm-based combat, where everything moves to the beat of an anchor.
</p>

---
## Contributions

_In each of these tasks I focused on building maintainable and designer-friendly systems rather than hardcoded solutions._

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

<p align="center"> <a href="#TOP"><strong>↑ Return to Top</strong></a> </p>

_____________________________________________________________

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

<p align="center"> <a href="#TOP"><strong>↑ Return to Top</strong></a> </p>

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

<details> <summary><strong>Realtime event recordning</strong></summary>
  
***The system is built around a clear separation between transient combat data and persistent run data,\
allowing stats to reset each round while still contributing to long-term analysis.***

<hr width="30%" align="left">

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

<p align="center"> <a href="#TOP"><strong>↑ Return to Top</strong></a> </p>

_____________________________________________________________

### 🧭 Tutorial

The game already included a basic tutorial system, where the parrot “Cracky” would occasionally appear to provide tips and guidance.\
However, the team wanted a more structured and comprehensive onboarding experience—especially for first-time players.

I was tasked with designing and implementing this improved tutorial. This involved reworking the existing “Cracky” system,\
as well as building supporting systems to track player progression and dynamically guide the player through key gameplay elements.

My contributions:

- Expanded the existing tutorial system to adapt based on player state (first-time vs. returning players)
- Designed and implemented a dynamic UI that highlights relevant gameplay elements in sync with tutorial instructions
- Developed a data-driven setup using Scriptable Objects to define tutorial steps and content

  <p align="center">
  <strong><em>See examples below</em></strong>
</p>

<details> <summary><strong>Tutorial System Implementation</strong></summary> <br>

_These snippets highlight the core architecture behind the tutorial system,\
including data-driven design, event handling, and dynamic UI highlighting._

<hr width="30%" align="left">

_Defines tutorial content as data rather than hardcoded logic.\
 This allows designers to configure tutorial flows directly in the editor\
 and makes the system scalable and easy to extend._
```csharp
[CreateAssetMenu(menuName = "Tutorials/Tutorial Content")]
public class TutorialContent : ScriptableObject
{
    public EEvent Event;
    public TutorialStep[] steps;
    public StringKey returningPlayerDialogue;

    public bool useGreeting = false;

    [HideInInspector] public bool autoSpawnCracky;
    [HideInInspector] public bool prepPhase = false;
}
```
_Converts a list of tutorial content into a dictionary for fast runtime lookup.\
This avoids expensive searches and ensures O(1) access when reacting to gameplay events._
```csharp
private void Awake()
{
    lookUp = tutorials.ToDictionary(tutorial => tutorial.Event);
}

public TutorialContent Get(EEvent evt) =>
    lookUp.TryGetValue(evt, out var tutorial) ? tutorial : null;
```
_Registers UI elements using string IDs instead of direct references.\
This decouples the tutorial system from specific UI objects,\
making it reusable across scenes and layouts._
```csharp
private void Awake()
{
    HighlightRegistry.Register(highlightID, GetComponent<RectTransform>(), isSquare);
}

private void OnDestroy()
{
    HighlightRegistry.Unregister(highlightID);
}
```
_Dynamically resolves and highlights UI elements based on IDs.\
This allows tutorial steps to target UI without hard references,\
and supports multiple highlights per step._
```csharp
public void HighlightByIDs(string[] ids)
{
    ClearHoles();

    if (ids == null || ids.Length == 0)
        return;

    foreach (var id in ids)
    {
        RectTransform target = HighlightRegistry.Get(id);
        bool isSquare = HighlightRegistry.IsSquare(id);

        if (target != null)
        {
            CreateHole(target, isSquare);
        }
        else
        {
            Debug.LogWarning($"No highlight target found for ID: {id}");
        }
    }
}
```
_Matches highlight masks to UI elements by converting world-space corners\
into canvas-local positions. Handles different resolutions and layouts._
```csharp
private void MatchToTarget(RectTransform hole, RectTransform target)
{
    Vector3[] worldCorners = new Vector3[4];
    target.GetWorldCorners(worldCorners);

    Vector2 min = RectTransformUtility.WorldToScreenPoint(canvas.worldCamera, worldCorners[0]);
    Vector2 max = RectTransformUtility.WorldToScreenPoint(canvas.worldCamera, worldCorners[2]);

    RectTransformUtility.ScreenPointToLocalPointInRectangle(
        maskContainer,
        min,
        canvas.worldCamera,
        out var localMin
    );

    RectTransformUtility.ScreenPointToLocalPointInRectangle(
        maskContainer,
        max,
        canvas.worldCamera,
        out var localMax
    );

    Vector2 size = localMax - localMin;
    Vector2 center = (localMax + localMin) / 2f;

    hole.anchoredPosition = center;
    hole.sizeDelta = new Vector2(Mathf.Abs(size.x), Mathf.Abs(size.y));
}
```
_Entry point for tutorial logic triggered by gameplay events.\
Adapts behavior depending on whether the player is new or returning._
```csharp
public void HandleEvent(EEvent evt)
{
    var content = database.Get(evt);
    if (content == null)
        return;

    activeContent = content;
    currentSteps = content.steps;
    currentStepIndex = 0;

    if (!useTutorial)
    {
        TryShowReturningGreetingOnce(evt, content);

        if (content.autoSpawnCracky)
        {
            parrotGuide.ShowDialogue(
                StringLookup.GetStrings(content.returningPlayerDialogue),
                true
            );
        }
    }
    else
    {
        spawnButton.interactable = false;

        if (currentSteps != null && currentSteps.Length > 0)
            ShowCurrentStep();
    }
}
```
  </details>
<p align="center"> <img src="https://github.com/ewigur/Portfolio-Emma-Wigur/blob/main/Anchor/GIFs/Tutorial.gif" width="500"/> </p>

<p align="center"> <a href="#TOP"><strong>↑ Return to Top</strong></a> </p>

_____________________________________________________________

### 🛡️ Resistance : Mechanic and item

Some_Txt

My contributions:

- Expl
- Expl
- Expl
- Expl

  <p align="center">
  <strong><em>See examples below</em></strong>
</p>

<details> <summary><strong>Resistance Mechanic</strong></summary>
  
***Expl***

<hr width="30%" align="left">

__
```csharp

```
__
```csharp

```
__
```csharp

```
__
```csharp

```
__
```csharp

```
__
```csharp

```
__
```csharp

```
  </details>

  | Mechanic in Combat | Resistance Item : _Prism_|
| ------------- | ------------- |
| ![](https://github.com/ewigur/Portfolio-Emma-Wigur/blob/main/Anchor/GIFs/Resistance.gif)  | ![](https://github.com/ewigur/Portfolio-Emma-Wigur/blob/main/Anchor/GIFs/item_prism.png) |

_____________________________________________________________

👥 Developed by
<p align="center"> <img src="https://github.com/ewigur/Portfolio-Emma-Wigur/blob/main/ThumbNails/ImperialPlaygroundsWhiteLogoFramed.png" width="300"/> </p>

<p align="center"> <a href="#TOP"><strong>↑ Return to Top</strong></a> </p>
