<a name="TOP"></a>

---
<p align="center">
Anchor's Lament is a multiplayer roguelite strategy game where team-building meets rhythm-based combat,\
where everything moves to the beat of an anchor.
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

<p align="center"> <img src="https://github.com/ewigur/Portfolio-Emma-Wigur/blob/main/Anchor/GIFs/LocShow.gif" width="600"/> </p>

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
<p align="center"> <img src="https://github.com/ewigur/Portfolio-Emma-Wigur/blob/main/Anchor/GIFs/Shells.gif" width="500"/> </p>

🏷️ Tag Shop

A new event I developed as an extension of the core shop system.

Players can filter units by tags, allowing more strategic team building.

My contributions:

- Designed a new event flow with multiple shop paths
- Implemented randomized tag selection via UI
- Connected spawning logic to tag-based filtering
- Built layered UI to support the event flow
<p align="center"> <img src="https://github.com/ewigur/Portfolio-Emma-Wigur/blob/main/Anchor/GIFs/TagShop.gif" width="500"/> </p>

_____________________________________________________________

👥 Developed by
<p align="center"> <img src="https://github.com/ewigur/Portfolio-Emma-Wigur/blob/main/ThumbNails/ImperialPlaygroundsWhiteLogoFramed.png" width="300"/> </p>
<p align="center"> <a href="#TOP"><strong>↑ Return to Top</strong></a> </p>
