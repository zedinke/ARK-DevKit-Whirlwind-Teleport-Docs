# Teleport Rendszer - Advanced Blueprint Projekt
## Dinó + Effektek + Multicast Replikáció

## 📌 Projekt Áttekintése

Komplex teleport rendszer, amely lehetővé teszi:
- ✅ Játékos teleportálása koordinátára
- ✅ Tamed dinó teleportálása a játékossal
- ✅ Vissza-teleportálás (home point)
- ✅ Multicast VFX/SFX effektek minden client-nek
- ✅ Smooth animációkkal és transzkióval

---

## 🎯 Rendszer Komponensei

### 1. **Teleport Point Actor** (Blueprint)
Azok a helyek, ahova teleportálni lehet.

**Komponensek:**
- Sphere Collision
- Billboard (editor megjelenítés)
- Particle System (optional)
- Light (szín + intenzitás)

**Tulajdonságok:**
```
- Teleport Location: Vector (X, Y, Z)
- Teleport Rotation: Rotator
- Particle Effect Type: Enum
- Point Name: String
- Icon Color: Linear Color
- Is Return Point: Boolean
```

### 2. **Teleport Manager** (Blueprint)
A rendszer főkoordinátora, amely kezeli a logikát.

**Funkcionalitás:**
- Teleport pont registry (nyilvántartás)
- Teleport végrehajtása
- Effect triggering
- Dinó kezelés

### 3. **Teleport Effect System** (Blueprint)
Az összes vizuális és hangi effektért felelős.

**Effekt Típusok:**
```
1. Departure Effect (indulás)
   - Particle System
   - Sound effect
   - Light pulse

2. Arrival Effect (érkezés)
   - Particle System
   - Sound effect
   - Screen shake
   - Light burst

3. Trail Effect (útvonal)
   - Particle bezier görbe
   - Optional: csak indulás és érkezés között
```

---

## 📊 Node Graph Flow - Teleport Logika

### **Main Teleport Flow**
```
[Player Interacts]
    ↓
[Get Target Teleport Point]
    ↓
[Get Mounted Dino?] 
    ├→ YES → [Unmount Player Safely]
    │        ↓
    └→ NO → ↓
[Get Departure Location]
    ↓
[Spawn Departure Effect] ← Multicast
    ↓
[Delay 0.5s]
    ↓
[Teleport Player to Target]
    ↓
[Teleport Dino to Target (offset)]
    ↓
[Delay 0.2s]
    ↓
[Remount Player on Dino] ← If was mounted
    ↓
[Spawn Arrival Effect] ← Multicast
    ↓
[Save Return Point]
    ↓
[Complete]
```

---

## 🎨 VFX Effekt Designok

### **Effekt 1: Departure (Indulás)**
```
Részletek:
├─ Particle System
│  ├─ Shape: Sphere expanding
│  ├─ Color: Cyan/Blue gradient
│  ├─ Duration: 0.8s
│  ├─ Spawn Rate: 200 particles/sec
│  └─ Size: 50-200 cm
│
├─ Light Component
│  ├─ Color: Cyan
│  ├─ Intensity: 2000 lm
│  ├─ Radius: 500 cm
│  └─ Fade out: 0.5s
│
└─ Sound Effect
   ├─ File: "SFX_Teleport_Departure"
   ├─ Volume: 0.8
   └─ Pitch: 1.0
```

### **Effekt 2: Arrival (Érkezés)**
```
Részletek:
├─ Particle System
│  ├─ Shape: Sphere imploding
│  ├─ Color: Golden/White flash
│  ├─ Duration: 1.0s
│  ├─ Spawn Rate: 300 particles/sec
│  └─ Size: 100-300 cm
│
├─ Light Component
│  ├─ Color: White/Yellow
│  ├─ Intensity: 3000 lm
│  ├─ Radius: 800 cm
│  └─ Fade in/out: Quick
│
├─ Sound Effect
│  ├─ File: "SFX_Teleport_Arrival"
│  ├─ Volume: 0.9
│  └─ Pitch: 0.95
│
└─ Camera Shake
   ├─ Intensity: 2.0
   ├─ Duration: 0.3s
   └─ Frequency: 10 Hz
```

### **Effekt 3: Trail (Útvonal - Optional)**
```
Részletek:
├─ Particle Bezier Path
│  ├─ Start: Departure point
│  ├─ End: Arrival point
│  ├─ Color: Glowing trail
│  ├─ Fade: Disappear után 2s
│  └─ Speed: 500 cm/s
│
└─ Sound Effect (opcionális)
   ├─ File: "SFX_Teleport_Travel"
   ├─ Volume: 0.5
   └─ 3D sound spacing
```

---

## 🔄 Multicast Replikáció Implementáció

### **NetMulticast RPC Function**
```cpp
Concept:
[Server Logic]
    ↓
[Validation]
    ↓
[NetMulticast RPC Call]
    ↓
[Broadcast to ALL Clients]
    ↓
[Execute Effect on Each Client]
```

**Blueprint Node Graph Replikáció:**
```
[Execute Teleport]
    ↓
[Is Server?]
    ├→ YES → [Perform Teleport]
    │        ↓
    │        [Call Multicast Effect]
    │        ↓
    │        [Broadcast to All Clients]
    │
    └→ NO → [Send to Server RPC]
            ↓
            [Server handles logic]
            ↓
            [Multicast back to clients]
```

---

## 🛠️ Blueprint Struktúra

### **BP_TeleportPoint**
```
Components:
├─ Root: Scene Component
├─ Collision Sphere
├─ Particle System (visual)
├─ Billboard (editor icon)
├─ Point Light
└─ Audio Component

Variables:
├─ TeleportLocation (Vector)
├─ TeleportRotation (Rotator)
├─ PointName (String)
├─ EffectType (Enum)
├─ IsReturnPoint (Boolean)
└─ PointColor (Linear Color)

Functions:
├─ Initialize Point
├─ Get Teleport Data
└─ Show Highlight
```

### **BP_TeleportManager**
```
Components:
├─ Root: Actor
└─ (csak logika, nincs visual)

Variables:
├─ TeleportPoints (Array of BP_TeleportPoint)
├─ PlayerReturnPoint (Vector)
├─ LastTeleportTime (Float)
├─ TeleportCooldown (Float) = 2.0s
├─ DismountOffset (Vector) = 150cm
└─ IsTeleporting (Boolean)

Functions:
├─ Register Teleport Point
├─ Find Teleport Point by Name
├─ Execute Teleport
├─ Teleport With Dino
├─ Unmount & Mount Logic
├─ Multicast Spawn Effect
├─ Get Departure Location
└─ Save Return Point
```

### **BP_TeleportEffectManager**
```
Components:
├─ Root: Actor
└─ Particle System Component

Variables:
├─ DepartureEffect (Particle System)
├─ ArrivalEffect (Particle System)
├─ DepartureSound (Sound)
├─ ArrivalSound (Sound)
├─ ScreenShakeIntensity (Float)
└─ EffectScale (Float)

Functions:
├─ Spawn Departure Effect
├─ Spawn Arrival Effect
├─ Spawn Trail Effect
├─ Play Sound (Multicast)
├─ Apply Screen Shake
├─ Cleanup Effects
└─ Multicast Replication Setup
```

---

## ⚙️ Dinó Kezelés Logika

### **Tamed Dino Teleportálás**

```
Flow:
[Get Mounted Dino]
    ↓
[Store Dino Reference]
    ↓
[Unmount Player] ← Safe dismount
    ↓
[Disable Dino Physics] ← Prevent falling
    ↓
[Teleport Dino] 
    ├─ Location: Target point
    ├─ Rotation: Keep or reset
    └─ Offset: 150cm back
    ↓
[Teleport Player]
    ├─ Location: Target point
    ├─ Rotation: Towards dino
    └─ Slight offset: 100cm
    ↓
[Enable Dino Physics]
    ↓
[Remount Player]
    ├─ Animation: Mount anim
    └─ Delay: 0.3s
    ↓
[Verify Position] ← Collision check
```

**Biztonsági Ellenőrzések:**
```
✓ Dino valóban mounted-e?
✓ Célpont érvényes-e?
✓ Dino health > 0?
✓ Player nem mort-e?
✓ Phasing/túlnyomást elkerüléshez space check
```

---

## 📍 Koordináta Rendszer

### **Tárolás**
```
HomePoint: Vector
├─ X: -123456.5
├─ Y: 87654.3
└─ Z: 350.0

TargetPoint: Vector
└─ Same structure

Return Offset:
├─ Back offset: 200cm
├─ Up offset: 50cm
└─ Side sway: 0-100cm random
```

### **Biztonsági Zóna Ellenőrzés**
```
[Get Target Location]
    ↓
[Check if in water?] → NO water spawns
    ↓
[Check if on ground?] → raycast down
    ↓
[Check collision?] → no actors present
    ↓
[Adjust Z height] ← trace ground level
    ↓
[Clear to teleport]
```

---

## 🎮 Játékos Interakció

### **Teleport UI/Input**

```
Opció 1: Teleport Point Actor-ra lépés
├─ Prompt: "Press [E] to Teleport"
└─ Input: E key → Interact

Opció 2: UI Menu
├─ Open TeleportMap (Tab)
├─ Select destination
└─ Confirm → Teleport

Opció 3: Command
├─ /teleport [pointname]
├─ /home
└─ /return
```

### **Feedback Visszajelzés**
```
Visual:
├─ Screen fadeout: 0.3s
├─ Screen fade in: 0.3s
└─ Black overlay alpha: 0-1.0

Audio:
├─ Departure sound: multicasted
├─ Arrival sound: multicasted
└─ Travel sound: optional

Haptic (ha van):
├─ Vibration departure
├─ Vibration arrival
└─ Intensity: medium
```

---

## 📊 Tesztelési Checklist

### **Alapfunkciók**
- [ ] Játékos egyedül teleportálható
- [ ] Játékos dinón teleportálható
- [ ] Dinó helyes offsettel teleportál
- [ ] Return pont mentve van
- [ ] Vissza-teleport működik

### **Effektek**
- [ ] Indulási VFX megjelenik (departure)
- [ ] Érkezési VFX megjelenik (arrival)
- [ ] Trail Effect működik (optional)
- [ ] Soundok játszódnak
- [ ] Screen shake működik

### **Multicast Replikáció**
- [ ] Minden player látja az effect-et
- [ ] Szinkronizált time-azás
- [ ] Delay nincs között a client-eknek
- [ ] Server authority helyes

### **Dinó Logika**
- [ ] Mount/Dismount smooth
- [ ] Dinó nem jut túlnyomásba
- [ ] Dinó fizikája helyes
- [ ] Collision helyes

### **Edge Cases**
- [ ] Dinó halott állapotban
- [ ] Játékos már teleportál (cooldown)
- [ ] Cél pont érvénytelen
- [ ] Water spawn kezelés
- [ ] PvP zóna ellenőrzés (optional)

---

## 📁 Szükséges Asset-ek

### **VFX Particle Systems**
```
1. PT_Teleport_Departure
   └─ Cyan expanding sphere

2. PT_Teleport_Arrival
   └─ White/Gold implosion

3. PT_Teleport_Trail (optional)
   └─ Glowing bezier path
```

### **Sound Effects**
```
1. SFX_Teleport_Departure
   └─ Sci-fi whoosh (0.8s)

2. SFX_Teleport_Arrival
   └─ Magic chime + shimmer (1.0s)

3. SFX_Teleport_Travel (optional)
   └─ Travel sound (2.0s loop)
```

### **Materials/Post Process**
```
1. PP_TeleportFade
   └─ Black screen fade effect

2. M_TeleportGlow
   └─ Glow material for particles
```

---

## 🔗 Multicast RPC Implementáció - Részletes

### **Server-side:**
```
[Execute Teleport Event]
    ↓
[Check Authority] ← Only Server
    ↓
[Teleport Player + Dino]
    ↓
[Call Multicast RPC]
    ↓
[Broadcast to All Clients]
```

### **Client-side:**
```
[Receive Multicast RPC]
    ↓
[Spawn VFX on Client]
    ↓
[Play Audio on Client]
    ↓
[Apply Screen Effects]
    ↓
[Cleanup after Duration]
```

---

## 📝 Összetevő-lépések a fejlesztéshez

### **1. FÁZIS: Blueprint Alap**
- [ ] BP_TeleportPoint létrehozása
- [ ] BP_TeleportManager létrehozása
- [ ] BP_TeleportEffectManager létrehozása

### **2. FÁZIS: Node Graph Logika**
- [ ] Execute Teleport flow
- [ ] Dinó kezelés logikája
- [ ] Coordinate validation

### **3. FÁZIS: VFX/SFX**
- [ ] Particle effectek importálása
- [ ] Sound effects importálása
- [ ] Effect blueprint-ek konfigurálása

### **4. FÁZIS: Multicast Replikáció**
- [ ] Multicast RPC implementálása
- [ ] Server authority setup
- [ ] Client-side effect spawning

### **5. FÁZIS: Dinó Integrációció**
- [ ] Mount/Dismount sistema
- [ ] Dinó offset kalkulus
- [ ] Collision safety checks

### **6. FÁZIS: Tesztelés & Polishing**
- [ ] Multiplayer tesztelés
- [ ] Edge case handling
- [ ] Performance optimization
- [ ] Animáció finomítás

---

## 💡 Opcionális Bővítmények

```
🔹 Home Beacon
   └─ Világít a home point felé

🔹 Teleport Network Map
   └─ UI megjelenítés az összes ponttal

🔹 Dinó Size Scaling
   └─ Nagy dinók extra effekttel

🔹 PvP Zóna Blokk
   └─ Bizonyos zónákban nem működik

🔹 Energy/Cooldown System
   └─ Mana-szerű resource

🔹 Destination Preview
   └─ Látvány az érkezési helyre
```

---

## 📞 Node Graph Referencia

A Node Graph mintákat lásd a `06-Node-Graph-Dokumentacio/node-graph-referencia.md` fájlban!

---

**Verzió**: 1.0  
**DevKit**: 5.5.4  
**Dátum**: 2026-06-22  
**Status**: 🟡 Tervezett
