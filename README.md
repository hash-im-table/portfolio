# The Purpose of this Portfolio
This (under-construction) portfolio is a place for me to showcase my Web & Game Development experiments in `Unity`, `Unreal` and `Blender`. The portfolio, is very much a WIP and will be updated regularly!

- [The Purpose of this Portfolio](#the-purpose-of-this-portfolio)
  * [Unity Experiments](#unity-experiments)
    + [Building my own collision detection system.](#building-my-own-collision-detection-system)
      - [The Code](#aabb-code)
    + [Mathematics of a Platformer Game](#Mathematics-of-a-Platformer-Game])
    + [Enemy Platform Tracing](#enemy-platform-tracing)
    + [Top-Down Collision Detection](#top-down-collision-detection)
      - [Explaining the Algorithm Breakdown](#explaining-the-algorithm-breakdown)
    + [Building a quick proof of concept](#building-a-quick-proof-of-concept)
      - [The Code](#the-code)
    + [Building a 3D PlayerController](#building-a-playercontroller)
      - [PlayerController Code](#playercontroller-code)
    + [Complex Enemy Behaviour](#complex-enemy-behaviour)

  * [Blender Experiments](#blender-experiments)
  * [Project Tracker](#project-tracker)


## Unity Experiments 
### Building my own collision detection system. 

The Unity engine is great for 3D games, the inbuilt Rigidbody physics system is well optimized for 3D player & enemy behaviour. However, for 2D games, the RigidBody2D system creates more sporadic and unpredictable behaviour when interacting with other physics gameobjects. The 2D Unity physics system is designed to replicate real world physics, which is not what you want for a 2D game. 

Another issue with the RigidBody2D system as a means of collision detection and movement, is that it creates this ‘float-y’ in-game gravity which for most games is an undesired feel. This is an example of that float-y, imprecise movement moving the player via a RigidBody2D can create.

<p align="center">
<img src="https://user-images.githubusercontent.com/66776230/86543782-87b38c00-bf19-11ea-8526-9ccf90760feb.gif" width="500" height="500"/>
</p>

To work around this I have been experimenting with building me own collision detection and raycasting system. Below is an example of my implementation:

<p align="center">
<img src="https://user-images.githubusercontent.com/66776230/84371494-a308d100-abd1-11ea-8479-d00a94a477e9.gif" width="500" height="500"/>
</p>

    
#### AABB Code

```c#
    void Start() {
        collider = GetComponent<BoxCollider2D> ();
        CalculateRaySpacing ();
    }
 
    public void Move(Vector3 velocity) {
        UpdateRaycastOrigins ();
        collisions.Reset ();
 
        if (velocity.x != 0) {
            HorizontalCollisions (ref velocity);
        }
        if (velocity.y != 0) {
            VerticalCollisions (ref velocity);
        }
 
        transform.Translate (velocity);
    }
 
    void HorizontalCollisions(ref Vector3 velocity) {
        float directionX = Mathf.Sign (velocity.x);
        float rayLength = Mathf.Abs (velocity.x) + skinWidth;
       
        for (int i = 0; i < horizontalRayCount; i ++) {
            Vector2 rayOrigin = (directionX == -1)?raycastOrigins.bottomLeft:raycastOrigins.bottomRight;
            rayOrigin += Vector2.up * (horizontalRaySpacing * i);
            RaycastHit2D hit = Physics2D.Raycast(rayOrigin, Vector2.right * directionX, rayLength, collisionMask);
 
            Debug.DrawRay(rayOrigin, Vector2.right * directionX * rayLength,Color.red);
 
            if (hit) {
                velocity.x = (hit.distance - skinWidth) * directionX;
                rayLength = hit.distance;
 
                float slopeAngle = Vector2.Angle(hit.normal, Vector2.up);
                if (i == 0 && slopeAngle <maxSlopeAngle)
                {
                    print(slopeAngle);
                    ClimbSlope(ref velocity, slopeAngle);
                }
 
                collisions.left = directionX == -1;
                collisions.right = directionX == 1;
            }
        }
    }
    
    void VerticalCollisions(ref Vector3 velocity) {
        float directionY = Mathf.Sign (velocity.y);
        float rayLength = Mathf.Abs (velocity.y) + skinWidth;
 
        for (int i = 0; i < verticalRayCount; i ++) {
            Vector2 rayOrigin = (directionY == -1)?raycastOrigins.bottomLeft:raycastOrigins.topLeft;
            rayOrigin += Vector2.right * (verticalRaySpacing * i + velocity.x);
            RaycastHit2D hit = Physics2D.Raycast(rayOrigin, Vector2.up * directionY, rayLength, collisionMask);
 
            Debug.DrawRay(rayOrigin, Vector2.up * directionY * rayLength,Color.red);
 
            if (hit) {
                velocity.y = (hit.distance - skinWidth) * directionY;
                rayLength = hit.distance;
 
                collisions.below = directionY == -1;
                collisions.above = directionY == 1;
            }
        }
    }
 
    void UpdateRaycastOrigins() {
        Bounds bounds = collider.bounds;
        bounds.Expand (skinWidth * -2);
 
        raycastOrigins.bottomLeft = new Vector2 (bounds.min.x, bounds.min.y);
        raycastOrigins.bottomRight = new Vector2 (bounds.max.x, bounds.min.y);
        raycastOrigins.topLeft = new Vector2 (bounds.min.x, bounds.max.y);
        raycastOrigins.topRight = new Vector2 (bounds.max.x, bounds.max.y);
    }
 
    void CalculateRaySpacing() {
        Bounds bounds = collider.bounds;
        bounds.Expand (skinWidth * -2);
 
        horizontalRayCount = Mathf.Clamp (horizontalRayCount, 2, int.MaxValue);
        verticalRayCount = Mathf.Clamp (verticalRayCount, 2, int.MaxValue);
 
        horizontalRaySpacing = bounds.size.y / (horizontalRayCount - 1);
        verticalRaySpacing = bounds.size.x / (verticalRayCount - 1);
    }
 
    struct RaycastOrigins {
        public Vector2 topLeft, topRight;
        public Vector2 bottomLeft, bottomRight;
    }
 
    public struct CollisionInfo {
        public bool above, below;
        public bool left, right;
 
        public void Reset() {
            above = below = false;
            left = right = false;
        }
    }
```

### Enemy Platform Tracing

I began tinkering with the above code to see other use cases for it. By using the code above to determine the bounds of a rectangular platform, I can essentially map a patrolling enemy around its extremities without the need to constantly set and re-set multiple waypoints. Here is what that looked like: 

<p align="center">
<img src="https://user-images.githubusercontent.com/66776230/84375124-faf60680-abd6-11ea-9a83-a6e3c67beefe.gif"/>
</p>

That’s what’s going on under the hood. If you overlay it with some fancy assets and tinker with the speed variable, this is what it would look like:

<p align="center">
<img src="https://user-images.githubusercontent.com/66776230/84427828-72e71f80-ac1d-11ea-819b-480c2453a050.gif"/>
</p>

### Top-Down Collision Detection

I also wanted to try and repurpose my code again for a top-down game, again leveraging the collision detection system I built earlier. This time I also added enemies that could pathfind using the A* algorithm

Below is what that looks like!

<p align="center">
<img src="https://user-images.githubusercontent.com/66776230/84375535-9d15ee80-abd7-11ea-9298-6ddc3e77166f.gif" width="500" height="500"/>
</p> 
<p align="center">*(Ignore the pink colour palate, I was trying to impress my little cousin. It didn’t work…)*</p> 

####  Explaining the Algorithm Breakdown
WIP

### Building a quick proof of concept

* Falling obstacles are dynamically generated based on player screen dimensions 
* Player screen wrapping is also dynamically calculated based on the size of the screen the game is being played on
* Falling obstacle angle, scale and speed are all randomly calculated variables that increase to a max difficulty over time.

<p align="center">
<img src="https://user-images.githubusercontent.com/66776230/86813777-7cc33d80-c078-11ea-86e1-fb8556055dbd.gif" width="250" height="350"/>
</p> 

#### The Code

```csharp
if (Time.time > nextSpawnTime) {
    float secondsBetweenSpawns = Mathf.Lerp (secondsBetweenSpawnsMinMax.y, secondsBetweenSpawnsMinMax.x, Difficulty.GetDifficultyPercent ());
    nextSpawnTime = Time.time + secondsBetweenSpawns;

    float spawnAngle = Random.Range (-spawnAngleMax, spawnAngleMax);
    float spawnSize = Random.Range (spawnSizeMinMax.x, spawnSizeMinMax.y);
    Vector2 spawnPosition = new Vector2 (Random.Range (-screenHalfSizeWorldUnits.x, screenHalfSizeWorldUnits.x), screenHalfSizeWorldUnits.y + spawnSize);
    GameObject newBlock = (GameObject)Instantiate (fallingBlockPrefab, spawnPosition, Quaternion.Euler(Vector3.forward * spawnAngle));
    newBlock.transform.localScale = Vector2.one * spawnSize;
    }
```

### Building a PlayerController

<p align="center">
<img src="https://user-images.githubusercontent.com/66776230/89227364-72dc1e00-d5d5-11ea-81aa-ccf9718e3803.gif"/>
</p>

#### PlayerController Code
```c# 
[RequireComponent (typeof (PlayerController))]
public class Player : MonoBehaviour {

	public float moveSpeed = 5;

	Camera viewCamera;
	PlayerController controller;
	
	void Start () {
		controller = GetComponent<PlayerController> ();
		viewCamera = Camera.main;
	}

	void Update () {
		Vector3 moveInput = new Vector3 (Input.GetAxisRaw ("Horizontal"), 0, Input.GetAxisRaw ("Vertical"));
		Vector3 moveVelocity = moveInput.normalized * moveSpeed;
		controller.Move (moveVelocity);

		Ray ray = viewCamera.ScreenPointToRay (Input.mousePosition);
		Plane groundPlane = new Plane (Vector3.up, Vector3.zero);
		float rayDistance;

		if (groundPlane.Raycast(ray,out rayDistance)) {
			Vector3 point = ray.GetPoint(rayDistance);
			Debug.DrawLine(ray.origin,point,Color.red);
			//Debug.DrawRay(ray.origin,ray.direction * 100,Color.red);
			controller.LookAt(point);
		}
	}
}
```

### Complex Enemy Behaviour

I have also started looking into enemy behaviour: 

The green pizza like slice represents what Bowser can see. Anything in that segment will alert Bowser. The radius and angle of the slice is are variable values. 

<p align="center">
<img src="https://user-images.githubusercontent.com/66776230/84420013-55ac5400-ac11-11ea-8a55-a651874af659.gif"/>
</p>

## Mathematics of a Platformer Game

Implementing climbing slope functionality using triginometry



## Blender Experiments
WIP

## Project Tracker
- [x] Building my own collision detection system. 
- [ ] Showcase a few Blender models
- [ ] A* pathfinding Alogorithm breakdown
