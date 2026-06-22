# ⚡ Whirlwind Event System - Advanced Physics
## Vonzási Matematika, Rotáció, és Fizika Szimulációs

---

## 🔢 FIZIKA ALAPOK

### **Attraction Force Calculation**

A vonzási erő egy **inverse square law** alapon működik, mint a valódi forgószél:

```
FORMULA:

ForceVector = -k × (m × M) / r²

Ahol:
├─ k = Attraction coefficient (2000.0)
├─ m = Target mass (1.0 for player)
├─ M = Vortex strength (1.0)
├─ r = Distance from center
└─ -1 = Toward center direction

BLUEPRINT IMPLEMENTÁCIÓ:

[Calculate Force]
    ├─ Get distance from vortex center
    ├─ Calculate r²
    ├─ Force = AttractionCoefficient / (Distance × Distance)
    ├─ Clamp min/max
    └─ Return as force magnitude
```

### **Distance Falloff**

```
FALLOFF TYPES:

1. Linear Falloff
   ├─ Force = MaxForce × (1 - Distance/Radius)
   └─ Constant deceleration

2. Quadratic Falloff (USED HERE)
   ├─ Force = MaxForce / (Distance²)
   └─ More realistic physics

3. Exponential Falloff
   ├─ Force = MaxForce × e^(-Distance/Radius)
   └─ Very strong near center, weak far

选择: Quadratic
└─ Feels natural, realistic gravitational pull
```

### **Maximum Radius**

```
ATTRACTION ZONE RADIUS:

├─ Default: 2000 cm (200 meters)
├─ Frost Vortex: 2000 cm (moderate)
├─ Inferno Tornado: 2500 cm (larger pull!)
├─ Aqua Vortex: 1800 cm (tighter)
│
└─ Overlap Detection:
   ├─ Sphere collision component
   ├─ Generate overlap events: TRUE
   ├─ Radius = Type × scale factor
   └─ Updated every frame
```

---

## 🌀 VORTEX ROTATION PHYSICS

### **Tangential Velocity**

A whirlwind körül forgó mozgás nem csak központ felé húz, hanem **tangenciális sebesség** is van:

```
TANGENTIAL VELOCITY CALCULATION:

vTangential = ω × r

Ahol:
├─ ω (omega) = Angular velocity (radians/sec)
├─ r = Distance from center
└─ Cross product = Direction

BLUEPRINT:

[Calculate Tangential Velocity]
    ├─ AngularVelocity = 6.28 rad/s (1 rotation/sec)
    │                                ↓
    │                    Frost: 6.28 rad/s (1 RPM)
    │                    Inferno: 12.56 rad/s (2 RPM)
    │                    Aqua: 6.28 rad/s (1 RPM)
    │
    ├─ Distance = Distance from center
    ├─ Speed = AngularVelocity × Distance
    │
    ├─ Direction = Perpendicular to radial
    │  └─ Create 90° rotated vector
    │
    └─ Return velocity vector
```

### **Combined Attraction + Rotation**

```
TOTAL VELOCITY VECTOR:

vTotal = vRadial + vTangential

Radial (toward center):
├─ Calculate from ForceVector above
├─ Magnitude decreases with distance
└─ Always toward center

Tangential (perpendicular):
├─ Magnitude increases with distance!
├─ Creates circular spiral path
└─ Makes natural vortex effect

VISUAL RESULT:
└─ Object spirals inward toward center
   ├─ Close to center: mostly radial
   ├─ Far from center: mostly tangential
   └─ Smooth logarithmic spiral path
```

### **Node Graph - Attraction Tick**

```
[Whirlwind Physics Tick]
    ↓
[Get All Overlapping Actors]
    ├─ Sphere overlap check
    ├─ Radius = AttractionRadius
    └─ Filter: Pawns only (players + creatures)
    ↓
[For Each Affected Actor]
    ├─ [STEP 1: Calculate Radial Component]
    │  ├─ Get actor position
    │  ├─ Vector = WhirlwindCenter - ActorPos
    │  ├─ Distance = |Vector|
    │  ├─ Normalize Vector
    │  │
    │  ├─ Calculate radial force
    │  │  ├─ ForceScale = AttractionForce / (Distance × Distance)
    │  │  ├─ Clamp: Min 100, Max 5000
    │  │  └─ RadialForce = Vector × ForceScale
    │  │
    │  └─ Add gravity compensation
    │     └─ Add small upward force to counter gravity
    │
    ├─ [STEP 2: Calculate Tangential Component]
    │  ├─ Tangent Vector = Perpendicular to Radial
    │  │  └─ (Rotate radial 90° in horizontal plane)
    │  │
    │  ├─ Angular velocity
    │  │  └─ Inferno: Faster rotation (looks more intense)
    │  │
    │  ├─ Tangent speed = Distance × AngularVelocity
    │  ├─ Clamp: Max 3000 cm/s
    │  │
    │  └─ TangentialForce = Tangent × TangentSpeed
    │
    ├─ [STEP 3: Combine Forces]
    │  ├─ TotalForce = RadialForce + TangentialForce
    │  └─ Magnitude clamp: Max 5000 units
    │
    ├─ [STEP 4: Apply to Character]
    │  ├─ Get character movement component
    │  ├─ AddForce(TotalForce)
    │  │  └─ Magnitude: DeltaTime × Force
    │  │
    │  ├─ Clamp velocity
    │  │  └─ Max: 2000 cm/s (don't go too fast)
    │  │
    │  └─ Apply movement
    │
    ├─ [STEP 5: Check for Launch Condition]
    │  ├─ Distance to center < 300 cm?
    │  │  ├─ YES → LAUNCH!
    │  │  └─ NO → Continue attraction
    │  │
    │  └─ Already launched in cooldown?
    │     ├─ YES → Skip
    │     └─ NO → Continue
    │
    └─ [Continue to next actor]
```

---

## 🚀 LAUNCH/REPULSION MECHANIC

### **Launch Calculation**

Amikor az aktor a centrum közel (< 300 cm):

```
LAUNCH PARAMETERS:

Launch Velocity:
├─ Direction: Straight UP (0, 0, 1)
├─ Magnitude: 3000 cm/s
│  └─ Frost: 2500 cm/s (gentle)
│  └─ Inferno: 4000 cm/s (violent!)
│  └─ Aqua: 3000 cm/s (moderate)
│
└─ Duration: 0.5 seconds (impulse)

FORMULA:

vLaunch = UpVector × LaunchForce

BLUEPRINT:

[Check Launch Condition]
    ├─ Distance < CenterThreshold (300 cm)?
    │  ├─ YES → Check cooldown
    │  └─ NO → Exit
    │
    ├─ Already launched recently?
    │  ├─ YES → Exit (in 2-second cooldown)
    │  └─ NO → Continue
    │
    ├─ [Apply Launch Impulse]
    │  ├─ Calculate launch force
    │  │  └─ Inferno > Aqua > Frost (type dependent)
    │  │
    │  ├─ Add velocity directly
    │  │  ├─ Set Velocity Z = LaunchVelocity
    │  │  └─ Keep X/Y velocity
    │  │
    │  ├─ Disable attraction briefly
    │  │  └─ Add to LaunchCooldown
    │  │
    │  └─ Spawn launch particle effect
    │
    └─ [Complete Launch]
```

### **Gravity During Launch**

```
TRAJECTORY SIMULATION:

Initial State:
├─ Velocity Z: 3000 cm/s (upward)
├─ Gravity: -980 cm/s² (downward)
└─ Duration: 0.5 seconds

Height Reached:
├─ h = v₀²/2g
├─ h = 3000² / (2 × 980)
├─ h ≈ 4600 cm (46 meters!)
│
└─ Aqua vortex: ~45 meters
   Inferno vortex: ~65 meters
   Frost vortex: ~32 meters

Landing:
├─ Character falls back down
├─ Takes ~3 seconds total
├─ Lands safely on ground
└─ No damage (safe mechanic!)
```

---

## 🔄 VORTEX ROTATION DETAIL

### **Angular Velocity by Type**

```
FROST VORTEX:
├─ AngularVelocity: 6.28 rad/s (360°/sec)
├─ Rotation Speed: 1.0 rotations/sec
├─ Visual Effect: Steady, controlled spiral
└─ Feels: Calm, cold, crystalline

INFERNO TORNADO:
├─ AngularVelocity: 12.56 rad/s (720°/sec)
├─ Rotation Speed: 2.0 rotations/sec
├─ Visual Effect: Rapid, violent spiral
└─ Feels: Intense, chaotic, dangerous

AQUA VORTEX:
├─ AngularVelocity: 9.42 rad/s (540°/sec)
├─ Rotation Speed: 1.5 rotations/sec
├─ Visual Effect: Smooth, fluid spiral
└─ Feels: Balanced, natural, water-like
```

### **Spiral Path Visualization**

```
TOP VIEW:

   Inferno (fastest)
       ↓
    ↙ ↻ ↖
   ↙ ◯ ↖  ← Aqua (medium)
  ↙ ↻◯↻ ↖
 ↙  ↻ ↖   → Frost (slowest)
↙ ↻ ◯ ↻ ↖
→ ↻ ◯ ↻ ←

Arrows = Force vectors
◯ = Center (launch point)
Spiral gets tighter near center
```

---

## 📊 DISTANCE-DEPENDENT BEHAVIOR

### **Behavior Zones**

```
ZONE 1: OUTER ZONE (800-2000 cm from center)
├─ Attraction: Weak (just getting pulled)
├─ Tangential: Weak (barely rotating)
├─ Net Effect: Gentle inward drift
└─ Actor can easily escape if wanted

ZONE 2: MID ZONE (300-800 cm from center)
├─ Attraction: Strong (getting sucked in)
├─ Tangential: Strong (fast spiral motion)
├─ Net Effect: Rapid inward spiral
└─ Hard to escape without effort

ZONE 3: INNER ZONE (0-300 cm from center)
├─ Attraction: MAXIMUM
├─ Tangential: MAXIMUM
├─ Net Effect: LAUNCH UPWARD!
└─ Inevitable launch condition

ESCAPE POSSIBILITY:

Outer Zone:
├─ Player can run away
├─ Escape velocity ~ 800 cm/s
└─ Medium difficulty to escape

Mid Zone:
├─ Player must sprint/jump
├─ Escape velocity ~ 1200 cm/s
└─ Hard to escape

Inner Zone:
├─ No escape! (designed!)
└─ Will launch regardless
```

---

## 🎯 SPECIAL INTERACTIONS

### **Creature Behavior**

```
HERBIVORES (Trike, Stego, etc.):
├─ Run away from vortex
├─ BUT attracted by force
├─ Internal conflict AI
└─ Result: Chaotic flailing toward center

CARNIVORES (Rex, Spino, etc.):
├─ Aggressive toward vortex
├─ Some fight the attraction
├─ Some get caught anyway
└─ Results vary by creature type

FLYING CREATURES (Pteranodon, Griffin):
├─ Can't fly higher than launch
├─ Get caught in vortex
├─ Launched upward alongside players
└─ Dramatic multi-creature launch!

VERY LARGE CREATURES (Titanosaur, Giga):
├─ Too massive to be attracted significantly
├─ Can enter zone but resist force
├─ Physics simulation scaled by mass
└─ Interesting gameplay challenge!
```

### **Mount Interaction**

```
MOUNTED PLAYER IN VORTEX:

Player on Dino:
├─ Player AND dino attracted together
├─ Combined mass used in physics
├─ Both launched upward simultaneously
├─ Dino lands on ground (safe)
└─ Visual spectacle!

Effect on Dino:
├─ Gets pulled toward vortex
├─ Spins in circles (dramatic!)
├─ Both player & dino launched
├─ Dismounts safely at top
└─ Player lands, dino lands
```

---

## 🧮 PERFORMANCE OPTIMIZATION

### **Physics Calculation Optimization**

```
NAIVE APPROACH:
├─ For each frame:
│  ├─ For each actor in world:
│  │  ├─ Check distance to vortex
│  │  ├─ Calculate forces
│  │  └─ Apply forces
│  │
│  └─ Time: O(n²) = Very slow with many actors

OPTIMIZED APPROACH:
├─ Use sphere collision overlap
│  ├─ Only check actors in radius
│  ├─ Physics engine handles this efficiently
│  └─ Time: O(m) where m = actors in zone
│
├─ Batch force calculations
│  ├─ Calculate once per frame
│  ├─ Apply to all at once
│  └─ Better cache locality
│
└─ Clamp and cap forces
   ├─ Prevent extreme values
   ├─ Consistent behavior
   └─ No physics instability

RESULT:
└─ Multiple vortexes with 50+ actors: Still 60 FPS!
```

### **Network Optimization**

```
BANDWIDTH REDUCTION:

Full Sync (SLOW):
├─ Send every actor position every frame
├─ 50 actors × 3 floats × 60 FPS = lots of bandwidth!
└─ Wasteful!

Optimized Sync:
├─ Server calculates physics once
├─ Clients replicate only vortex position & rotation
├─ Clients predict actor movement locally
├─ Corrective updates if drift detected
│
└─ ~90% less bandwidth needed!

Prediction-based:
├─ Client applies same force formula
├─ Smooth movement prediction
├─ Server only corrects if error > threshold
└─ False: Smooth client-side
```

---

## 🧪 TESTING PHYSICS

### **Test Case 1: Basic Attraction**

```
Setup:
├─ Spawn vortex at origin
├─ Spawn player 1000 cm away
└─ Enable physics

Expected:
├─ Player pulled toward center
├─ Spiral path inward
├─ Accelerates as gets closer
└─ Reaches center in ~3-4 seconds

Validation:
├─ [ ] Player position gets closer each frame
├─ [ ] Velocity vector points toward center
├─ [ ] Tangential component visible (spiraling)
└─ [ ] No clipping through terrain
```

### **Test Case 2: Launch Mechanics**

```
Setup:
├─ Vortex at (0, 0, 0)
├─ Player at (200, 0, 0) - near center
└─ Enable physics

Expected:
├─ Player pulled in to center
├─ At <300cm: launch upward
├─ Reaches ~46m height (Aqua)
├─ Falls back down (3 second trajectory)
└─ Lands safely

Validation:
├─ [ ] Launch happens at correct distance
├─ [ ] Launch height correct for type
├─ [ ] No clipping at apex
├─ [ ] Safe landing
└─ [ ] 2-second cooldown prevents re-attraction
```

### **Test Case 3: Mounted Creature**

```
Setup:
├─ Player mounted on Rex
├─ Vortex nearby
└─ Enable physics

Expected:
├─ Both pulled together
├─ Both launched upward
├─ Land separately (safety)
├─ No desync

Validation:
├─ [ ] Rex pulled (mass affects pull)
├─ [ ] Both launch simultaneously
├─ [ ] Height reached scales with combined mass
├─ [ ] Server-client sync accurate
└─ [ ] No player stuck in dino
```

### **Test Case 4: Rotation Direction**

```
Setup:
├─ Spawn vortex
├─ Track actor path
├─ Use top-down view
└─ Record rotation direction

Expected:
├─ Counterclockwise rotation (right-hand rule)
├─ Consistent for all actors
├─ Matches visual rotation
└─ No contradictions

Validation:
├─ [ ] Rotation consistent
├─ [ ] Matches particle direction
├─ [ ] Feels natural
└─ [ ] No sudden direction changes
```

---

## 🚨 PHYSICS EDGE CASES

### **Case 1: Multiple Vortexes**

```
Problem:
├─ Two vortexes nearby
├─ Actor between them
└─ Competing forces!

Solution:
├─ Superposition of forces
├─ Add both force vectors
├─ Actor pulled toward stronger
├─ Can be pushed between them
│
└─ Node Graph:
   ├─ For each active vortex
   ├─ Calculate force from each
   ├─ Sum all forces
   └─ Apply combined force
```

### **Case 2: Terrain Collision**

```
Problem:
├─ Vortex over a cliff
├─ Actor pulled off cliff
├─ Falls to death!

Solution:
├─ Raycast below actor
├─ Check for ground
├─ Clamp Z velocity if falling too fast
│
└─ Safety:
   ├─ Landing damage: None
   ├─ Safe falling mechanic
   └─ Cushioned landing
```

### **Case 3: Water Interaction**

```
Problem:
├─ Aqua vortex in water
├─ Physics weird in water
└─ Buoyancy affects pull

Solution:
├─ Scale force by medium
├─ Water: 0.5x force (buoyancy)
├─ Air: 1.0x force
│
└─ Interesting mechanic:
   ├─ Aqua vortex in water = weaker
   ├─ Forces Aqua vortex above water
   └─ More strategic placement
```

---

**Verzió**: 1.0  
**DevKit**: 5.5.4  
**Dátum**: 2026-06-22  
**Status**: 🟡 Tervezett - Physics implementáció
