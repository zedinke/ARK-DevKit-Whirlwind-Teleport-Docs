# Teleport Rendszer - UI Widget (UMG) Implementáció
## Teleport Point Selection Interface

---

## 📌 UI Komponensek Áttekintése

Az teljes teleport UI rendszer UMG (Unreal Motion Graphics) widget-ekből áll:

1. **WBP_TeleportMainMenu** - Fő menü widget
2. **WBP_TeleportPointButton** - Pont kiválasztás gomb
3. **WBP_TeleportMap** - Térkép megjelenítés
4. **WBP_ConfirmationDialog** - Megerősítés dialógus
5. **WBP_LoadingScreen** - Teleportálás közben
6. **WBP_ErrorMessage** - Hiba megjelenítés

---

## 🎮 WBP_TeleportMainMenu - Fő Widget

### **Hierarchia**

```
Canvas Panel (Root)
├─ Background Image
│  └─ Brush: Dark transparent overlay (0, 0, 0, 0.7)
│
├─ Content Panel (Horizontal Box)
│  ├─ Title Text
│  │  └─ "TELEPORT PONTOK"
│  │     ├─ Font: Title large
│  │     ├─ Color: Cyan
│  │     └─ Shadow: TRUE
│  │
│  ├─ Description Text
│  │  └─ "Válassz egy teleport pontot..."
│  │     ├─ Font: Normal
│  │     ├─ Color: White (transparent)
│  │     └─ Margin: 10px bottom
│  │
│  ├─ Scroll Box (Teleport Points List)
│  │  ├─ Max height: 400px
│  │  ├─ Scroll Bar Style: Custom
│  │  │
│  │  └─ Vertical Box (Dynamic)
│  │     └─ [Filled by Blueprint]
│  │        ├─ Point Button 1
│  │        ├─ Point Button 2
│  │        ├─ Point Button 3
│  │        └─ ...Point Button N
│  │
│  ├─ Divider Line
│  │  └─ Brush: Cyan horizontal line
│  │
│  ├─ Button Panel (Horizontal Box)
│  │  ├─ Button: "HOME" (Multicast Gomb)
│  │  │  ├─ Size: 150 x 50px
│  │  │  ├─ Color: Gold
│  │  │  └─ On Click: TeleportHome
│  │  │
│  │  ├─ Button: "MEGERŐSÍTÉS" (Confirm)
│  │  │  ├─ Size: 150 x 50px
│  │  │  ├─ Color: Cyan (gomb)
│  │  │  └─ On Click: ConfirmTeleport
│  │  │
│  │  └─ Button: "MÉGSEM" (Cancel)
│  │     ├─ Size: 150 x 50px
│  │     ├─ Color: Red/Gray
│  │     └─ On Click: CloseMenu
│  │
│  ├─ Map Preview (Optional)
│  │  ├─ Image: Mini map
│  │  ├─ Overlay: Teleport point markers
│  │  └─ Interactive: Click pont-ra teleportálni
│  │
│  └─ Cooldown Display
│     ├─ Text: "Cooldown: X.Xs"
│     ├─ Progress Bar: Cooldown vizualizáció
│     └─ Disabled when available
```

### **Blueprint Variables**

```
Variables:

1. SelectedTeleportPoint
   ├─ Type: BP_TeleportPoint
   ├─ Tooltip: "Kiválasztott teleport pont"
   └─ Default: NULL

2. TeleportManager
   ├─ Type: BP_TeleportManager
   ├─ Tooltip: "Referencia a manager-re"
   └─ Cache on Begin Play

3. TeleportPointButtons
   ├─ Type: Array (Button references)
   ├─ Dynamic: Created at runtime
   └─ One per teleport point

4. CurrentCooldownTime
   ├─ Type: Float
   ├─ Replicated: YES
   └─ Track cooldown remaining

5. bIsMenuOpen
   ├─ Type: Boolean
   ├─ Default: FALSE
   └─ Control visibility

6. PointButtonClass
   ├─ Type: Class (WBP_TeleportPointButton)
   ├─ Blueprint reference
   └─ Spawn buttons from this
```

### **Event Graph - WBP_TeleportMainMenu**

#### **Event: Construct (UI Initialization)**

```
[Construct Event]
    ↓
[Get Player Character]
    ├─ Cast to PlayerCharacter
    └─ Store reference
    ↓
[Find Teleport Manager]
    ├─ Get All Actors of Class
    │  └─ BP_TeleportManager
    │
    ├─ Select first one
    └─ Store in TeleportManager
    ↓
[Populate Teleport Points]
    ├─ Get TeleportManager.TeleportPoints
    ├─ For Each in Array
    │  ├─ Create Button Widget
    │  │  ├─ Widget class: WBP_TeleportPointButton
    │  │  ├─ Parent: ScrollBox
    │  │  │
    │  │  └─ Initialize Button
    │  │     ├─ Set Point Data
    │  │     ├─ Set Button Text
    │  │     ├─ Setup On Click
    │  │     └─ Add to array
    │  │
    │  └─ Continue to next
    │
    └─ [Complete]
```

#### **Event: Tick (Cooldown Update)**

```
[Tick Event]
    ├─ Every Frame
    │
    ├─ Get Current Time
    ├─ Calculate time since last teleport
    ├─ Remaining time = Cooldown - elapsed
    │
    ├─ If Remaining > 0
    │  ├─ Update Cooldown Text
    │  ├─ Update Progress Bar
    │  └─ Disable Teleport Buttons
    │
    └─ If Remaining <= 0
       ├─ Hide Cooldown Display
       └─ Enable Teleport Buttons
```

#### **Event: On Point Selected (Button Click)**

```
[On Point Button Clicked]
    ├─ Input: Clicked Button Index
    │
    ├─ Get Array Element
    │  └─ TeleportPointButtons[Index]
    │
    ├─ Get Point Data
    │  ├─ Location
    │  ├─ Rotation
    │  ├─ Name
    │  └─ Effect Type
    │
    ├─ Store in SelectedTeleportPoint
    │
    ├─ Update UI
    │  ├─ Highlight selected button
    │  ├─ Show point details
    │  ├─ Update description text
    │  └─ Enable confirm button
    │
    └─ [Complete]
```

#### **Event: On Confirm Teleport**

```
[Confirm Button Clicked]
    ↓
[Validate Selection]
    ├─ Is point selected?
    │  ├─ YES → Continue
    │  └─ NO → Show error + return
    │
    ├─ Is on cooldown?
    │  ├─ YES → Show error + return
    │  └─ NO → Continue
    │
    └─ Is player alive?
       ├─ YES → Continue
       └─ NO → Show error + return
    ↓
[Prepare Teleport]
    ├─ Get selected point location
    ├─ Get selected point rotation
    ├─ Get player character
    ├─ Check if mounted dino
    │
    └─ Collect parameters
    ↓
[Show Loading Screen]
    ├─ Fade in loading widget
    │  └─ "Teleportálás..."
    │
    ├─ Disable input
    └─ Play loading sound
    ↓
[Call Server RPC]
    └─ ServerExecuteTeleport(
       ├─ TeleportManager reference
       ├─ Selected Location
       ├─ Selected Rotation
       ├─ Has Dino?
       └─ Player Controller
       )
    ↓
[Wait for Server Response]
    ├─ Timeout: 5 seconds
    │
    └─ On Response:
       ├─ Close Loading Screen
       ├─ Close Teleport Menu
       ├─ Enable input
       └─ Show success message
```

#### **Event: On Home Button Clicked**

```
[Home Button Clicked]
    ↓
[Get Teleport Manager]
    ├─ Get saved return point
    │  └─ PlayerReturnPoint
    │
    └─ If valid:
       ├─ Use as target location
       │
       ├─ Call Confirm Teleport logic
       │  └─ Reuse ServerExecuteTeleport
       │
       └─ Show "Teleporting Home..."
```

#### **Event: On Cancel (Close Menu)**

```
[Cancel Button Clicked]
    ↓
[Remove from Parent]
    ├─ Close widget
    └─ Restore previous UI
    ↓
[Reset Selection]
    ├─ SelectedTeleportPoint = NULL
    ├─ Unhighlight buttons
    └─ Clear description
    ↓
[Enable Player Input]
    ├─ Set input mode
    └─ Show mouse
```

---

## 🔘 WBP_TeleportPointButton - Pont Gomb Widget

### **Hierarchia**

```
Button (Root)
├─ SizeBox
│  └─ Width: 100%, Height: 60px
│
├─ HorizontalBox (Content)
│  ├─ Image (Point Icon)
│  │  ├─ Size: 40x40px
│  │  ├─ Brush: Colored dot (cyan by default)
│  │  └─ Margin: 10px
│  │
│  ├─ VerticalBox (Text)
│  │  ├─ Text: Point Name
│  │  │  ├─ Font: Medium
│  │  │  ├─ Color: Cyan
│  │  │  └─ Bold: TRUE
│  │  │
│  │  ├─ Text: Point Description
│  │  │  ├─ Font: Small
│  │  │  ├─ Color: White (50% transparent)
│  │  │  └─ Max lines: 1
│  │  │
│  │  └─ Text: Distance
│  │     ├─ Font: Small
│  │     ├─ Color: Gold
│  │     └─ Format: "X.X km away"
│  │
│  └─ Image (Right Arrow)
│     ├─ Size: 20x20px
│     ├─ Color: Cyan
│     └─ Visibility: Unhovered
│
└─ Border (Highlight)
   ├─ Brush: Cyan border
   ├─ Color: Transparent (changes on hover/select)
   └─ Thickness: 2px
```

### **Blueprint Variables**

```
Variables:

1. AssignedTeleportPoint
   ├─ Type: BP_TeleportPoint
   └─ Store which point this button is for

2. PointLocation
   ├─ Type: Vector
   └─ Cache location for distance calc

3. bIsSelected
   ├─ Type: Boolean
   ├─ Default: FALSE
   └─ Track selection state

4. ParentMenu
   ├─ Type: WBP_TeleportMainMenu
   └─ Reference to parent widget

5. DistanceToPoint
   ├─ Type: Float
   └─ Calculated at runtime
```

### **Event Graph - WBP_TeleportPointButton**

```
[Construct]
    ↓
[Initialize with Point Data]
    ├─ Input: TeleportPoint (BP_TeleportPoint)
    │
    ├─ Set Point Name (Text)
    │  └─ AssignedTeleportPoint.PointName
    │
    ├─ Set Point Icon Color
    │  └─ AssignedTeleportPoint.PointColor
    │
    ├─ Calculate Distance
    │  ├─ Get Player Location
    │  ├─ Get Point Location
    │  ├─ Calculate distance
    │  └─ Update Text: "X.X km away"
    │
    └─ Cache point location
       └─ PointLocation = AssignedTeleportPoint.Location

[On Hovered]
    ├─ Change button color (lighter)
    ├─ Show arrow
    └─ Play hover sound

[On Unhovered]
    ├─ Restore button color
    └─ Hide arrow

[On Clicked]
    ├─ Set bIsSelected = TRUE
    │
    ├─ Change border to Cyan
    └─ Notify parent
       └─ Call OnPointSelected in parent menu
```

---

## 🗺️ WBP_TeleportMap - Mini Mapa Widget

### **Hierarchia (Optional Advanced)**

```
Canvas Panel (Root)
├─ Map Image
│  └─ Scaled map of zone
│
├─ For Each Teleport Point
│  └─ Icon Button
│     ├─ Position: Relative to map coordinates
│     ├─ Color: Cyan
│     ├─ Size: 30x30px
│     └─ On Click: Select point
│
├─ Player Position Indicator
│  ├─ Icon: Blinking dot
│  ├─ Color: Green
│  └─ Updates position (real-time)
│
└─ Distance Scale Text
   └─ "1 Grid = X km"
```

### **Features**

```
Functionality:
├─ Click on point → Select in list
├─ Hover point → Show name tooltip
├─ Zoom in/out (Mouse wheel)
├─ Drag map (Right mouse button)
├─ Auto-center on player
│  └─ Button: "Center on Me"
│
└─ Color coding:
   ├─ Cyan: Available points
   ├─ Gold: Home point
   ├─ Red: Restricted zone
   └─ Green: Player location
```

---

## ✅ WBP_ConfirmationDialog - Megerősítés

### **Content**

```
Dialog Box:
├─ Title: "Megerősítés"
├─ Message: "Teleportálni szeretnél ide: [Point Name]?"
│
├─ Character Fate:
│  └─ "Dinó települ veled: [YES/NO]"
│
├─ Buttons:
│  ├─ "IGEN - Teleportálj!"
│  ├─ "NEM - Mégsem"
│  └─ "OTTHON" (Home)
│
└─ Effects Preview:
   └─ Show VFX preview animation
```

---

## ⏳ WBP_LoadingScreen - Teleportálás Közben

### **Content**

```
Full Screen Overlay:
├─ Background: Dark with blur
├─ Center: Loading spinner
├─ Text: "Teleportálás folyamatban..."
│  └─ Animated dots: "... .....  ...."
│
├─ Progress Bar:
│  ├─ 0.0 - 0.4s: Departure effect
│  ├─ 0.4 - 0.6s: Transition
│  └─ 0.6 - 1.0s: Arrival effect
│
├─ Tip Text (Random):
│  ├─ "Dinók könnyebben teleportálhatók!"
│  ├─ "Teleportálj otthonba a HOME gombbal"
│  └─ "Energet fogyaszt a teleportálás"
│
└─ Auto-close after 1.0s
```

---

## ❌ WBP_ErrorMessage - Hiba Widget

### **Error Types**

```
ErrorDialog Box:
├─ Icon: Red X
├─ Title: "HIBA"
├─ Message: [Error specific]
│
├─ Error Messages:
│  ├─ InvalidLocation
│  │  └─ "A célhelyzet érvénytelen!"
│  │
│  ├─ OnCooldown
│  │  └─ "Még X.X másodperc a következő teleportálásig!"
│  │
│  ├─ NotAlive
│  │  └─ "Nem teleportálhatsz halálra!"
│  │
│  ├─ InCombat
│  │  └─ "Küzdelemben nem teleportálhatsz!"
│  │
│  ├─ LocationRestricted
│  │  └─ "Ez a hely tiltott zóna!"
│  │
│  └─ Unknown
│     └─ "Ismeretlen hiba történt!"
│
├─ Button: "OK"
│  └─ Closes dialog
│
└─ Auto-close: 5 seconds (unless critical)
```

---

## 🔌 Blueprint Integration

### **Player Character Blueprint - Input Binding**

```
[Event Setup Input Component]
    ↓
[Bind Key Input]
    ├─ Key: TAB (Teleport menu toggle)
    │  └─ Call "Toggle Teleport Menu"
    │
    ├─ Key: H (Home - Quick)
    │  └─ Call "Quick Teleport Home"
    │
    └─ Key: ESC (Close menu)
       └─ Call "Close All Teleport UI"

[Toggle Teleport Menu Event]
    ├─ Is menu open?
    │  ├─ YES → Close menu
    │  └─ NO → Open menu
    │
    ├─ Create WBP_TeleportMainMenu
    ├─ Add to viewport
    ├─ Set input mode: UI Only
    │
    └─ Store menu reference
       └─ bTeleportMenuOpen = TRUE/FALSE

[Quick Teleport Home]
    ├─ If on cooldown → Show error
    ├─ Else → Call teleport to home
```

### **HUD Integration**

```
Main HUD:
├─ Teleport Button (Screen corner)
│  ├─ Text: "TAB - Teleport"
│  ├─ Icon: Portal symbol
│  ├─ Highlight: When available
│  └─ Grayed out: On cooldown
│
├─ Cooldown Display
│  ├─ Show remaining time
│  └─ Progress bar visualization
│
└─ Recent Teleport Notification
   ├─ Show when player teleports
   ├─ "Teleported to: [Location]"
   └─ Fade out after 3 seconds
```

---

## 🎨 Style & Theme

### **Color Scheme**

```
Primary: Cyan (0, 255, 255)
├─ Menu background
├─ Buttons
└─ Text highlights

Secondary: Gold (255, 240, 100)
├─ Home button
├─ Special text
└─ Emphasis

Accent: White (255, 255, 255)
├─ Main text
├─ Descriptions
└─ Info

Background: Dark (20, 20, 30, 0.8)
├─ Menu overlay
├─ Dialog background
└─ Transparency

Hover: Light Cyan (100, 255, 255)
├─ Button hover
└─ Interactive element feedback
```

### **Font Setup**

```
Title Font:
├─ Family: Bold (any sci-fi style)
├─ Size: 32px
├─ Color: Cyan

Header Font:
├─ Family: Semi-bold
├─ Size: 24px
├─ Color: Cyan

Body Font:
├─ Family: Regular
├─ Size: 16px
├─ Color: White

Small Font:
├─ Family: Regular
├─ Size: 12px
├─ Color: White (50% transparent)
```

---

## 🔊 UI Sound Effects

### **Audio Cues Needed**

```
1. UI_MenuOpen.wav
   └─ Soft sci-fi sound when menu opens

2. UI_ButtonHover.wav
   └─ Subtle click/beep on button hover

3. UI_ButtonClick.wav
   └─ Confirmation sound on click

4. UI_Teleport_Initiated.wav
   └─ When teleport starts

5. UI_Error.wav
   └─ Error notification sound

6. UI_Cooldown_Ready.wav
   └─ Notification when cooldown expires
```

---

## 📋 Implementation Checklist

### **Phase 1: Core Widgets**
- [ ] WBP_TeleportMainMenu (basic structure)
- [ ] WBP_TeleportPointButton (button logic)
- [ ] Point list population
- [ ] Selection handling

### **Phase 2: Advanced Features**
- [ ] WBP_TeleportMap (mini map)
- [ ] Distance calculation
- [ ] Real-time distance updates
- [ ] Zoom/pan controls

### **Phase 3: Dialogs**
- [ ] WBP_ConfirmationDialog
- [ ] WBP_LoadingScreen
- [ ] WBP_ErrorMessage
- [ ] Error message mapping

### **Phase 4: Polish**
- [ ] Animations (fade in/out)
- [ ] Sound effects
- [ ] Visual feedback (hover, click)
- [ ] Color theme consistency

### **Phase 5: Integration**
- [ ] Player Character input binding
- [ ] HUD integration
- [ ] Cooldown display
- [ ] Notification system

---

**Verzió**: 1.0  
**DevKit**: 5.5.4  
**Dátum**: 2026-06-22  
**Status**: 🟡 Tervezett - UMG szerkesztés szükséges
