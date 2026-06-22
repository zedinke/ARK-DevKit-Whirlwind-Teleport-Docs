# Teleport Effektek - VFX & SFX Szerkesztési Útmutató

## 📌 Effekt Rendszer Áttekintése

Három fő effekt típus:
1. **Departure Effect** - Induláskor (Cyan)
2. **Arrival Effect** - Érkezéskor (White/Gold)
3. **Trail Effect** - Útvonal (opcionális, Glowing)

---

## 🎨 VFX EFFEKTEK - PARTICLE SYSTEMS

### **EFFEKT 1: PT_Teleport_Departure (Indulás)**

#### **Létrehozás**
```
Content Browser → New → Particle System
└─ Name: PT_Teleport_Departure
└─ Double-click to open Cascade Editor
```

#### **Emitter Beállítások**

**Main Emitter:**

| Beállítás | Érték | Megjegyzés |
|-----------|-------|-----------|
| **Duration** | 0.8 | Effekt teljes időtartama |
| **Loop Count** | 1 | Egyszer lejátszódik |
| **Lifetime (min)** | 0.5 | Részecske élettartama |
| **Lifetime (max)** | 0.8 | Részecske élettartama |

**Spawn Rate:**
```
Rate Over Time: 200
└─ Min: 180
└─ Max: 220
```

**Initial Velocity:**
```
Direction: 0, 0, 1 (felfelé)
Speed:
├─ Min: 500 cm/s
├─ Max: 1500 cm/s
└─ Randomize: 50%
```

**Initial Size:**
```
Size:
├─ Min: (20, 20, 20)
├─ Max: (80, 80, 80)
└─ Growth over time: 150 cm
```

**Initial Color:**
```
Color: Cyan
├─ R: 0
├─ G: 1.0
├─ B: 1.0
├─ A: 1.0 (start)
└─ A: 0.0 (end, fade out)
```

**Color Over Life:**
```
Timeline:
├─ Start: Cyan opaque
├─ 50%: Cyan medium
├─ 100%: Cyan transparent
└─ Use Curve: TRUE
```

**Material:**
```
Material: M_TeleportGlow_Cyan
├─ Emissive: YES
├─ Additive blend: YES
└─ Particle size affects brightness
```

---

### **EFFEKT 2: PT_Teleport_Arrival (Érkezés)**

#### **Létrehozás**
```
Content Browser → New → Particle System
└─ Name: PT_Teleport_Arrival
└─ Double-click to open Cascade Editor
```

#### **Emitter Beállítások**

**Burst Emitter (White Flash):**

| Beállítás | Érték |
|-----------|-------|
| **Duration** | 0.3 |
| **Loop Count** | 1 |
| **Lifetime** | 0.3 |

**Spawn Rate:**
```
Burst Rate: 400 particles single burst
```

**Initial Velocity:**
```
Direction: Random 360°
Speed:
├─ Min: 300 cm/s
├─ Max: 800 cm/s
└─ Outward radial
```

**Initial Size:**
```
Size:
├─ Min: (50, 50, 50)
├─ Max: (150, 150, 150)
└─ Decrease: 100 cm over life
```

**Initial Color:**
```
Color: White/Yellow
├─ R: 1.0
├─ G: 0.95
├─ B: 0.3
├─ A: 1.0 (start)
└─ A: 0.0 (end)
```

**Material:**
```
Material: M_TeleportGlow_White
├─ Emissive: MAXIMUM
├─ Additive blend: YES
└─ Brightness: 2.0x
```

**Second Emitter (Ring Expansion):**

```
Shape: Ring/Disc expanding
├─ Start Size: (200, 200, 0)
├─ End Size: (800, 800, 0)
├─ Duration: 0.5s
├─ Color: Cyan → White fade
└─ Opacity: 1.0 → 0.0
```

---

### **EFFEKT 3: PT_Teleport_Trail (Útvonal - Opcionális)**

#### **Létrehozás**
```
Content Browser → New → Particle System
└─ Name: PT_Teleport_Trail
└─ Bezier path emitter
```

#### **Beállítások**

**Bezier Path:**
```
Start Point: Departure location
End Point: Arrival location
└─ Duration: 2.0 seconds

Path Shape: Curved bezier
├─ Control Point 1: Mid height +200cm
└─ Control Point 2: Mid height +200cm

Spawn Behavior:
├─ Spawn Rate: 100 particles/sec
├─ Lifetime: 2.0s
└─ Path Speed: 500 cm/s
```

**Particle Properties:**
```
Size:
├─ Start: (30, 30, 30)
├─ End: (10, 10, 10)
└─ Taper: Fade at ends

Color:
├─ Start: Glowing Blue
├─ Middle: Cyan
├─ End: Transparent cyan
└─ Emissive: YES

Material:
├─ Glow type: Additive
├─ Trail blur: Slight motion blur
└─ Brightness: 1.5x
```

---

## 🔊 SFX EFFEKTEK - AUDIO SETUP

### **SOUND 1: SFX_Teleport_Departure**

#### **Audio File Properties**
```
Duration: 0.8 seconds
Format: WAV or MP3
Sample Rate: 44100 Hz
Channels: Stereo

Characteristics:
├─ Whoosh/Sci-fi sound
├─ Frequency: 2000-8000 Hz (mid-high)
├─ Attack: Fast (0.05s)
├─ Decay: Medium (0.3s)
├─ Sustain: Short
└─ Release: Fast (0.2s)

Example sound design:
├─ Synth ascending pitch
├─ Wind/whoosh layer
└─ Crystalline chime underneath
```

#### **Sound Cue Setup (DevKit)**
```
1. Import audio file
   └─ /Content/Sounds/SFX_Teleport_Departure

2. Create Sound Cue
   └─ Name: SC_Teleport_Departure
   └─ Right-click → Duplicate

3. Sound Cue Graph:
   ├─ Output Node
   └─ Wave Player
      ├─ Sound Wave: SFX_Teleport_Departure
      ├─ Volume: 0.8
      └─ Pitch: 1.0

4. Properties:
   ├─ Volume: 0.8
   ├─ Pitch: 1.0
   ├─ Priority: Normal
   └─ 3D Spatialization: YES
      ├─ Radius: 1000 cm
      └─ Attenuation: Linear
```

---

### **SOUND 2: SFX_Teleport_Arrival**

#### **Audio File Properties**
```
Duration: 1.0 seconds
Format: WAV or MP3
Sample Rate: 44100 Hz
Channels: Stereo

Characteristics:
├─ Magic chime/shimmer
├─ Frequency: 4000-12000 Hz (high)
├─ Attack: Very fast (0.02s)
├─ Decay: Medium (0.4s)
├─ Sustain: Medium
└─ Release: Medium (0.3s)

Example sound design:
├─ High-pitched bell/chime
├─ Shimmer/sparkle layer
├─ Resonance tail
└─ Slight reverb
```

#### **Sound Cue Setup**
```
1. Import audio file
   └─ /Content/Sounds/SFX_Teleport_Arrival

2. Create Sound Cue
   └─ Name: SC_Teleport_Arrival

3. Sound Cue Graph:
   ├─ Output Node
   └─ Wave Player
      ├─ Sound Wave: SFX_Teleport_Arrival
      ├─ Volume: 0.9
      └─ Pitch: 0.95

4. Properties:
   ├─ Volume: 0.9
   ├─ Pitch: 0.95 (lower, warmer)
   ├─ Priority: High
   └─ 3D Spatialization: YES
      ├─ Radius: 1200 cm
      └─ Attenuation: Logarithmic
```

---

### **SOUND 3: SFX_Teleport_Travel (Opcionális)**

#### **Audio File Properties**
```
Duration: 2.0 seconds (loopable)
Format: WAV or MP3
Characteristics:
├─ Ethereal/space travel sound
├─ Frequency: 1000-4000 Hz (low-mid)
├─ Ambient, niet percussive
└─ Loop-friendly (no clicks)

Example sound design:
├─ Deep drone base
├─ Ambient whoosh cycling
└─ Subtle modulation/chorus
```

#### **Sound Cue Setup**
```
1. Import audio file
   └─ /Content/Sounds/SFX_Teleport_Travel

2. Create Sound Cue
   └─ Name: SC_Teleport_Travel

3. Sound Cue Graph:
   ├─ Output Node
   └─ Wave Player
      ├─ Sound Wave: SFX_Teleport_Travel
      ├─ Volume: 0.5
      ├─ Pitch: 1.0
      └─ Loop: TRUE

4. Properties:
   ├─ Volume: 0.5 (background)
   ├─ Attenuation: Very wide (2000 cm)
   └─ Pan Spread: 360° (surround)
```

---

## 💡 MATERIALS KONFIGURÁLÁSA

### **Material 1: M_TeleportGlow_Cyan**

#### **Material Setup**
```
1. Content Browser → New → Material
   └─ Name: M_TeleportGlow_Cyan

2. Material Details:
   ├─ Blend Mode: Additive
   ├─ Light Mode: Unlit
   ├─ Uses World Position Offset: FALSE
   └─ Material Domain: Surface

3. Material Graph:
   ├─ Base Color (Cyan)
   │  └─ Vector3(0, 1, 1)
   │
   ├─ Emissive Color
   │  └─ Multiply Base Color × 2.0
   │
   ├─ Opacity
   │  └─ Connect to Particle Alpha
   │
   ├─ Metallic: 0.5
   ├─ Specular: 0.5
   ├─ Roughness: 0.3
   └─ Normal: Flat (0, 0, 1)

4. Particle Specific:
   ├─ Use Particle Color: TRUE
   ├─ Use Particle Opacity: TRUE
   └─ Use Particle Size: FALSE
```

### **Material 2: M_TeleportGlow_White**

#### **Material Setup**
```
1. Content Browser → New → Material
   └─ Name: M_TeleportGlow_White

2. Material Details:
   ├─ Blend Mode: Additive
   ├─ Light Mode: Unlit
   └─ Emissive Light Radius: 10000

3. Material Graph:
   ├─ Base Color: White (1, 1, 1)
   │
   ├─ Emissive Color
   │  └─ Multiply Base Color × 3.0 (very bright)
   │
   ├─ Opacity: Particle Alpha + Fade
   │
   └─ Use with Peak Brightness: YES

4. Properties:
   ├─ Brightness Boost: 2.0x
   ├─ Glow Intensity: Maximum
   └─ Particle Size influences brightness
```

---

## 🎬 SCREENSHAKE & POST PROCESS

### **Camera Shake - Arrival Effect**

#### **Blueprint Setup**
```cpp
Details:
├─ Shake Scale: 2.0
├─ Location Amplitude
│  ├─ X: 5.0
│  ├─ Y: 5.0
│  └─ Z: 3.0
│
├─ Rotation Amplitude
│  ├─ Pitch: 0.5
│  ├─ Yaw: 1.5
│  └─ Roll: 0.3
│
├─ Frequency: 10.0 Hz
├─ Duration: 0.3 seconds
└─ Falloff Distance: 500 cm
```

#### **Implementation in Blueprint**
```
[Spawn Arrival Effect]
    ↓
[Get Player Camera Manager]
    ├─ Play Camera Shake
    │  ├─ Shake Class: CameraShake_TeleportArrival
    │  ├─ Scale: 2.0
    │  └─ Duration: 0.3s
    │
    └─ Complete
```

---

## 🎯 TESTING & TWEAKING

### **Departure Effect Tesztelés**

Checklist:
```
✓ Particles spawn at correct location
✓ Cyan color is visible
✓ Expansion is smooth and fast
✓ Sound plays without clicks
✓ Light flashes appropriately
✓ Duration matches (0.8s)
✓ No performance drops
✓ Looks good in multiplayer
```

### **Arrival Effect Tesztelés**

Checklist:
```
✓ Particles burst at target location
✓ White/gold color is bright
✓ Ring expansion looks good
✓ Sound is clear and chimey
✓ Screen shake feels punchy
✓ Light burst is visible
✓ Duration matches (1.0s)
✓ Feels satisfying to player
```

### **Trail Effect Tesztelés (If Using)**

Checklist:
```
✓ Bezier path is smooth
✓ Particles follow path correctly
✓ Glow effect is visible
✓ No gaps in the trail
✓ Fades at both ends
✓ Performance acceptable
```

---

## 🔧 PERFORMANCE OPTIMIZATION

### **Particle System Optimization**

```
Settings:
├─ Use Fixed Bounds: TRUE
│  └─ Bounds: 2000 x 2000 x 2000
│
├─ Disable with Occlusion: TRUE
│  └─ Occlusion Bounds Threshold: 10000
│
├─ Use Fixed Relative Bounding Box: TRUE
│
├─ Max Particles: 1000 (total)
│  ├─ Departure: 300
│  ├─ Arrival: 400
│  └─ Trail: 300
│
└─ Particles Per Frame Cap: 100
```

### **Audio Optimization**

```
Sound Cue Properties:
├─ Priority: Normal
├─ Max Concurrent Plays: 2
├─ Attenuation Distance: 2500 cm
└─ 3D Spatialization: Critical sounds only
```

---

## 📋 ASSET CHECKLIST

### **VFX Assets Szükségesek**
```
□ PT_Teleport_Departure (Cyan expanding)
□ PT_Teleport_Arrival (White burst)
□ PT_Teleport_Trail (Glowing path, optional)
□ M_TeleportGlow_Cyan (Material)
□ M_TeleportGlow_White (Material)
```

### **SFX Assets Szükségesek**
```
□ SFX_Teleport_Departure.wav (0.8s whoosh)
□ SFX_Teleport_Arrival.wav (1.0s chime)
□ SFX_Teleport_Travel.wav (2.0s ambient, optional)
□ SC_Teleport_Departure (Sound Cue)
□ SC_Teleport_Arrival (Sound Cue)
□ SC_Teleport_Travel (Sound Cue, optional)
```

### **Camera/Post Process**
```
□ CameraShake_TeleportArrival (Blueprint)
□ PP_TeleportScreenFade (Post Process Material)
```

---

## 🎨 COLOR REFERENCES

### **Cyan (Departure)**
```
RGB: (0, 255, 255)
Hex: #00FFFF
HSL: (180°, 100%, 50%)
Unreal: FLinearColor(0.0f, 1.0f, 1.0f, 1.0f)
```

### **White/Gold (Arrival)**
```
RGB: (255, 240, 100)
Hex: #FFF064
HSL: (50°, 100%, 70%)
Unreal: FLinearColor(1.0f, 0.94f, 0.39f, 1.0f)
```

### **Glow Enhancement**
```
Emissive Multiplier: 2.0 - 3.0x
Brightness Boost: 2.0x
Bloom Intensity: High
```

---

## 📝 DEVELOPMENT PHASES

```
Phase 1: Asset Creation
├─ [ ] Import/create audio files
├─ [ ] Create particle systems
└─ [ ] Create materials

Phase 2: Configuration
├─ [ ] Tune particle parameters
├─ [ ] Configure sound cues
├─ [ ] Setup camera shake
└─ [ ] Test materials

Phase 3: Integration
├─ [ ] Add to BP_TeleportEffectManager
├─ [ ] Test in Blueprint
├─ [ ] Network replication test
└─ [ ] Multiplayer validation

Phase 4: Polish
├─ [ ] Timing adjustments
├─ [ ] Color fine-tuning
├─ [ ] Sound mixing
├─ [ ] Performance optimization
└─ [ ] Final playtest
```

---

**Verzió**: 1.0  
**DevKit**: 5.5.4  
**Dátum**: 2026-06-22  
**Status**: 🟡 Tervezett - Asset létrehozás szükséges
