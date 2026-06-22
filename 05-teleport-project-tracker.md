# Teleport Rendszer - Projekt Fejlesztési Útmutató
## Progress Tracker & Implementation Schedule

---

## 📌 Projekt Típus

**Komplexitás**: ⭐⭐⭐⭐⭐ (Nagyon összetett)  
**Becsült munkaidő**: 80-120 óra  
**Fejlesztési szakaszok**: 8 fázis  
**Team méret ajánlott**: 2-3 fejlesztő

---

## 🎯 VÉGLEGES PROJEKT CÉLOK

```
✅ TELJES RENDSZER:

Functionality:
├─ Játékos teleportálása koordinátára
├─ Dinó teleportálása a játékossal (offset)
├─ Vissza-teleportálás (home pont)
├─ Teleport pontok felfedezése/regisztrálása
├─ Cooldown rendszer
└─ Multicast replikáció (összes játékos látja)

Visual Effects:
├─ Departure effect (Cyan expanding sphere)
├─ Arrival effect (White/Gold burst)
├─ Optional trail effect (Glowing path)
├─ Screen shake (csak lokális játékos)
└─ Light flashes

Audio:
├─ Departure sound (Whoosh)
├─ Arrival sound (Chime)
├─ Travel sound (optional ambient)
└─ UI sounds (menu interactions)

User Interface:
├─ Teleport menu (TAB gomb)
├─ Teleport point lista
├─ Mini map (opcionális)
├─ Confirmation dialógus
├─ Cooldown display
└─ Error messages

Networking:
├─ Server authority
├─ Multicast RPC effects
├─ Position replication
├─ Animation synchronization
└─ Latency handling

Dinó Integration:
├─ Mount/Dismount animációk
├─ Safe positioning
├─ Offset calculations
├─ Physics handling
└─ Remount logic
```

---

## 📅 FÁZISOK ÉS ÜTEMEZÉS

### **FÁZIS 1: Foundation & Setup (Hét 1)**

**Cél**: Alapvető blueprint struktúrák és alap logika

```
Feladatok:
├─ [ ] BP_TeleportPoint létrehozása
│   ├─ Components (Collision, Light, Particle)
│   ├─ Variables (Location, Rotation, Name)
│   └─ Basic functions
│
├─ [ ] BP_TeleportManager létrehozása
│   ├─ Point registration system
│   ├─ Array management
│   └─ Basic teleport logic (no effects yet)
│
├─ [ ] BP_TeleportEffectManager létrehozása
│   ├─ Component structure
│   ├─ Variable setup
│   └─ Placeholder functions
│
└─ [ ] Test level készítése
    ├─ Spawn test points
    ├─ Add to scene
    └─ Basic manual testing

Dokumentáció:
├─ Update progress trackers
├─ Note any issues/blockers
└─ Update estimated timeline

Estimated Time: 16-20 óra
Deliverable: Basic blueprint structure, manual testing working
```

### **FÁZIS 2: Core Teleport Logic (Hét 1-2)**

**Cél**: Alapvető teleportálás logika működő

```
Feladatok:
├─ [ ] Execute Teleport Node Graph
│   ├─ Player teleport logic
│   ├─ Validate target location
│   ├─ Handle location adjustments
│   └─ Server authority checks
│
├─ [ ] Return Point System
│   ├─ Save departure location
│   ├─ Retrieve return point
│   └─ Home teleport button
│
├─ [ ] Cooldown System
│   ├─ Track last teleport time
│   ├─ Check cooldown validity
│   └─ Display remaining time
│
├─ [ ] Basic Player Input
│   ├─ Key binding (TAB)
│   ├─ Simple UI toggle
│   └─ Console command (testing)
│
└─ [ ] Single Player Testing
    ├─ Teleport works locally
    ├─ Return point works
    ├─ Cooldown works
    └─ No crashes

Estimated Time: 20-24 óra
Deliverable: Working teleport in single player, no effects
```

### **FÁZIS 3: Dino Integration (Hét 2-3)**

**Cél**: Dinó teleportálása a játékossal

```
Feladatok:
├─ [ ] Detect Mounted Dino
│   ├─ Get current mount reference
│   ├─ Validate dino is valid
│   ├─ Check dino health
│   └─ Check dino is tamed
│
├─ [ ] Safe Dismount Logic
│   ├─ Play dismount animation
│   ├─ Disable dino movement
│   ├─ Store dino reference
│   └─ Wait for animation
│
├─ [ ] Dino Offset Calculations
│   ├─ Calculate offset position
│   ├─ Apply rotation to offset
│   ├─ Ground level validation
│   └─ Collision checking
│
├─ [ ] Dino Teleportation
│   ├─ Disable physics
│   ├─ Teleport to offset location
│   ├─ Enable physics after settle
│   └─ Validate position
│
├─ [ ] Safe Remount Logic
│   ├─ Play mount animation
│   ├─ Attach player to dino
│   ├─ Re-enable controls
│   └─ Update camera
│
└─ [ ] Dino Testing
    ├─ Mount/dismount smooth
    ├─ Offset calculations correct
    ├─ No clipping/stuck issues
    └─ Remount works

Estimated Time: 20-28 óra
Deliverable: Complete dino teleport system, animations synced
```

### **FÁZIS 4: VFX & SFX Implementation (Hét 3-4)**

**Cél**: Teljes effekt rendszer

```
Feladatok:
├─ [ ] Asset Creation/Import
│   ├─ Particle system templates (3)
│   ├─ Sound effects (3)
│   ├─ Materials (2)
│   └─ Post-process materials
│
├─ [ ] Particle System Configuration
│   ├─ PT_Teleport_Departure
│   │   ├─ Emitter setup
│   │   ├─ Particle behavior
│   │   └─ Color/size tweaks
│   │
│   ├─ PT_Teleport_Arrival
│   │   ├─ Burst emitter
│   │   ├─ Ring expansion
│   │   └─ Glow effects
│   │
│   └─ PT_Teleport_Trail (optional)
│       ├─ Bezier path setup
│       └─ Color fade
│
├─ [ ] Sound Cue Setup
│   ├─ SC_Teleport_Departure
│   │   ├─ Wave player
│   │   ├─ Volume/pitch settings
│   │   └─ Attenuation
│   │
│   └─ SC_Teleport_Arrival
│       ├─ Wave player
│       ├─ Volume/pitch settings
│       └─ Attenuation
│
├─ [ ] Effect Manager Blueprint
│   ├─ Spawn departure effect
│   ├─ Spawn arrival effect
│   ├─ Spawn trail effect
│   ├─ Camera shake setup
│   └─ Light components
│
├─ [ ] Effect Testing
│   ├─ Particles visible
│   ├─ Sounds play correctly
│   ├─ Timing is good
│   ├─ Colors are right
│   └─ Performance acceptable

Estimated Time: 24-32 óra
Deliverable: All effects working, asset-ready
```

### **FÁZIS 5: Multicast RPC & Networking (Hét 4-5)**

**Cél**: Network replikáció teljes szinkronizáció

```
Feladatok:
├─ [ ] Multicast RPC Setup
│   ├─ ServerExecuteTeleport RPC
│   ├─ MulticastSpawnDepartureEffect
│   ├─ MulticastSpawnArrivalEffect
│   └─ Error handling RPC
│
├─ [ ] Replication Variables
│   ├─ LastTeleportLocation
│   ├─ LastTeleportedCharacter
│   ├─ LastTeleportTime
│   └─ OnRep callbacks
│
├─ [ ] Network Authority
│   ├─ Server-side validation
│   ├─ Client permission checks
│   ├─ Anti-cheat measures
│   └─ Error propagation
│
├─ [ ] Multiplayer Testing
│   ├─ 2-player testing
│   ├─ 4-player testing
│   ├─ Effect synchronization
│   ├─ No desync issues
│   └─ Latency handling (test 500ms+)
│
└─ [ ] Network Debugging
    ├─ Network stats logging
    ├─ RPC timing checks
    └─ Bandwidth analysis

Estimated Time: 20-24 óra
Deliverable: Full network sync, multiplayer tested
```

### **FÁZIS 6: UI Implementation (Hét 5-6)**

**Cél**: Teljes User Interface

```
Feladatok:
├─ [ ] WBP_TeleportMainMenu
│   ├─ Layout hierarchy
│   ├─ Teleport point list
│   ├─ Scroll box setup
│   ├─ Button styling
│   ├─ Cooldown display
│   └─ Logic implementation
│
├─ [ ] WBP_TeleportPointButton
│   ├─ Button layout
│   ├─ Text/icons
│   ├─ Hover effects
│   ├─ Click handlers
│   └─ Color coding
│
├─ [ ] WBP_TeleportMap (optional)
│   ├─ Mini map layout
│   ├─ Point markers
│   ├─ Player indicator
│   ├─ Zoom/pan controls
│   └─ Click to select
│
├─ [ ] Dialog Widgets
│   ├─ WBP_ConfirmationDialog
│   ├─ WBP_LoadingScreen
│   ├─ WBP_ErrorMessage
│   └─ Styling/colors
│
├─ [ ] Player Input Binding
│   ├─ TAB key for menu
│   ├─ H key for home
│   ├─ ESC for close
│   ├─ Enter for confirm
│   └─ Test all inputs
│
├─ [ ] HUD Integration
│   ├─ Teleport button display
│   ├─ Cooldown indicator
│   ├─ Notification system
│   └─ Error messages
│
└─ [ ] UI Testing
    ├─ All widgets display
    ├─ Buttons work
    ├─ No input lag
    ├─ Performance good
    └─ Looks polished

Estimated Time: 24-28 óra
Deliverable: Complete UI, fully functional
```

### **FÁZIS 7: Polish & Optimization (Hét 6-7)**

**Cél**: Performance és visual/audio szépítés

```
Feladatok:
├─ [ ] Animation Polish
│   ├─ Timing adjustments
│   ├─ Easing functions
│   ├─ Camera transitions
│   └─ Fade effects
│
├─ [ ] Audio Mixing
│   ├─ Volume balancing
│   ├─ Frequency EQ
│   ├─ Spatial audio (3D)
│   └─ Master volume
│
├─ [ ] Visual Refinement
│   ├─ Particle tweaks
│   ├─ Light adjustments
│   ├─ Color grading
│   ├─ Screen effects
│   └─ Glow/bloom
│
├─ [ ] Performance Optimization
│   ├─ Particle system optimization
│   ├─ Draw call reduction
│   ├─ Network bandwidth optimization
│   ├─ Memory profiling
│   └─ Frame rate testing
│
├─ [ ] Edge Case Handling
│   ├─ Error recovery
│   ├─ Lag compensation
│   ├─ Crash prevention
│   └─ Data validation
│
└─ [ ] Final Testing Pass
    ├─ Solo play complete run
    ├─ Multiplayer stress test
    ├─ Long session stability
    └─ Build check

Estimated Time: 16-20 óra
Deliverable: Polished, optimized, stable system
```

### **FÁZIS 8: Documentation & Finalization (Hét 7-8)**

**Cél**: Teljes dokumentáció és final review

```
Feladatok:
├─ [ ] Code Documentation
│   ├─ Blueprint comments
│   ├─ Variable descriptions
│   ├─ Function documentation
│   └─ Node graph labels
│
├─ [ ] Asset Organization
│   ├─ Folder structure
│   ├─ Naming conventions
│   ├─ Version control
│   └─ Asset dependencies
│
├─ [ ] Video Documentation
│   ├─ How-to guides
│   ├─ Troubleshooting video
│   ├─ Feature overview
│   └─ Network setup guide
│
├─ [ ] Final QA Pass
│   ├─ Feature completeness check
│   ├─ Bug sweep
│   ├─ Performance metrics
│   └─ Security review
│
└─ [ ] Release Preparation
    ├─ Build release version
    ├─ Final testing
    ├─ README.md update
    └─ GitHub upload

Estimated Time: 8-12 óra
Deliverable: Complete package, ready for use
```

---

## 📊 OVERALL TIMELINE

```
Heti Breakdown:

Hét 1:  FÁZIS 1 + FÁZIS 2 Start
        ├─ Foundation setup
        ├─ Basic teleport logic
        └─ 36-44 óra

Hét 2:  FÁZIS 2 Completion + FÁZIS 3 Start
        ├─ Cooldown system
        ├─ Dino integration begins
        └─ 40-48 óra

Hét 3:  FÁZIS 3 Completion + FÁZIS 4 Start
        ├─ Dino animations
        ├─ Asset creation
        └─ 44-52 óra

Hét 4:  FÁZIS 4 Completion + FÁZIS 5 Start
        ├─ Effect implementation
        ├─ Network setup
        └─ 44-56 óra

Hét 5:  FÁZIS 5 Completion + FÁZIS 6 Start
        ├─ Multicast testing
        ├─ UI development
        └─ 44-52 óra

Hét 6:  FÁZIS 6 Completion + FÁZIS 7 Start
        ├─ UI polish
        ├─ Visual refinement
        └─ 40-48 óra

Hét 7:  FÁZIS 7 Completion + FÁZIS 8 Start
        ├─ Optimization
        ├─ Documentation
        └─ 36-44 óra

Hét 8:  FÁZIS 8 Completion
        ├─ Final QA
        ├─ Release prep
        └─ 24-32 óra

TOTAL: 80-120+ óra (10-15 óra/nap, 2 hónap)
```

---

## ✅ MILESTONES

```
Milestone 1: Foundation Complete ✓
├─ Blueprint structure complete
├─ Basic teleport works (local)
└─ Target: End of Hét 1

Milestone 2: Core Features Working ✓
├─ Teleport + dino system
├─ Cooldown working
├─ Single player complete
└─ Target: End of Hét 3

Milestone 3: Network Ready ✓
├─ Multicast RPC working
├─ Effects synchronized
├─ Multiplayer tested
└─ Target: End of Hét 5

Milestone 4: Feature Complete ✓
├─ UI complete
├─ All features implemented
├─ Polish pass done
└─ Target: End of Hét 7

Milestone 5: Release Ready ✓
├─ Documentation done
├─ Final QA passed
├─ Performance optimized
└─ Target: End of Hét 8
```

---

## 🛑 CRITICAL BLOCKERS

```
Must Fix Before Proceeding:

1. Animation Synchronization
   └─ If montages don't sync → can't progress to multiplayer
   
2. Multicast RPC Issues
   └─ If network doesn't replicate → entire system breaks
   
3. Dino Physics
   └─ If physics broken → dino placement fails
   
4. Memory Leaks
   └─ If spawned actors not destroyed → crash risk
```

---

## 🔧 TROUBLESHOOTING GUIDE

### **Common Issues & Solutions**

```
Issue 1: Dino teleports but player doesn't
Solution:
├─ Check player reference validity
├─ Check server authority (should be server only)
├─ Verify coordinates
└─ Check collision sphere size

Issue 2: Effects don't show for all players
Solution:
├─ Verify Multicast RPC is called
├─ Check RPC reliability (should be Reliable)
├─ Test with simulated latency
├─ Check Effect Manager reference

Issue 3: Dinó stuck in geometry
Solution:
├─ Increase offset distance (try 200cm instead of 150)
├─ Add raycast validation before/after
├─ Check ground level detection
└─ Increase collision tolerance

Issue 4: Animation montages don't play
Solution:
├─ Check animation blueprint exists
├─ Verify montage is imported
├─ Check play rate/timing
├─ Verify character has mesh component

Issue 5: Cooldown not syncing
Solution:
├─ Replicate LastTeleportTime
├─ Check server time synchronization
├─ Verify cooldown calculation
└─ Test on 500ms latency
```

---

## 📈 SUCCESS METRICS

```
✓ Performance Targets:
├─ Frame rate: 60 FPS minimum
├─ Network latency: < 5 second total
├─ Memory footprint: < 500 MB
└─ Draw calls: < 2000

✓ Functionality:
├─ Teleport accuracy: < 10cm error
├─ Network sync: < 100ms delay
├─ Error recovery: 100% working
└─ Feature completeness: 100%

✓ User Experience:
├─ Menu response: < 100ms
├─ Teleport duration: 1.0 second
├─ Animation smoothness: 60 FPS
└─ Visual quality: High

✓ Code Quality:
├─ Crash rate: 0%
├─ Memory leaks: 0
├─ Code documentation: 100%
└─ Comment coverage: > 80%
```

---

## 📝 DAILY LOG TEMPLATE

```
Dátum: YYYY-MM-DD
Fázis: [FÁZIS X]
Team Member: [Name]

Completed Today:
├─ [ ] Task 1 - X óra
├─ [ ] Task 2 - Y óra
└─ [ ] Task 3 - Z óra

Total Hours: X hours

Issues Encountered:
├─ Issue 1: Description
├─ Issue 2: Description
└─ Issue 3: Description

Solutions Applied:
├─ Solution 1
├─ Solution 2
└─ Solution 3

Tomorrow's Goals:
├─ Task 1
├─ Task 2
└─ Task 3

Blockers:
├─ Blocker 1
└─ Blocker 2

Notes:
[Any additional notes]
```

---

## 🎓 RESOURCES & REFERENCES

### **Documentation Links**
```
ARK DevKit:
└─ https://ark.gamepedia.com/Modding

Unreal Engine:
├─ Blueprint basics: [Epic Games docs]
├─ Network replication: [Epic Games docs]
├─ UMG widgets: [Epic Games docs]
└─ Animation systems: [Epic Games docs]

Community:
├─ ARK Modding Discord
├─ UE4/UE5 Forums
├─ YouTube tutorials
└─ This documentation!
```

### **Tools Needed**
```
Software:
├─ ARK DevKit 5.5.4
├─ Unreal Engine 4.27+
├─ Visual Studio or Unreal IDE
├─ A good text editor
└─ Git/GitHub (version control)

Optional:
├─ Wwise (advanced audio)
├─ Substance Painter (materials)
├─ Blender (asset creation)
└─ DaVinci Resolve (video docs)
```

---

## 📞 SUPPORT & QUESTIONS

```
For Issues:
1. Check troubleshooting guide above
2. Review relevant documentation file
3. Check Unreal Engine documentation
4. Ask on ARK modding community

This Project:
├─ Questions about system: See documentation files
├─ Blueprint help: Check implementation guides
├─ VFX questions: See effects documentation
└─ Network issues: See multicast RPC guide
```

---

**Verzió**: 1.0  
**DevKit**: 5.5.4  
**Utolsó frissítés**: 2026-06-22  
**Status**: 🟡 Aktív Fejlesztés

---

## 📋 QUICK STATUS UPDATE

```
Last Updated: 2026-06-22

Current Phase: PREPARATION
├─ Documentation: ✅ COMPLETE
├─ Planning: ✅ COMPLETE
├─ Asset List: ✅ READY
└─ Ready to start: ✅ YES

Next Steps:
├─ Phase 1: Foundation Setup
├─ Create base blueprints
├─ Setup test level
└─ Begin development

Good luck! 🚀
```
