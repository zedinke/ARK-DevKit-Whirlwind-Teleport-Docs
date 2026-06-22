# Teleport Rendszer - Advanced Multicast RPC Implementáció
## Network Replikáció Mélységes Útmutató

---

## 📌 Multicast RPC Alapok

Az ARK DevKit-ben a multicast RPC lehetővé teszi, hogy egy szerver-oldali akcióból a kliensek egy időben látják az effekteket.

---

## 🔄 RPC Típusok Összehasonlítása

| RPC Típus | Server | Client | Multicast | Megbízható |
|-----------|--------|--------|-----------|-----------|
| **Server RPC** | ✅ Fut | ❌ | ❌ | Client → Server |
| **Client RPC** | ❌ | ✅ Fut | ❌ | Server → 1 Client |
| **Multicast RPC** | ✅ Fut | ✅ Fut | ✅ | Server → Összes |

---

## 🎯 MULTICAST RPC SETUP - BP_TeleportManager

### **1. Multicast Function - Execute Teleport Network**

#### **Function Declaration**

Node Graph az Event Graph-ban:
```
[Custom Event: ServerExecuteTeleport]
├─ Input Pin: Server Only
├─ Parameters:
│  ├─ TeleportPlayer (Character)
│  ├─ TargetLocation (Vector)
│  ├─ TargetRotation (Rotator)
│  ├─ bWithDino (Boolean)
│  └─ OwningPlayerController (PlayerController)
│
└─ Replication:
   ├─ Type: Server Only
   ├─ Reliable: TRUE
   └─ Run in Editor: FALSE
```

#### **Implementation Node Graph**

```
[ServerExecuteTeleport]
    ↓
[Check is Server?]
    ├─ NO → Exit
    │
    └─ YES:
       ├─ [Get Mounted Dino]
       │  └─ Store in LocalDinoRef
       │
       ├─ [Departure Effect - Multicast Call]
       │  └─ Call MulticastSpawnDepartureEffect
       │     ├─ EffectLocation: Current Position
       │     ├─ EffectType: Enum value
       │     └─ InstigatingPlayer: OwningPlayerController
       │
       ├─ [Delay 0.5s]
       │
       ├─ [Teleport Sequence]
       │  ├─ Set Character Location = TargetLocation
       │  ├─ Set Character Rotation = TargetRotation
       │  │
       │  ├─ If LocalDinoRef is Valid
       │  │  ├─ Calculate Offset Position
       │  │  ├─ Teleport Dino to Offset
       │  │  └─ Set Dino Rotation = TargetRotation - 180°
       │  │
       │  └─ Delay 0.2s
       │
       ├─ [Remount Logic]
       │  ├─ If bWithDino AND LocalDinoRef Valid
       │  │  ├─ Play Mount Animation
       │  │  ├─ Mount Character on Dino
       │  │  └─ Delay 0.3s
       │  │
       │  └─ Otherwise → Skip
       │
       ├─ [Arrival Effect - Multicast Call]
       │  └─ Call MulticastSpawnArrivalEffect
       │     ├─ EffectLocation: TargetLocation
       │     ├─ ScreenShakeIntensity: 2.0
       │     └─ InstigatingPlayer: OwningPlayerController
       │
       ├─ [Save Return Point]
       │  └─ PlayerReturnPoints.Add(OwningPlayerController, CurrentLocation)
       │
       ├─ [Broadcast to All Clients - Replicate Position]
       │  └─ Replicate Variables
       │     ├─ LastTeleportedCharacter
       │     ├─ LastTeleportLocation
       │     └─ LastTeleportTime
       │
       └─ [Complete]
```

---

### **2. Multicast Function - Spawn Departure Effect**

#### **Function Declaration**

```cpp
Function Type: Multicast Reliable RPC
Return Value: VOID

Parameters:
├─ EffectLocation (Vector)
├─ EffectRotation (Rotator) = (0, 0, 0)
├─ EffectType (ETeleportEffectType)
├─ InstigatingController (PlayerController)
└─ EffectScale (Float) = 1.0
```

#### **Node Graph Implementation**

```
[MulticastSpawnDepartureEffect]
    ↓
[Check if Valid Location]
    ├─ Is EffectLocation valid 3D point?
    │  ├─ YES → Continue
    │  └─ NO → Return
    │
    ├─ Spawn Particle System
    │  ├─ Asset: PT_Teleport_Departure
    │  ├─ Location: EffectLocation
    │  ├─ Rotation: EffectRotation
    │  ├─ Scale: EffectScale
    │  ├─ Store ref: DepartureParticleRef
    │  └─ Auto Destroy: TRUE
    │     └─ Duration: 1.0s (particle duration + buffer)
    │
    ├─ [Spawn Light Component]
    │  ├─ Create Light
    │  ├─ Attach to Particle
    │  ├─ Light Color: Cyan (0, 1, 1)
    │  ├─ Intensity: 2000 lm
    │  ├─ Attenuation Radius: 500 cm
    │  │
    │  ├─ [Light Fade Timeline]
    │  │  ├─ 0.0s: Intensity 2000
    │  │  ├─ 0.8s: Intensity 0
    │  │  └─ Play from start
    │  │
    │  └─ Auto Destroy: TRUE
    │
    ├─ [Spawn Sound at Location]
    │  ├─ Sound: SC_Teleport_Departure
    │  ├─ Location: EffectLocation
    │  ├─ Volume: 0.8
    │  ├─ Pitch: 1.0
    │  ├─ AttenuationDistance: 1000 cm
    │  ├─ 3D Spatialization: TRUE
    │  │
    │  └─ [All Clients Hear]
    │     └─ Replicated via multicast
    │
    ├─ [Optional: Create Decal]
    │  ├─ Decal Material: M_TeleportDecal_Cyan
    │  ├─ Size: 200 x 200 x 50
    │  ├─ Life Time: 0.8s
    │  └─ Fade out at end
    │
    └─ [Log Network Event (Debug)]
       └─ Print: "Departure Effect spawned at [Location]"
          └─ Only when BP_DebugNetworking = TRUE
```

---

### **3. Multicast Function - Spawn Arrival Effect**

#### **Function Declaration**

```cpp
Function Type: Multicast Reliable RPC
Return Value: VOID

Parameters:
├─ EffectLocation (Vector)
├─ EffectRotation (Rotator) = (0, 0, 0)
├─ bPlayScreenShake (Boolean) = TRUE
├─ InstigatingController (PlayerController)
├─ EffectScale (Float) = 1.0
└─ PlayScreenShakeForLocalOnly (Boolean) = TRUE
```

#### **Node Graph Implementation**

```
[MulticastSpawnArrivalEffect]
    ↓
[Validate Location]
    ├─ Check if in playable area
    │
    └─ If valid:
       ├─ [Particle System - Burst]
       │  ├─ Asset: PT_Teleport_Arrival
       │  ├─ Location: EffectLocation
       │  ├─ Rotation: EffectRotation
       │  ├─ Scale: EffectScale × 1.2
       │  ├─ Reference: ArrivalParticleRef
       │  └─ Auto Destroy: TRUE (1.2s)
       │
       ├─ [Particle System - Ring Expansion]
       │  ├─ Asset: PT_Teleport_Ring
       │  ├─ Location: EffectLocation
       │  ├─ Scale: 0.1 → 5.0 (Timeline)
       │  ├─ Color fade: Cyan → White → Transparent
       │  └─ Duration: 0.5s
       │
       ├─ [Light Burst]
       │  ├─ Create Point Light
       │  ├─ Color: White/Gold (1, 0.94, 0.39)
       │  ├─ Intensity: 3000 lm (peak)
       │  │
       │  ├─ [Intensity Timeline]
       │  │  ├─ 0.0s: 0 lm (start invisible)
       │  │  ├─ 0.05s: 3000 lm (peak)
       │  │  ├─ 0.3s: 0 lm (fade)
       │  │  └─ Auto destroy
       │  │
       │  └─ Source Radius: 100 cm
       │
       ├─ [Sound - Arrival Chime]
       │  ├─ Asset: SC_Teleport_Arrival
       │  ├─ Location: EffectLocation
       │  ├─ Volume: 0.9
       │  ├─ Pitch: 0.95
       │  ├─ Attenuation: 1200 cm
       │  │
       │  └─ [Volume Timeline]
       │     ├─ Fade in: 0.05s
       │     ├─ Sustain: 0.5s
       │     └─ Fade out: 0.4s
       │
       ├─ [Screen Shake - Local Player Only]
       │  ├─ Is Local Player?
       │  │  ├─ Get Instigating Player Controller
       │  │  └─ Compare with Local Player Controller
       │  │
       │  ├─ If bPlayScreenShake AND is local
       │  │  ├─ Get Camera Manager
       │  │  └─ Play Camera Shake
       │  │     ├─ Shake Scale: 2.0
       │  │     ├─ X Amplitude: 5.0
       │  │     ├─ Y Amplitude: 5.0
       │  │     ├─ Z Amplitude: 3.0
       │  │     ├─ Duration: 0.3s
       │  │     └─ Frequency: 10 Hz
       │  │
       │  └─ If not local → Skip (no double shake)
       │
       ├─ [Particle Trail - Optional]
       │  ├─ If bShowTrail = TRUE
       │  ├─ Asset: PT_Teleport_Trail
       │  ├─ Scale: 0.8
       │  └─ Auto Destroy: TRUE (3.0s)
       │
       └─ [Complete - Debug Log]
          └─ Print: "Arrival Effect at [Location]"
```

---

## 🌐 Network Replication Setup

### **Replicated Variables in BP_TeleportManager**

Variables → Details Panel:
```
Replication:

1. LastTeleportLocation
   ├─ Replication: Replicated
   ├─ On Rep Notify: OnRep_TeleportLocation
   ├─ Category: Networking
   └─ Comment: "Broadcast for all clients to know"

2. LastTeleportedCharacter
   ├─ Replication: Replicated
   ├─ On Rep Notify: OnRep_CharacterTeleported
   └─ Type: Character Reference

3. LastTeleportTime
   ├─ Replication: Replicated
   ├─ Type: Float
   └─ Use for cooldown sync

4. IsTeleportingNetwork
   ├─ Replication: Replicated
   ├─ Type: Boolean
   └─ Prevents duplicate calls
```

### **OnRep Functions**

**OnRep_TeleportLocation Event Graph:**

```
[OnRep_TeleportLocation]
    ↓
[Client Side Execution]
    ├─ Only runs on clients when variable changes
    │
    ├─ [Update UI - Show Recent Teleport]
    │  ├─ Get Teleport UI Widget
    │  ├─ Display location indicator
    │  └─ Show on map (if applicable)
    │
    ├─ [Sync Audio/Visual Effects]
    │  ├─ If not already local effect
    │  ├─ Spawn missed arrival effect
    │  └─ Ensure all clients in sync
    │
    └─ [Log for Debugging]
       └─ Print: "Client received teleport location: [Vector]"
```

---

## 🔐 Authority Validation

### **Server-Side Validation**

Node Graph in ServerExecuteTeleport:
```
[Before Teleport Execution]
    ↓
[Validate Server Authority]
    ├─ Is Server? → Continue
    ├─ Is not server? → Return (reject)
    │
    └─ Validate Gameplay State:
       ├─ [ ] Character is valid?
       ├─ [ ] Character is alive?
       ├─ [ ] Character not in combat?
       ├─ [ ] Not in restricted zone?
       ├─ [ ] Teleport not on cooldown?
       ├─ [ ] Target location valid?
       ├─ [ ] Target location not blocked?
       ├─ [ ] Location in map bounds?
       └─ [ ] All checks pass?
          ├─ YES → Execute Teleport
          └─ NO → Reject + Send Error to Client
```

### **Client-Side Permission Check**

```
[Player Requests Teleport]
    ↓
[Local Validation]
    ├─ Can this player teleport?
    ├─ Is teleport button enabled?
    ├─ Is player not already teleporting?
    │
    └─ If valid:
       ├─ Send Server RPC
       │  └─ ServerExecuteTeleport(...)
       │
       └─ Show local feedback (UI)
          └─ "Teleporting..." message
```

---

## 📡 RPC CALL ORDER - Full Sequence

### **Complete Network Flow**

```
CLIENT SIDE:
[Player presses Teleport button]
    ↓
[Local Input Handler]
    ├─ Disable button (prevent spam)
    ├─ Show "Teleporting..." UI
    │
    └─ Call Server RPC
       └─ ServerExecuteTeleport(
          ├─ Player Character
          ├─ Target Location
          ├─ Target Rotation
          ├─ HasDino?
          └─ PlayerController
          )
         ↓
SERVER SIDE:
[ServerExecuteTeleport Received]
    ↓
[Server Validation]
    ├─ Check authority
    ├─ Validate gameplay state
    ├─ Validate target location
    │
    └─ If all valid:
       ├─ [Departure Multicast]
       │  └─ MulticastSpawnDepartureEffect(
       │     ├─ Current position
       │     ├─ Cyan effect type
       │     └─ Broadcast to ALL
       │     )
       │
       ├─ [Delay 0.5s]
       │
       ├─ [Perform Teleport]
       │  ├─ Teleport Character
       │  ├─ Teleport Dino (offset)
       │  ├─ Mount player
       │  │
       │  └─ Replicate new positions
       │
       ├─ [Arrival Multicast]
       │  └─ MulticastSpawnArrivalEffect(
       │     ├─ Target location
       │     ├─ White/Gold effect type
       │     ├─ Screen shake (local only)
       │     └─ Broadcast to ALL
       │     )
       │
       └─ Update Replicated Variables
          ├─ LastTeleportLocation = new location
          ├─ LastTeleportTime = current time
          └─ OnRep callbacks trigger on clients
             ↓
ALL CLIENTS:
[Multicast RPC Received]
    ↓
[Effect Execution]
    ├─ Spawn particles
    ├─ Play audio
    ├─ Apply light
    ├─ Update UI
    │
    └─ OnRep Notify Triggers
       ├─ Update location display
       ├─ Sync with latest state
       └─ Log confirmation

[Result: All players see same effects at same time!]
```

---

## 🎮 Latency Handling

### **Predicted vs Replicated Movement**

```
Local Player (Instigator):
├─ Immediate feedback
│  ├─ Disable input
│  ├─ Play local effect
│  └─ Move character instantly
│
└─ Wait for server confirmation
   └─ If rejected → rollback + show error

Remote Players:
├─ See departure effect
├─ See player disappear
├─ Wait for server to send arrival
├─ See arrival effect + player appear
│
└─ Position replicated from server
   └─ Smooth movement to final position
```

### **Handshake Protocol**

```
[Client] → [Send Request]
    ↓
[Server] → [Process + Multicast Departure]
    ↓
[All Clients] → [Show Departure Effect]
    ↓
[Server] → [Execute Teleport]
    ↓
[Server] → [Multicast Arrival]
    ↓
[All Clients] → [Show Arrival + Update Position]
    ↓
[Server] → [Send Confirmation to Requesting Client]
    ↓
[Client] → [Enable Input + Remove "Teleporting" UI]
```

---

## ⚠️ ERROR HANDLING

### **Network Error Cases**

**Case 1: Server Rejects Teleport**

```
[Server Validation Failed]
    ↓
[Call Client-Specific RPC]
    └─ ClientReceiveTeleportError(...)
       ├─ ErrorType: ETeleportErrorType
       ├─ ErrorMessage: FString
       └─ ErrorCode: Int32
       ↓
[Client Response]
    ├─ Display error message
    ├─ Re-enable teleport button
    ├─ Log rejection reason
    └─ Revert any local changes
```

**Client RPC for Error Handling:**

```cpp
Function: ClientReceiveTeleportError
Type: Client Unreliable RPC
Parameters:
├─ ErrorType (ETeleportErrorType)
│  ├─ InvalidLocation
│  ├─ TooFarAway
│  ├─ OnCooldown
│  ├─ InCombat
│  └─ LocationRestricted
│
└─ ErrorMessage (String)

Implementation:
├─ Show error popup
├─ Log: "[TeleportError] " + ErrorMessage
├─ Play negative sound effect
└─ Revert UI state
```

**Case 2: Network Packet Loss**

```
Mitigation:
├─ Use Reliable RPC for teleport execution
├─ Use Reliable Multicast for effects
├─ Server resends if client doesn't confirm
└─ Timeout: 5 seconds → force sync
```

---

## 🧪 MULTIPLAYER DEBUG SETUP

### **Debug Variables in BP_TeleportManager**

```
Variables:
├─ bDebugNetworking: Boolean = FALSE
├─ bLogAllRPCs: Boolean = FALSE
├─ bShowEffectTimings: Boolean = FALSE
├─ DebugPlayerName: String = "ALL"
└─ NetworkLatencySimulation: Float = 0.0 (0-5 seconds)
```

### **Debug Node Graph Nodes**

```
[Throughout Execute Teleport]
    ├─ If bDebugNetworking = TRUE
    │  ├─ Print to screen:
    │  │  ├─ [NETWORK] Function called
    │  │  ├─ [AUTHORITY] Is Server? [YES/NO]
    │  │  ├─ [VALIDATE] Location valid? [YES/NO]
    │  │  ├─ [MULTICAST] Departure sent
    │  │  ├─ [TELEPORT] Position updated
    │  │  ├─ [MULTICAST] Arrival sent
    │  │  └─ [COMPLETE] Teleport finished
    │  │
    │  └─ Log to file: "NetworkDebug_Teleport.log"
    │
    └─ If bShowEffectTimings = TRUE
       ├─ Display on screen:
       │  ├─ Effect spawn time
       │  ├─ RPC latency
       │  └─ Total duration
       │
       └─ Graph with timeline
```

### **Network Simulation**

```
[Simulate Packet Loss]
    ├─ If NetworkLatencySimulation > 0
    │  └─ Delay multicast calls
    │     └─ By: NetworkLatencySimulation seconds
    │
    └─ Test how effects look with lag
       ├─ Local effect plays immediately
       ├─ Remote effects delayed
       └─ Helps debug synchronization
```

---

## 🔧 TESTING CHECKLIST - Networking

### **Single Player Tests**
- [ ] Teleport works without network
- [ ] Dinó teleports with player
- [ ] Effects play correctly
- [ ] Sound effects audible
- [ ] No crashes

### **Host/Client Tests**
- [ ] Host can teleport
- [ ] Client can teleport
- [ ] Host sees client teleport effects
- [ ] Client sees host teleport effects
- [ ] Both see each other's effects simultaneously

### **Multicast Verification**
- [ ] 2 players: Both see effects at same time
- [ ] 4 players: All see synchronized effects
- [ ] 8+ players: No desync issues
- [ ] Effects appear within 100ms of each other

### **Edge Cases**
- [ ] Network disconnect during teleport → rollback
- [ ] High latency (500ms) → effects still sync
- [ ] Packet loss → effects still appear
- [ ] Teleport + another player action → no conflicts
- [ ] Server lag → client can still request

### **Performance Under Load**
- [ ] 10 simultaneous teleports → no crash
- [ ] Constant teleporting → frame rate stable
- [ ] Multiple effects → GPU handles it
- [ ] Network bandwidth acceptable

---

## 📊 NETWORK PROFILING

### **Monitor in DevKit**

PIE (Play in Editor) Console:
```
stat network
stat unit

Shows:
├─ Frame time
├─ Network traffic
├─ RPC calls/sec
├─ Bandwidth used
└─ Latency
```

---

## 🎓 Best Practices Összefoglalása

```
✅ DO:
├─ Use Reliable for critical teleports
├─ Validate on server side
├─ Use Multicast for visible effects
├─ Log all network events (debug mode)
├─ Handle errors gracefully
├─ Test with simulated latency
├─ Cache references (don't search each call)
└─ Use structs for parameter packing

❌ DON'T:
├─ Trust client input completely
├─ Use Unreliable for important data
├─ Spawn effects on client only
├─ Ignore network errors
├─ Assume zero latency
├─ Send unvalidated positions
├─ Loop complex logic in multicast
└─ Forget to cleanup actors
```

---

**Verzió**: 1.0  
**DevKit**: 5.5.4  
**Dátum**: 2026-06-22  
**Status**: 🟢 Implementálható
