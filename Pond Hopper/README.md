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

Developed with a focus on core programming patterns — state machine, object pooling, and scriptable objects. It was my first solo-completed game, and it's one I'm genuinely proud of.
The game features a double jump mechanic for balance, two types of collectible flies (a common fly and a rarer, high-value firefly), and a local leaderboard that saves the top scores on the device.


## Mechanics

### 🐸 Jumping
The core mechanic of this game is pretty straight forward - touch, drag and release to make the frog jump and devour the flies.
The indicator shows direction of the jump, which gives the player control over how far the frog will go.
The double jump was the best way to balance the mechanic, since this little critter can't swim a miscalculated jump == drowning.

<p align="center">________________________________________________________________________________________________________________________________________________________</p>

### 💎 Pickups
Each collected fly adds to the score. The base of the fly is built on a scriptable object, which was the best approach for managing the attributes.\
I decided on having one common fly, and one firefly that would be more rare but with a higher score on collection. I quickly noticed that the firefly
caught the eye of the people testing the game,and it became more interesting gameplay as the subjects were more likely to chase the shiny fly instead of
diving into the cloud of common flies that might even give them a higher score.

<details>
<summary>PickUpItem.cs - Scriptable Object</summary>
<br>
  
```ruby
/*NOTE: This is the item data container for the pickups (flies).
In addition to defining what kind of item this is, this is also used
by the object pool to calculate which of the two pickup items to choose  - based on spawnProbability*/

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
  
```ruby
/*
  NOTE: This is a snippet of what happens under the hood as the game changes states.
        I created enums for each state
*/
      public enum GameStates
    {
        MainMenu,
        GameLoop,
        GamePaused,
        GameResumed,
        GameRestarted,
        GameOver,
    }

________________________

*/
    NOTE: As soon as the game state changes, the corresponding components listens to that.
          Below is a snippet from under the hood upon player death...
*/

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

________________________

/*
    NOTE: ...and a bunch of happens in correlation with the state change.
             (UI managing, this case.)
*/

(from "InGameStatesHandler")

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

Since were already on the topic of the flies - they also have their very own object pool. Since the gameloop goes on and on,\
it would be irresposible of me not to implement a _circle of life_ kind of functionality. The flies spawn from a pool of preloaded\
prefabs, and when the player collects them they return to the pool to be released again.

- I used Unity's built in OP, and it takes information from the behavioural script created for the flies,\
which in turn is based off of the scriptable object that contains all the data.

- The pool takes the "spawnProbability" (from the scriptable object) into account,\
and releases a set amount of flies based on weight and amount of flies already excisting in the scene.

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

Another system I wanted to implement was a leaderboard. I decided to only make it local, since this game was more about making it for myself and a fun thing to show friends and family (and, of course, you). 
If the player reaches a score higher than the last 8, they will be prompted to add their name in the textbox upon the frogs final death. The highscore is saved on the local device, and the leaderboard will be updated and available in the main menu of the game.

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
