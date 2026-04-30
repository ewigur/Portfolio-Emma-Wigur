<a name="TOP"></a>
🔗 [Back to Overview](https://github.com/ewigur/Portfolio-Emma-Wigur/blob/main/README.md)

<p align="center">
  <strong>Links</strong><br>
 | <a href="https://yrgo-game-creator.itch.io/pogopirates">Download Pogo Pirates</a> |
  <a href="https://www.youtube.com/watch?v=AxzvmTWsCbA">Trailer</a> |
</p>

![PogoLogo](https://github.com/ewigur/Portfolio/blob/main/Pogo%20Pirates/Img/pogologo.png)

<p align="center">
  <strong>Contents</strong><br>
  <a href="#-camera">🎥 Camera</a> |
  <a href="#-angled-platforms">📐 Angled Platforms</a> |
  <a href="#-punchy-ui--programming-and-animations">⚡ Punchy UI</a> |
  <a href="#-menu-and-tutorial">📜 Menu and Tutorial</a>
</p>

<p align="center">________________________________________________________________________________________________________________________________________________________</p>

<p align="center"><b>Game Summary</b></p>
<p align="center"><i>Pogo Pirates is all about being the King of the hill. Eliminate your friends (or enemies) with your mighty pogo stick; the perfect party game for couch hangouts!</i></p>

<p align="center">________________________________________________________________________________________________________________________________________________________</p>

*This project was the starting point of getting comfortable with building games, as well as working in a\
team. The latter wasn't hard, since every single one in the team really kept on encouraging and supporting each other,\
which also made the first point not as intimidating.*

*This was the first game I had ever been working on, and I couldn't be more proud of how it turned out*

## A Selection of Contributions

### 🎥 Camera
The camera system is based on fast paced action PvP games like *"Super Smash Bros"*. The camera keeps track of how many players are currently in the game, and adjusts a bounding box accordingly.\
The camera moves smoothly and zooms to get the feeling of depth. A dynamic camera was important to get the right feel of the game, and it was a good experience to keep that as my main focus.

**Contributions:**
- Implementing a dynamic bounding box that tracks all active players simultaneously
- Smooth camera follow with speed scaling based on distance from center
- Auto zoom that adjusts based on the spread of players on screen
- Arrow indicators that activate and point toward players when they leave the camera bounds
- Subscribing to player join and death events to dynamically update camera targets

</details>

<details>
<summary>CameraManager.cs</summary>
<br>
 
  _Sets up player targets, camera reference, and arrow indicators on start.\
Subscribes to player join and death events to keep the camera targets up to date._
 
```csharp
namespace PogoPirates.Cameras
{
    public class CameraManager : MonoBehaviour
    {
        
        public void Start()
        {
            cam = GetComponent<Camera>();

            foreach (GameObject playerObject in InstanceManager.Instance.playerObjects)
            {
                if (playerObject is null)
                    continue;
                
                Player player = playerObject.GetComponent<Player>();
                targets[player.id] = playerObject;
            }

            for (int i = 0; i < targets.Length; i++)
            {
                Transform arrowTransform = transform.Find($"Arrow_{i}");
                
                if (arrowTransform != null)
                {
                    arrows[i] = arrowTransform.gameObject;
                    arrows[i].GetComponent<SpriteRenderer>().enabled = false;
                }
            }

            FollowPlayers();           
        }

        private void OnEnable()
        {
            Player.onJoin += PlayerJoins;
            Player.onDeath += OnPlayerDeath;
        }
```
_Calculates a bounding box that fits all active players.\
The camera smoothly follows and zooms to keep everyone in frame.\
The camera will move faster when players are further from center to keep up with the action._
```csharp
        private void FollowPlayers()
        {   
            if (targets.All(item => item is null))
            {
                playerBounds = new Bounds(Vector3.zero, Vector3.zero);
            }
            else if (targets.Count(item => item is not null) == 1)
            {
                GameObject target = targets.First(item => item is not null);
                
                playerBounds = new Bounds(target.transform.position, Vector3.zero);
            }
            else
            {
                GameObject target = targets.Where(item => item is not null).Skip(1).First();

                playerBounds = new Bounds(target.transform.position, Vector3.zero);
            }

            for (int i = 0; i < targets.Length; i++)
            {
                if (targets[i] is null)
                    continue;
                
                playerBounds.Encapsulate(targets[i].transform.position);
                ArrowIndicator(targets[i].GetComponent<Player>());
            }
            
            playerBounds.Expand(cameraExpand);
            
            boundsCenter.x = Mathf.Clamp(boundsCenter.x, -cam.orthographicSize * cam.aspect, cam.orthographicSize * cam.aspect);
            boundsCenter.y = Mathf.Clamp(boundsCenter.y, -cam.orthographicSize, cam.orthographicSize);

            Vector3 moveOffset = playerBounds.center - transform.localPosition;
            moveOffset.z = 0;

            if(moveOffset.sqrMagnitude > 4)
            {
                transform.localPosition += moveOffset * (offsetMultiplier * 20);
            }
            else if(moveOffset.sqrMagnitude > 2)
            {
                transform.localPosition += moveOffset * (offsetMultiplier * 10);
            }
            else if(moveOffset.sqrMagnitude > 0.1f)
            {
                transform.localPosition += moveOffset * offsetMultiplier;
            }

            if (playerBounds.size.x > playerBounds.size.y)
            {
                float orthSize = Mathf.Lerp(Mathf.Clamp(playerBounds.extents.x / cam.aspect, minValue, maxValue), cam.orthographicSize, zoomSmoother);
                
                cam.orthographicSize = orthSize;
            }
            else
            {
                float orthSize = Mathf.Lerp(Mathf.Clamp(playerBounds.extents.y, minValue, maxValue), cam.orthographicSize, zoomSmoother);
                cam.orthographicSize = orthSize;
            }              
        }
```
</details>

<p align="center">
  <img src="https://github.com/ewigur/Portfolio/blob/main/Pogo%20Pirates/GIFs/CamShow.gif" width="720"/>
</p>


<p align="center"> <a href="#TOP"><strong>↑ Return to Top</strong></a> </p>
<p align="center">________________________________________________________________________________________________________________________________________________________</p>

### 📐 Angled Platforms
To keep things interesting the platforms at the far end of the ship on this level has an angled bounce effect.\
Landing on them gives the player an angled push, that can be adjusted to line up with the angle of the platform.

**Contributions:**
- Designing and implementing the angled bounce mechanic
- Calculating and applying directional launch force based on platform angle
- Exposing tweakable values in the Inspector for easy angle and to easily tweak adjustment during playtesting

</details>

<details>
<summary>AngledPlatform.cs</summary>
<br>

_Defines the platform's angle and the force applied to players on contact.\
Both values are tweakable in the Inspector within safe ranges._  
```csharp

    public class AngledPlatform : MonoBehaviour
    {
        [Tooltip("The angle of the platform (game object)")]
        [Range(-180, 180)]
        [SerializeField] 
        private int angle = 45;

        [Tooltip("The force of which the player is pushed away on contact")]
        [Range(15f, 50f)]
        [SerializeField]
        private float forceStrength = 30f;
```
_When a player collides with the platform, they are rotated to match its angle\
and launched in the platform's upward direction with an instant burst of force,\
sending them flying like a proper smash hit._
```csharp
        public void OnCollisionEnter2D(Collision2D collider) 
        {
            
            if (!collider.gameObject.TryGetComponent(out Player player))
                return;

            Rigidbody2D playerRigidbody = player.GetComponent<Rigidbody2D>();
            if (playerRigidbody == null)
                return;

            Quaternion platformRotation = Quaternion.Euler(0, 0, angle);
            player.transform.rotation = platformRotation;

            Vector2 forceDirection = transform.up;
            playerRigidbody.AddForce(forceDirection * forceStrength, ForceMode2D.Impulse);
        }
    }

```
</details>

<p align="center">
  <img src="https://github.com/ewigur/Portfolio/blob/main/Pogo%20Pirates/GIFs/AngledPlatforms.gif" width="720"/>
</p>

<p align="center"> <a href="#TOP"><strong>↑ Return to Top</strong></a> </p>
<p align="center">________________________________________________________________________________________________________________________________________________________</p>

### ⚡ Punchy UI : Programming and Animations
Players don't have a health bar, instead the hit percentage goes up as players hit one another.\
I worked with the hit progression UI.

Since the game is very fast paced we needed a way for players to easily see their character.\
The solution for that was an arrow shows up to indicate where a player is when they go outside of the camera bounds.

**Contributions:**
- Color lerp and animation on numbers when numbers change
- Plank shake animation
- Create an indicator based on the characters main color
- Implement a system that tracks if the player is outside of camera bounds that would trigger the indicator

</details>

<details>
<summary>Arrow Indicator</summary>
<br>
 
_If a player goes off screen, their arrow indicator activates and points toward them.\
The arrow is clamped to the edge of the viewport and rotates to face the player's direction._
```csharp
        private void ArrowIndicator(Player player)
        {
            Vector3 viewportPos = cam.WorldToViewportPoint(player.transform.position);

            GameObject arrow = arrows[player.id];

            bool isOutsideView = viewportPos.x < 0 || viewportPos.x > 1 || viewportPos.y < 0 || viewportPos.y > 1;
            arrows[player.id].SetActive(isOutsideView);
            
            if (isOutsideView)
                arrow.GetComponent<SpriteRenderer>().enabled = true;
            else
                arrow.GetComponent<SpriteRenderer>().enabled = false;

            if (isOutsideView)
            {               
                Vector3 arrowPosition = viewportPos;
                arrowPosition.x = Mathf.Clamp(arrowPosition.x, arrowoffset1, arrowoffset2);
                arrowPosition.y = Mathf.Clamp(arrowPosition.y, arrowoffset1, arrowoffset2);

                Vector3 worldPosition = cam.ViewportToWorldPoint(new Vector3(arrowPosition.x, arrowPosition.y, cam.nearClipPlane));
                arrow.transform.position = worldPosition;

                Vector3 direction = player.transform.position - arrow.transform.position;
                float angle = Mathf.Atan2(direction.y, direction.x) * Mathf.Rad2Deg;
                arrow.transform.rotation = Quaternion.Euler(0, 0, angle);
            }
        }

        public void FixedUpdate()
        {
            FollowPlayers();
        }

```
</details>


| Hit Percent  | Indicator |
| ------------- | ------------- |
| ![](https://github.com/ewigur/Portfolio/blob/main/Pogo%20Pirates/GIFs/UIShake.gif)  | ![](https://github.com/ewigur/Portfolio/blob/main/Pogo%20Pirates/GIFs/Indicator.gif) |

<p align="center"> <a href="#TOP"><strong>↑ Return to Top</strong></a> </p>
<p align="center">________________________________________________________________________________________________________________________________________________________</p>

### 📜 Menu and Tutorial
The main animations for the menu and tutorial is made by one of the artists on the project.\
To make the menu more alive we decided to add programmed UI animations as well.\
This was to create a punchier feel to each aspect of the game.

**Contributions:**
- Button shakes in menus
- Show/Hide tutorial side panels
- Menu behaviours
- Tutorial panel behaviours

| Menu  | Tutorial |
| ------------- | ------------- |
| ![](https://github.com/ewigur/Portfolio/blob/main/Pogo%20Pirates/GIFs/UI_1.gif)  | ![](https://github.com/ewigur/Portfolio/blob/main/Pogo%20Pirates/GIFs/UI_2.gif) |

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
