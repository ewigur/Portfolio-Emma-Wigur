<a name="TOP"></a>

🔗 [Back to Overview](https://github.com/ewigur/Portfolio-Emma-Wigur/blob/main/README.md)

<p align="center">
  <strong>Links</strong><br>
 | <a href="https://ewigur.itch.io/pond-hoppe">Play Pond Hopper</a> |

<p align="center">
  <img src=https://github.com/ewigur/Portfolio/blob/main/Pond%20Hopper/GIFs/PH.gif />
</p>

<p align="center">
  <strong>Contents</strong><br>
  <a href="#-jumping">🐸 Jumping</a> |
  <a href="#-pickups">💎 Pickups</a> |
  <a href="#-object-pooling">♻️ Object Pooling</a> |
  <a href="#-highscore-and-leaderboard">🏅 Highscore and Leaderboard</a> |
  <a href="#-graphics">🖌️ Graphics</a>
</p>

<p align="center">____________________________________________________________</p>
<p align="center"><b>Game Summary</b></p>

<i><p align="center">Pond Hopper is a 2D mobile game built for Android, developed solo as a school project at Yrgo Game Creator Programmer in early 2025. The player controls a frog that jumps between platforms by dragging and releasing on the screen, catching flies to score points while avoiding falling into the water. It's available to play in browser on itch.io.</p></i>

<p align="center">___________________________________________________________________________________________________________________________________</p>

Developed with a focus on core programming patterns - state machine, object pooling, and scriptable objects. It was my first solo-completed game, and it's one I'm genuinely proud of.
The game features a double jump mechanic for balance, two types of collectible flies (a common fly and a rarer, high-value firefly), and a local leaderboard that saves the top scores on the device.


## Mechanics

### 🐸 Jumping
The core mechanic is simple touch/click, drag, and release to make the frog jump and devour flies. A directional indicator lets you control both the angle and distance of each leap,\
so every jump is a deliberate choice.

The catch?\
This little critter can't swim. A miscalculated jump means drowning, which is exactly why the double jump exists — it's your second chance to course-correct before ending up at the bottom of the pond.

<p align="center">________________________________________________________________________________________________________________________________________________________</p>

### 💎 Pickups
Every fly you catch adds to your score - but not all flies are created equal. There's a common fly for steady scoring, and a rarer firefly worth more points on collection.\
Each type is built on a scriptable object, keeping their attributes clean and easy to tweak.

The firefly turned out to be more than just a scoring variant.\
During playtesting, it consistently caught people's attention — and they'd chase it even when a cluster of common flies nearby would've scored them more.\
That instinct to go for the shiny thing made the gameplay feel more dynamic without any extra design work.

<details>
<summary>Pickup</summary>
<br>

_This is the data container for collectible flies. Rather than hardcoding fly attributes, each fly type is defined as a ScriptableObject asset-\
meaning you can create and tweak fly variants (name, animation, value, spawn chance) directly in the Unity editor without touching code.\
The spawnProbability field is also what the object pool reads to decide which fly to spawn._

```csharp
[CreateAssetMenu(fileName = "PickUp", menuName = "ScriptableObjects/PickUp Item", order = 1)]
public class PickUpItem : ScriptableObject
{
    public string itemName;
    
    public Animator pickUpAnimator;
    public float flockMovement;
    public GameObject prefab;
    public int spawnAmount;
    public int value;
    
    [Range(0f, 1f)]
    public float spawnProbability;
}

```
</details>

<details>
<summary>State Machine</summary>
<br>

_This defines all possible states the game can be in as named constants.\
Using an enum here keeps state management readable and type-safe - instead of passing magic numbers or strings around.\
The code always deals in explicit, meaningful names like GameOver or GamePaused._

```csharp
      public enum GameStates
    {
        MainMenu,
        GameLoop,
        GamePaused,
        GameResumed,
        GameRestarted,
        GameOver,
    }
```
_The central state dispatcher. When a new state is requested, this switch routes to the right behaviour.\
The GameOver case shown here disables input, triggers the pause music event, and freezes time (timeScale = 0) - cleanly separating what happens on game over from who decides it's game over._

```csharp

    private void HandleStates(GameStates newState)
    {
        switch (newState)
        {
            
            case GameStates.GameOver:
                onToggleInput?.Invoke(false);
                TriggerPauseMusic?.Invoke();
                Time.timeScale = 0f;
                break;
        }
    }
```
_The call site that actually triggers the game over flow. It hands off to the state machine (ChangeState), then handles the UI side - hiding the lives display and pause button, and showing the game over menu.\
Keeping the UI update here and the game logic in HandleStates is a clean separation of concerns._

```csharp

{
    private void GameOver()
    {
        GMInstance.ChangeState(GameStates.GameOver);
        livesDisplay.SetActive(false);
        pauseButton.SetActive(false);
        gameOverMenu.SetActive(true);
    }
}

```

</details>

<p align="center">
  <img src="https://github.com/ewigur/Portfolio/blob/main/Pond%20Hopper/GIFs/PH_GamePlay.gif" width="600"/>
</p>

<p align="center">________________________________________________________________________________________________________________________________________________________</p>

### ♻️ Object Pooling

Since we're already on the topic of the flies - they also have their very own object pool. Since the gameloop goes on and on,\
it would be irresponsible of me not to implement a _circle of life_ kind of functionality. The flies spawn from a pool of preloaded\
prefabs, and when the player collects them they return to the pool to be released again.

- I used Unity's built in OP, and it takes information from the behavioural script created for the flies,\
which in turn is based off of the scriptable object that contains all the data.

- The pool takes the "spawnProbability" (from the scriptable object) into account,\
and releases a set amount of flies based on weight and amount of flies already existing in the scene.

<details>
<summary>Object Pooling</summary>
<br>
  
_Picks a fly type by running a weighted lottery across all items' spawn probabilities - the higher the probability, the more likely it is to win._

```csharp
    private PickUpItem GetRandomPickUpItem()
    {
        float totalWeight = pickUpItems.Sum(item => item.spawnProbability);
        float randomValue = Random.Range(0f, totalWeight);
        float cumulativeWeight = 0f;

        foreach (var item in pickUpItems)
        {
            cumulativeWeight += item.spawnProbability;
            if (randomValue <= cumulativeWeight)
                return item;
        }

        return pickUpItems[0];
    }
```

_Pulls a pickup from the pool and places it in the scene,\
skips spawning entirely if the active pickup limit has already been reached._

```csharp
    private void Spawn()
    {
        if (currentActivePickUps >= maxActivePickUps) 
            return;

        var randomPickUpItem = GetRandomPickUpItem();

        for (var i = 0; i < randomPickUpItem.spawnAmount; i++)
        {
            var pickUp = pickUpPools[randomPickUpItem].Get();
            currentActivePickUps++;

            pickUp.transform.position = GetRandomSpawnPosition();
            pickUp.Initialize(randomPickUpItem);
            pickUp.OnReturn += DisablePrefab;
        }
    }
```

_Returns a collected pickup to its pool for reuse, and unsubscribes from its return event to keep things clean._

```csharp
    private void DisablePrefab(PickUpBehaviour pickUp)
    {
        if (pickUpPools.TryGetValue(pickUp.GetItemData(), out var pool))
        {
            currentActivePickUps--;
            pool.Release(pickUp);
            pickUp.OnReturn -= DisablePrefab;
        }
    }
}

```

</details>

<p align="center">
  <img src="https://github.com/ewigur/Portfolio/blob/main/Pond%20Hopper/GIFs/PH_ObjectPool.gif" width="600"/>
</p>

<p align="center">________________________________________________________________________________________________________________________________________________________</p>

### 🏅 Highscore and Leaderboard

The game includes a local leaderboard that tracks the top scores. Keeping it local was a deliberate choice - this was a personal project meant to be shared with friends and family, and an online leaderboard would've been overkill.
When the frog meets its final fate and the player has earned a spot on the board, they're prompted to enter their name. The leaderboard is saved to the device and accessible from the main menu.
What I built:

- Local leaderboard storing the top 8 scores
- End-of-game prompt for name entry, triggered only when the score qualifies
- Persistent save to device storage
- Leaderboard display integrated into the main menu

<details>
<summary>Score Management</summary>
<br>
  
_Score is tracked live and persists across lives.\
The ScoreManager subscribes to game events on enable and unsubscribes on disable - so it only listens when it's active.\
Each collected fly adds its value to the score, which is immediately written to PlayerPrefs so it survives scene changes and life losses._

```csharp
private void OnEnable()
{
    PlayerCollision.OnScoreCollected += ScoreCollected;
    PlayerHasDied += FinalScore;
    OnLifeLost += SavedScore;
}

private void ScoreCollected(PickUpItem pickUpItem)
{
    score += pickUpItem.value;
    scoreText.text = "Score: " + score;
    PlayerPrefs.SetInt("currentScore", score);
}

private void SavedScore()
{
    score = PlayerPrefs.GetInt("currentScore", 0);
    scoreText.text = "Score: " + score;
}
```

_The score is handed off to the leaderboard on death.\
When the frog dies, FinalScore passes the current score to the HighScoreManager.\
This is the key connection between the two systems — the score manager doesn't know anything about leaderboards, it just hands the value off._

```csharp
private void FinalScore()
{
    if (HSInstance != null)
    {
        HSInstance.AddHighScore(score);
    }
}
```

_Only qualifying scores trigger the name prompt.\
Before anything leaderboard-related happens, AddHighScore checks whether the score actually earns a spot - either the list isn't full yet, or the new score beats the lowest entry.\
Only then does it fire the event that prompts the player to enter their name._

```csharp
public void AddHighScore(int newScore)
{
    LoadScore();
    if (newScore <= 0)
        return;

    if (highScores.Count < MaxListedScores || newScore > highScores[highScores.Count - 1].Value)
    {
        pendingHighScore = newScore;
        OnNewHighScore?.Invoke(newScore);
        TriggerHighScoreSound?.Invoke();
    }
}

```
_The name is attached, the list sorted, trimmed and saved.\
Once the player enters their name, SaveHighScoreToList pairs it with the pending score, sorts the full list in descending order, and trims anything beyond the cap.\
SaveScores then writes each entry to PlayerPrefs as indexed key-value pairs._

```csharp
private void SaveHighScoreToList(string playerName)
{
    highScores.Add(new KeyValuePair<string, int>(playerName, pendingHighScore));
    highScores.Sort((a, b) => b.Value.CompareTo(a.Value));

    if (highScores.Count > MaxListedScores)
    {
        highScores.RemoveAt(MaxListedScores);
    }

    SaveScores();
    pendingHighScore = 0;
}

private void SaveScores()
{
    PlayerPrefs.SetInt("ScoreCount", highScores.Count);
    for (int i = 0; i < highScores.Count; i++)
    {
        PlayerPrefs.SetInt($"HighScore{i}", highScores[i].Value);
        PlayerPrefs.SetString($"HighScoreName{i}", highScores[i].Key);
    }
    PlayerPrefs.Save();
}

```
_The leaderboard is reconstructed from storage on startup.\
LoadScore rebuilds the in-memory list from PlayerPrefs each time it's called — ensuring the leaderboard always reflects what's actually saved, even after the app is closed and reopened._

```csharp
private void LoadScore()
{
    highScores.Clear();
    int scoreCount = PlayerPrefs.GetInt("ScoreCount", 0);
    for (int i = 0; i < scoreCount; i++)
    {
        int score = PlayerPrefs.GetInt($"HighScore{i}", 0);
        string playerName = PlayerPrefs.GetString($"HighScoreName{i}");

        if (score > 0)
            highScores.Add(new KeyValuePair<string, int>(playerName, score));
    }
}

```
_Saved data is turned into ranked UI entries.\
DisplayLeaderBoard clears any existing entries, fetches the saved scores, and instantiates a UI template for each one - positioning them vertically by index.\
It also handles the ordinal formatting (1ST, 2ND, 3RD...) with a clean switch expression._

```csharp
private void DisplayLeaderBoard()
{
    foreach (Transform child in entryContainer)
    {
        if (child != entryTemplate)
            Destroy(child.gameObject);
    }

    List<KeyValuePair<string, int>> highScores = highScoreManager.GetHighScores();
    entryTemplate.gameObject.SetActive(true);

    for (int i = 0; i < highScores.Count && i < maxEntries; i++)
    {
        Transform entryTransform = Instantiate(entryTemplate, entryContainer);
        RectTransform entryRect = entryTransform.GetComponent<RectTransform>();
        entryRect.anchoredPosition = new Vector2(0, -tempHeight * i);
        entryTransform.gameObject.SetActive(true);

        int rank = i + 1;
        string rankTag = rank switch
        {
            1 => "ST",
            2 => "ND",
            3 => "RD",
            _ => "TH"
        };

        var posCountText = entryTransform.Find("PosCountText")?.GetComponent<TMPro.TextMeshProUGUI>();
        var scoreCountText = entryTransform.Find("ScoreCountText")?.GetComponent<TMPro.TextMeshProUGUI>();
        var playerName = entryTransform.Find("PlayerNameText")?.GetComponent<TMPro.TextMeshProUGUI>();

        if (posCountText != null) posCountText.text = rank + rankTag;
        if (scoreCountText != null) scoreCountText.text = highScores[i].Value.ToString();
        if (playerName != null) playerName.text = highScores[i].Key;
    }
}

```

</details>

<p align="center">
  <img src="https://github.com/ewigur/Portfolio/blob/main/Pond%20Hopper/GIFs/PH_HS.gif" width="600"/>
</p>

<p align="center">________________________________________________________________________________________________________________________________________________________</p>

### 🖌️ Graphics

All graphics are created by me.\
The only exception is the level background,\
which is an AI-generated image (Adobe Firefly) that I repainted and cut into three different pieces to layer the game scene.

| Fly  | Firefly |
| ------------- | ------------- |
| ![](https://github.com/ewigur/Portfolio/blob/main/Pond%20Hopper/Graphics/Fly.gif)  | ![](https://github.com/ewigur/Portfolio/blob/main/Pond%20Hopper/Graphics/FireFly.gif) |

| Level  | Frog |
| ------------- | ------------- |
| ![](https://github.com/ewigur/Portfolio/blob/main/Pond%20Hopper/Graphics/Level.gif)  |  ![](https://github.com/ewigur/Portfolio/blob/main/Pond%20Hopper/Graphics/PH_Frog.gif) |

| Platforms | 
| ------------- |
| ![](https://github.com/ewigur/Portfolio/blob/main/Pond%20Hopper/Graphics/PH_Log_Stone.png) |

<p align="center"> <a href="#TOP"><strong>↑ Return to Top</strong></a> </p>
<p align="center">_____________________________________________________________________________________</p>

<p align="center"><b><i>Made possible by</i></b></p>

<p align="center">
  <img src="https://github.com/ewigur/Portfolio/blob/main/ThumbNails/Yrgo.png"/>
</p>

<p align="center"><i>Higher Vocational Education - Game Creator Programmer, Göteborg</i></p>

<p align="center">_____________________________________________________________________________________</p>

<p align="center"> <a href="#TOP"><strong>↑ Return to Top</strong></a> </p>

🔗 [Back to Overview](https://github.com/ewigur/Portfolio-Emma-Wigur/blob/main/README.md)
