# What is this portfolio?
This WIP portfolio is a place for me to showcase my Game Development experiments in Unity, Unreal and Blender. The portfolio, is very much a WIP and will be updated regularly!


Table of contents
=================

<!--ts-->
   * [Unity](#unity-experiments)
      * [Building my own collision detection system](#Building-my-own-collision-detection-system)
      * [Enemy Platform Tracing](#Enemy-Platform-Tracing)
      * [Top-Down Collision Detection](#Top-Down-Collision-Detection)
      * [Complex Enemy Behaviour](#Complex-Enemy-Behaviour)
   * [Project Tracker](#Project-Tracker)
<!--te-->

## Unity Experiments
### Building my own collision detection system. 

The Unity engine is great for 3D games, the inbuilt Rigidbody physics system is well optimized for 3D player & enemy behaviour. However, for 2D games, the RigidBody2D system creates more sporatic and unpredicatable behaviour when interatcting with other physics gameobjects. The 2D Unity physics system is designed to replicate real world physics, which is not what you want for a 2D game. Another issue with the RigidBody2D system as a means of collision detection and movement, is that it creates this ‘float-y’ in-game gravity which for most games is an undesired feel.

To work around this I have been experimenting with building me own collision detection and raycasting system. Below is an example of my implementation:

<p align="center">
<img src="https://user-images.githubusercontent.com/66776230/84371494-a308d100-abd1-11ea-8479-d00a94a477e9.gif" width="500" height="500"/>
</p>

    {::options parse_block_html="true" /}
    <details><summary markdown="span">Let's see some code!</summary>
    
    ```c#
    public LayerMask collisionMask;
 
    const float skinWidth = .015f;
    public int horizontalRayCount = 4;
    public int verticalRayCount = 4;
 
    float horizontalRaySpacing;
    float verticalRaySpacing;
 
    BoxCollider2D collider;
    RaycastOrigins raycastOrigins;
    public CollisionInfo collisions;
 
    private float maxSlopeAngle = 80f;
 
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
    {::options parse_block_html="false" /}

### Enemy Platform Tracing

I began tinkering with the above code to see other use cases for it. By using the code above to determine the bounds of a rectangular platform, I can essentially map a patrolling enemy around its extremities without the need to constantly set and re-set multiple waypoints. Here is what that looked like: 

![PatorllingEnemy](https://user-images.githubusercontent.com/66776230/84375124-faf60680-abd6-11ea-9a83-a6e3c67beefe.gif)

Thats whats going on under the hood. If you overlay it with some fancy assets and tinker with the speed variable, this is what it would look like:

![showcase](https://user-images.githubusercontent.com/66776230/84427828-72e71f80-ac1d-11ea-819b-480c2453a050.gif)

### Top-Down Collision Detection

I also wanted to try and repurpose my code again for a top-down game, again leveraging the collision detection system I built earlier. 

Below is what that looks like!

![WIP1](https://user-images.githubusercontent.com/66776230/84375535-9d15ee80-abd7-11ea-9298-6ddc3e77166f.gif)

*(Ignore the pink colour palate, I was trying to impress my little cousin. It didn’t work…)*

## Complex Enemy Behaviour

I have also started looking into enemy behaviour: 

The green pizza like slice represents what Bowser can see. Anything in that segment will alert Bowser. The radius and angle of the slice is are variable values. 

![Bowser](https://user-images.githubusercontent.com/66776230/84420013-55ac5400-ac11-11ea-8a55-a651874af659.gif)

# Project Tracker
- [x]  building my own collision detection system. 
- [ ]  build a few Blender models
- [ ]  a* pathfinding Alogorithm 
