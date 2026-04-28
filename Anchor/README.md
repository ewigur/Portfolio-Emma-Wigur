<a name="TOP"></a>

_____________________________________________________________________________________

## A Selection of Contributions

### 🌍 Localization Work

One of the bigger systems I worked on in this project was **localization**.

The main challenge was that the game had already been in development for months and relied heavily on text embedded directly in code\
often through override functions that generated the descriptive text content dynamically.

To address this, I:
```
- Went through scripts containing hardcoded text  
- Refactored them into reusable localization functions  
- Replaced inline strings with localized entries  
```
This allowed the project to scale across multiple languages without rewriting core systems.

***See examples below***
</details>

<details>
<summary>Damage Action : Snippet</summary>
<br>
  
_This is how the description for dealing damage looked before localization._
```ruby

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
```
_The section below is what was shown in description box, which was limited to english._
```ruby
        return $"Deals {dmgNr} <color=#FF0000>damage</color> to the enemy player";
        return Damage[(int)fish.Tier].ToString();
    }

```

</details>

<details>
<summary>Localized Damage Action : Snippet</summary>
<br>
  

_I used the already established override functions and added a helper in the
class from which the action inherited from._
```ruby
 public override string GetDescription()
    {
        return GetLocalizedDescription(Loc(("damage", "X")));
    }

    public override string GetDescription(Fish fish, List<ActionInstance.FieldOverride> fieldOverrides = null)
    {
        string dmgNr = GetDamageValueForFish(fish, fieldOverrides);
        return GetLocalizedDescription(Loc(("damage", dmgNr)));
    }

    public override string GetDescription(EnchantType enchant, ERarity rarity, List<ActionInstance.FieldOverride> fieldOverrides = null)
    {
        string dmgNr = GetDamageValueForEnchant(enchant, rarity, fieldOverrides);
        return GetLocalizedDescription(Loc(("damage", dmgNr)));
    }
```
_This is from the parent class of which the snippets above get their localization from._
```ruby
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
_I put this debugging condition here to make it easier to see what was missing 
when testing localization for the many different scripts deriving from this class
It saved me a lot of headache many times!_

```ruby

        if (string.IsNullOrEmpty(result))
        {
            Logger.LogWarning($"Localized string was empty for key '{LocalizationKey}' in table '{LocalizationTable}'");
            return $"[{LocalizationKey}]";
        }

        return result;
    }
```

</details>

***Visual Result***\
![](https://github.com/ewigur/Portfolio-Emma-Wigur/blob/main/Anchor/GIFs/LocShow.gif)
_____________________________________________________________________________________

### *Developed by*

![](https://github.com/ewigur/Portfolio-Emma-Wigur/blob/main/ThumbNails/ImperialPlaygroundsWhiteLogoFramed.png)
_____________________________________________________________________________________


[RETURN TO TOP](#TOP)
             <a name="TOP"></a>  

