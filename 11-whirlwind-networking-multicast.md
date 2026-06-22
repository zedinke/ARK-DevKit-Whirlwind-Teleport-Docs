# ⚡ Whirlwind Event System - Networking & Multicast RPC
## Server Authority és Network Replikáció

---

## 📌 Network Architecture Overview

Az whirlwind esemény rendszer **server-authoritative** architecture-et használ, ahol a szerver teljes kontroll alatt tartja az eseményeket, és a klienseknek csak az effekteket mutatja.

```
NETWORK FLOW:

[Client]
    ├─ Only sees effects
    ├─ No local physics
    └─ Receives position updates

[Server]
    ├─ Runs event scheduling
    ├─ Calculates physics (attraction)
    ├─ Manages spawning/despawning
    └─ Sends updates to all clients
```

---

## 🔄 EVENT REPLICATION FLOW

### **Step 1: Whirlwind Spawn Event**

```
[Server: CheckEventTrigger]
    ├─ Event time reached
    ├─ Spawn BP_WhirlwindActor
    ├─ Initialize properties
    └─ Call Multicast RPC
    ↓
[Multicast: SpawnWhirlwindEffects]
    ├─ All clients receive call
    ├─ Spawn particle effects
    ├─ Play audio loops
    ├─ Create light glow
    └─ Update UI
    ↓
[Result: All players see event simultaneously!]
```

### **Step 2: Whirlwind Position Updates**

```
[Server Tick]
    ├─ Calculate new position
    ├─ Update actor location
    ├─ Replicate position via variable
    └─ Every 0.1 seconds
    ↓
[Client Tick]
    ├─ Receive position update
    ├─ Smooth interpolation
    ├─ Update visual position
    └─ Vortex moves smoothly
```

### **Step 3: Player Attraction**

```
[Server: ApplyAttractionPhysics]
    ├─ Calculate forces
    ├─ Apply to player character
    ├─ Replicate new velocity
    └─ Every frame
    ↓
[Client: Receive Replication]
    ├─ Get updated velocity
    ├─ Player moves visually
    ├─ Smooth movement
    └─ Predicts locally
```

### **Step 4: Whirlwind Despawn**

```
[Server: OnWhirlwindExpire]
    ├─ 300 seconds elapsed
    ├─ Stop movement
    ├─ Call Multicast RPC
    └─ Cleanup on server
    ↓
[Multicast: DespawnWhirlwindEffects]
    ├─ Fade out particles
    ├─ Fade audio
    ├─ Fade light
    ├─ Remove from UI
    └─ All clients cleanup
```

---

## 🔐 SERVER AUTHORITY IMPLEMENTATION

### **Replication Variables - BP_WhirlwindEventManager**

```
Variables that must be Replicated:

1. ActiveWhirlwinds (Array of BP_WhirlwindActor)
   ├─ Replication: Replicated
   ├─ Owned by: Server
   └─ Tells clients which whirlwinds exist

2. LastEventTriggerTime (Float)
   ├─ Replication: Replicated
   ├─ On Rep Notify: OnRep_EventTriggered
   └─ Tells clients event just happened

3. MaxSimultaneousWhirlwinds (Int32)
   ├─ Replication: Replicated
   ├─ Owner: Server
   └─ Config shared with clients

4. EventScheduleEnabled (Boolean)
   ├─ Replication: Replicated
   ├─ Owner: Server
   └─ Can toggle remotely
```

### **Replication Variables - BP_WhirlwindActor**

```
Critical Variables:

1. WhirlwindType (EWhirlwindType)
   ├─ Replication: Replicated
   ├─ Set once at creation
   └─ Determines visuals on clients

2. ActorLocation (Vector)
   ├─ Replicated automatically by engine
   ├─ Movement synced every 0.1s
   └─ Used for positioning effects

3. RemainingDuration (Float)
   ├─ Replication: Replicated
   ├─ On Rep Notify: OnRep_DurationChanged
   └─ Clients know when expiring

4. PatrolPath (Array of Vector)
   ├─ Replication: Replicated
   ├─ Set at spawn
   └─ Clients know movement path

5. CurrentPathIndex (Int32)
   ├─ Replication: Replicated
   ├─ On Rep Notify: OnRep_WaypointChanged
   └─ Smooth visual waypoint transitions
```

---

## 📡 MULTICAST RPC FUNCTIONS

### **Function 1: Server Execute → Client Multicast**

**Function: MulticastSpawnWhirlwindEffects**

```
Function Declaration:
├─ Name: MulticastSpawnWhirlwindEffects
├─ Type: Multicast Reliable RPC
├─ Access: Public
└─ Runs on: All clients + server

Input Parameters:
├─ WhirlwindType (EWhirlwindType)
├─ SpawnLocation (Vector)
├─ ParticleTemplate (Class reference)
├─ AudioCue (Sound Cue)
└─ LightColor (Linear Color)

Execution Scope:
├─ Server: Executes first
├─ All Clients: Receive and execute
└─ Order: Server processes first, then broadcasts
```

### **Multicast Implementation - Node Graph**

```
[MulticastSpawnWhirlwindEffects]
    ↓
[Validation]
    ├─ Is SpawnLocation valid?
    │  ├─ YES → Continue
    │  └─ NO → Return (invalid)
    │
    └─ Is WhirlwindType valid?
       ├─ YES → Continue
       └─ NO → Return (invalid)
    ↓
[Spawn Particle Effect]
    ├─ Get World
    ├─ Spawn Emitter Attached
    │  ├─ Emitter Class: ParticleTemplate
    │  ├─ Location: SpawnLocation
    │  ├─ Attach to Owner: this actor
    │  └─ Duration: 300.0 (match event)
    │
    ├─ Activate emitter
    └─ Store reference for cleanup
    ↓
[Play Audio Loop]
    ├─ Get Audio Component
    ├─ Set Sound Cue: AudioCue
    ├─ Set Volume Multiplier:
    │  ├─ FrostVortex: 0.7
    │  ├─ InfernoTornado: 0.85
    │  └─ AquaVortex: 0.75
    │
    ├─ Set Pitch Multiplier:
    │  ├─ FrostVortex: 1.0
    │  ├─ InfernoTornado: 1.1
    │  └─ AquaVortex: 0.95
    │
    ├─ Set Loop: TRUE
    ├─ Activate audio
    └─ Audio starts immediately
    ↓
[Create Light Glow]
    ├─ Create Point Light Component
    ├─ Set Light Color: LightColor
    ├─ Set Intensity:
    │  ├─ FrostVortex: 2000 lm
    │  ├─ InfernoTornado: 3500 lm
    │  └─ AquaVortex: 2200 lm
    │
    ├─ Set Attenuation Radius: 2500 cm
    ├─ Attach to this actor
    └─ Light glows immediately
    ↓
[Update UI]
    ├─ Get HUD reference
    ├─ Add whirlwind to active list
    ├─ Update counter (X active events)
    └─ Show notification
    ↓
[Log Event (Debug)]
    ├─ If verbose logging enabled:
    │  ├─ Print: "Whirlwind spawned: [Type]"
    │  └─ Print location
    │
    └─ Network debug info
    ↓
[Complete Spawn - All Clients Done!]
```

### **Function 2: Despawn Multicast**

**Function: MulticastDespawnWhirlwindEffects**

```
Function Declaration:
├─ Name: MulticastDespawnWhirlwindEffects
├─ Type: Multicast Reliable RPC
├─ Access: Public
└─ Runs on: All clients + server

Input Parameters:
├─ DespawnLocation (Vector)
├─ WhirlwindType (EWhirlwindType)
└─ WhirlwindID (Integer - for reference)

Implementation:
├─ [Fade Out Particles]
│  ├─ Get particle component
│  ├─ Deactivate emitter
    ├─ Fade duration: 1.0 second
│  └─ Particles drift and fade
│
├─ [Fade Out Audio]
│  ├─ Get audio component
│  ├─ Fade volume: 1.0 → 0.0
│  ├─ Duration: 0.5 seconds
│  └─ Stop audio gracefully
│
├─ [Fade Out Light]
│  ├─ Get light component
│  ├─ Fade intensity: Current → 0
│  ├─ Duration: 0.5 seconds
│  └─ Light dims and disappears
│
├─ [Cleanup]
│  ├─ Destroy particle component
│  ├─ Destroy audio component
│  ├─ Destroy light component
│  └─ Remove from visual list
│
├─ [Update UI]
│  ├─ Remove from active list
│  ├─ Update counter
│  └─ Show "Event ended" message
│
└─ [Complete Cleanup]
```

---

## 🎯 PLAYER ATTRACTION REPLICATION

### **Server-Side Physics**

```
[Server: ApplyAttractionPhysics - Tick]
    ↓
[Calculate Forces]
    ├─ Get overlapping actors
    ├─ For each actor:
    │  ├─ Calculate attraction vector
    │  ├─ Apply force
    │  └─ Update velocity
    │
    └─ Update character velocity
    ↓
[Replicate to Clients]
    ├─ Character velocity updated
    ├─ Engine replicates automatically
    │  └─ Via CharacterMovement component
    │
    └─ Clients receive updates every frame
```

### **Client-Side Prediction**

```
[Client: Receives Position Update]
    ↓
[Predict Movement]
    ├─ Get current velocity
    ├─ Apply physics locally
    │  └─ Character moves client-side
    │
    └─ Wait for server correction
    ↓
[Server Sends Update]
    ├─ If difference > threshold:
    │  ├─ Send correction
    │  └─ Snap to server position
    │
    └─ Otherwise: Trust prediction
    ↓
[Result: Smooth movement with minimal updates]
```

---

## ⚡ LAUNCH/EJECTION SYNCHRONIZATION

### **Launch Event Network Flow**

```
[Server: LaunchActor Function]
    ↓
[Calculate Launch Velocity]
    ├─ Get actor position
    ├─ Distance to center < 300?
    ├─ Calculate launch force (type-dependent)
    └─ Apply velocity
    ↓
[Replicate Launch]
    ├─ Character velocity changed
    │  └─ Replicated via movement component
    │
    ├─ Call Multicast RPC for effects
    │  └─ Launch sound + particles
    │
    └─ All clients see launch simultaneously
    ↓
[Client Side]
    ├─ Receives velocity update
    ├─ Character moves upward
    ├─ Hears launch sound
    ├─ Sees particles burst
    └─ Visible on all players!
```

---

## 📊 BANDWIDTH OPTIMIZATION

### **Network Traffic Reduction**

```
NAIVE APPROACH (TOO MUCH BANDWIDTH):
├─ Send every actor position every frame
├─ 50 actors × 3 floats (X,Y,Z) × 60 FPS
│  └─ = ~54,000 values/sec = TOO MUCH!
│
└─ Would lag multiplayer!

OPTIMIZED APPROACH (USED HERE):
├─ Server-only physics calculation
├─ Only broadcast key events:
│  ├─ Whirlwind spawned (once)
│  ├─ Position updates (0.1s interval, not per frame)
│  ├─ Launch events (per actor, once)
│  └─ Whirlwind despawn (once)
│
├─ Client predicts movement between updates
│  └─ Smooth interpolation locally
│
└─ ~90% bandwidth reduction!
```

### **Position Update Frequency**

```
Options:

Every Frame (BAD):
├─ 60 FPS × 3 floats × 50 vortexes
├─ Massive bandwidth
└─ Network congestion

Every 0.1 Seconds (GOOD - USED):
├─ 10 updates/sec × 3 floats × 50 vortexes
├─ Much better bandwidth
├─ Smooth visual movement
└─ Client prediction fills gaps

Every 0.5 Seconds (ACCEPTABLE):
├─ 2 updates/sec
├─ Lower bandwidth
└─ Client prediction more obvious

Compromise: 0.1 seconds (10 Hz)
├─ Bandwidth: ~30% of per-frame
├─ Smoothness: Excellent
└─ Best balance
```

---

## 🔐 ANTI-CHEAT & VALIDATION

### **Server-Side Validation**

```
[Before Applying Attraction Force]
    ├─ Verify actor is in game
    ├─ Check actor is alive
    ├─ Verify actor in range
    │  └─ Distance < 2500 cm
    │
    ├─ Check no speed cap exceeded
    │  └─ Velocity < 2000 cm/s
    │
    └─ Apply force if all checks pass
```

### **Client Anti-Cheat**

```
WHAT CLIENT CANNOT DO:
├─ Affect attraction physics
├─ Change vortex position
├─ Avoid launch mechanic
├─ Modify event timing
└─ Cheat event spawning

WHY SAFE:
├─ Server calculates all physics
├─ Clients only display visuals
├─ Position updates from server
├─ No client-side decision authority
```

---

## 🧪 NETWORK TESTING

### **Test Case 1: Spawn Synchronization**

```
Setup:
├─ 2 players connected
├─ Trigger whirlwind event
└─ Both observe

Expected:
├─ Both see spawn at same time
├─ Particles identical
├─ Audio starts together
├─ Light appears simultaneously

Verification:
├─ [ ] Spawn within 100ms
├─ [ ] Visuals match
├─ [ ] Audio sync tight
└─ [ ] Light timing correct
```

### **Test Case 2: Movement Synchronization**

```
Setup:
├─ Whirlwind spawned
├─ Monitor position sync
├─ Watch movement

Expected:
├─ Position updates smooth
├─ Movement path same for both
├─ No jittering
├─ Prediction works

Verification:
├─ [ ] Smooth motion (no snapping)
├─ [ ] Path matches waypoints
├─ [ ] Latency handled well
├─ [ ] No position desync
```

### **Test Case 3: Attraction with Latency**

```
Setup:
├─ Simulate 500ms latency
├─ Player in vortex attraction zone
├─ Monitor movement

Expected:
├─ Player still attracted
├─ Force applied correctly
├─ Smooth despite latency
├─ Client prediction works

Verification:
├─ [ ] Attraction works
├─ [ ] Not "rubber-banding"
├─ [ ] Client prediction good
├─ [ ] Server authority correct
```

### **Test Case 4: Launch Synchronization**

```
Setup:
├─ 2 players in vortex
├─ Both reach launch point
└─ Observe launch

Expected:
├─ Both launch at same time
├─ Both get launch sound
├─ Visual effects match
├─ Landing simultaneous

Verification:
├─ [ ] Launch sync (< 200ms)
├─ [ ] Sound plays for both
├─ [ ] Particles spawn both
├─ [ ] Jump height same
└─ [ ] Landing synced
```

---

## 📋 NETWORKING CHECKLIST

```
REPLICATION SETUP:

Variables Replicated:
├─ [ ] ActiveWhirlwinds (Array)
├─ [ ] LastEventTriggerTime (Float)
├─ [ ] MaxSimultaneousWhirlwinds (Int32)
├─ [ ] EventScheduleEnabled (Boolean)
├─ [ ] WhirlwindType (Enum)
├─ [ ] RemainingDuration (Float)
├─ [ ] PatrolPath (Array)
└─ [ ] CurrentPathIndex (Int32)

On Rep Notifies:
├─ [ ] OnRep_EventTriggered
├─ [ ] OnRep_DurationChanged
├─ [ ] OnRep_WaypointChanged
└─ [ ] Others as needed

MULTICAST RPC FUNCTIONS:

├─ [ ] MulticastSpawnWhirlwindEffects
├─ [ ] MulticastDespawnWhirlwindEffects
├─ [ ] MulticastLaunchActor (optional)
└─ [ ] All marked Reliable

SERVER AUTHORITY:

├─ [ ] Physics calculated on server only
├─ [ ] Position updates from server
├─ [ ] Clients predict locally
├─ [ ] Server validates corrections

TESTING:

├─ [ ] Solo play works
├─ [ ] 2-player sync works
├─ [ ] 4+ player stable
├─ [ ] High latency tested (500ms+)
├─ [ ] Bandwidth monitoring
└─ [ ] Anti-cheat verified
```

---

**Verzió**: 1.0  
**DevKit**: 5.5.4  
**Dátum**: 2026-06-22  
**Status**: 🟡 Implementálható - Networking guide
