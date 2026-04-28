<a name="TOP"></a>

_____________________________________________________________________________________

## A Selection of Contributions

**LOCALIZATION**\
One of the bigger jobs I did on this game was Localization. I had never worked with translations in games before, so th first thing I did was to research the different ways to localize a game with Unity.\
Unity does have a great tool for Localizing games, and had I started a fresh I would probably use the same setup thoughout the game. The main big thing in this situation was that the game had already\
been developed for months, and relied heavily on descriptive text that for the most part was nested into code, using override functions to determine what the text blocks would show while hovering over game objects.\
To tackle this I went through each script with hardcoded text and created functions for localizing the text - using key value pairs to match the action with the Loclaization Tables created for that purpose.

![](https://github.com/ewigur/Portfolio-Emma-Wigur/blob/main/Anchor/GIFs/LocShow.gif)

</details>

<details>
<summary>BoostPerAdjacentWithTag</summary>
<br>
  
```ruby
public class FA_BoostPerAdjacentWithTag : FishAction
{
    [ActionSetting] public EFishTag TagToCheck;
    [ActionSetting] public StatToBuff StatToBoost;
    [ActionSetting] EAdjacentMode adjacentMode;
    [ActionSetting] public TieredInt BoostAmount;

    public override void Initialize(TeamInstance team, FishInstance fish)
    {
        base.Initialize(team, fish);

        int stat = BoostAmount[(int)fish.Tier];
        foreach (FishInstance _fish in fish.GetAdjacentFishes())
        {
            if (stat > 0 && _fish.HasTag(TagToCheck))
            {
                switch (StatToBoost)
                {
                    case StatToBuff.damage:
                        fish.BonusDamage += stat;
                        fish.Damage += stat;
                        break;
                    case StatToBuff.heal:
                        fish.BonusHealing += stat;
                        fish.Healing += stat;
                        break;
                    case StatToBuff.armor:
                        fish.BonusBlock += stat;
                        fish.Block += stat;
                        break;
                    case StatToBuff.crit:
                        fish.BaseCritChance += stat;
                        break;
                }
            }
        }
    }

    public override string GetDescription()
    {
        return GetLocalizedDescription(
            Loc(
                ("boost", GetLocalizedBoostStat(StatToBoost)),
                ("value", "X"),
                ("tag", StringLookup.GetTag(TagToCheck))
            )
        );
    }
    public override string GetDescription(Fish fish, List<ActionInstance.FieldOverride> fieldOverrides = null)
    {
        List<ActionInstance.FieldOverride> overrides = fieldOverrides ?? GetFieldOverrides(fish, this);

        EFishTag selectedTag = TagToCheck;
        StatToBuff selectedStat = StatToBoost;
        string valueStr = null;

        foreach (var ovr in fieldOverrides)
        {
            if (ovr.FieldName == "TagToCheck")
            {
                selectedTag = (EFishTag)ovr.GetValue();
            }
            else if (ovr.FieldName == "BoostAmount")
            {
                TieredInt ar = (TieredInt)ovr.GetValue();
                valueStr = ar[(int)fish.Tier].ToString();
            }
            else if (ovr.FieldName == "StatToBoost")
            {
                selectedStat = (StatToBuff)ovr.GetValue();
            }
        }

        if (string.IsNullOrEmpty(valueStr))
            valueStr = BoostAmount[(int)fish.Tier].ToString();

        return GetLocalizedDescription(
            Loc(
                ("boost", GetLocalizedBoostStat(selectedStat)),
                ("value", valueStr),
                ("tag", StringLookup.GetTag(selectedTag))
            )
        );
    }

    public override string GetDescription(EnchantType enchantType, ERarity rarity, List<ActionInstance.FieldOverride> fieldOverrides = null)
    {
        List<ActionInstance.FieldOverride> overrides = fieldOverrides ?? GetFieldOverrides(enchantType, this);

        EFishTag selectedTag = TagToCheck;
        StatToBuff selectedStat = StatToBoost;
        string valueStr = null;

        foreach (var ovr in overrides)
        {
            if (ovr.FieldName == "TagToCheck")
            {
                selectedTag = (EFishTag)ovr.GetValue();
            }
            else if (ovr.FieldName == "BoostAmount")
            {
                TieredInt ar = (TieredInt)ovr.GetValue();
                valueStr = ar[(int)rarity].ToString();
            }
            else if (ovr.FieldName == "StatToBoost")
            {
                selectedStat = (StatToBuff)ovr.GetValue();
            }
        }

        if (string.IsNullOrEmpty(valueStr))
            valueStr = BoostAmount[(int)rarity].ToString();

        return GetLocalizedDescription(
            Loc(
                ("boost", GetLocalizedBoostStat(selectedStat)),
                ("value", valueStr),
                ("tag", StringLookup.GetTag(selectedTag))
            )
        );
    }

    private string GetLocalizedBoostStat(StatToBuff statToBuff)
    {
        return statToBuff switch
        {
            StatToBuff.damage => L.Get("MiscKeywords", "STAT_DAMAGE"),
            StatToBuff.heal => L.Get("MiscKeywords", "STAT_HEAL"),
            StatToBuff.crit => L.Get("MiscKeywords", "STAT_CRIT"),
            StatToBuff.armor => L.Get("MiscKeywords", "STAT_ARMOR"),
            _ => statToBuff.ToString()
        };
    }
}


```

</details>

_____________________________________________________________________________________

### *Developed by*

![](https://github.com/ewigur/Portfolio-Emma-Wigur/blob/main/ThumbNails/ImperialPlaygroundsWhiteLogoFramed.png)
_____________________________________________________________________________________


[RETURN TO TOP](#TOP)
             <a name="TOP"></a>  

