🔗 [Back to Overview](https://github.com/ewigur/Portfolio-Emma-Wigur/blob/main/README.md)

<a name="TOP"></a>
<p align="center">
  <strong>Links</strong><br>
 | <a href="https://yrgo-game-creator.itch.io/train-tale">Download Train Tale</a> |
  <a href="https://www.youtube.com/watch?v=okvqh6uOwDE">Trailer</a> |
</p>

<p align="center"> <img src="https://github.com/ewigur/Portfolio/blob/main/Train%20Tale/GIFs/Sample_01.gif" width="600"/> </p>

<p align="center">
  <strong>Contents</strong><br>
  <a href="#️-following-eyes">👁️ Following Eyes</a> |
  <a href="#-nail-jump-sequence">🏃 Nail Jump Sequence</a> |
  <a href="#-player-movement">🎮 Player Movement</a> |
  <a href="#-sound-design">🔉 Sound Design</a>
</p>

# Brief Game Summary and Vision
Train Tale is a cozy, yet suspenseful 3D sidescroller where you play as a little gnomelike creature who lives among traveling human passengers on a train.\
 \
Our vision while building this game was to create characters that are easy to adore, as well as make an impactful impression on the player as the story unfolds.

*I had the pleasure to work with a wonderful team of fellow students in creating this stunning piece of work.\
It was challenging, but most of all it was fun.*

<p align="center">________________________________________________________________________________________________________________________________________________________</p>

*Working on this project, one of my personal goals was to get more comfortable with taking a step back and make sure I could read and understand my own code.\
One of my teachers gave me a valuable tip that I will take with me wherever I go - grab a pen and paper and map out the steps.\
Looking back I can pat myself on the back and say that I reached that goal.*

## A Selection of Contributions

### 👁️ Following Eyes
The first sinister encounter in the game are a pair of eyes following the player's movement. I coded the eyes to track the player based on their position and distance.\
Since the player has restricted movement on the trolley, it slows down to really capture the eerie stare of the stalking eyes.

<p align="center">
  <img src="https://github.com/ewigur/Portfolio/blob/main/Train%20Tale/GIFs/Following_Eyes_01.gif"/>
</p>

</details>

<details>
<summary>FollowingEyes.cs</summary>
<br>

_Stores references to each eyeball object, the player, and the tracking boundaries.\
Saves the eyes' starting rotation to return to when the player is out of range._
```csharp
public class FollowingEyes : MonoBehaviour
{
    public GameObject[] eyeBalls;

    [SerializeField]
    private GameObject player;

    [SerializeField] private float maxDistance = 5f;
    [SerializeField] private float minDistance = -5f;
    private float rotationTime = 1f;
    private Vector3 playerPos;
    private Quaternion startRotation;

    private void Start()
    {
        startRotation = transform.rotation;
    }
```
_Every frame, checks if the player is hiding. If not, the eyes will track them._
```csharp
    void Update()
    {
        if (!FindAnyObjectByType<CheckForObstacle>().IsPlayerHiding())
        {
            CalculateRotation();
        }
    }
```
_Loops through each eyeball and calculates how far the player is.\
If the player is within range, the eye smoothly rotates to look at them.\
If out of range, it slowly drifts back to its original position._
```csharp
    private void CalculateRotation()
    {
        foreach (GameObject eyeball in eyeBalls)
        {
            playerPos = player.transform.position;
            Vector3 distance = transform.position - playerPos;
            Quaternion resetRotation = Quaternion.Euler(0, 0, 0);
            Quaternion lookRotation = Quaternion.LookRotation((playerPos - eyeball.transform.position).normalized);

            if (distance.x < maxDistance && distance.x! > minDistance ||
                distance.x > minDistance && distance.x! < maxDistance)
            {
                eyeball.transform.rotation = Quaternion.Slerp(eyeball.transform.rotation,
                lookRotation, rotationTime * Time.deltaTime);
            }

            else
            {
                eyeball.transform.rotation = Quaternion.Slerp(eyeball.transform.rotation,
                resetRotation, rotationTime * 0.5f * Time.deltaTime);
            }
        }
    }
```
_Smoothly resets the first eyeball back to its starting rotation.\
Called externally when the eyes should stop tracking._
```csharp
    public void GetStartRotation()
    {
        eyeBalls[0].transform.rotation = Quaternion.Slerp(transform.rotation,
                startRotation, rotationTime * Time.deltaTime);
    }
}

```
</details>

<p align="center"> <a href="#TOP"><strong>↑ Return to Top</strong></a> </p>
<p align="center">________________________________________________________________________________________________________________________________________________________</p>

### 🏃 Nail Jump Sequence
The game relies on an interaction system to progress with the story.\
Most of the interactions triggers a cutscene that takes the player from A to B.\
The "Nail Jump" is one of them.

Initially I coded this to be an interaction where you press the interact button for each individual nail the player could jump on.\
It was a lot of tweaking to get the snap points to match with the jumping animation.\
We finally decided on making the nail jump into a single interaction,\
and the result was a smooth transition between the two levels.

**Contributions:**
- Level Design : positioning the nails
- Implementing the snap points for easier tweaking of the jump sequence
- Implementing the ability to interact with the nail jump using an Interface
- Implementing a timer for easier tweaking when creating the animation sequence

<p align="center">
  <img src="https://github.com/ewigur/Portfolio/blob/main/Train%20Tale/GIFs/Nail_Jump_01.gif" width="400"/>
</p>

*NOTE: The script I created has since been rewritten - as we programmers ended up collaborating in many different aspects of the game.\
The functionality remains the same, and below are code snippets from the original script written by me.*

</details>

<details>
<summary>NailJump.cs</summary>
<br>

_Called when the player interacts with the nail jump trigger.\
Checks that the player exists, the object is interactable,\
and the previous jump is complete before starting the next jump._
```csharp
 public void Interact()
    {
        if (playerRef == null)
            return;

        if (isInteractable)
        {
            if (isTimerDone == true && hasJumped == true)
            {
                EnumChecker();
                StartCoroutine(TimerBetweenJumps());
                CalculateJump(nextJumpPoint);
                DoJump(nextJumpPoint);
                StopCoroutine(TimerBetweenJumps());
            }
        }
    }
```
_Locks the player in place and disables input while waiting between jumps.\
Once the timer is done, re-enables the jump sequence._
```csharp
    private IEnumerator TimerBetweenJumps()
    {
        if (isTimerDone == false)
        {
            playerRb.isKinematic = true;
            CutsceneManager.Instance.DisableInput();

            yield return new WaitForSeconds(timerDuration);

            isTimerDone = true;
        }
    }
```
_Animates the player jumping to the target position using DOTween.\
Re-enables player control once they land on the floor._
```csharp
    public void DoJump(Vector3 nextJumpPoint)
    {
        if (playerRb == null || playerRef == null)
            return;
        

        playerRb.DOJump(nextJumpPoint, jumpForce, numbOfJumps, jumpDuration, snapping: false)
        .SetEase(Ease.Linear)
        .OnComplete(() =>
        {
            if (type == PointType.onFloor)
            {
                CutsceneManager.Instance.EnableInput();
                playerRb.isKinematic = false;
            }
        });

        hasJumped = true;
    }
```
_Calculates the exact world position of the next jump point,\
adjusting for a vertical offset so the player lands at the right height._
```csharp
    private void CalculateJump(Vector3 jumpTo)
    {
        Vector3 offsetY = new(0, offset, 0);
        Vector3 jumpPoint = nextPoint.transform.position - offsetY;
        nextJumpPoint = new Vector3(jumpPoint.x, jumpPoint.y, jumpPoint.z);
    }
```
</details>

<p align="center"> <a href="#TOP"><strong>↑ Return to Top</strong></a> </p>
<p align="center">________________________________________________________________________________________________________________________________________________________</p>

### 🎮 Player Movement
A character and story driven game is nothing without smooth navigation. We went back and forth many times to figure out the best type of movement in our game.\
It started with a blend between a type of locked (rail like) movement and free movement,\
but we finally decided on using free movement.

**Contributions:**
- Overall coding the movement system with the other programmers in the team
- Creating and implementing footstep sound - using animation events
- Implementing and tweaking walk/stop with animations
- Creating input action maps
- Collisions

<p align="center">
  <img src="https://github.com/ewigur/Portfolio/blob/main/Train%20Tale/GIFs/PlayerWalk_01.gif" width="400"/>
</p>

<p align="center"> <a href="#TOP"><strong>↑ Return to Top</strong></a> </p>
<p align="center">________________________________________________________________________________________________________________________________________________________</p>

### 🔉 Sound Design
I chose to take on the sound design. A big task that really gave me more respect for how intricate sound in games are - as well as the importance of it.

</details>

<details>
<summary>Disclaimer</summary>
<br>
- Apart from two SFX audio ("Dying Hand" and "Dead Eye Awakens"), all SFX are recorded by me
- All SFX and Ambience are mixed by me
- Tools: Audacity and Adobe Audition
</details>

**Contributions:**
- Mixing audio based on game object material, size and surroundings
- Matching audio with animation using event-based triggers
- Creating systems to implementing audio in the game
- Spatial audio based on direction and distance

<p align="center"> <a href="#TOP"><strong>↑ Return to Top</strong></a> </p>

<p align="center">_____________________________________________________________________________________</p>

<p align="center"><b><i>Developer</i></b></p>

<p align="center">
  <img src="https://github.com/ewigur/Portfolio/blob/main/Train%20Tale/GIFs/Carneval.gif" width="300"/>
</p>
<p align="center">_____________________________________________________________________________________</p>

<p align="center"><b><i>Made possible by</i></b></p>

![Image](https://github.com/ewigur/Portfolio/blob/main/ThumbNails/Yrgo.png)

<p align="center"><i>Higher Vocational Education - Game Creator Programmer, Göteborg</i></p>

<p align="center">_____________________________________________________________________________________</p>

<p align="center"> <a href="#TOP"><strong>↑ Return to Top</strong></a> </p>

🔗 [Back to Overview](https://github.com/ewigur/Portfolio-Emma-Wigur/blob/main/README.md)

