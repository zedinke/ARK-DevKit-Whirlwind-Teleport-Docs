# ⚡ Whirlwind Event System - Forgatott Szél Viharok
## Időalapú Dinamikus Térképi Esemény Rendszer

---

## 📌 Projekt Összefoglaló

Egy **dinamikus weather event rendszer**, amely:

- ✅ Megadott időn jelenik meg (Day/Night cycle szerinti időzítés)
- ✅ Három variáció: **Jeges**, **Tüzes**, **Vizes**
- ✅ Forgatott szél vortex (whirlwind) fizikával
- ✅ Vonzási mágnes effekt (radius-n belül attracts)
- ✅ Kilövési effekt (repulsion) - felfelé lökés
- ✅ Mozgás a térképen (path-based patrolling)
- ✅ Max 5 perc időtartam
- ✅ Developer kontroll: hány egyszerre
- ✅ Multicast replikáció (összes játékos látja)
- ✅ Teljes effekt rendszer

---

## 🌪️ RENDSZER KOMPONENSEI

### **1. Event Types (Variációk)**

```
JEGES WHIRLWIND (Frost Vortex)
├─ Szín: Kék/Cián
├─ Effekt: Jégkristályok, hóvihar
├─ Sérülés: Cold damage (opcional)
├─ Sebességcsökkentés: TRUE
└─ Hang: Hideg szél

TÜZES WHIRLWIND (Inferno Tornado)
├─ Szín: Vörös/Narancssárga
├─ Effekt: Tűz, szikrák, füst
├─ Sérülés: Fire damage (opcional)
├─ Sebességcsökkentés: FALSE (felgyorsít)
└─ Hang: Tüzes füstöt suhogás

VIZES WHIRLWIND (Aqua Vortex)
├─ Szín: Zöld/Világoskék
├─ Effekt: Víz spirál, cseppek
├─ Sérülés: None (safe)
├─ Sebességcsökkentés: MEDIUM
└─ Hang: Víz csobogás
```

---

## ⚙️ MANAGER BLUEPRINT - BP_WhirlwindEventManager

### **Fő Komponensek**

```
Blueprint Variables:

1. EventTriggerTime
   ├─ Type: Float (0-1440 minutes, 24h cycle)
   ├─ Default: 1080 (19:00 / 7 PM)
   ├─ Editable in editor
   └─ Day cycle synchronized

2. MaxSimultaneousWhirlwinds
   ├─ Type: Int32
   ├─ Default: 3
   ├─ Editable in editor
   ├─ Developer kontrollja
   └─ Min: 1, Max: 10

3. WhirlwindDuration
   ├─ Type: Float
   ├─ Default: 300.0 (5 minutes)
   ├─ Locked: TRUE (nem szabad módosítani)
   └─ Hard limit

4. WhirlwindSpawnLocations
   ├─ Type: Array (Vector)
   ├─ Possible spawn points on map
   ├─ Define in editor
   └─ Random selection

5. ActiveWhirlwinds
   ├─ Type: Array (BP_WhirlwindActor)
   ├─ Tracks active instances
   ├─ Managed by system
   └─ Auto-cleanup on expire

6. WhirlwindTypes
   ├─ Type: Array (Enum)
   ├─ FrostVortex
   ├─ InfernoTornado
   └─ AquaVortex

7. EventScheduleEnabled
   ├─ Type: Boolean
   ├─ Default: TRUE
   └─ Can disable entire event system

8. DayCycleManagerRef
   ├─ Type: BP_DayCycleManager
   ├─ Get current time
   └─ Synchronize events

9. LastEventTriggerTime
   ├─ Type: Float
   ├─ Track cooldown
   └─ Prevent spam triggers
```

### **Manager Functions**

#### **Function 1: Initialize Manager**

Node Graph:
```
[Initialize Manager]
    ↓
[Get or Create Day Cycle Manager]
    ├─ Find in world
    └─ Store reference
    ↓
[Setup Whirlwind Spawn Points]
    ├─ Define map locations
    ├─ Create random path between points
    └─ Store patrol route
    ↓
[Setup Event Schedule]
    ├─ Convert time to minutes
    ├─ Set trigger time
    └─ Start event loop
    ↓
[Initialize Event Loop]
    ├─ Set timer
    └─ Check every 10 seconds
```

#### **Function 2: Check Event Schedule**

```
[Check Event Trigger]
    ↓
[Get Current Game Time]
    ├─ From Day Cycle Manager
    └─ In minutes (0-1440)
    ↓
[Calculate Time Until Event]
    ├─ EventTriggerTime - CurrentTime
    ├─ Handle day wrap-around
    │  └─ If negative → Add 1440 minutes
    │
    └─ TimeUntilEvent = Result
    ↓
[Check if Should Trigger]
    ├─ Is EventScheduleEnabled?
    │  ├─ NO → Exit
    │  └─ YES → Continue
    │
    ├─ TimeUntilEvent < 1.0 minute?
    │  ├─ NO → Exit
    │  └─ YES → Should trigger!
    │
    ├─ ActiveWhirlwinds count < MaxSimultaneous?
    │  ├─ NO → Already at max, exit
    │  └─ YES → Spawn new!
    │
    └─ LastEventTriggerTime > 10 minutes?
       ├─ Cooldown check
       └─ Prevent spam
    ↓
[Spawn Event]
    └─ Call SpawnWhirlwindEvent
```

#### **Function 3: Spawn Whirlwind Event**

```
[Spawn Whirlwind Event]
    ↓
[Input Parameters]
    ├─ SelectedType (Enum: Frost/Inferno/Aqua)
    ├─ SpawnLocation (Vector)
    └─ PatrolPath (Array of Vectors)
    ↓
[Create Whirlwind Actor]
    ├─ Spawn BP_WhirlwindActor
    ├─ At random location from list
    ├─ Set whirlwind type
    └─ Store reference in ActiveWhirlwinds array
    ↓
[Initialize Whirlwind]
    ├─ Set event type (frost/inferno/aqua)
    ├─ Set patrol route
    ├─ Set duration (300 seconds)
    ├─ Start movement
    └─ Start attraction physics
    ↓
[Broadcast Event to All Clients]
    ├─ Multicast RPC call
    ├─ Show VFX on all clients
    ├─ Play sound on all clients
    └─ Synchronized
    ↓
[Update Active Count]
    ├─ Add to array
    ├─ Log event
    └─ Set expiration timer
```

#### **Function 4: On Whirlwind Expire**

```
[Whirlwind Expires (5 min)]
    ↓
[Cleanup Whirlwind]
    ├─ Stop attraction physics
    ├─ Spawn exit effect
    ├─ Broadcast disappearance
    └─ Multicast to all clients
    ↓
[Remove from Active List]
    ├─ Remove from ActiveWhirlwinds array
    └─ Log removal
    ↓
[Destroy Actor]
    ├─ Safe cleanup
    ├─ Free memory
    └─ Nullify references
```

---

## 🌪️ BP_WhirlwindActor - Az Event Aktőr

### **Komponensek**

```
Root Structure:

Scene Root (Default)
├─ Sphere Collision (Attraction Zone)
│  ├─ Radius: 2000 cm (adjustable)
│  ├─ Generate Overlap Events: TRUE
│  ├─ Ignore Player: FALSE
│  └─ Ignore Creatures: FALSE
│
├─ Particle System (Visual Vortex)
│  ├─ Template: PT_Whirlwind_Base
│  ├─ Rotation: Spinning (Y-axis)
│  ├─ Scale: Size-type based
│  └─ Auto-update with type
│
├─ Point Light (Glow)
│  ├─ Color: Type-dependent
│  ├─ Intensity: 3000 lm
│  ├─ Radius: 2000 cm
│  └─ Dynamic: FALSE (perf)
│
├─ Audio Component
│  ├─ Sound: Type-specific
│  ├─ Loop: TRUE
│  ├─ Volume: 1.0
│  └─ Attenuation: 3000 cm
│
└─ Arrow Component (Debug)
   └─ Show direction of movement
```

### **Variables**

```
Variables:

1. WhirlwindType
   ├─ Type: EWhirlwindType Enum
   ├─ FrostVortex
   ├─ InfernoTornado
   └─ AquaVortex

2. WhirlwindColors
   ├─ Frost: Cyan (0, 1, 1)
   ├─ Inferno: Red (1, 0.3, 0)
   └─ Aqua: Green (0, 1, 0.5)

3. AttractionRadius
   ├─ Type: Float
   ├─ Default: 2000 cm
   └─ Type affects radius

4. AttractionForce
   ├─ Type: Float
   ├─ Default: 2000.0 cm/s²
   └─ Pull towards center

5. CurrentLifetime
   ├─ Type: Float
   ├─ Tracks how long alive
   └─ Used for fade-out timing

6. RemainingDuration
   ├─ Type: Float
   ├─ Max 300 seconds
   └─ Countdown timer

7. PatrolPath
   ├─ Type: Array (Vector)
   ├─ Waypoints to visit
   └─ Random between points

8. CurrentPathIndex
   ├─ Type: Int32
   ├─ Tracking current waypoint
   └─ For movement

9. MovementSpeed
   ├─ Type: Float
   ├─ Default: 500 cm/s
   └─ How fast whirlwind moves

10. AffectedActors
    ├─ Type: Array (Actor)
    ├─ Currently being attracted
    └─ For tracking/networking
```

---

## 📊 PHYSICS & ATTRACTION SYSTEM

### **Attraction Mechanics - Node Graph**

```
[Whirlwind Event Tick]
    ├─ Every frame
    │
    ├─ [Get Attracted Actors]
    │  ├─ Sphere overlap check
    │  ├─ Radius: AttractionRadius
    │  └─ Get all pawns in radius
    │
    ├─ [For Each Affected Actor]
    │  ├─ Check if player or creature
    │  │
    │  ├─ [Calculate Attraction Vector]
    │  │  ├─ Direction: WhirlwindCenter → Actor
    │  │  ├─ Distance: Actual distance
    │  │  ├─ Falloff: Quadratic distance falloff
    │  │  │  └─ Force = (MaxForce / Distance²)
    │  │  │
    │  │  └─ Apply toward center
    │  │
    │  ├─ [Add Rotational Force]
    │  │  ├─ Vortex rotation around center
    │  │  ├─ Tangential velocity
    │  │  └─ Makes circular motion
    │  │
    │  ├─ [Apply Forces]
    │  │  ├─ Get character movement component
    │  │  ├─ Add force vector
    │  │  ├─ Add gravity compensation
    │  │  └─ Clamp max speed
    │  │
    │  ├─ [Update Position Tracking]
    │  │  ├─ Add to AffectedActors array
    │  │  └─ Mark for upward launch (later)
    │  │
    │  └─ Continue to next actor
    │
    └─ [Complete Frame]
```

### **Repulsion (Launch Upward) Mechanic**

```
[When Actor Reaches Center]
    ├─ Distance to center < 300 cm?
    │  ├─ YES → Trigger launch
    │  └─ NO → Continue pulling
    │
    ├─ [Apply Repulsion Force]
    │  ├─ Direction: Straight UP (Z-axis)
    │  ├─ Force: 3000 cm/s² upward
    │  ├─ Duration: 0.5 seconds
    │  └─ Visual: Particle burst
    │
    ├─ [Eject from Vortex]
    │  ├─ Remove from AffectedActors
    │  ├─ Disable attraction for 2 seconds
    │  ├─ Add cooldown
    │  └─ Prevent re-attraction immediately
    │
    └─ [Complete Launch]
```

---

## 🚶 MOVEMENT SYSTEM - Whirlwind Mozgás

### **Patrol Path Movement**

```
[Whirlwind Movement Tick]
    ↓
[Get Current Target Waypoint]
    ├─ PatrolPath[CurrentPathIndex]
    └─ Get vector position
    ↓
[Calculate Direction to Target]
    ├─ Target - Current Position
    ├─ Normalize
    └─ Get unit vector
    ↓
[Apply Movement]
    ├─ New Position = Current + (Direction × MovementSpeed × DeltaTime)
    ├─ Set Location
    └─ Update all clients
    ↓
[Check Distance to Waypoint]
    ├─ Distance < 200 cm?
    │  ├─ YES → Next waypoint
    │  ├─ CurrentPathIndex += 1
    │  └─ Check if at end → wrap to 0
    │
    └─ NO → Continue moving
```

### **Spawn Point Selection**

```
[On Manager Init]
    ├─ Define map patrol points
    │  ├─ Point A: Center field
    │  ├─ Point B: North edge
    │  ├─ Point C: South edge
    │  ├─ Point D: East area
    │  └─ Point E: West area
    │
    ├─ Create patrol routes
    │  └─ Random waypoint sequence
    │
    └─ Store in WhirlwindSpawnLocations array
```

---

## 🎨 VFX EFFEKT RENDSZER

### **Particle Systems (3 Variáció)**

#### **1. PT_Whirlwind_Frost (Jeges)**

```
Characteristics:
├─ Base Shape: Vertical spiral cone
├─ Color: Cyan/Blue gradient
│  ├─ Start: Bright cyan
│  ├─ Middle: Medium blue
│  └─ End: Dark transparent
│
├─ Particles:
│  ├─ Ice crystals
│  ├─ Snow flakes
│  ├─ Mist/fog
│  └─ Sparkles
│
├─ Animation:
│  ├─ Rotation: 360° per second (Y-axis)
│  ├─ Float up: Gentle upward movement
│  ├─ Spawn rate: 300 particles/sec
│  └─ Duration: Permanent (loop)
│
├─ Lifetime:
│  ├─ Min: 2.0 seconds
│  ├─ Max: 5.0 seconds
│  └─ Fade out: Last 0.5s transparent
│
├─ Size:
│  ├─ Start: 10-30 cm
│  ├─ End: 5-10 cm
│  └─ Taper toward center
│
└─ Material:
   ├─ Additive blend
   ├─ Emissive: Cyan glow
   └─ Brightness: 1.5x
```

#### **2. PT_Whirlwind_Inferno (Tüzes)**

```
Characteristics:
├─ Base Shape: Vertical fiery tornado
├─ Color: Red/Orange/Yellow gradient
│  ├─ Core: Bright orange
│  ├─ Middle: Red
│  └─ Outer: Dark smoke
│
├─ Particles:
│  ├─ Fire sparks
│  ├─ Smoke wisps
│  ├─ Embers
│  └─ Heat shimmer
│
├─ Animation:
│  ├─ Rotation: 540° per second (faster!)
│  ├─ Rise: Animated upward flow
│  ├─ Spawn rate: 400 particles/sec
│  └─ Duration: Permanent
│
├─ Lifetime:
│  ├─ Min: 1.5 seconds (shorter)
│  ├─ Max: 4.0 seconds
│  └─ Fade: Quick disappear
│
├─ Size:
│  ├─ Larger particles
│  └─ More chaotic spread
│
└─ Material:
   ├─ Additive blend
   ├─ Emissive: Red/Yellow glow
   └─ Brightness: 2.0x (very bright!)
```

#### **3. PT_Whirlwind_Aqua (Vizes)**

```
Characteristics:
├─ Base Shape: Smooth water spiral
├─ Color: Green/Blue/Cyan
│  ├─ Surface: Light blue
│  ├─ Core: Deeper blue
│  └─ Mist: Transparent cyan
│
├─ Particles:
│  ├─ Water droplets
│  ├─ Splash effects
│  ├─ Mist/steam
│  └─ Bubble effects
│
├─ Animation:
│  ├─ Rotation: 360° per second
│  ├─ Wave motion: Horizontal flow
│  ├─ Spawn rate: 350 particles/sec
│  └─ Duration: Permanent
│
├─ Lifetime:
│  ├─ Min: 2.5 seconds
│  ├─ Max: 6.0 seconds
│  └─ Fade: Smooth transition
│
├─ Size:
│  ├─ Medium particles
│  └─ Smooth distribution
│
└─ Material:
   ├─ Additive blend (water-like)
   ├─ Emissive: Cyan glow
   └─ Brightness: 1.2x
```

---

## 🔊 AUDIO SYSTEM

### **Sound Effects (3 Types)**

#### **Frost Vortex Audio**

```
Primary Sound: SFX_Whirlwind_Frost_Loop
├─ Duration: 2.0 seconds (loopable)
├─ Frequency: Low-mid (500-3000 Hz)
├─ Characteristics:
│  ├─ Howling wind sound
│  ├─ Crystalline shimmer
│  ├─ Cold, eerie tone
│  └─ Slight echo
│
├─ Volume: 0.7
├─ Pitch: 1.0
├─ Attenuation: 5000 cm
└─ 3D Spatialization: TRUE

Enhancement Sounds:
├─ SFX_Whirlwind_Frost_Attract
│  └─ Ice crackle on attraction
│
└─ SFX_Whirlwind_Frost_Launch
   └─ Crystalline burst on ejection
```

#### **Inferno Tornado Audio**

```
Primary Sound: SFX_Whirlwind_Inferno_Loop
├─ Duration: 1.5 seconds (loopable)
├─ Frequency: Mid-high (2000-8000 Hz)
├─ Characteristics:
│  ├─ Roaring fire sound
│  ├─ Crackling flames
│  ├─ Hot rushing wind
│  └─ Chaotic, intense
│
├─ Volume: 0.9
├─ Pitch: 1.1 (slightly higher)
├─ Attenuation: 4000 cm
└─ 3D Spatialization: TRUE

Enhancement Sounds:
├─ SFX_Whirlwind_Inferno_Attract
│  └─ Flames ignite sound
│
└─ SFX_Whirlwind_Inferno_Launch
   └─ Explosive burst sound
```

#### **Aqua Vortex Audio**

```
Primary Sound: SFX_Whirlwind_Aqua_Loop
├─ Duration: 2.5 seconds (loopable)
├─ Frequency: Low-mid (300-4000 Hz)
├─ Characteristics:
│  ├─ Swirling water sound
│  ├─ Splash effects
│  ├─ Gentle rushing
│  └─ Calm but powerful
│
├─ Volume: 0.8
├─ Pitch: 0.95 (slightly lower)
├─ Attenuation: 5000 cm
└─ 3D Spatialization: TRUE

Enhancement Sounds:
├─ SFX_Whirlwind_Aqua_Attract
│  └─ Water suction sound
│
└─ SFX_Whirlwind_Aqua_Launch
   └─ Water splash ejection
```

---

## 🔄 MULTICAST REPLIKÁCIÓ

### **Network Events**

#### **Event 1: Whirlwind Spawned (Multicast)**

```
[On Whirlwind Created]
    ↓
[Server-side]
    ├─ Create actor instance
    ├─ Calculate position
    └─ Call Multicast RPC
    ↓
[Call MulticastSpawnWhirlwind]
    ├─ Whirlwind Type
    ├─ Location
    ├─ Rotation
    ├─ Patrol Path
    └─ Broadcast to ALL clients
    ↓
[All Clients Receive]
    ├─ Spawn particle system
    ├─ Start audio loop
    ├─ Update UI (show active event)
    └─ Add to visual list
```

#### **Event 2: Whirlwind Expired (Multicast)**

```
[On Duration Expires]
    ↓
[Server-side]
    ├─ Stop movement
    ├─ Stop attraction
    └─ Call Multicast RPC
    ↓
[Call MulticastDespawnWhirlwind]
    ├─ Whirlwind ID
    ├─ Despawn location
    └─ Broadcast to ALL clients
    ↓
[All Clients Receive]
    ├─ Play exit effect
    ├─ Stop audio gracefully
    ├─ Fade out particles
    ├─ Update UI (remove from list)
    └─ Remove from visual representation
```

#### **Event 3: Player Attracted (Replicated)**

```
[On Actor Enters Attraction Zone]
    ↓
[Server validates attraction]
    ├─ Apply physics force
    ├─ Track in AffectedActors
    └─ Replicate position updates
    ↓
[All Clients See]
    ├─ Character being pulled
    ├─ Animation state (being pushed)
    ├─ Particle effects on character
    └─ Real-time position sync
```

---

## 🎮 DEVELOPER KONTROLL & EDITOR

### **Editable Parameters**

Az editor-ben editable:

```
Whirlwind Event Manager Details:

Public Variables (editable):
├─ ☐ Event Schedule Enabled
│  └─ Checkbox to toggle entire system
│
├─ 🕐 Event Trigger Time
│  ├─ In minutes (0-1440)
│  ├─ Example: 1080 = 18:00 (6 PM)
│  └─ Dropdown: Preset times
│
├─ 📊 Max Simultaneous Whirlwinds
│  ├─ Slider: 1-10
│  ├─ Default: 3
│  └─ Info text: "Max concurrent events"
│
├─ 📍 Spawn Location Points
│  ├─ Array editor
│  ├─ Add/remove points
│  └─ Visualize in viewport
│
├─ 🎨 Available Whirlwind Types
│  ├─ Checkboxes for each:
│  │  ├─ Frost Vortex
│  │  ├─ Inferno Tornado
│  │  └─ Aqua Vortex
│  │
│  └─ Random selection from enabled
│
└─ 📁 Asset References
   ├─ Particle systems
   ├─ Sound cues
   └─ Material assignments
```

### **Developer Console Commands**

```
Available Commands:

1. Trigger Event Manually
   └─ /whirlwind start [FrostVortex|InfernoTornado|AquaVortex]
      └─ Immediately spawn event

2. Set Max Count
   └─ /whirlwind max 5
      └─ Change simultaneously limit

3. Enable/Disable System
   └─ /whirlwind enabled true/false
      └─ Toggle entire event system

4. Test Attraction
   └─ /whirlwind testattraaction
      └─ Apply attraction force to self

5. List Active Events
   └─ /whirlwind list
      └─ Show all active whirlwinds

6. Set Event Time
   └─ /whirlwind time 1080
      └─ Set trigger time (minutes)
```

---

## 📋 TESZTELÉSI CHECKLIST

### **Functionality Tests**

```
Schedule & Timing:
- [ ] Event triggers at correct time
- [ ] Day cycle sync works
- [ ] Cooldown prevents spam
- [ ] Max simultaneous limit respected
- [ ] 5-minute expiration works

Attraction Physics:
- [ ] Actors pulled toward center
- [ ] Falloff calculation correct
- [ ] Rotation around center smooth
- [ ] No physics clipping
- [ ] Smooth interpolation

Launch Mechanic:
- [ ] Actors launched upward properly
- [ ] Launch velocity correct
- [ ] Cooldown on re-attraction working
- [ ] Launch animation smooth
- [ ] Safe landing handling

Movement:
- [ ] Whirlwind moves along path
- [ ] Waypoint transitions smooth
- [ ] Speed consistent
- [ ] Path loops properly
- [ ] No teleporting

Multicast/Network:
- [ ] All clients see event simultaneously
- [ ] Attraction synced for all players
- [ ] Position updates smooth
- [ ] No desync issues
- [ ] Works with 500ms latency

VFX/SFX:
- [ ] Correct particle type (frost/inferno/aqua)
- [ ] Correct sound loop
- [ ] Colors visible
- [ ] Light glow visible
- [ ] Audio mixes well
- [ ] No performance drops

UI:
- [ ] Event shows on HUD
- [ ] Information displayed correctly
- [ ] Timer counting down
- [ ] Removal when expires
- [ ] No errors

Developer Controls:
- [ ] Console commands work
- [ ] Editor parameters adjust correctly
- [ ] Manual trigger works
- [ ] Max count changes applied
- [ ] Toggle enable/disable works
```

---

## 🎯 PROJEKT FÁZISOK

### **Fázis 1: Foundation (Hét 1-2) - 20-24 óra**

```
✓ Create BP_WhirlwindEventManager
  ├─ Variables setup
  ├─ Initialize function
  └─ Basic structure

✓ Create BP_WhirlwindActor
  ├─ Components (sphere, particle, light)
  ├─ Variables
  └─ Basic lifecycle

✓ Setup spawning system
  ├─ Spawn locations
  └─ Basic actor creation
```

### **Fázis 2: Core Mechanics (Hét 2-3) - 24-28 óra**

```
✓ Attraction physics
  ├─ Gravity calculation
  ├─ Force application
  ├─ Falloff system
  └─ Actor tracking

✓ Repulsion/Launch system
  ├─ Center detection
  ├─ Upward force
  └─ Cooldown tracking

✓ Movement system
  ├─ Patrol path following
  ├─ Waypoint navigation
  └─ Speed control
```

### **Fázis 3: Scheduling (Hét 3) - 16-20 óra**

```
✓ Day cycle integration
  ├─ Time sync
  ├─ Trigger calculation
  └─ Schedule checking

✓ Manager event loop
  ├─ Periodic checks
  ├─ Event spawning
  └─ Max count enforcement
```

### **Fázis 4: Multicast Network (Hét 3-4) - 16-20 óra**

```
✓ Server RPC functions
  ├─ Spawn/despawn multicast
  ├─ Position replication
  └─ State synchronization

✓ Client replication
  ├─ Receive spawn events
  ├─ Update positions
  └─ Show effects
```

### **Fázis 5: Effects (Hét 4-5) - 20-24 óra**

```
✓ VFX particles (3 types)
  ├─ Frost vortex
  ├─ Inferno tornado
  └─ Aqua vortex

✓ SFX audio (3 types)
  ├─ Frost loop + enhancers
  ├─ Inferno loop + enhancers
  └─ Aqua loop + enhancers

✓ Light & visual effects
  ├─ Dynamic lights
  ├─ Color transitions
  └─ Glow effects
```

### **Fázis 6: Developer Controls (Hét 5) - 12-16 óra**

```
✓ Editor parameters
  ├─ Trigger time slider
  ├─ Max count slider
  ├─ Type checkboxes
  └─ Location point array

✓ Console commands
  ├─ Manual spawn
  ├─ Parameter changes
  ├─ Enable/disable
  └─ Debug info

✓ UI display
  ├─ Active event indicator
  ├─ Timer display
  └─ Information panel
```

### **Fázis 7: Testing & Polish (Hét 6) - 16-20 óra**

```
✓ Multiplayer testing
  ├─ 2-4 player tests
  ├─ Attraction sync
  ├─ Launch mechanics
  └─ Network latency

✓ Physics refinement
  ├─ Attraction feel
  ├─ Launch velocity
  ├─ No bugs/clipping
  └─ Smooth animations

✓ Performance optimization
  ├─ Particle count
  ├─ Physics calculations
  ├─ Network bandwidth
  └─ Frame rate
```

### **Fázis 8: Documentation (Hét 6-7) - 8-12 óra**

```
✓ Complete documentation
  ├─ Setup guide
  ├─ Configuration guide
  ├─ Troubleshooting
  └─ Code comments
```

**TELJES PROJEKT: 80-120 óra**

---

## 📁 DOKUMENTÁCIÓ KELL

1. **whirlwind-rendszer-általános.md** - Teljes koncepció
2. **whirlwind-blueprint-guide.md** - Lépésről-lépésre
3. **whirlwind-physics-advanced.md** - Vonzás/repulzió matematika
4. **whirlwind-effects-vfx-sfx.md** - Particle & audio setup
5. **whirlwind-networking-multicast.md** - Network replikáció
6. **whirlwind-scheduling-timing.md** - Time-based events
7. **whirlwind-project-tracker.md** - Menedzselés & progress

---

**Verzió**: 1.0  
**DevKit**: 5.5.4  
**Dátum**: 2026-06-22  
**Status**: 🟡 Tervezett - Dokumentáció indítása
