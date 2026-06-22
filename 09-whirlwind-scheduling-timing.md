# ⚡ Whirlwind Event System - Scheduling & Time Integration
## Day/Night Cycle Manager Integráció

---

## 📌 Time System Alapok

Az ARK DevKit-ben az idő kezelése a **Day Cycle Manager** segítségével történik, amely 0-1440 perc közötti értékeket kezel (24 óra = 1440 perc).

```
TIME CONVERSION:

0 perc = 00:00 (éjfél)
60 perc = 01:00
360 perc = 06:00 (hajnal)
720 perc = 12:00 (dél)
1080 perc = 18:00 (este 6-kor)
1440 perc = 24:00 (majd újra 0)

DEFAULT SETTINGS ARK-BAN:
├─ Daylength: ~30 valós percre (game time)
├─ Day speed: ~1.5x (1 valós perc = 1.5 game perc)
├─ Night speed: ~1.0x (1 valós perc = 1.0 game perc)
└─ Cycle: Nap - Sürgős (1 óra előtti) - Éjszaka - Hajnal
```

---

## 🕐 Day Cycle Manager Integration

### **Blueprint: BP_DayCycleManager (Exists in ARK)**

A DevKit már tartalmaz egy `BP_DayCycleManager` vagy hasonló blueprint-et. Ez a mi `DayCycleManagerRef`-ünknek a célja.

```
Lekérdezése:

[Get Day Cycle Manager]
    ├─ Get All Actors of Class
    │  └─ Class: BP_DayCycleManager
    │
    ├─ Get (first item)
    └─ This is our reference

Usage:

[Get Current Time]
    ├─ Call function on manager
    ├─ Return: Float (0-1440 minutes)
    └─ Use for trigger checking
```

---

## ⏰ EVENT TRIGGER LOGIC

### **Time-Based Trigger Calculation**

**Jelenet: Whirlwind este 6-kor (18:00 = 1080 perc)**

```
FORMULA:

CurrentTime = DayCycleManager.GetCurrentTime()
TriggerTime = 1080 (configured)

If CurrentTime >= 1079 AND CurrentTime <= 1081:
    └─ TRIGGER EVENT!

TimeUntilEvent = TriggerTime - CurrentTime

If TimeUntilEvent < 0:
    └─ TimeUntilEvent += 1440 (next day)
```

### **Node Graph: Calculate Time Until Event**

```
[Calculate Time Until Event]
    ↓
[Get Current Game Time]
    ├─ Call DayCycleManagerRef.GetCurrentTime()
    └─ Returns: Float (0-1440 minutes)
    ↓
[Calculate Difference]
    ├─ Difference = EventTriggerTime - CurrentTime
    │
    ├─ If Difference < 0:
    │  │ (Event already passed today)
    │  └─ Difference += 1440 (next day)
    │
    └─ TimeUntilEvent = Difference
    ↓
[Clamp to Realistic Range]
    ├─ Clamp Min: 0.0
    ├─ Clamp Max: 1440.0
    └─ Sanity check
    ↓
[Result: TimeUntilEvent]
    └─ Minutes until next trigger
```

### **Example Timeline**

```
SCENARIO 1: It's 17:00 (1020 min), event at 18:00 (1080 min)
├─ Difference = 1080 - 1020 = 60 minutes
└─ Event in 60 minutes ✓

SCENARIO 2: It's 19:00 (1140 min), event at 18:00 (1080 min)
├─ Difference = 1080 - 1140 = -60
├─ Since negative: -60 + 1440 = 1380 minutes
└─ Event tomorrow in 1380 minutes ✓

SCENARIO 3: It's 17:59 (1079 min), event at 18:00 (1080 min)
├─ Difference = 1080 - 1079 = 1 minute
├─ Within trigger window! (< 1.0 minute)
└─ TRIGGER EVENT! ✓
```

---

## 🔄 EVENT CHECK LOOP

### **Periodic Event Checking**

A `CheckEventTrigger` függvényt **periodikusan** hívjuk, nem continuousban (hatékonyabb).

```
SETUP:

[Initialize Manager]
    ├─ Get World Timer Manager
    ├─ Set Timer by Event
    │  ├─ Function: CheckEventTrigger
    │  ├─ Rate: 10.0 seconds
    │  ├─ Loop: TRUE
    │  └─ Store Timer Handle
    │
    └─ Check every 10 seconds

BEHAVIOR:

Every 10 real-world seconds:
├─ Call CheckEventTrigger
├─ Check if time to trigger
├─ If YES → Spawn whirlwind
├─ If NO → Continue waiting
└─ Repeat
```

### **Trigger Window**

```
Trigger Window: 1 minute (60 seconds)

If EventTriggerTime = 1080 (18:00):
├─ Trigger zone: 1079-1081 minutes
└─ 1 minute before & after exact time

This allows for:
├─ Game server lag compensation
├─ Multiple check cycles to catch trigger
└─ Smooth event spawning (not frame-dependent)

IMPORTANT:
├─ Don't use == exactly!
│  └─ (CurrentTime == 1080) might never be true
│
└─ Use range check:
   └─ (CurrentTime >= 1079 && CurrentTime <= 1081)
```

---

## 🎯 TRIGGER TIME EDITOR SETUP

### **Blueprint Variable: EventTriggerTime**

**Editable in Details Panel:**

```
Details Panel Setup:

Variable: EventTriggerTime
├─ Type: Float
├─ Default: 1080.0
├─ Category: "Event Schedule"
├─ Editable: TRUE ✓
│
├─ Slider Range:
│  ├─ Min: 0.0
│  ├─ Max: 1440.0
│  └─ Snap to 1.0 minute increments
│
├─ Display Name: "Event Trigger Time (minutes)"
├─ Tooltip: "Time in minutes (0-1440). 1080=18:00"
│
└─ Advanced Details:
   ├─ Show 3D Widget: FALSE
   └─ Edit Condition: Always
```

### **Preset Times Dropdown (Optional)**

```
Create custom function to help set common times:

[Set Event to Preset Time]
    ├─ Enum: EPresetEventTime
    │  ├─ Dawn = 360 (06:00)
    │  ├─ Noon = 720 (12:00)
    │  ├─ Afternoon = 900 (15:00)
    │  ├─ Dusk = 1080 (18:00)
    │  ├─ Night = 1200 (20:00)
    │  └─ Midnight = 1440 (00:00)
    │
    └─ Switch statement to set time

Usage in editor:
├─ Select Manager actor
├─ Choose preset from dropdown
└─ Automatically set correct minute value
```

---

## 🔐 COOLDOWN PREVENTION

### **Event Spam Prevention**

Mivel a trigger window 2 percen át tart, valamint az check loop 10 másodpercenként fut, az esemény többször is indulhat. Ezt megakadályozzuk egy **cooldown** segítségével.

```
COOLDOWN MECHANISM:

Variable: LastEventTriggerTime
├─ Type: Float
├─ Default: 0.0
├─ Stores last spawn time
└─ In game minutes

Check Logic:

[Check Cooldown]
    ├─ CurrentTime = DayCycleManager.GetCurrentTime()
    ├─ LastSpawn = LastEventTriggerTime
    │
    ├─ Elapsed = CurrentTime - LastSpawn
    │
    ├─ If Elapsed > 10 minutes:
    │  ├─ Cooldown expired!
    │  └─ Event can spawn again
    │
    └─ Otherwise:
       └─ Event already triggered, skip
```

### **Timeline Example**

```
TIME: 17:50 (1070 min)
├─ Check event
├─ No trigger yet (10 min away)
└─ LastEventTriggerTime = 0

TIME: 17:59:50 (1079.8 min)
├─ CheckEventTrigger called
├─ Time check: 1079.8 >= 1079 ✓
├─ Cooldown check: (1079.8 - 0) > 10 ✓
├─ SPAWN EVENT!
├─ LastEventTriggerTime = 1079.8
└─ Whirlwind spawned

TIME: 18:00 (1080 min)
├─ CheckEventTrigger called again (10 sec later)
├─ Time check: 1080 >= 1079 ✓
├─ Cooldown check: (1080 - 1079.8) = 0.2 min
├─ 0.2 < 10 min → Cooldown active!
└─ SKIP (event already spawned)

TIME: 18:10 (1090 min)
├─ CheckEventTrigger called
├─ Time check: 1090 NOT in range
└─ Event window passed, skip

TIME: 18:01 to 05:59 (Next day)
├─ Check every 10 seconds
├─ No triggers (wrong time)
└─ Waiting for next 18:00

TIME: 17:59 next day (1079.8 min next cycle)
├─ CheckEventTrigger called
├─ Time check: 1079.8 >= 1079 ✓
├─ Cooldown check: (1079.8 - 1079.8 from prev day) > 10 ✓
├─ SPAWN EVENT AGAIN!
└─ Cycle repeats
```

---

## ⚙️ CONFIGURABLE SPAWN TIMES

### **Multiple Events Same Day**

Ha szeretnél több whirlwind eseményt ugyanazon a nap különböző időkben:

```
OPTION 1: Multiple Managers
├─ Create BP_WhirlwindEventManager_Morning (06:00)
├─ Create BP_WhirlwindEventManager_Afternoon (15:00)
├─ Create BP_WhirlwindEventManager_Evening (18:00)
└─ Place each separately in level

Each manager:
├─ Independent trigger time
├─ Separate cooldown tracking
├─ All spawn whirlwinds normally
└─ Can have multiple simultaneous

OPTION 2: Single Manager with Array
├─ EventTriggerTimes: Array of Float
│  ├─ [0] = 360 (06:00)
│  ├─ [1] = 900 (15:00)
│  └─ [2] = 1080 (18:00)
│
├─ For each time in array:
│  └─ Check if trigger
│
└─ More complex, but cleaner
```

### **Dynamic Configuration**

```
Console Command: Set Event Time

[Command: /whirlwind time 1200]
    ├─ Parse input: 1200
    ├─ Convert to valid range (0-1440)
    ├─ Set EventTriggerTime = 1200
    ├─ Print: "Event time set to 20:00 (1200 min)"
    └─ Next check cycle will use new time

Blueprint Implementation:

[Set Event Time]
    ├─ Input: NewTime (Float)
    │
    ├─ Validation:
    │  ├─ If NewTime < 0 → Set to 0
    │  ├─ If NewTime > 1440 → Set to 1440
    │  └─ Clamp valid range
    │
    ├─ Set EventTriggerTime = Clamped value
    ├─ Reset LastEventTriggerTime = 0
    │  └─ Allow immediate trigger if valid
    │
    └─ Log: "Event trigger time changed to X"
```

---

## 🌍 REAL-WORLD TIME SCALING

### **Day Length Configuration**

Az ARK-ban az egész nap szimulálódik úgy, hogy nem kell 24 valós óra. Ezt be kell venni az évényadási logikánkba.

```
EXAMPLE SCENARIO:

Game Settings:
├─ Day Length: 30 real minutes = 1 game day
├─ Day Speed: 1.5x
├─ Night Speed: 1.0x

Calculation:
├─ 1 real minute = 1.5 game minutes (daytime)
├─ 1 real minute = 1.0 game minutes (nighttime)

EVENT TIMING:

Target: 18:00 game time
├─ 1080 game minutes from midnight
├─ At 1.5x speed: 1080 / 1.5 = 720 real minutes
└─ = 12 real-world minutes after day start

On 30 min day cycle:
├─ Real minutes to event from 00:00
│  └─ = 720 / (60 min per hour) = 12 real minutes
│
└─ Event at: 00:00 game + 12 real min = XX time

If day starts at 06:00 real time:
├─ 06:00 + 12 min = 06:12 real time
└─ Whirlwind appears at 06:12 real!
```

### **Sync with Day Cycle Manager**

```
The DayCycleManager handles all this timing!

Our job:
├─ Get Current Time (it handles scaling)
├─ Compare with EventTriggerTime (both in game minutes)
└─ Don't worry about real-world scaling
    └─ Manager handles it for us

So our code stays simple:
├─ Get CurrentTime (0-1440 game min)
├─ Compare with EventTriggerTime (0-1440 game min)
├─ Spawn if equal (within window)
└─ Done!
```

---

## 🧪 TESTING TIME INTEGRATION

### **Test Case 1: Manual Time Setting**

```
Blueprint Test Setup:

[Event Begin Play]
    ├─ Spawn Manager
    ├─ Set EventTriggerTime = 1 minute from now
    │  └─ This allows quick testing!
    │
    ├─ Set DayLength to very short (5 real minutes)
    │  └─ For testing speedup
    │
    └─ Start monitoring
    ↓
[Every 1 second]
    ├─ Print current time
    ├─ Print time until event
    └─ Watch countdown
    ↓
[When trigger happens]
    ├─ Whirlwind should spawn
    ├─ ActiveWhirlwinds count increases
    └─ Verify success!
```

### **Test Case 2: Day Cycle Verification**

```
Setup:
├─ Set game to 17:55 (1075 min) manually
├─ Set EventTriggerTime = 18:00 (1080 min)
├─ Let game run normally
└─ Watch for spawn

Expected:
├─ At 17:59 (1079 min)
│  └─ Event check window active
│
├─ Whirlwind spawns
├─ Persists for 5 minutes
└─ Despawns at ~18:04

Verification Checklist:
├─ [ ] Event spawned at right time
├─ [ ] Duration was 5 minutes
├─ [ ] Cooldown prevented re-spawn
├─ [ ] Next event next day at 18:00
└─ [ ] Timer logic correct
```

### **Test Case 3: Cooldown Testing**

```
Setup:
├─ Set time to 1079.8 (right in trigger window)
├─ Manually call CheckEventTrigger twice
│  └─ Rapid successive calls
│
└─ Test cooldown prevention

Expected:
├─ First call: Spawn event
├─ Second call (10 sec later): No spawn
│  └─ Cooldown active!
│
├─ Log:
│  ├─ "Event spawned at 1079.8"
│  └─ "Cooldown active, skipping spawn"
│
└─ Verify: Only 1 whirlwind, not 2
```

---

## 📊 DAY/NIGHT AWARE SPAWNING

### **Optional: Time-Specific Behavior**

```
ADVANCED: Different whirlwinds based on time

[Determine Whirlwind Type by Time]
    ├─ Morning (06:00-12:00):
    │  └─ Frost Vortex (cold morning)
    │
    ├─ Afternoon (12:00-18:00):
    │  └─ Aqua Vortex (moderate)
    │
    └─ Evening/Night (18:00-06:00):
       └─ Inferno Tornado (dangerous night)

Implementation:

[Select Whirlwind Type]
    ├─ Get Current Time
    ├─ If Time >= 360 AND < 720:
    │  └─ Type = FrostVortex
    │
    ├─ Else If Time >= 720 AND < 1080:
    │  └─ Type = AquaVortex
    │
    └─ Else:
       └─ Type = InfernoTornado
```

---

## 🎮 CONSOLE DEBUG COMMANDS

### **Time Management Commands**

```
Commands for testing:

/whirlwind time [minutes]
├─ Set trigger time directly
└─ Example: /whirlwind time 1080 (18:00)

/whirlwind currenttime
├─ Print current game time
└─ Debug output

/whirlwind spawn [type]
├─ Force spawn whirlwind immediately
└─ Example: /whirlwind spawn inferno

/whirlwind skip
├─ Skip forward to next trigger time
├─ Useful for testing without waiting
└─ Internally increases game time

/whirlwind reset
├─ Reset last trigger time
├─ Allow immediate trigger
└─ Clear cooldown

/whirlwind list
├─ Print all active whirlwinds
├─ Locations, durations
└─ Debug info
```

### **Implementation Example**

```
[Parse Console Command]
    ├─ Command: "/whirlwind time 1200"
    │
    ├─ Extract parameters:
    │  ├─ Command = "whirlwind"
    │  ├─ SubCommand = "time"
    │  └─ Parameter = "1200"
    │
    ├─ Switch on SubCommand:
    │  ├─ "time":
    │  │  ├─ Convert to float
    │  │  ├─ Call SetEventTriggerTime(1200)
    │  │  └─ Print "Event time set to 20:00"
    │  │
    │  ├─ "currenttime":
    │  │  ├─ Get time from manager
    │  │  ├─ Format as HH:MM
    │  │  └─ Print to console
    │  │
    │  └─ ... other commands
    │
    └─ Success!
```

---

## 📝 TIMING TROUBLESHOOTING

### **Common Issues**

```
ISSUE 1: Event never triggers
├─ Cause: EventTriggerTime never equals CurrentTime
├─ Solution:
│  ├─ Use range check (±1 minute)
│  ├─ Verify DayCycleManager integrated
│  ├─ Check trigger window logic
│  └─ Use debug output
│
└─ Debug:
   ├─ Print current time every 10 sec
   ├─ Watch countdown
   └─ Verify trigger window

ISSUE 2: Event triggers multiple times
├─ Cause: Cooldown not working
├─ Solution:
│  ├─ Verify LastEventTriggerTime updates
│  ├─ Check cooldown duration (> 10 min)
│  ├─ Verify cooldown check logic
│  └─ Add logging to debug
│
└─ Debug:
   ├─ Print LastEventTriggerTime
   ├─ Print elapsed time
   └─ Verify > 10 min check

ISSUE 3: Event time wrong on live server
├─ Cause: Day length not same as test
├─ Solution:
│  ├─ Verify DayLength setting
│  ├─ Check day speed multiplier
│  ├─ Ensure same settings test & live
│  └─ Use game minutes (not real)
│
└─ Debug:
   ├─ Check game settings
   ├─ Print DayLength value
   └─ Verify consistency

ISSUE 4: Event doesn't respect day/night
├─ Cause: Not using DayCycleManager time
├─ Solution:
│  ├─ Ensure getting time from manager
│  ├─ Not using world time
│  ├─ Not using real-world time
│  └─ Always use manager GetTime()
│
└─ Debug:
   ├─ Verify manager reference
   ├─ Check GetTime() return
   └─ Compare with game time UI
```

---

**Verzió**: 1.0  
**DevKit**: 5.5.4  
**Dátum**: 2026-06-22  
**Status**: 🟡 Implementálható - Time integration guide
