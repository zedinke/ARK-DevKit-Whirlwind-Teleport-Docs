# Teleport Rendszer - Dinó Animációk & Mount/Dismount Logika
## Advanced Creature Handling

---

## 📌 Dinó Integrációs Áttekintése

Az ARK-ban a montált dinók speciális logikát igényelnek:
- Mount/Dismount animációk
- Pozicionálás (player sáron)
- Dinó offset kezelése
- Animation replication

---

## 🦕 Dinó Típusok Kezelése

### **Mountable vs Non-Mountable Dinos**

```
Mountable Dinos:
├─ Rex
├─ Trike
├─ Griffin
├─ Wyvern
├─ Phoenix
└─ ... stb

Non-Mountable:
├─ Small creatures (Dodo)
├─ Flying creatures (Ptera, Argent)
└─ Boss creatures

Detection Blueprint:
[Check if Mountable]
    ├─ Get Creature class
    ├─ Check bCanBeTamed
    ├─ Check bCanBeMounted
    └─ Return TRUE/FALSE
```

---

## 🎬 MOUNT ANIMÁCIÓ SETUP

### **Animation Setup az ARK-ban**

#### **Required Animations**

```
Character Blueprint Animation States:

1. Mount_Start
   ├─ Duration: 0.8-1.0 seconds
   ├─ Start: Player standing next to dino
   ├─ End: Player on dino saddle
   └─ Transition: Smooth interpolation

2. Mounted_Idle
   ├─ Duration: Loop
   ├─ Player sitting on dino
   ├─ Can override with dino look
   └─ Blend with camera movement

3. Dismount_Start
   ├─ Duration: 0.5-0.8 seconds
   ├─ Start: Player on dino
   ├─ End: Player falling/sliding off
   └─ Transition: Smooth

4. Dismount_End
   ├─ Duration: 0.3-0.5 seconds
   ├─ Start: Player falling
   ├─ End: Player landing on ground
   └─ Impact animation
```

#### **Animation Blueprint Node Graph**

```
[Character Blueprint Animation Graph]
    ↓
[State Machine]
    ├─ State: Idle
    ├─ State: Walking
    ├─ State: Running
    ├─ State: Falling
    ├─ State: Mounted
    │  ├─ Sub-state: Mount_Start
    │  ├─ Sub-state: Mounted_Idle
    │  └─ Sub-state: Dismount_Start
    │
    └─ Transitions:
       ├─ Idle → Mount_Start
       │  └─ Condition: bMounted = TRUE
       │
       ├─ Mount_Start → Mounted_Idle
       │  └─ Time: > 1.0s
       │
       ├─ Mounted_Idle → Dismount_Start
       │  └─ Condition: bDismounting = TRUE
       │
       └─ Dismount_Start → Idle
          └─ Time: > 0.8s
```

---

## 🐴 SAFE MOUNT/DISMOUNT PROTOCOL

### **Mount Sequence - Detailed Node Graph**

```
[Player Requests Teleport with Dino]
    ↓
[STEP 1: Validate Dino]
    ├─ Get Currently Mounted Pawn
    ├─ Is Valid?
    │  ├─ NO → Skip to player-only teleport
    │  └─ YES → Continue
    │
    ├─ Is Dino Alive?
    │  ├─ NO → Unmount + skip dino teleport
    │  └─ YES → Continue
    │
    └─ Is Dino Tamed?
       ├─ NO → Show error "Not tamed!"
       └─ YES → Continue
    ↓
[STEP 2: Prepare Dismount]
    ├─ Disable dino movement controls
    │  └─ Set dino bIgnoreMovementInput = TRUE
    │
    ├─ Disable dino jumping
    │  └─ Set dino bCanJump = FALSE
    │
    └─ Lock dino rotation
       └─ Set dino bUseRootMotion = TRUE
    ↓
[STEP 3: Play Dismount Animation]
    ├─ Get Player Character
    ├─ Get Animation Blueprint
    ├─ Play Animation Montage
    │  ├─ Montage: Dismount_Start
    │  ├─ Duration: 0.8 seconds
    │  └─ Sync on all clients
    │     └─ Multicast animation call
    │
    └─ Wait for animation to complete
       └─ Delay: 0.8s
    ↓
[STEP 4: Perform Dismount]
    ├─ Get Player Character
    ├─ Unmount from Pawn
    │  └─ UCharacterMovementComponent::UnmountFromCharacter()
    │
    ├─ Set Player position
    │  ├─ Offset: 150cm behind dino
    │  ├─ Height: Ground level
    │  └─ Avoid collision
    │
    ├─ Enable player movement
    │  └─ Set character input enabled
    │
    └─ Store Dino Reference
       └─ CachedMountedDino = Current mounted dino
    ↓
[STEP 5: Disable Dino Physics (Important!)]
    ├─ Get Dino Physics Component
    ├─ Set dino gravity = 0 (floating)
    ├─ Set collision = off
    │  └─ Prevent falling through world
    │
    └─ This prevents teleport errors
    ↓
[READY FOR TELEPORT]
```

### **Teleport Execution (with Dino)**

```
[Server Performs Teleport]
    ↓
[STEP 1: Teleport Player]
    ├─ Set Player Location = TargetLocation
    ├─ Set Player Rotation = TargetRotation
    │
    └─ Multicast Position Update
       └─ All clients see player appear
    ↓
[STEP 2: Teleport Dino]
    ├─ Calculate Dino Offset
    │  ├─ Offset vector: (150, 0, 0) cm
    │  ├─ Apply target rotation to offset
    │  │  └─ Rotate offset by TargetRotation
    │  │
    │  └─ Final Dino Position = Target + Rotated Offset
    │
    ├─ Set Dino Location = Calculated Position
    ├─ Set Dino Rotation = TargetRotation + 180°
    │  └─ Face opposite direction of player
    │
    └─ Multicast Dino Position
       └─ All clients see dino appear
    ↓
[STEP 3: Validate Positions]
    ├─ Check dino not in water
    ├─ Check dino not in collision
    ├─ Check ground level under dino
    │
    └─ If issue → Adjust dino Z height
    ↓
[STEP 4: Re-enable Dino Physics]
    ├─ Set dino gravity = 1.0 (normal)
    ├─ Set collision = on
    ├─ Set physics = enabled
    │
    └─ Wait small delay (0.2s)
       └─ Let physics settle
    ↓
[READY FOR REMOUNT]
```

### **Remount Sequence**

```
[STEP 1: Position Validation]
    ├─ Is Dino standing on ground?
    │  ├─ Raycast down from dino
    │  ├─ Check hit distance < 100cm
    │  │
    │  └─ If not on ground → Wait + retry
    │
    └─ Is Player next to Dino?
       ├─ Check distance < 200cm
       └─ If too far → Move closer
    ↓
[STEP 2: Play Mount Animation]
    ├─ Get Player Animation Blueprint
    ├─ Play Animation Montage
    │  ├─ Montage: Mount_Start
    │  ├─ Duration: 1.0 seconds
    │  └─ Multicast (all see it)
    │
    └─ Wait for mount animation
       └─ Delay: 1.0s
    ↓
[STEP 3: Perform Mount]
    ├─ Get Mounted Dino Reference
    ├─ Mount Character on Pawn
    │  └─ UCharacterMovementComponent::MountCharacter(Dino)
    │
    ├─ Set proper saddle position
    │  ├─ Offset: Center of saddle
    │  ├─ Height: Proper seated height
    │  └─ Rotation: Aligned with dino
    │
    ├─ Update player camera
    │  ├─ Mount camera on dino head
    │  └─ Offset: 200cm up, behind
    │
    └─ Replicate mount state
    ↓
[STEP 4: Re-enable Controls]
    ├─ Re-enable dino movement
    │  └─ Set bIgnoreMovementInput = FALSE
    │
    ├─ Re-enable jumping
    │  └─ Set bCanJump = TRUE
    │
    ├─ Re-enable player input
    │  └─ Set input enabled
    │
    └─ Restore camera control
    ↓
[MOUNT COMPLETE]
```

---

## 🎯 OFFSET CALCULATIONS

### **Dino Positioning Logic**

```
Offset Calculation:
┌─────────────────────────────────────┐
│   Base Offset Vector                │
│   X: 150 cm (behind player view)    │
│   Y: 0 cm (centered)                │
│   Z: 0 cm (same height)             │
└─────────────────────────────────────┘
    ↓
[Apply Target Rotation]
    ├─ Get target facing direction
    ├─ Create rotation matrix
    ├─ Multiply offset × rotation matrix
    │
    └─ Result: World-space offset
    ↓
[Calculate Final Position]
    ├─ Final Pos = Target Location + Rotated Offset
    │
    └─ Example:
       ├─ Target: (1000, 500, 300)
       ├─ Rotation: 90° (facing +Y)
       ├─ Offset: (150, 0, 0) rotated = (0, 150, 0)
       └─ Final: (1000, 650, 300)
```

### **Blueprint Node Graph - Calculate Offset**

```
[Calculate Dino Offset]
    ↓
[Inputs]
    ├─ TargetLocation (Vector)
    ├─ TargetRotation (Rotator)
    └─ BaseOffset (Vector) = (150, 0, 0)
    ↓
[Convert Rotation to Matrix]
    ├─ Create Rotation Matrix from Rotator
    └─ Store in RotationMatrix
    ↓
[Transform Offset]
    ├─ Vector Transformed = RotationMatrix × BaseOffset
    │
    └─ This rotates offset to match facing
    ↓
[Calculate Final Position]
    ├─ FinalPosition = TargetLocation + Transformed
    │
    └─ Clamp to map bounds
    ↓
[Validate Ground Level]
    ├─ Raycast down from FinalPosition
    ├─ Check collision distance
    ├─ Adjust Z if needed
    │  └─ FinalPosition.Z = Ground Level + 10cm
    │
    └─ Return FinalPosition
```

---

## 🔄 MOUNT STATE REPLICATION

### **Networked Variables**

```
Character Blueprint:

Variables:
├─ bIsMounted
│  ├─ Replication: Replicated
│  ├─ On Rep Notify: OnRep_IsMounted
│  └─ Sync across network
│
├─ MountedDino
│  ├─ Type: Pawn reference
│  ├─ Replication: Replicated
│  ├─ On Rep Notify: OnRep_MountedDino
│  └─ Tells clients which dino is mounted
│
├─ DinoSaddlePosition
│  ├─ Type: Vector
│  ├─ Replication: Replicated
│  └─ Where player sits on dino
│
└─ MountAnimationPlaying
    ├─ Type: Boolean
    ├─ Replication: Replicated
    └─ Sync animations on all clients
```

### **OnRep Functions**

**OnRep_IsMounted:**

```
[IsMounted Replicated]
    ↓
[Client Receives Change]
    ├─ If TRUE (just mounted)
    │  ├─ Update animation state
    │  ├─ Switch to mounted camera
    │  └─ Disable ground movement anim
    │
    └─ If FALSE (just dismounted)
       ├─ Update animation state
       ├─ Switch to first-person camera
       └─ Enable movement animations
```

**OnRep_MountedDino:**

```
[MountedDino Changed]
    ↓
[Client Updates Mount Info]
    ├─ Get Mounted Dino reference
    ├─ Verify dino is valid
    ├─ Update UI (show dino name)
    ├─ Update camera attachment
    │  └─ Follow dino head position
    │
    └─ Multicast animation sync
       └─ Make sure all clients show mount anim
```

---

## ⚠️ ERROR HANDLING - Dinó Problémák

### **Problem 1: Dino Dies During Teleport**

```
Detection:
[In Mount Sequence]
    ├─ Before teleport: Health > 0
    ├─ During/After teleport: Health <= 0
    │
    └─ Problem detected!
    ↓
[Handle Death]
    ├─ Stop teleport sequence
    ├─ Dismount player (safe)
    ├─ Don't teleport dino
    ├─ Show error message
    │  └─ "Your dino died!"
    │
    └─ Teleport player to location anyway
       └─ Dino stays at origin point
```

### **Problem 2: Dino Stuck in Geometry**

```
Detection:
[After Teleport]
    ├─ Raycast check around dino
    ├─ Check for collision blocks
    │
    └─ Stuck detected!
    ↓
[Resolve Stuck]
    ├─ Move dino up Z by 200cm
    │  └─ FinalPos.Z += 200
    │
    ├─ Raycast down again
    ├─ Settle on new position
    │
    └─ If still stuck:
       ├─ Teleport dino back to origin
       ├─ Dismount player
       └─ Show warning
```

### **Problem 3: Remount Position Invalid**

```
Detection:
[Before Remount]
    ├─ Check mount socket validity
    ├─ Check dino skeleton
    │
    └─ If invalid → Can't mount!
    ↓
[Handle Invalid Mount]
    ├─ Use alternative mount point
    │  └─ Use dino root socket
    │
    ├─ Adjust player position
    │  ├─ Offset from dino center
    │  └─ Place on saddle manually
    │
    └─ Complete mount
```

---

## 🎮 ANIMATION BLUEPRINT DETAILS

### **Animation Montage Setup**

```
Animation Montage: Montage_Mount_Start
├─ Track: Default Slot
├─ Duration: 1.0 seconds
│
├─ Frames:
│  ├─ 0.0s: Character_Mount_01
│  ├─ 0.5s: Character_Mount_02
│  ├─ 0.8s: Character_Mount_03
│  └─ 1.0s: Character_Mounted_Idle
│
├─ Notifies:
│  └─ At frame 0.8s
│     └─ EventNotify_MountComplete
│        ├─ Trigger: Perform actual mount
│        ├─ Called from Node Graph
│        └─ Complete animation before sync
│
└─ Sections: None (Linear)

Animation Montage: Montage_Dismount_Start
├─ Track: Default Slot
├─ Duration: 0.8 seconds
│
├─ Frames:
│  ├─ 0.0s: Character_Mounted_Idle
│  ├─ 0.3s: Character_Dismount_01
│  ├─ 0.6s: Character_Dismount_02
│  └─ 0.8s: Character_Landing_Impact
│
├─ Notifies:
│  └─ At frame 0.3s
│     └─ EventNotify_DismountStart
│        ├─ Trigger: Detach from dino
│        └─ Begin falling
│
└─ Sections: None (Linear)
```

### **Montage Playback in Blueprint**

```
[Play Mount Animation]
    ├─ Get Animation Blueprint instance
    ├─ Montage to play: Montage_Mount_Start
    │
    ├─ Parameters:
    │  ├─ Starting position: 0.0
    │  ├─ Play rate: 1.0
    │  ├─ Loop count: 1
    │  └─ Stop when finished: TRUE
    │
    ├─ Event Delegates:
    │  ├─ OnMontageBegan
    │  │  └─ Disable player input
    │  │
    │  ├─ OnMontageInterrupted
    │  │  └─ Handle if cancelled
    │  │
    │  └─ OnMontageEnded
    │     └─ Re-enable input
    │
    └─ Wait for montage to complete
```

---

## 🐴 SADDLE POSITION SETUP

### **Dino Skeleton Sockets**

```
Dino Skeleton:
├─ Socket: Saddle_Position (mount point)
│  ├─ Location: Center of back
    ├─ Rotation: Aligned with dino forward
│  └─ Offset: 0, 0, 150cm up
│
├─ Socket: Camera_Mount
│  ├─ Location: Top of dino head
│  ├─ Offset: 0, 0, 100cm up
│  └─ For mounted camera
│
└─ Socket: Rider_Left / Rider_Right
   └─ For multiple riders (future)
```

### **Player Socket Attachment**

```
Node Graph:
[Mount Player on Dino]
    ├─ Get Player Character
    ├─ Get Dino (Pawn)
    │
    ├─ Get Dino Mesh component
    ├─ Get Socket: "Saddle_Position"
    │
    ├─ Attach Character to Dino
    │  ├─ Target: Dino mesh
    │  ├─ Socket name: "Saddle_Position"
    │  ├─ Location rule: SnapToTarget
    │  ├─ Rotation rule: KeepRelativeRotation
    │  └─ Maintain world rotation: FALSE
    │
    └─ Player now follows dino movement
       ├─ Dino moves → Player moves with it
       ├─ Dino rotates → Player rotates with it
       └─ Dino attacks → Player seated through it
```

---

## 📊 TESTING CHECKLIST - Dinó Kezelés

### **Mount Tests**
- [ ] Player mounts dino without lag
- [ ] Mount animation plays on all clients
- [ ] Player properly seated on saddle
- [ ] Camera follows dino head (mounted view)
- [ ] Input controls work while mounted
- [ ] Dino can move while player mounted

### **Dismount Tests**
- [ ] Dismount animation smooth
- [ ] Player lands safely on ground
- [ ] No collision bugs with dino
- [ ] Player can move after dismount
- [ ] Dino movement continues normal

### **Teleport + Dinó Tests**
- [ ] Dismount before teleport works
- [ ] Dino teleports with correct offset
- [ ] Remount after teleport smooth
- [ ] All clients see dino teleport
- [ ] Dinó collision correct after teleport
- [ ] No clipping issues

### **Edge Cases**
- [ ] Dino dies mid-teleport
- [ ] Dino becomes stuck in geometry
- [ ] Mount position is invalid
- [ ] Player too far from dino
- [ ] Dino too high above ground
- [ ] Water spawn handling

---

**Verzió**: 1.0  
**DevKit**: 5.5.4  
**Dátum**: 2026-06-22  
**Status**: 🟡 Tervezett - Animation setup szükséges
