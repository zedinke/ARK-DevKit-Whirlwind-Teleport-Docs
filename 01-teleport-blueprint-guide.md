# Teleport Rendszer - Blueprint Implementáció Útmutató
## Lépésről-lépésre a DevKit-ben

---

## 🎯 FÁZIS 1: Blueprint Alap Felépítése

### **1.1 BP_TeleportPoint Létrehozása**

**Lépések:**
```
1. Content Browser → New → Blueprint
   └─ Parent Class: Actor
   └─ Name: BP_TeleportPoint

2. Blueprint Editor megnyitása
   └─ Components panel → Add Component

3. Komponensek hozzáadása:
   ✓ Scene Root (Default)
   ✓ Sphere Collision
   ✓ Particle System
   ✓ Point Light
   ✓ Billboard Icon (editor)
   ✓ Audio Component
```

**Sphere Collision beállítása:**
```
Details panel:
├─ Shape
│  ├─ Sphere Radius: 100.0
│  └─ Generate Overlap Events: TRUE
│
├─ Collision
│  ├─ Object Type: WorldStatic
│  ├─ Collision Responses: OverlapAll
│  └─ Generate Physics: FALSE
│
└─ Physics
   └─ Simulate Physics: FALSE
```

**Point Light beállítása:**
```
Details panel:
├─ Light
│  ├─ Intensity: 2000.0 lumen
│  ├─ Light Color: Cyan (0, 255, 255)
│  └─ Visible: TRUE
│
├─ Attenuation Radius: 500.0
└─ Source Radius: 50.0
```

---

### **1.2 BP_TeleportPoint Variables**

**Variables panel → New Variable hozzáadása:**

| Variable Name | Type | Default | Description |
|--------------|------|---------|-------------|
| TeleportLocation | Vector | (0,0,0) | Cél koordináta |
| TeleportRotation | Rotator | (0,0,0) | Cél irányultság |
| PointName | String | "Teleport Point" | Pont elnevezése |
| EffectType | Enum | Cyan | Effekt típusa |
| IsReturnPoint | Boolean | FALSE | Vissza pont? |
| PointColor | LinearColor | (0,1,1) | Pont szín |
| IsActive | Boolean | TRUE | Aktív-e? |

**Variable Detáil Beállítások:**
```
TeleportLocation:
├─ Details
│  ├─ Show 3D Widget: TRUE
│  ├─ Category: "Teleport Settings"
│  └─ Tooltip: "Target location for teleportation"

PointName:
├─ Category: "Teleport Settings"
├─ Tooltip: "Unique name for UI/reference"
└─ Default Value: "New Teleport Point"
```

---

### **1.3 BP_TeleportPoint Functions**

**Új Function: Initialize Point**

Node Graph:
```
[Event Initialize Point]
    ↓
[Get Particle System] → [Set Particle System at Location]
    ├─ Location: TeleportLocation
    └─ Rotation: TeleportRotation
    ↓
[Get Point Light] → [Set Light Color]
    └─ New Color: PointColor
    ↓
[Set Location] ← Actor
    └─ New Location: TeleportLocation
    ↓
[Set Rotation] ← Actor
    └─ New Rotation: TeleportRotation
    ↓
[Complete]
```

**Új Function: Get Teleport Data**

Node Graph:
```
[Input]
    ↓
[Return as Struct] ← Make Teleport Data
├─ Location: TeleportLocation
├─ Rotation: TeleportRotation
├─ PointName: PointName
└─ EffectType: EffectType
```

**Event Graph:**
```
[Event BeginPlay]
    ↓
[Call Initialize Point]
    ↓
[Complete]
```

---

## 🎯 FÁZIS 2: BP_TeleportManager

### **2.1 Létrehozás és Komponensek**

```
1. Content Browser → New → Blueprint
   └─ Parent Class: Actor
   └─ Name: BP_TeleportManager

2. Komponensek
   └─ Scene Root (Default)
   └─ (Logika-alapú, nincs visual komponens szükséges)
```

### **2.2 BP_TeleportManager Variables**

| Variable Name | Type | Default | Description |
|--------------|------|---------|-------------|
| TeleportPoints | Array (BP_TeleportPoint) | [] | Összes TP pont |
| PlayerReturnPoint | Vector | (0,0,0) | Játékos home |
| LastTeleportTime | Float | 0.0 | Utolsó TP idő |
| TeleportCooldown | Float | 2.0 | Cooldown (mp) |
| DismountOffset | Vector | (150,0,0) | Dinó offset |
| IsTeleporting | Boolean | FALSE | Teleportálás alatt? |
| CharacterRef | Character | NULL | Játékos ref |
| CachedMountedDino | Pawn | NULL | Mounted dinó |

---

### **2.3 BP_TeleportManager Functions**

**Function 1: Register Teleport Point**

Paraméterek:
```
Input:
├─ InTeleportPoint (BP_TeleportPoint)
└─ AutoInitialize (Boolean)

Output:
└─ Success (Boolean)
```

Node Graph:
```
[Register Teleport Point Event]
    ↓
[Is Valid?] → InTeleportPoint
    ├─ YES:
    │  ├─ Add to Array (TeleportPoints)
    │  ├─ If AutoInitialize → Call Initialize Point
    │  └─ Return TRUE
    │
    └─ NO:
       └─ Return FALSE
```

**Function 2: Find Teleport Point by Name**

Paraméterek:
```
Input:
└─ PointName (String)

Output:
└─ FoundPoint (BP_TeleportPoint)
```

Node Graph:
```
[Find Teleport Point Event]
    ↓
[For Each Loop] → Array (TeleportPoints)
    ├─ Get Array Element
    │  ├─ Get PointName
    │  └─ Compare String
    │     ├─ EQUAL → Return element
    │     └─ NOT EQUAL → Continue loop
    │
    └─ Loop Complete → Return NULL
```

**Function 3: Execute Teleport (MAIN LOGIC)**

Paraméterek:
```
Input:
├─ TargetPoint (Vector)
├─ TargetRotation (Rotator)
├─ InCharacter (Character)
└─ bWithDino (Boolean)

Output:
└─ bSuccess (Boolean)
```

Node Graph:
```
[Execute Teleport Event]
    ↓
[Check Cooldown?]
    ├─ TOO SOON → Print "Cooldown active" → Return FALSE
    │
    └─ OK:
       ├─ Set IsTeleporting = TRUE
       ├─ Get Mounted Dino Reference
       │  ├─ Store in CachedMountedDino
       │  └─ If mounted → Unmount Character
       │
       ├─ Delay 0.2s ← Dismount anim
       │
       ├─ [Spawn Departure Effect]
       │  └─ Call "Multicast Spawn Departure"
       │     ├─ Location: Current position
       │     └─ EffectType: Enum value
       │
       ├─ Delay 0.5s ← Effect duration
       │
       ├─ [Teleport Sequence]
       │  ├─ Teleport Character
       │  │  ├─ Set Location: TargetPoint
       │  │  ├─ Set Rotation: TargetRotation
       │  │  └─ Is Valid?
       │  │
       │  ├─ If CachedMountedDino is Valid
       │  │  ├─ Calculate Dino Offset
       │  │  │  ├─ Vector Offset = (150, 0, 0)
       │  │  │  ├─ Apply Rotation
       │  │  │  └─ Final Position = Target + Offset
       │  │  │
       │  │  ├─ Disable Dino Physics
       │  │  ├─ Teleport Dino to offset position
       │  │  ├─ Enable Dino Physics
       │  │  └─ Delay 0.2s
       │  │
       │  └─ If not mounted, just teleport player
       │
       ├─ [Remount Logic]
       │  ├─ If CachedMountedDino is Valid
       │  │  ├─ Get Character → Mount Dino
       │  │  │  └─ Use Mount Animation
       │  │  └─ Delay 0.3s ← Mount anim
       │  │
       │  └─ Otherwise → Skip
       │
       ├─ [Spawn Arrival Effect]
       │  └─ Call "Multicast Spawn Arrival"
       │     ├─ Location: TargetPoint
       │     ├─ EffectType: Enum value
       │     └─ Play Screen Shake
       │
       ├─ [Save Return Point]
       │  ├─ Get Character Current Location
       │  └─ Store in PlayerReturnPoint
       │
       ├─ Set LastTeleportTime = Current Time
       ├─ Set IsTeleporting = FALSE
       └─ Return TRUE
```

---

## 🎯 FÁZIS 3: Effekt Rendszer

### **3.1 BP_TeleportEffectManager Létrehozása**

```
1. Content Browser → New → Blueprint
   └─ Parent Class: Actor
   └─ Name: BP_TeleportEffectManager

2. Komponensek:
   ├─ Scene Root
   ├─ Particle System (Departure)
   ├─ Particle System (Arrival)
   └─ Audio Component
```

### **3.2 Variables**

| Variable Name | Type | Default |
|--------------|------|---------|
| DepartureEffectClass | Class (Particle) | PT_Teleport_Departure |
| ArrivalEffectClass | Class (Particle) | PT_Teleport_Arrival |
| TrailEffectClass | Class (Particle) | PT_Teleport_Trail |
| DepartureSoundCue | Sound | SFX_Teleport_Departure |
| ArrivalSoundCue | Sound | SFX_Teleport_Arrival |
| ScreenShakeIntensity | Float | 2.0 |
| EffectScale | Float | 1.0 |

### **3.3 Multicast RPC Functions**

**Function: Multicast Spawn Departure Effect**

```cpp
Replication:
├─ RepNotify: Multicast
├─ Reliable: YES
└─ Authority: ServerOnly

Parameters:
├─ EffectLocation (Vector)
├─ EffectRotation (Rotator)
└─ EffectType (Enum)
```

Node Graph:
```
[Multicast Spawn Departure]
    ↓
[Spawn Particle System]
    ├─ Template: DepartureEffectClass
    ├─ Location: EffectLocation
    ├─ Rotation: EffectRotation
    └─ Scale: EffectScale
    ↓
[Spawn Sound at Location]
    ├─ Sound: DepartureSoundCue
    ├─ Location: EffectLocation
    ├─ Volume: 0.8
    └─ Pitch: 1.0
    ↓
[Spawn Light]
    ├─ Color: Cyan
    ├─ Intensity: 2000
    ├─ Radius: 500
    └─ Duration: 0.5s fade
    ↓
[Complete]
```

**Function: Multicast Spawn Arrival Effect**

```cpp
Replication:
├─ RepNotify: Multicast
├─ Reliable: YES
└─ Authority: ServerOnly

Parameters:
├─ EffectLocation (Vector)
├─ EffectRotation (Rotator)
└─ bPlayScreenShake (Boolean)
```

Node Graph:
```
[Multicast Spawn Arrival]
    ↓
[Spawn Particle System]
    ├─ Template: ArrivalEffectClass
    ├─ Location: EffectLocation
    ├─ Rotation: EffectRotation
    └─ Scale: EffectScale (1.2x)
    ↓
[Spawn Sound at Location]
    ├─ Sound: ArrivalSoundCue
    ├─ Location: EffectLocation
    ├─ Volume: 0.9
    └─ Pitch: 0.95
    ↓
[Spawn Light (Burst)]
    ├─ Color: White/Gold
    ├─ Intensity: 3000
    ├─ Radius: 800
    └─ Duration: 0.3s
    ↓
[If bPlayScreenShake]
    ├─ Get Player Controller
    ├─ Play Camera Shake
    │  ├─ Intensity: ScreenShakeIntensity
    │  ├─ Duration: 0.3s
    │  └─ Frequency: 10Hz
    │
    └─ Complete
```

---

## 🎯 FÁZIS 4: Dinó Kezelés

### **4.1 Mount/Dismount Sistem**

**Function: Safe Dismount Character**

```
Input:
└─ InCharacter (Character)

Output:
└─ DidDismount (Boolean)
```

Node Graph:
```
[Safe Dismount Event]
    ↓
[Get Mounted Character]
    ├─ Is Valid?
    │  ├─ YES:
    │  │  ├─ Play Dismount Animation
    │  │  ├─ Delay 0.2s
    │  │  ├─ Unmount
    │  │  └─ Return TRUE
    │  │
    │  └─ NO:
    │     └─ Return FALSE
    │
└─ Complete
```

**Function: Safe Mount Character**

```
Input:
├─ InCharacter (Character)
├─ TargetDino (Pawn)
└─ bPlayAnimation (Boolean)

Output:
└─ DidMount (Boolean)
```

Node Graph:
```
[Safe Mount Event]
    ↓
[Validate Dino]
    ├─ Is Valid?
    ├─ Is Alive?
    └─ Is Tamed?
    ↓
[If Valid]
    ├─ If bPlayAnimation
    │  ├─ Play Mount Animation
    │  └─ Delay 0.3s
    │
    ├─ Mount Character on Pawn
    │  └─ Target Pawn: TargetDino
    │
    └─ Return TRUE
    ↓
[If Not Valid]
    └─ Return FALSE
```

---

## 🎯 FÁZIS 5: Koordináta Validáció

**Function: Validate Teleport Location**

```
Input:
└─ LocationToCheck (Vector)

Output:
└─ IsValid (Boolean)
```

Node Graph:
```
[Validate Location Event]
    ↓
[Check 1: Is in Water?]
    ├─ Line Trace down from location
    ├─ Get trace result
    └─ Check if water
       ├─ YES → Adjust Z upward
       └─ NO → Continue
    ↓
[Check 2: Ground Level]
    ├─ Line Trace down 10000 units
    ├─ Get impact point
    └─ Adjust location to ground + offset
    ↓
[Check 3: Collision/Obstruction]
    ├─ Sphere Trace at location
    ├─ Radius: 200cm
    └─ Check for collision
       ├─ Found object → Adjust X/Y
       └─ Clear → Continue
    ↓
[Check 4: Final Validation]
    ├─ Is location reachable?
    ├─ Is location safe?
    └─ Return TRUE/FALSE
```

---

## 🎮 FÁZIS 6: Player Input Beépítése

**Gamemode/Character Blueprint:**

Event Graph:
```
[Setup Input Component]
    ↓
[Bind Action Input]
    ├─ Input Name: "Teleport"
    ├─ Input Key: E
    ├─ Event: IE_Pressed → Call "Initiate Teleport"
    │
    └─ Input Name: "TeleportHome"
       ├─ Input Key: H
       └─ Event: IE_Pressed → Call "Teleport Home"
```

**Initiate Teleport Event:**

```
[Initiate Teleport]
    ↓
[Get Teleport Manager] ← Reference
    ↓
[Open Teleport UI]
    ├─ Show point list
    ├─ Player selects point
    │
    └─ On Selection Confirmed:
       ├─ Get Selected Point
       ├─ Call Execute Teleport
       │  ├─ TargetPoint: Selected Location
       │  ├─ TargetRotation: Selected Rotation
       │  ├─ InCharacter: PlayerCharacter
       │  └─ bWithDino: IsMounted?
       │
       └─ Complete
```

---

## 🧪 TESZTELÉSI NODE GRAPH SETUP

**Test Character Blueprint Event Graph:**

```
[Event BeginPlay]
    ↓
[Find Teleport Manager]
    ├─ Get All Actors of Class
    └─ Store reference
    ↓
[Spawn Test Teleport Points]
    ├─ Create Point 1 (Red)
    │  ├─ Location: (1000, 0, 500)
    │  └─ Register
    │
    ├─ Create Point 2 (Blue)
    │  ├─ Location: (-1000, 0, 500)
    │  └─ Register
    │
    └─ Create Point 3 (Green)
       ├─ Location: (0, 1000, 500)
       └─ Register
    ↓
[Setup Input]
    ├─ T Key → Test Teleport to Point 1
    ├─ Y Key → Test Teleport to Point 2
    └─ U Key → Test Teleport to Point 3
```

---

## 📝 Szükséges Asset-ek Listája

### **Particle Systems (Import/Create)**
```
□ PT_Teleport_Departure
  ├─ Cyan sphere expanding
  └─ Duration: 0.8s

□ PT_Teleport_Arrival
  ├─ White burst incoming
  └─ Duration: 1.0s

□ PT_Teleport_Trail (optional)
  ├─ Glowing path between points
  └─ Duration: 2.0s
```

### **Sounds (Import)**
```
□ SFX_Teleport_Departure (0.8s)
□ SFX_Teleport_Arrival (1.0s)
□ SFX_Teleport_Travel (2.0s, optional)
```

### **Post Process (Create/Configure)**
```
□ PP_TeleportFade
  └─ Black fade in/out material
```

---

## ✅ Fejlesztési Sorrendiseg

1. **BP_TeleportPoint** - Pont létrehozása
2. **BP_TeleportManager** - Manager logika
3. **BP_TeleportEffectManager** - Effect rendszer
4. **Multicast RPC** - Network replikáció
5. **Dinó kezelés** - Mount/Dismount
6. **Input kötések** - Player interakció
7. **Tesztelés** - Multiplayer validáció
8. **Polishing** - Animációk, timing, hangok

---

**Verzió**: 1.0  
**DevKit**: 5.5.4  
**Dátum**: 2026-06-22
