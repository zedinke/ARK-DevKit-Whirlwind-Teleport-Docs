# ⚡ Whirlwind Event System - Blueprint Implementáció Útmutató
## Lépésről-lépésre a DevKit-ben

---

## 🎯 FÁZIS 1: Foundation Blueprints

### **1.1 BP_WhirlwindEventManager Létrehozása**

**Lépések:**

```
1. Content Browser → New → Blueprint
   └─ Parent Class: Actor
   └─ Name: BP_WhirlwindEventManager
   └─ Folder: /Blueprints/WhirlwindEvent/

2. Blueprint Editor megnyitása
   └─ Jobb klikk → Edit Blueprint

3. Komponensek hozzáadása
   ├─ Scene Root (Default - OK)
   └─ Nincs szükség visual komponensekre
      └─ Ez csak logika aktőr

4. Event Manager megjegyzése
   ├─ Details panel: Description
   ├─ Írj: "Manager for all whirlwind events"
   └─ Comment: "Time-based event spawning"
```

### **1.2 BP_WhirlwindEventManager Variables**

**Nyisd Details panel → My Blueprint → Variables**

```
Variables hozzáadása:

1. EventTriggerTime
   ├─ Type: Float
   ├─ Default Value: 1080.0
   ├─ Tooltip: "Time in minutes (0-1440). 1080 = 18:00"
   ├─ Category: "Event Schedule"
   ├─ Editable: TRUE ✓
   └─ Slider: 0-1440

2. MaxSimultaneousWhirlwinds
   ├─ Type: Integer
   ├─ Default Value: 3
   ├─ Tooltip: "Max concurrent whirlwind events"
   ├─ Category: "Event Configuration"
   ├─ Editable: TRUE ✓
   └─ Slider: 1-10

3. EventScheduleEnabled
   ├─ Type: Boolean
   ├─ Default Value: TRUE
   ├─ Tooltip: "Enable/disable entire event system"
   ├─ Category: "Event Schedule"
   ├─ Editable: TRUE ✓
   └─ Show 3D Widget: FALSE

4. WhirlwindSpawnLocations
   ├─ Type: Array (Vector)
   ├─ Default: Empty
   ├─ Tooltip: "Possible spawn locations on map"
   ├─ Category: "Spawn Locations"
   ├─ Editable: TRUE ✓
   └─ Array elements: Add in editor (5-10 points)

5. ActiveWhirlwinds
   ├─ Type: Array (BP_WhirlwindActor)
   ├─ Default: Empty
   ├─ Tooltip: "Currently active whirlwind instances"
   ├─ Category: "Internal"
   ├─ Editable: FALSE ✗
   └─ Replicated: TRUE

6. DayCycleManagerRef
   ├─ Type: BP_DayCycleManager (reference)
   ├─ Default: NULL
   ├─ Tooltip: "Reference to day/night cycle manager"
   ├─ Category: "Integration"
   ├─ Editable: FALSE ✗
   └─ This is found at runtime

7. LastEventTriggerTime
   ├─ Type: Float
   ├─ Default: 0.0
   ├─ Tooltip: "Tracks last spawn time (prevents spam)"
   ├─ Category: "Internal"
   ├─ Editable: FALSE ✗
   └─ Replicated: FALSE

8. EventCheckInterval
   ├─ Type: Float
   ├─ Default: 10.0
   ├─ Tooltip: "Check event schedule every X seconds"
   ├─ Category: "Configuration"
   ├─ Editable: TRUE ✓
   └─ Range: 5-30 seconds
```

### **1.3 BP_WhirlwindActor Létrehozása**

**Lépések:**

```
1. Content Browser → New → Blueprint
   └─ Parent Class: Actor
   └─ Name: BP_WhirlwindActor
   └─ Folder: /Blueprints/WhirlwindEvent/

2. Blueprint Editor megnyitása
   └─ Jobb klikk → Edit Blueprint

3. Komponensek hozzáadása

   Root: Scene Component
   ├─ Name: Root
   └─ Default component

   Child 1: Sphere Collision
   ├─ Name: AttractionZone
   ├─ Set Radius: 2000.0
   ├─ Generate Overlap Events: TRUE
   ├─ Collision → Object Type: WorldDynamic
   ├─ Collision → Collision Enabled: QueryOnly
   └─ Collision → Ignore Pawn: FALSE

   Child 2: Particle System Component
   ├─ Name: WhirlwindParticles
   ├─ Asset: PT_Whirlwind_Aqua (starts with aqua)
   ├─ Set Relative Rotation (0, 0, 0)
   ├─ Loop: TRUE
   ├─ Auto Activate: TRUE
   └─ Relative Location: (0, 0, 0)

   Child 3: Point Light
   ├─ Name: WhirlwindLight
   ├─ Light Color: (0, 1, 1) Cyan
   ├─ Intensity: 2000.0
   ├─ Attenuation Radius: 2000.0
   ├─ Visibility: TRUE
   └─ Relative Location: (0, 0, 500)

   Child 4: Audio Component
   ├─ Name: WhirlwindAudio
   ├─ Sound: SC_Whirlwind_Aqua_Loop (starts with aqua)
   ├─ Volume Multiplier: 0.8
   ├─ Pitch Multiplier: 1.0
   ├─ Loop: TRUE
   ├─ Auto Activate: FALSE (start manually)
   └─ Attenuation Settings: Enabled (3D spatial)

   Child 5: Arrow Component (Debug)
   ├─ Name: MovementDirection
   ├─ Relative Location: (0, 0, 200)
   └─ This shows direction actor is moving
```

### **1.4 BP_WhirlwindActor Variables**

**Nyisd Details panel → My Blueprint → Variables**

```
Variables hozzáadása:

1. WhirlwindType
   ├─ Type: EWhirlwindType (Create Enum first!)
   ├─ Enum Values:
   │  ├─ FrostVortex
   │  ├─ InfernoTornado
   │  └─ AquaVortex
   ├─ Default: AquaVortex
   ├─ Category: "Whirlwind Type"
   └─ Editable: TRUE ✓

2. AttractionRadius
   ├─ Type: Float
   ├─ Default: 2000.0
   ├─ Category: "Physics"
   ├─ Editable: TRUE ✓
   └─ Varies by type (1800-2500)

3. AttractionForce
   ├─ Type: Float
   ├─ Default: 2000.0
   ├─ Category: "Physics"
   ├─ Editable: TRUE ✓
   └─ Force magnitude multiplier

4. RemainingDuration
   ├─ Type: Float
   ├─ Default: 300.0
   ├─ Category: "Lifetime"
   ├─ Editable: FALSE ✗
   └─ Countdown to expiration

5. CurrentLifetime
   ├─ Type: Float
   ├─ Default: 0.0
   ├─ Category: "Lifetime"
   ├─ Editable: FALSE ✗
   └─ How long alive

6. PatrolPath
   ├─ Type: Array (Vector)
   ├─ Default: Empty
   ├─ Category: "Movement"
   ├─ Editable: FALSE ✗
   └─ Waypoints to follow

7. CurrentPathIndex
   ├─ Type: Integer
   ├─ Default: 0
   ├─ Category: "Movement"
   ├─ Editable: FALSE ✗
   └─ Current waypoint

8. MovementSpeed
   ├─ Type: Float
   ├─ Default: 500.0
   ├─ Category: "Movement"
   ├─ Editable: TRUE ✓
   └─ cm/s movement speed

9. AffectedActors
   ├─ Type: Array (Actor)
   ├─ Default: Empty
   ├─ Category: "Internal"
   ├─ Editable: FALSE ✗
   └─ Track who's being attracted

10. AngularVelocity
    ├─ Type: Float
    ├─ Default: 6.28 (rad/s)
    ├─ Category: "Physics"
    ├─ Editable: TRUE ✓
    └─ Vortex rotation speed

11. LaunchCooldownTime
    ├─ Type: Float
    ├─ Default: 2.0
    ├─ Category: "Launch"
    ├─ Editable: TRUE ✓
    └─ Cooldown after launch
```

### **1.5 Enum Létrehozása**

```
1. Content Browser → New → Enum
   └─ Name: EWhirlwindType

2. Enum Editor megnyitása
   └─ Double-click on enum

3. Enum Values hozzáadása
   ├─ FrostVortex (Index 0)
   ├─ InfernoTornado (Index 1)
   └─ AquaVortex (Index 2)

4. Save & Close
   └─ Compile
```

---

## 🔧 FÁZIS 2: Core Functions

### **2.1 BP_WhirlwindEventManager - Initialize Function**

**Új Function létrehozása: InitializeManager**

```
Function Declaration:
├─ Name: InitializeManager
├─ Inputs: None
├─ Outputs: None
├─ Access: Private
└─ Graph: Create Node Graph

Node Graph Implementation:

[Initialize Manager Event]
    ↓
[Find Day Cycle Manager]
    ├─ Get All Actors of Class
    │  └─ Class: BP_DayCycleManager
    │
    ├─ Get First
    │  └─ From array
    │
    └─ Set DayCycleManagerRef
       └─ Store reference
    ↓
[Setup Event Loop Timer]
    ├─ Get World
    ├─ Get Timer Manager
    ├─ Set Timer by Event
    │  ├─ Function: CheckEventTrigger
    │  ├─ Rate: EventCheckInterval (10 sec)
    │  └─ Loop: TRUE
    │
    └─ Store timer handle (optional debug)
    ↓
[Debug Log]
    ├─ If blueprint logs enabled
    ├─ Print: "Whirlwind Manager initialized"
    └─ Also log spawn locations count
    ↓
[Complete]
```

### **2.2 BP_WhirlwindEventManager - Check Event Trigger**

**Új Function: CheckEventTrigger**

```
Function Declaration:
├─ Name: CheckEventTrigger
├─ Inputs: None
├─ Outputs: None
├─ Access: Private
└─ Callable from timer

Node Graph:

[Check Event Trigger]
    ↓
[Validation Checks]
    ├─ Is EventScheduleEnabled? → NO → Exit
    ├─ Is DayCycleManagerRef valid? → NO → Exit
    └─ Is ActiveWhirlwinds < MaxSimultaneous? → NO → Exit
    ↓
[Get Current Game Time]
    ├─ Get DayCycleManagerRef
    ├─ Get Current Time (minutes)
    │  └─ Should be float (0-1440)
    │
    └─ Store in CurrentTime variable
    ↓
[Calculate Time Until Event]
    ├─ TimeDifference = EventTriggerTime - CurrentTime
    │
    ├─ If negative (event passed today):
    │  └─ TimeDifference += 1440 (next day)
    │
    └─ Store in TimeUntilEvent
    ↓
[Check Trigger Conditions]
    ├─ Is TimeUntilEvent < 1.0 minute? (window)
    │  ├─ NO → Exit
    │  └─ YES → Continue
    │
    ├─ Is (CurrentTime - LastEventTriggerTime) > 10.0?
    │  ├─ NO → Cooldown active, exit
    │  └─ YES → Continue
    │
    └─ All checks pass!
    ↓
[Spawn Whirlwind]
    ├─ Select random spawn location
    │  ├─ Get WhirlwindSpawnLocations
    │  ├─ Get random element
    │  └─ Store as SpawnLocation
    │
    ├─ Select random type
    │  ├─ Get random enum value
    │  │  └─ FrostVortex, InfernoTornado, AquaVortex
    │  │
    │  └─ Store in SelectedType
    │
    ├─ Create patrol path
    │  ├─ Generate 3-5 random waypoints
    │  ├─ Within map bounds
    │  └─ Store in PatrolPath array
    │
    └─ Call SpawnWhirlwindEvent
       ├─ Input: SelectedType
       ├─ Input: SpawnLocation
       ├─ Input: PatrolPath
       └─ Continue to next step
    ↓
[Update Last Trigger Time]
    ├─ LastEventTriggerTime = CurrentTime
    └─ Prevent spam
    ↓
[Debug Log]
    ├─ Print: "Whirlwind spawned!"
    └─ Include type and location
```

### **2.3 BP_WhirlwindEventManager - Spawn Whirlwind Event**

**Új Function: SpawnWhirlwindEvent**

```
Function Declaration:
├─ Inputs:
│  ├─ WhirlwindType (EWhirlwindType)
│  ├─ SpawnLocation (Vector)
│  └─ PatrolPath (Array of Vector)
│
├─ Outputs: None
├─ Access: Private
└─ Graph: Create Node Graph

Node Graph:

[Spawn Whirlwind Event]
    ↓
[Spawn Actor]
    ├─ Get World
    ├─ Spawn Actor from Class
    │  ├─ Class: BP_WhirlwindActor
    │  ├─ Location: SpawnLocation
    │  ├─ Rotation: (0, 0, 0)
    │  ├─ Owner: Self
    │  └─ Instigator: NULL
    │
    └─ Cast to BP_WhirlwindActor
       └─ Store in WhirlwindRef
    ↓
[Initialize Whirlwind]
    ├─ Set WhirlwindType
    │  └─ Call SetWhirlwindType function (next)
    │
    ├─ Set Patrol Path
    │  └─ WhirlwindRef.PatrolPath = PatrolPath array
    │
    ├─ Set Duration
    │  └─ WhirlwindRef.RemainingDuration = 300.0
    │
    └─ Begin Whirlwind (custom event)
       └─ Start movement & physics
    ↓
[Add to Active List]
    ├─ ActiveWhirlwinds.Add(WhirlwindRef)
    └─ Track for cleanup
    ↓
[Setup Expiration Timer]
    ├─ Get World Timer Manager
    ├─ Set Timer by Event
    │  ├─ Function: OnWhirlwindExpire
    │  ├─ Instigator: WhirlwindRef
    │  ├─ Rate: 300.0 (5 minutes)
    │  └─ Loop: FALSE
    │
    └─ Store timer handle
    ↓
[Broadcast to Clients]
    ├─ Call MulticastSpawnWhirlwind
    │  ├─ Type: WhirlwindType
    │  ├─ Location: SpawnLocation
    │  ├─ Whirlwind Ref: WhirlwindRef
    │  └─ Broadcast to all clients
    │
    └─ All players see event simultaneously
    ↓
[Complete]
```

### **2.4 BP_WhirlwindActor - Set Whirlwind Type**

**Új Function: SetWhirlwindType**

```
Function Declaration:
├─ Inputs: WhirlwindType (EWhirlwindType)
├─ Outputs: None
├─ Access: Public
└─ Graph: Create Node Graph

Node Graph:

[Set Whirlwind Type]
    ↓
[Store Type]
    └─ WhirlwindType = Input
    ↓
[Update Components Based on Type]
    ├─ [Switch on WhirlwindType]
    │
    ├─ Case: FrostVortex
    │  ├─ Set Particle System: PT_Whirlwind_Frost
    │  ├─ Set Audio: SC_Whirlwind_Frost_Loop
    │  ├─ Set Light Color: (0, 1, 1) Cyan
    │  ├─ Set Attraction Radius: 2000.0
    │  ├─ Set Angular Velocity: 6.28
    │  ├─ Set Launch Velocity: 2500.0
    │  └─ Store type-specific values
    │
    ├─ Case: InfernoTornado
    │  ├─ Set Particle System: PT_Whirlwind_Inferno
    │  ├─ Set Audio: SC_Whirlwind_Inferno_Loop
    │  ├─ Set Light Color: (1, 0.4, 0) Orange
    │  ├─ Set Attraction Radius: 2500.0
    │  ├─ Set Angular Velocity: 12.56
    │  ├─ Set Launch Velocity: 4000.0
    │  └─ Store type-specific values
    │
    └─ Case: AquaVortex
       ├─ Set Particle System: PT_Whirlwind_Aqua
       ├─ Set Audio: SC_Whirlwind_Aqua_Loop
       ├─ Set Light Color: (0, 0.8, 1) Light Blue
       ├─ Set Attraction Radius: 1800.0
       ├─ Set Angular Velocity: 9.42
       ├─ Set Launch Velocity: 3000.0
       └─ Store type-specific values
    ↓
[Complete]
```

---

## 🌀 FÁZIS 3: Physics Implementation

### **3.1 BP_WhirlwindActor - Attraction Tick**

**Event Graph: Tick Event**

```
[Event Tick]
    ├─ DeltaSeconds: Float
    │
    └─ Call ApplyAttractionPhysics
       └─ Pass DeltaSeconds as parameter

[Apply Attraction Physics]
    ↓
[Get Overlapping Actors]
    ├─ Get AttractionZone sphere collision
    ├─ Get Overlapping Components
    │  └─ Filter: Pawns only (players + creatures)
    │
    └─ For Each Overlapping Component → Owner Actor
    ↓
[For Each Affected Actor Loop]
    ├─ [STEP 1: Get Actor Data]
    │  ├─ Cast to Pawn
    │  ├─ Get Position
    │  ├─ Get Velocity
    │  ├─ Get Movement Component
    │  └─ Validate all not null
    │
    ├─ [STEP 2: Calculate Radial Force]
    │  ├─ Direction Vector = ThisLocation - ActorLocation
    │  ├─ Distance = Vector Length
    │  │
    │  ├─ If Distance < 100:
    │  │  ├─ Mark for launch (see below)
    │  │  └─ Continue (don't apply attraction, launch instead)
    │  │
    │  ├─ Normalize Direction
    │  ├─ Force Magnitude = AttractionForce / (Distance × Distance)
    │  │  └─ Quadratic falloff formula
    │  │
    │  ├─ Clamp Force:
    │  │  ├─ Min: 100.0
    │  │  ├─ Max: 5000.0
    │  │  └─ Prevent extreme values
    │  │
    │  └─ RadialForce = Direction × ForceMagnitude
    │
    ├─ [STEP 3: Calculate Tangential Force]
    │  ├─ Tangent = Perpendicular to Radial
    │  │  └─ Cross product in horizontal plane
    │  │
    │  ├─ TangentSpeed = Distance × AngularVelocity
    │  ├─ Clamp: Max 3000 cm/s
    │  │
    │  └─ TangentialForce = Tangent × TangentSpeed
    │
    ├─ [STEP 4: Combine Forces]
    │  ├─ TotalForce = RadialForce + TangentialForce
    │  ├─ Magnitude = Length(TotalForce)
    │  │
    │  └─ Clamp Max Magnitude:
    │     ├─ If Magnitude > 5000:
    │     │  └─ Normalize and scale to 5000
    │     └─ Prevent super-high forces
    │
    ├─ [STEP 5: Apply to Character]
    │  ├─ Get CharacterMovement component
    │  ├─ Velocity = CurrentVelocity + (TotalForce × DeltaSeconds)
    │  │
    │  ├─ Clamp Velocity:
    │  │  ├─ Max Speed: 2000 cm/s
    │  │  └─ Prevent unrealistic speed
    │  │
    │  ├─ SetVelocity(NewVelocity)
    │  └─ Apply movement
    │
    ├─ [STEP 6: Check Launch Condition]
    │  ├─ If Distance < 300.0:
    │  │  ├─ Check if not in cooldown
    │  │  ├─ Launch actor (see next section)
    │  │  └─ Add to launch cooldown
    │  │
    │  └─ Otherwise continue attraction
    │
    └─ Continue to next affected actor
    ↓
[Tick Complete]
```

### **3.2 BP_WhirlwindActor - Launch Mechanic**

**Custom Function: LaunchActor**

```
Function Declaration:
├─ Inputs: ActorToLaunch (Pawn)
├─ Outputs: None
├─ Access: Private
└─ Graph: Create Node Graph

Node Graph:

[Launch Actor]
    ↓
[Get Launch Parameters]
    ├─ Switch on WhirlwindType
    │  ├─ FrostVortex: LaunchVel = 2500.0
    │  ├─ InfernoTornado: LaunchVel = 4000.0
    │  └─ AquaVortex: LaunchVel = 3000.0
    │
    └─ Up Vector = (0, 0, 1)
    ↓
[Apply Launch Impulse]
    ├─ Get Character Movement Component
    ├─ Get Current Velocity
    ├─ Keep X/Y, replace Z
    │
    ├─ NewVelocity.Z = LaunchVelocity
    └─ SetVelocity(NewVelocity)
    ↓
[Disable Attraction Briefly]
    ├─ Add actor to launch cooldown
    ├─ Store in LaunchCooldownActors array
    ├─ Set timer: 2.0 seconds
    │  └─ Remove from array after timeout
    │
    └─ Check in attraction logic: if in cooldown → skip
    ↓
[Spawn Launch Effect]
    ├─ Spawn Particle System
    │  ├─ Particle: PT_Whirlwind_Launch (create this!)
    │  ├─ Location: Actor position
    │  └─ Duration: 0.5 seconds
    │
    ├─ Play Sound (multicast)
    │  ├─ Switch on Type:
    │  │  ├─ Frost: SFX_Whirlwind_Frost_Launch
    │  │  ├─ Inferno: SFX_Whirlwind_Inferno_Launch
    │  │  └─ Aqua: SFX_Whirlwind_Aqua_Launch
    │  │
    │  └─ Broadcast to all clients
    │
    └─ Camera Shake (local player only)
       ├─ If Actor = Player
       ├─ Play Camera Shake
       │  ├─ Intensity: 2.0
       │  ├─ Duration: 0.3s
       │  └─ Frequency: 10 Hz
       │
       └─ Dramatic effect!
    ↓
[Complete Launch]
```

### **3.3 BP_WhirlwindActor - Movement System**

**Event Graph: Tick Event (Movement part)**

```
[Event Tick]
    ├─ DeltaSeconds: Float
    │
    └─ Call UpdateWhirlwindPosition
       └─ Pass DeltaSeconds

[Update Whirlwind Position]
    ↓
[Get Current Target Waypoint]
    ├─ PatrolPath[CurrentPathIndex]
    └─ Store as TargetWaypoint
    ↓
[Calculate Direction]
    ├─ Direction = TargetWaypoint - Current Position
    ├─ Distance = |Direction|
    │
    ├─ Normalize Direction
    └─ Store as MoveDirection
    ↓
[Apply Movement]
    ├─ New Position = Current + (MoveDirection × MovementSpeed × DeltaSeconds)
    ├─ SetActorLocation(NewPosition)
    │
    └─ Update on all clients (replicated)
    ↓
[Check Waypoint Reached]
    ├─ If Distance < 200.0 cm:
    │  ├─ Current waypoint reached!
    │  ├─ CurrentPathIndex += 1
    │  │
    │  └─ If CurrentPathIndex >= PathArray.Length:
    │     └─ CurrentPathIndex = 0 (loop to start)
    │
    └─ Otherwise continue moving
    ↓
[Rotate Actor Toward Movement]
    ├─ Get rotation to face movement direction
    ├─ Smooth interpolation (not instant)
    │  └─ Use Rotation Interp
    │
    └─ SetActorRotation(NewRotation)
    ↓
[Complete Movement]
```

---

## ⏰ FÁZIS 4: Lifetime & Expiration

### **4.1 BP_WhirlwindActor - Lifetime Tracking**

**Event Graph: Tick Event (Duration tracking)**

```
[Event Tick]
    ├─ DeltaSeconds: Float
    │
    ├─ [Update Lifetime]
    │  ├─ CurrentLifetime += DeltaSeconds
    │  ├─ RemainingDuration = 300.0 - CurrentLifetime
    │  │
    │  └─ If RemainingDuration <= 0:
    │     ├─ Duration expired!
    │     └─ Call OnWhirlwindExpire
    │
    └─ [Fade Out Effect (Last 30 seconds)]
       ├─ If RemainingDuration < 30.0:
       │  ├─ Calculate fade factor
       │  │  └─ Alpha = RemainingDuration / 30.0
       │  │
       │  ├─ Update Particle Alpha
       │  ├─ Fade Light Intensity
       │  ├─ Fade Audio Volume
       │  │
       │  └─ Create disappearing effect
       │
       └─ Visual feedback of ending event
```

### **4.2 BP_WhirlwindEventManager - On Whirlwind Expire**

**Custom Event: OnWhirlwindExpire**

```
[Whirlwind Expired]
    ↓
[Input: ExpiredWhirlwind (BP_WhirlwindActor)]
    ↓
[Cleanup Whirlwind]
    ├─ Stop all physics
    ├─ Stop all effects
    ├─ Broadcast despawn multicast
    │
    └─ Call MulticastDespawnWhirlwind
       ├─ Type: ExpiredWhirlwind.WhirlwindType
       ├─ Location: ExpiredWhirlwind.Location
       └─ Broadcast to all clients
    ↓
[Remove from Active List]
    ├─ ActiveWhirlwinds.RemoveItem(ExpiredWhirlwind)
    ├─ Update count
    └─ Log removal
    ↓
[Destroy Actor]
    ├─ DestroyActor(ExpiredWhirlwind)
    ├─ Free memory
    └─ Cleanup complete
    ↓
[Update UI]
    ├─ Remove from HUD active events
    └─ Notify player event ended
```

---

## 📊 TESTING NODE GRAPH

### **Simple Test Setup**

**Create BP_WhirlwindTestLevel**

```
[Event Begin Play]
    ↓
[Manual Event Setup]
    ├─ Spawn BP_WhirlwindEventManager
    │  └─ Location: (0, 0, 0)
    │
    ├─ Set spawn locations
    │  ├─ Point A: (0, 0, 0)
    │  ├─ Point B: (5000, 0, 0)
    │  └─ Point C: (0, 5000, 0)
    │
    ├─ Call InitializeManager
    └─ Setup complete
    ↓
[Input Testing]
    ├─ Key T: Spawn whirlwind manually
    │  └─ Call SpawnWhirlwindEvent directly
    │
    ├─ Key Y: List active whirlwinds
    │  └─ Print count + positions
    │
    ├─ Key U: Remove all whirlwinds
    │  └─ Cleanup for testing
    │
    └─ Ready for manual QA
```

---

## 📋 DEBUGGING NODE GRAPH

**Optional: Add Debug Display**

```
[Event Tick]
    ├─ If bDebugMode = TRUE:
    │
    │  ├─ [Draw Debug Info]
    │  │  ├─ Draw Attraction Radius
    │  │  │  └─ Sphere outline
    │  │  │
    │  │  ├─ Draw Force Vectors
    │  │  │  └─ Lines showing direction
    │  │  │
    │  │  ├─ Draw Patrol Path
    │  │  │  └─ Line between waypoints
    │  │  │
    │  │  └─ Draw Affected Actors
    │  │     └─ Circle around each
    │  │
    │  └─ Print Debug Info
    │     ├─ Distance to each actor
    │     ├─ Force applied
    │     ├─ Time remaining
    │     └─ Current waypoint
    │
    └─ Visibility toggle with console cmd
```

---

## ✅ COMPLETION CHECKLIST

```
FÁZIS 1-4 COMPLETION:

[ ] BP_WhirlwindEventManager
    [ ] Variables setup (11 variables)
    [ ] Initialize function
    [ ] CheckEventTrigger function
    [ ] SpawnWhirlwindEvent function

[ ] BP_WhirlwindActor
    [ ] Components (5 total)
    [ ] Variables setup (11 variables)
    [ ] SetWhirlwindType function
    [ ] ApplyAttractionPhysics function
    [ ] LaunchActor function
    [ ] UpdateWhirlwindPosition function
    [ ] Lifetime tracking

[ ] Testing
    [ ] Manual spawn works
    [ ] Attraction pulls actors
    [ ] Launch ejects upward
    [ ] Movement works
    [ ] Expiration after 5 min
    [ ] No crashes

NEXT: Move to Fázis 5 (VFX/SFX Integration)
```

---

**Verzió**: 1.0  
**DevKit**: 5.5.4  
**Dátum**: 2026-06-22  
**Status**: 🟡 Implementálható - Blueprint lépések
