<a name="TOP"></a>

![](https://github.com/ewigur/Portfolio/blob/main/Train%20Tale/GIFs/Sample_01.gif)


# Brief Game Summary and Vision
Train Tale is a cozy, yet suspenseful 3D sidescroller where you play as a little gnomelike creature who lives among traveling human passengers on a train.\
 \
Our vision while building this game was to create characters that are easy to adore, as well as make an impactful impression on the player as the story unfolds.

## 
[Itch Page](https://yrgo-game-creator.itch.io/train-tale)\
[Trailer](https://www.youtube.com/watch?v=okvqh6uOwDE)
## 



*I had the pleasure to work with a wonderful team of fellow students in creating this stunning piece of game.\
It was challenging, but most of all it was fun - and I really hit the jackpot when I signed up to work with **Carneval**, a team of aspiring game devs!*

 \
*8 week 3D Project - at Yrgo Game Creator*\
*Developed in Spring/Summer of 2025*
_____________________________________________________________________________________
*Working on this project, one of my personal goals was to get more comfortable with taking a step back and make sure I could read and understand my own code. One of my teachers gave me a valuable tip that I will take with me wherever I go - bring out the old trusty pen and paper and write down the steps I needed to take to get where I wanted. Looking back I can tap myself on the shoulder and say that I reached that goal, and I learned so much more in the process.*

## A Selection of Contributions

**1. Following Eyes**\
The first sinister encounter in the game are a pair of eyes following the player's movement. I coded the eyes to track the player based on their position/distance. Since the player has restricted movement on the trolley, it slows down to really capture the eerie stare of the stalking eyes.

![](https://github.com/ewigur/Portfolio/blob/main/Train%20Tale/GIFs/Following_Eyes_01.gif)

</details>

<details>
<summary>FollowingEyes.cs</summary>
<br>
  
```ruby
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

    void Update()
    {
        if (!FindAnyObjectByType<CheckForObstacle>().IsPlayerHiding())
        {
            CalculateRotation();
        }
    }

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

    public void GetStartRotation()
    {
        eyeBalls[0].transform.rotation = Quaternion.Slerp(transform.rotation,
                startRotation, rotationTime * Time.deltaTime);
    }
}

```

</details>

_____________________________________________________________________________________

**2. Nail Jump Sequence**\
The game relies on an interaction system to progress with the story. Most of the interactions triggers a cutscene that takes the player from A to B. The "Nail Jump" is one of them; initially I coded this to be an interaction where you press the interact button for each individual nail the player could jump on. It was a lot of tweaking to get the snap points to match with the jumping animation.\
We finally decided on making the nail jump into a single interaction, and it resulted in a smooth transition between the two levels.

![](https://github.com/ewigur/Portfolio/blob/main/Train%20Tale/GIFs/Nail_Jump_01.gif)

*NOTE: The script I created has since been rewritten - as we programmers ended up collaborating in many different aspects of the game.\
The functionality remains the same, and below are code snippets from the original script written by me.*

</details>

<details>
<summary>NailJump.cs</summary>
<br>
  
```ruby
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

    private void CalculateJump(Vector3 jumpTo)
    {
        Vector3 offsetY = new(0, offset, 0);
        Vector3 jumpPoint = nextPoint.transform.position - offsetY;
        nextJumpPoint = new Vector3(jumpPoint.x, jumpPoint.y, jumpPoint.z);
    }

```

</details>

_____________________________________________________________________________________

**3. Player Moving**\
A character and story driven game is nothing without smooth navigation. We went back and foth many times to figure out the best type of movement in our game. It started with a blend between a type of locked (rail like) movement and free movement,\
but we finally decided on using free movement.

**My work with movement included:**
- Overall coding the movement system with the other programmers in the team
- Creating and implementing footstep sound - using animation events
- Implementing and tweaking walk/stop with animations
- Creating input action maps
- Collisions

![](https://github.com/ewigur/Portfolio/blob/main/Train%20Tale/GIFs/PlayerWalk_01.gif)

_____________________________________________________________________________________

**4. Sound Design**

I chose to take on the sound design. A big task that really gave me more respect for how intricate sound in games are - as well as the importance of it.


- *Apart from two SFX audio ("Dying Hand" and "Dead Eye Awakens"), all SFX are recorded by me*
- *All SFX and Ambience are mixed by me*
- *Tools: Audacity and Adobe Audition*

 In addition to dusting off old skills in recording, I worked on:
- Mixing audio based on game object material, size and surroundings
- Matching audio with animation using event-based triggers
- Creating systems to implementing audio in the game
- Spacial audio based on direction and distance
_____________________________________________________________________________________




### *Developed by*
![](https://github.com/ewigur/Portfolio/blob/main/Train%20Tale/GIFs/Carneval.gif)
_____________________________________________________________________________________
### *Made Possible by*
![Image](https://github.com/ewigur/Portfolio/blob/main/ThumbNails/Yrgo.png)
*Higher Vocational Education - Game Creator Programmer, Göteborg*
_____________________________________________________________________________________

[RETURN TO TOP](#TOP)
             <a name="TOP"></a>  

