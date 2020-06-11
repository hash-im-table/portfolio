# My (WIP) portfolio

AABB

![CollisionDetectionTest](https://user-images.githubusercontent.com/66776230/84371494-a308d100-abd1-11ea-8479-d00a94a477e9.gif)

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
 
    void ClimbSlope(ref Vector3 velocity, float slopeAngle)
    {
        // Note: // treat vel.x as total distance up slope. Use bottom most ray to determine slope angle
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


