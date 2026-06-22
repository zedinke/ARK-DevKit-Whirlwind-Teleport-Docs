# ⚡ Whirlwind Event System - Project Tracker
## Fejlesztési Ütemezés & Progress Management

---

## 📌 PROJEKT CÉLJA

```
Cél: Egy enterprise-grade event system létrehozása az ARK DevKit-hez,
amely időalapú, forgatott szél viharokkal kell megjelenniük a pályán,
vonzási és repulzációs mechanikal, fejlesztői kontroll lehetőséggel,
és teljes multiplay szinkronizációval.

Költségvetés: 100-160 óra fejlesztés
Csapatméret: 2-3 fejlesztő ideális
Időtartam: 4-6 hét intensz fejlesztés
```

---

## 📊 8 FÁZIS RÉSZLETES LEBONTÁSA

### **FÁZIS 1: Foundation & Blueprint Setup (Hét 1)**

**Cél**: Alapvető blueprint struktúrák és komponensek

```
Feladatok:

├─ [ ] BP_WhirlwindEventManager
│   ├─ [ ] Create blueprint
│   ├─ [ ] Add 11 variables
│   ├─ [ ] Setup details panel
│   └─ [ ] Configure sliders/arrays
│
├─ [ ] BP_WhirlwindActor
│   ├─ [ ] Create blueprint
│   ├─ [ ] Add 5 components
│   │   ├─ Sphere Collision
│   │   ├─ Particle System
│   │   ├─ Point Light
│   │   ├─ Audio Component
│   │   └─ Arrow (debug)
│   ├─ [ ] Add 11 variables
│   └─ [ ] Setup details panel
│
├─ [ ] EWhirlwindType Enum
│   ├─ [ ] Create enum
│   └─ [ ] Add 3 values
│
├─ [ ] Test Level
│   ├─ [ ] Create basic test map
│   ├─ [ ] Place manager actor
│   ├─ [ ] Add spawn points (5-10)
│   └─ [ ] Setup basic testing
│
└─ [ ] Documentation
    ├─ [ ] Update progress tracker
    ├─ [ ] Note any blockers
    └─ [ ] Prepare for Fázis 2

Deliverable: Basic blueprint structure, compilable without errors
Testing: Compile check, no runtime crashes
Time: 16-20 óra
Success Criteria: All blueprints created, all variables visible in editor
```

### **FÁZIS 2: Core Event Management (Hét 1-2)**

**Cél**: Event scheduling és spawning logika

```
Feladatok:

├─ [ ] Initialize Manager Function
│   ├─ [ ] Find DayCycleManager reference
│   ├─ [ ] Setup event loop timer
│   └─ [ ] Configure timer interval
│
├─ [ ] Check Event Trigger Function
│   ├─ [ ] Get current game time
│   ├─ [ ] Calculate time until event
│   ├─ [ ] Implement trigger window
│   ├─ [ ] Add cooldown check
│   └─ [ ] Call spawn on trigger
│
├─ [ ] Spawn Whirlwind Event Function
│   ├─ [ ] Spawn actor at location
│   ├─ [ ] Select random type
│   ├─ [ ] Generate patrol path
│   ├─ [ ] Initialize whirlwind
│   ├─ [ ] Add to active list
│   └─ [ ] Setup expiration timer
│
├─ [ ] On Whirlwind Expire Function
│   ├─ [ ] Stop physics
│   ├─ [ ] Remove from active list
│   ├─ [ ] Destroy actor safely
│   └─ [ ] Cleanup memory
│
├─ [ ] Testing
│   ├─ [ ] Manual event triggering works
│   ├─ [ ] Events spawn at correct time
│   ├─ [ ] Max count respected
│   ├─ [ ] Cooldown prevents spam
│   ├─ [ ] 5-minute duration works
│   └─ [ ] No crashes on expiration
│
└─ [ ] Single Player Testing
    ├─ [ ] Create test scenario
    ├─ [ ] Manually set event time
    ├─ [ ] Watch event spawn
    ├─ [ ] Watch event expire
    └─ [ ] Verify all mechanics

Deliverable: Event spawning works in single player
Testing: 1 player can trigger events manually
Time: 20-24 óra
Success Criteria: Events spawn and despawn correctly, no memory leaks
```

### **FÁZIS 3: Physics Implementation (Hét 2-3)**

**Cél**: Attraction, rotation, és launch mechanics

```
Feladatok:

├─ [ ] Attraction Physics Tick
│   ├─ [ ] Get overlapping actors
│   ├─ [ ] Calculate radial force
│   │   ├─ Inverse square law formula
│   │   ├─ Distance falloff
│   │   └─ Force clamping
│   │
│   ├─ [ ] Calculate tangential force
│   │   ├─ Angular velocity
│   │   ├─ Perpendicular vector
│   │   └─ Tangent speed
│   │
│   ├─ [ ] Combine forces
│   │   ├─ Sum radial + tangential
│   │   └─ Total force magnitude clamp
│   │
│   └─ [ ] Apply to character
│       ├─ Get movement component
│       ├─ Add force
│       └─ Clamp velocity
│
├─ [ ] Launch Mechanics
│   ├─ [ ] Detect center proximity
│   ├─ [ ] Calculate launch velocity
│   ├─ [ ] Apply upward force
│   ├─ [ ] Add cooldown tracking
│   ├─ [ ] Spawn launch effects
│   └─ [ ] Disable attraction briefly
│
├─ [ ] Movement System
│   ├─ [ ] Patrol path following
│   ├─ [ ] Waypoint navigation
│   ├─ [ ] Speed control
│   ├─ [ ] Smooth transitions
│   └─ [ ] Rotation toward movement
│
├─ [ ] Lifetime Tracking
│   ├─ [ ] Track elapsed time
│   ├─ [ ] Countdown remaining
│   ├─ [ ] Fade out effect (last 30s)
│   └─ [ ] Trigger expiration
│
├─ [ ] Physics Testing
│   ├─ [ ] Attraction pulls actors
│   ├─ [ ] Spiral motion visible
│   ├─ [ ] Launch at correct distance
│   ├─ [ ] Launch height correct
│   ├─ [ ] Movement smooth
│   ├─ [ ] No clipping with terrain
│   └─ [ ] No physics instability
│
└─ [ ] Debugging
    ├─ [ ] Add debug draw forces
    ├─ [ ] Add debug print info
    ├─ [ ] Visualize attraction zone
    └─ [ ] Verify calculations

Deliverable: Full physics system working in single player
Testing: Actor gets pulled in, spirals, launches upward
Time: 24-32 óra
Success Criteria: Physics feels natural, no bugs/clipping, smooth motion
```

### **FÁZIS 4: Scheduling Integration (Hét 3)**

**Cél**: Day Cycle Manager integráció és time-based triggering

```
Feladatok:

├─ [ ] Day Cycle Manager Integration
│   ├─ [ ] Locate manager in level
│   ├─ [ ] Get time function working
│   ├─ [ ] Verify time format (0-1440)
│   └─ [ ] Test multiple times of day
│
├─ [ ] Event Trigger Configuration
│   ├─ [ ] Setup EventTriggerTime variable
│   ├─ [ ] Configure editor slider (0-1440)
│   ├─ [ ] Create preset times enum
│   ├─ [ ] Add dropdown selection
│   └─ [ ] Test multiple trigger times
│
├─ [ ] Cooldown System
│   ├─ [ ] Implement 10-min cooldown
│   ├─ [ ] Track LastEventTriggerTime
│   ├─ [ ] Test cooldown logic
│   └─ [ ] Verify spam prevention
│
├─ [ ] Console Commands (Optional)
│   ├─ [ ] /whirlwind time [min]
│   ├─ [ ] /whirlwind currenttime
│   ├─ [ ] /whirlwind spawn [type]
│   ├─ [ ] /whirlwind reset
│   └─ [ ] Test all commands
│
├─ [ ] Scheduling Testing
│   ├─ [ ] Manual time setting works
│   ├─ [ ] Event triggers at right time
│   ├─ [ ] Cooldown prevents repeats
│   ├─ [ ] Works across day cycles
│   └─ [ ] Multiple triggers per day work
│
└─ [ ] UI Integration
    ├─ [ ] Display current time
    ├─ [ ] Show time until event
    ├─ [ ] Show active events count
    └─ [ ] Update UI on state change

Deliverable: Time-based scheduling fully working
Testing: Events spawn on schedule, cooldown works
Time: 16-20 óra
Success Criteria: Events trigger at configured time, repeat daily
```

### **FÁZIS 5: Multicast Networking (Hét 4)**

**Cél**: Server authority és network replication

```
Feladatok:

├─ [ ] Replication Setup
│   ├─ [ ] Mark variables Replicated
│   │   ├─ ActiveWhirlwinds array
│   │   ├─ LastEventTriggerTime
│   │   ├─ WhirlwindType
│   │   ├─ RemainingDuration
│   │   └─ PatrolPath
│   │
│   ├─ [ ] Setup On Rep Notifies
│   │   ├─ OnRep_EventTriggered
│   │   ├─ OnRep_DurationChanged
│   │   └─ OnRep_WaypointChanged
│   │
│   └─ [ ] Configure replication settings
│       ├─ NetUpdateFrequency
│       ├─ NetPriority
│       └─ Update frequency
│
├─ [ ] Multicast RPC Functions
│   ├─ [ ] MulticastSpawnWhirlwindEffects
│   │   ├─ Broadcast to all clients
│   │   ├─ Spawn particles
│   │   ├─ Play audio
│   │   └─ Create light
│   │
│   ├─ [ ] MulticastDespawnWhirlwindEffects
│   │   ├─ Broadcast to all clients
│   │   ├─ Fade out effects
│   │   └─ Cleanup
│   │
│   └─ [ ] Test RPC calls
│       ├─ Broadcast works
│       ├─ All clients receive
│       └─ Timing correct
│
├─ [ ] Server Authority
│   ├─ [ ] Physics only on server
│   ├─ [ ] Position updates from server
│   ├─ [ ] Client prediction setup
│   └─ [ ] Validation checks
│
├─ [ ] Multiplayer Testing
│   ├─ [ ] 2-player: Spawn sync
│   ├─ [ ] 2-player: Effects sync
│   ├─ [ ] 4-player: Stability check
│   ├─ [ ] High latency test (500ms)
│   ├─ [ ] Attraction replication
│   ├─ [ ] Launch replication
│   └─ [ ] No desync issues
│
└─ [ ] Network Debugging
    ├─ [ ] Log RPC calls
    ├─ [ ] Monitor bandwidth
    ├─ [ ] Check for over-replication
    └─ [ ] Verify authority

Deliverable: Full network synchronization working
Testing: 2+ players see effects simultaneously, physics synced
Time: 20-24 óra
Success Criteria: Multiplayer tested, no desyncs, authority correct
```

### **FÁZIS 6: VFX & SFX Effects (Hét 4-5)**

**Cél**: Particle systems és audio setup

```
Feladatok:

├─ [ ] Particle Systems
│   ├─ [ ] PT_Whirlwind_Frost
│   │   ├─ Create particle system
│   │   ├─ Setup emitter (cyan spiral)
│   │   ├─ Color over lifetime
│   │   ├─ Size over lifetime
│   │   └─ Create material M_Whirlwind_Frost
│   │
│   ├─ [ ] PT_Whirlwind_Inferno
│   │   ├─ Create particle system
│   │   ├─ Setup emitter (fire tornado)
│   │   ├─ Color over lifetime (red/orange)
│   │   ├─ Size over lifetime
│   │   └─ Create material M_Whirlwind_Inferno
│   │
│   └─ [ ] PT_Whirlwind_Aqua
│       ├─ Create particle system
│       ├─ Setup emitter (water spiral)
│       ├─ Color over lifetime (blue/green)
│       ├─ Size over lifetime
│       └─ Create material M_Whirlwind_Aqua
│
├─ [ ] Audio Setup
│   ├─ [ ] SFX_Whirlwind_Frost_Loop
│   │   ├─ Import audio file
│   │   ├─ Create sound cue SC_Whirlwind_Frost_Loop
│   │   ├─ Setup attenuation (5000 cm)
│   │   └─ Test audio playback
│   │
│   ├─ [ ] SFX_Whirlwind_Inferno_Loop
│   │   ├─ Import audio file
│   │   ├─ Create sound cue
│   │   ├─ Setup attenuation
│   │   └─ Test volume/pitch
│   │
│   ├─ [ ] SFX_Whirlwind_Aqua_Loop
│   │   ├─ Import audio file
│   │   ├─ Create sound cue
│   │   ├─ Setup attenuation
│   │   └─ Test audio playback
│   │
│   ├─ [ ] Enhancement Sounds (x3 types)
│   │   ├─ Attract sounds
│   │   └─ Launch sounds
│   │
│   └─ [ ] Audio Mixing
│       ├─ Volume balancing
│       ├─ Frequency EQ
│       ├─ 3D spatialization
│       └─ Master levels
│
├─ [ ] Light Effects
│   ├─ [ ] Frost light (cyan, 2000 lm)
│   ├─ [ ] Inferno light (orange, 3500 lm)
│   ├─ [ ] Aqua light (blue, 2200 lm)
│   └─ [ ] Attenuation setup
│
├─ [ ] Effects Testing
│   ├─ [ ] Particles visible & correct
│   ├─ [ ] Audio plays correctly
│   ├─ [ ] Lights glow properly
│   ├─ [ ] All synced via multicast
│   ├─ [ ] Multiplayer sync verified
│   └─ [ ] No performance drops
│
└─ [ ] Polish Effects
    ├─ [ ] Timing adjustments
    ├─ [ ] Color fine-tuning
    ├─ [ ] Audio mixing
    ├─ [ ] Visual polish
    └─ [ ] Performance optimize

Deliverable: All VFX/SFX working, synced across network
Testing: Effects visible, audio audible, synced in multiplayer
Time: 24-32 óra
Success Criteria: All effects correct, no performance issues
```

### **FÁZIS 7: Developer Controls & Polish (Hét 5)**

**Cél**: Editor parameters és optimization

```
Feladatok:

├─ [ ] Editor Parameter Setup
│   ├─ [ ] EventTriggerTime slider
│   │   ├─ Range 0-1440
│   │   ├─ Visual slider in editor
│   │   └─ Real-time update
│   │
│   ├─ [ ] MaxSimultaneousWhirlwinds
│   │   ├─ Slider 1-10
│   │   ├─ Live update
│   │   └─ Enforce limit
│   │
│   ├─ [ ] EventScheduleEnabled toggle
│   │   ├─ Enable/disable system
│   │   └─ Works in play
│   │
│   ├─ [ ] WhirlwindSpawnLocations array
│   │   ├─ Add/edit/remove points
│   │   ├─ Visual in viewport
│   │   └─ Preview positions
│   │
│   └─ [ ] Type selection checkboxes
│       ├─ Frost enabled/disabled
│       ├─ Inferno enabled/disabled
│       └─ Aqua enabled/disabled
│
├─ [ ] Console Commands
│   ├─ [ ] /whirlwind time [min]
│   ├─ [ ] /whirlwind currenttime
│   ├─ [ ] /whirlwind spawn [type]
│   ├─ [ ] /whirlwind list
│   ├─ [ ] /whirlwind reset
│   └─ [ ] /whirlwind enabled true/false
│
├─ [ ] UI Improvements
│   ├─ [ ] Display active events
│   ├─ [ ] Show timer countdown
│   ├─ [ ] Event notifications
│   ├─ [ ] Dev mode display (debug info)
│   └─ [ ] Error messages
│
├─ [ ] Performance Optimization
│   ├─ [ ] Particle count optimization
│   │   └─ Max 1000 particles/vortex
│   │
│   ├─ [ ] Physics optimization
│   │   └─ Force calc < 1ms
│   │
│   ├─ [ ] Network optimization
│   │   └─ Update rate 0.1s
│   │
│   ├─ [ ] Memory optimization
│   │   └─ Auto-cleanup on expire
│   │
│   └─ [ ] FPS Target: 60+
│
├─ [ ] Animation Polish
│   ├─ [ ] Timing refinement
│   ├─ [ ] Easing functions
│   ├─ [ ] Smooth transitions
│   └─ [ ] Visual feedback
│
└─ [ ] Audio Polish
    ├─ [ ] Volume balancing
    ├─ [ ] EQ tuning
    ├─ [ ] Compression settings
    └─ [ ] 3D audio verify

Deliverable: Dev-friendly system, optimized, polished
Testing: Editor controls work, no lag, console commands functional
Time: 16-20 óra
Success Criteria: Developer can easily configure, system optimized
```

### **FÁZIS 8: Documentation & QA (Hét 5-6)**

**Cél**: Testing, documentation, release prep

```
Feladatok:

├─ [ ] Comprehensive QA Testing
│   ├─ [ ] Solo play testing
│   │   ├─ Event spawn at time
│   │   ├─ Physics attraction
│   │   ├─ Launch mechanic
│   │   ├─ Movement system
│   │   ├─ 5 min duration
│   │   └─ All no crashes
│   │
│   ├─ [ ] Multiplayer testing
│   │   ├─ 2-player full cycle
│   │   ├─ 4-player stability
│   │   ├─ Effects sync verified
│   │   ├─ Audio sync tight
│   │   ├─ High latency test
│   │   └─ No desyncs
│   │
│   ├─ [ ] Edge case testing
│   │   ├─ Multiple events simultaneous
│   │   ├─ Max count enforcement
│   │   ├─ Cooldown prevention
│   │   ├─ Memory cleanup
│   │   └─ No memory leaks
│   │
│   └─ [ ] Stress testing
│       ├─ 10 simultaneous vortexes
│       ├─ 50+ affected actors
│       ├─ 1-hour session stability
│       └─ FPS maintained 60+
│
├─ [ ] Documentation
│   ├─ [ ] Code comments added
│   ├─ [ ] Function documentation
│   ├─ [ ] Variable descriptions
│   ├─ [ ] Node graph labels
│   └─ [ ] Troubleshooting guide
│
├─ [ ] Asset Organization
│   ├─ [ ] Folder structure clean
│   ├─ [ ] Naming conventions follow
│   ├─ [ ] All assets named properly
│   └─ [ ] Dependencies clear
│
├─ [ ] Final Build
│   ├─ [ ] Compile check
│   ├─ [ ] No warnings
│   ├─ [ ] Blueprint validation
│   └─ [ ] Package for release
│
└─ [ ] Release Prep
    ├─ [ ] README.md update
    ├─ [ ] Version number set
    ├─ [ ] Changelog created
    ├─ [ ] GitHub upload (if applicable)
    └─ [ ] Final QA pass

Deliverable: Production-ready, fully documented, tested
Testing: Comprehensive QA passed, stress tests successful
Time: 12-16 óra
Success Criteria: Zero known bugs, fully documented, release-ready
```

---

## ⏱️ WEEKLY TIMELINE

```
HÉT 1: FÁZIS 1-2
├─ Mon-Tue: Foundation blueprints
├─ Wed-Thu: Event management logic
├─ Fri: Testing & debug
└─ Total: 36-44 óra

HÉT 2: FÁZIS 2-3 Start
├─ Mon-Tue: Core event logic complete
├─ Wed-Fri: Physics implementation begin
└─ Total: 40-48 óra

HÉT 3: FÁZIS 3-4
├─ Mon-Tue: Physics complete
├─ Wed-Fri: Scheduling integration
└─ Total: 40-48 óra

HÉT 4: FÁZIS 5-6 Start
├─ Mon-Tue: Networking setup
├─ Wed-Fri: VFX/SFX begin
└─ Total: 40-48 óra

HÉT 5: FÁZIS 6-7
├─ Mon-Tue: VFX/SFX complete
├─ Wed-Fri: Polish & optimization
└─ Total: 40-48 óra

HÉT 6: FÁZIS 8
├─ Mon-Tue: QA testing
├─ Wed-Thu: Documentation
├─ Fri: Final release prep
└─ Total: 24-32 óra

TOTAL: 100-160+ ÓRA
```

---

## 📋 DAILY LOG TEMPLATE

```
═══════════════════════════════════════════════════════
DATE: [YYYY-MM-DD]
PHASE: [FÁZIS X]
DEVELOPER: [Name]
═══════════════════════════════════════════════════════

TODAY'S WORK:
├─ [ ] Task 1 ..................... X óra
├─ [ ] Task 2 ..................... Y óra
└─ [ ] Task 3 ..................... Z óra

TOTAL HOURS: [X hours]
CUMULATIVE: [X hours] / 160 total

ISSUES ENCOUNTERED:
├─ Issue 1: [Description]
├─ Issue 2: [Description]
└─ Issue 3: [Description]

SOLUTIONS APPLIED:
├─ Solution 1: [Description]
├─ Solution 2: [Description]
└─ Solution 3: [Description]

TOMORROW'S GOALS:
├─ [ ] Goal 1
├─ [ ] Goal 2
└─ [ ] Goal 3

BLOCKERS:
├─ Blocker 1: [Description]
└─ Blocker 2: [Description]

NOTES:
[Any additional notes or observations]

═══════════════════════════════════════════════════════
```

---

## 🎯 SUCCESS METRICS

```
Project Success Defined By:

Functionality:
├─ [ ] All 8 features implemented
├─ [ ] Scheduling works daily
├─ [ ] Physics realistic
├─ [ ] Multiplay synced
└─ [ ] 100% feature complete

Performance:
├─ [ ] 60 FPS minimum (solo)
├─ [ ] 60 FPS minimum (4-player)
├─ [ ] Network latency < 100ms
├─ [ ] Memory stable (no leaks)
└─ [ ] GPU/CPU balanced

Quality:
├─ [ ] Zero critical bugs
├─ [ ] Zero known warnings
├─ [ ] Code well-documented
├─ [ ] Polished feel
└─ [ ] Production-ready

Testing:
├─ [ ] Solo play: 100% pass
├─ [ ] Multiplayer: 100% pass
├─ [ ] Stress test: 100% pass
├─ [ ] Edge cases: 100% handled
└─ [ ] Security: Validated
```

---

## 📞 SUPPORT & ESCALATION

```
Questions/Issues during development:

Level 1: Self-Help
├─ [ ] Check documentation
├─ [ ] Review Node Graph examples
├─ [ ] Check error messages
└─ [ ] Try common fixes

Level 2: Team Help
├─ [ ] Ask team member
├─ [ ] Pair program if stuck
├─ [ ] Code review help
└─ [ ] Design feedback

Level 3: Escalation
├─ [ ] Report blocker
├─ [ ] Document issue
├─ [ ] Adjust timeline if needed
└─ [ ] Request help from lead

Response Time SLA:
├─ Critical blockers: 1 hour
├─ Major issues: 4 hours
├─ Minor questions: 24 hours
└─ Nice-to-haves: Best effort
```

---

## ✅ SIGN-OFF CHECKLIST

**Before marking project COMPLETE:**

```
FINAL VERIFICATION:

Code Quality:
├─ [ ] All functions documented
├─ [ ] Comments on complex logic
├─ [ ] No unused code
├─ [ ] Consistent naming
└─ [ ] Follows project standards

Testing:
├─ [ ] All tests passed
├─ [ ] Stress tested
├─ [ ] Edge cases handled
├─ [ ] Multiplayer verified
└─ [ ] Zero crash reports

Documentation:
├─ [ ] README complete
├─ [ ] Inline comments added
├─ [ ] Function docs written
├─ [ ] Troubleshooting guide done
└─ [ ] Architecture diagram updated

Performance:
├─ [ ] Memory profiling done
├─ [ ] Network bandwidth ok
├─ [ ] GPU utilization good
├─ [ ] CPU efficient
└─ [ ] 60 FPS maintained

Release:
├─ [ ] Build tested
├─ [ ] No warnings
├─ [ ] Version bumped
├─ [ ] Changelog updated
└─ [ ] Ready to ship

SIGN-OFF:
Lead Developer: _________________ Date: _______
QA Lead: _____________________ Date: _______
Project Manager: ______________ Date: _______
```

---

**Verzió**: 1.0  
**DevKit**: 5.5.4  
**Dátum**: 2026-06-22  
**Status**: 🟡 Tracking & Schedule Ready

---

**Jó szerencsét a fejlesztéshez!** 🎮⚡
