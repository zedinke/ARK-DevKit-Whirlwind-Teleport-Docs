# ⚡ Whirlwind Event System - VFX & SFX Effects
## Particle Systems, Audio Setup, és Visual Effects Implementáció

---

## 📌 Effects Overview

Az whirlwind esemény három fő vizuális típussal rendelkezik, mindegyiknek egyedi effektekkel:

```
FROST VORTEX (Jeges)
├─ Szín: Cyan/Blue
├─ Particlé: Jégkristályok, hó, mist
├─ Audio: Hideg szél, kristályzaj
├─ Fény: Cyan kék
└─ Hangulat: Erős, ellenséges

INFERNO TORNADO (Tüzes)
├─ Szín: Red/Orange/Yellow
├─ Particlé: Tűz, szikrák, füst
├─ Audio: Égő hang, robbanás
├─ Fény: Narancssárga/vörös
└─ Hangulat: Intenzív, kaotikus

AQUA VORTEX (Vizes)
├─ Szín: Blue/Green/Cyan
├─ Particlé: Víz cseppek, buborékok
├─ Audio: Víz suhogás, csobogás
├─ Fény: Világoskék
└─ Hangulat: Kiegyensúlyozott, természetes
```

---

## 🎨 PARTICLE SYSTEM - FROST VORTEX

### **PT_Whirlwind_Frost Létrehozása**

**Lépések:**

```
1. Content Browser → New → Particle System
   └─ Name: PT_Whirlwind_Frost
   └─ Folder: /Particles/WhirlwindEvents/

2. Cascade Editor megnyitása
   └─ Double-click on particle system

3. Default Emitter beállítása
   └─ Right-click → Edit Module Details
```

### **Emitter Properties - Frost**

```
Main Emitter Settings:

Duration:
├─ Duration: 300.0 (5 minutes, matches event)
├─ Loop Count: 0 (infinite loop)
└─ Pre-roll Time: 0.0

Spawn Rate:
├─ Rate Over Time: 250.0
│  ├─ Range: 200-300
│  └─ Makes particle count dynamic
│
└─ Bursts: None (continuous spawn)

Lifetime:
├─ Particle Lifetime:
│  ├─ Min: 2.0 seconds
│  ├─ Max: 5.0 seconds
│  └─ Variation creates organic feel
│
└─ Fade Out Duration: 0.5 seconds

Initial Size:
├─ Start Size:
│  ├─ Min: 20.0 cm
│  ├─ Max: 50.0 cm
│  └─ Random variation
│
├─ Growth Over Lifetime:
│  ├─ Growth: 100.0 cm
│  └─ Particles expand slightly
│
└─ End Size:
   ├─ Scale: 0.3x (shrink at end)
   └─ Fade effect

Initial Velocity:
├─ Direction: 0, 0, 1 (UPWARD!)
├─ Speed:
│  ├─ Min: 400.0 cm/s
│  ├─ Max: 1000.0 cm/s
│  └─ Fast upward movement
│
└─ Randomize: 30%

Initial Color:
├─ Start Color: Cyan (0, 1, 1, 1)
├─ Intensity: 1.5x (bright)
├─ Alpha: 1.0 (opaque)
└─ Variations:
   ├─ R: ±0.2
   ├─ G: ±0.1
   └─ B: ±0.2
```

### **Color Over Lifetime - Frost**

```
Emitter Module: Color Over Life

Timeline:
├─ 0.0s: Cyan opaque (0, 1, 1, 1)
├─ 0.5s: Cyan medium (0, 0.8, 1, 0.8)
├─ 1.5s: Cyan fading (0.2, 0.9, 1, 0.4)
├─ 2.0s: Cyan transparent (0.5, 0.5, 1, 0.2)
└─ 5.0s: Invisible (0, 0, 0, 0)

Curve Type: SMOOTH INTERPOLATION
└─ Natural fade, not abrupt
```

### **Size Over Lifetime - Frost**

```
Module: Size Over Life

Scale Curve:
├─ 0.0s: 100% (original size)
├─ 0.5s: 120% (expand)
├─ 1.0s: 110%
├─ 2.0s: 80% (shrink)
└─ 5.0s: 30% (tiny at end)

Effect:
└─ Particles grow then shrink
   └─ Creates organic spiral appearance
```

### **Velocity Over Lifetime - Frost**

```
Module: Velocity Over Life

Movement Pattern:
├─ Initial velocity: 400-1000 cm/s UP
├─ Over time: Gradual slowdown
│  └─ Gravity affects particles
│
├─ X/Y drift: Slight wind effect
│  ├─ Horizontal sway
│  └─ Makes spiral look natural
│
└─ Z velocity: Decreases over time
   └─ Particles eventually float down
```

### **Material - Frost**

```
Create Material: M_Whirlwind_Frost

Material Settings:
├─ Blend Mode: Additive
│  └─ Particles glow/blend nicely
│
├─ Light Mode: Unlit
│  └─ Self-illuminated
│
├─ Base Color:
│  ├─ Cyan: (0, 1, 1)
│  └─ Adjust saturation
│
├─ Emissive Color:
│  ├─ Same as base color
│  ├─ Multiplied by 2.0x
│  └─ Makes it glow
│
├─ Opacity:
│  ├─ Connected to particle alpha
│  ├─ Fade in/out
│  └─ Smooth transitions
│
└─ Special:
   ├─ Normal Map: Default
   ├─ Metallic: 0.5
   └─ Roughness: 0.3
```

---

## 🎨 PARTICLE SYSTEM - INFERNO TORNADO

### **PT_Whirlwind_Inferno Létrehozása**

**Emitter Properties - Inferno (Key Differences):**

```
Spawn Rate:
├─ Rate Over Time: 350.0
│  ├─ More particles (more chaotic)
│  └─ Range: 300-400
└─ Makes intense effect

Lifetime:
├─ Particle Lifetime:
│  ├─ Min: 1.0 seconds (SHORTER!)
│  └─ Max: 3.5 seconds
│  └─ Fast burn-out effect
│
└─ Fade Out: 0.3 seconds (faster)

Initial Size:
├─ Start Size:
│  ├─ Min: 40.0 cm (LARGER!)
│  ├─ Max: 100.0 cm
│  └─ Bigger, more chaotic particles
│
├─ Growth: 150.0 cm (more expansion)
│
└─ End Size: 0.1x (shrink fast)

Initial Velocity:
├─ Direction: 0, 0, 1 (UP)
├─ Speed:
│  ├─ Min: 600.0 cm/s (FASTER!)
│  ├─ Max: 1500.0 cm/s
│  └─ Violent upward rush
│
└─ Randomize: 50% (more chaos)

Initial Color:
├─ Base: Red (1, 0.3, 0)
├─ Intensity: 2.0x (VERY BRIGHT!)
├─ Alpha: 1.0
└─ Variations:
   ├─ R: ±0.3
   ├─ G: ±0.2 (vary orange)
   └─ B: ±0.1
```

### **Color Over Lifetime - Inferno**

```
Timeline:
├─ 0.0s: Bright red (1, 0.2, 0, 1)
├─ 0.3s: Orange (1, 0.5, 0, 1)
├─ 0.8s: Orange dimming (0.8, 0.3, 0, 0.6)
├─ 1.5s: Dark orange (0.4, 0.1, 0, 0.2)
└─ 3.5s: Black (0, 0, 0, 0)

Effect:
└─ Quick bright burst → rapid fade
   └─ Like real fire burning out
```

### **Material - Inferno**

```
Create Material: M_Whirlwind_Inferno

Settings:
├─ Base Color: Red/Orange (1, 0.3, 0)
├─ Emissive: Multiplied by 3.0x (VERY BRIGHT!)
│  └─ Fire glow should be intense
│
├─ Opacity: Particle alpha
├─ Metallic: 0.0 (not metallic)
└─ Roughness: 0.5 (medium rough)

Texture Options (if available):
├─ Normal Map: Optional fire noise
├─ Smoke texture for some particles
└─ Creates layered effect
```

---

## 🎨 PARTICLE SYSTEM - AQUA VORTEX

### **PT_Whirlwind_Aqua Létrehozása**

**Emitter Properties - Aqua (Balanced):**

```
Spawn Rate:
├─ Rate Over Time: 300.0
│  ├─ Balanced particle count
│  └─ Range: 250-350
└─ Not too chaotic, not too calm

Lifetime:
├─ Particle Lifetime:
│  ├─ Min: 2.5 seconds
│  ├─ Max: 6.0 seconds (LONGER!)
│  └─ Longer lasting = more visual
│
└─ Fade Out: 0.8 seconds (smooth)

Initial Size:
├─ Start Size:
│  ├─ Min: 25.0 cm
│  ├─ Max: 60.0 cm
│  └─ Balanced
│
├─ Growth: 80.0 cm
│
└─ End Size: 0.4x (slower shrink)

Initial Velocity:
├─ Direction: 0, 0, 1 (UP)
├─ Speed:
│  ├─ Min: 300.0 cm/s (SLOWER!)
│  ├─ Max: 800.0 cm/s
│  └─ Graceful upward flow
│
└─ Randomize: 40%

Initial Color:
├─ Base: Light Blue (0, 0.8, 1)
├─ Intensity: 1.2x (moderate glow)
├─ Alpha: 1.0
└─ Variations:
   ├─ R: ±0.1
   ├─ G: ±0.2
   └─ B: ±0.3
```

### **Color Over Lifetime - Aqua**

```
Timeline:
├─ 0.0s: Light blue (0, 0.8, 1, 1)
├─ 1.0s: Cyan blue (0.1, 0.9, 1, 0.9)
├─ 2.5s: Medium blue (0.2, 0.6, 1, 0.6)
├─ 4.0s: Fading blue (0.3, 0.5, 0.8, 0.3)
└─ 6.0s: Transparent (0, 0, 0, 0)

Effect:
└─ Smooth color transition
   └─ Water-like, natural fade
```

### **Material - Aqua**

```
Create Material: M_Whirlwind_Aqua

Settings:
├─ Base Color: Light Blue (0, 0.8, 1)
├─ Emissive: 1.5x multiplier
│  └─ Water glow (subtle)
│
├─ Opacity: Particle alpha
├─ Metallic: 0.3 (water reflection)
└─ Roughness: 0.7 (water surface)

Optional Effects:
├─ Refraction: Small amount
├─ Distortion: Water ripple effect
└─ Normal map: Water surface normal
```

---

## 🔊 AUDIO - FROST VORTEX

### **SFX_Whirlwind_Frost_Loop (Primary)**

**Audio File Specifications:**

```
Duration: 2.5 seconds (loopable)
Format: WAV or MP3
Sample Rate: 44100 Hz
Channels: Stereo

Sound Characteristics:
├─ Howling wind (primary layer)
│  ├─ Frequency: 500-3000 Hz
│  ├─ Eerie, sustained tone
│  └─ Wind noise synthesized or recorded
│
├─ Crystalline shimmer (secondary layer)
│  ├─ Frequency: 4000-8000 Hz
│  ├─ Subtle, high-pitched
│  └─ Ice/crystal chime sounds
│
├─ Bass rumble (optional tertiary)
│  ├─ Frequency: 60-200 Hz
│  ├─ Very subtle, felt more than heard
│  └─ Adds weight/presence
│
└─ ADSR Envelope:
   ├─ Attack: 0.2s (quick start)
   ├─ Decay: 0.5s
   ├─ Sustain: 1.5s (held middle section)
   └─ Release: 0.3s (quick fade)
```

### **SC_Whirlwind_Frost_Loop Setup**

**Sound Cue Creation:**

```
1. Import Audio File
   └─ Path: /Sounds/WhirlwindEvents/SFX_Whirlwind_Frost_Loop

2. Create Sound Cue
   └─ Name: SC_Whirlwind_Frost_Loop

3. Sound Cue Graph:
   ├─ Output Node
   └─ Wave Player
      ├─ Sound Wave: SFX_Whirlwind_Frost_Loop
      ├─ Volume: 0.7
      ├─ Pitch: 1.0
      └─ Loop: TRUE
```

### **3D Audio Settings:**

```
Attenuation:
├─ Distance Attenuation: Logarithmic
├─ Min Distance: 100 cm
├─ Max Distance: 5000 cm
│  └─ Heard from far away
│
├─ Volume Falloff:
│  ├─ Close: 1.0x volume
│  ├─ Medium: 0.7x
│  └─ Far: 0.3x
│
└─ Doppler Effect: Enabled

Spatialization:
├─ Spatial Audio: TRUE (3D)
├─ Pan Spread: 360°
│  └─ Surrounds listener
│
└─ Priority: Normal
```

### **Enhancement Sounds - Frost**

**SFX_Whirlwind_Frost_Attract (On pull):**

```
Duration: 0.5 seconds
Sound: Ice crackle/snap
├─ Frequency: 2000-5000 Hz
├─ Attack: Percussive
├─ Decay: Medium
└─ Creates feedback when actors pulled

Usage:
├─ Triggered when actor enters zone
├─ Not every frame (too many!)
│  └─ Once per 1 second max
├─ Adds interactive feedback
└─ Multicast to all clients
```

**SFX_Whirlwind_Frost_Launch (On launch):**

```
Duration: 0.8 seconds
Sound: Crystalline burst/shimmer
├─ Frequency: 3000-12000 Hz
├─ Attack: Very fast (explosion)
├─ Sustain: Brief shimmer
└─ Release: Trailing crystals

Usage:
├─ Triggered when actor ejected
├─ Single play per launch
├─ Satisfying "pop" sound
└─ Multicast to all clients
```

---

## 🔊 AUDIO - INFERNO TORNADO

### **SFX_Whirlwind_Inferno_Loop (Primary)**

**Audio File Specifications:**

```
Duration: 2.0 seconds (loopable, shorter for intensity)
Format: WAV or MP3
Sample Rate: 44100 Hz
Channels: Stereo

Sound Characteristics:
├─ Roaring fire (primary)
│  ├─ Frequency: 300-2000 Hz
│  ├─ Deep, intense roar
│  ├─ Recorded or synthesized flame
│  └─ Very prominent
│
├─ Crackling flames (secondary)
│  ├─ Frequency: 1000-6000 Hz
│  ├─ Popping/snapping sounds
│  ├─ Rapid bursts
│  └─ Makes it sound chaotic
│
├─ Hot rushing wind (tertiary)
│  ├─ Frequency: 2000-8000 Hz
│  ├─ Fast, aggressive whoosh
│  └─ Adds intensity
│
└─ ADSR Envelope:
   ├─ Attack: 0.1s (very quick, intense start!)
   ├─ Decay: 0.3s
   ├─ Sustain: 1.2s (aggressive middle)
   └─ Release: 0.4s (sharp cutoff)
```

### **SC_Whirlwind_Inferno_Loop Setup**

```
Sound Cue Configuration:
├─ Volume: 0.85 (LOUDER than frost!)
├─ Pitch: 1.1 (slightly higher = more intense)
├─ Loop: TRUE
└─ Priority: High (important to hear)
```

### **Enhancement Sounds - Inferno**

**SFX_Whirlwind_Inferno_Attract:**

```
Duration: 0.6 seconds
Sound: Flames ignite/whoosh
├─ Quick flame burst
├─ Aggressive sound
├─ Shows intensity
└─ Triggered on attraction
```

**SFX_Whirlwind_Inferno_Launch:**

```
Duration: 1.0 seconds
Sound: Explosive burst/roar
├─ Loud explosion-like sound
├─ Very satisfying
├─ Peak loudness moment
└─ Triggered on ejection
```

---

## 🔊 AUDIO - AQUA VORTEX

### **SFX_Whirlwind_Aqua_Loop (Primary)**

**Audio File Specifications:**

```
Duration: 2.8 seconds (loopable)
Format: WAV or MP3
Sample Rate: 44100 Hz
Channels: Stereo

Sound Characteristics:
├─ Swirling water sound (primary)
│  ├─ Frequency: 200-1500 Hz
│  ├─ Whooshing, flowing
│  ├─ Smooth, natural water sound
│  └─ Calming
│
├─ Bubble/splash layer (secondary)
│  ├─ Frequency: 1000-4000 Hz
│  ├─ Occasional bubbles popping
│  ├─ Splash effects
│  └─ Water droplets
│
├─ Ambient dripping (tertiary)
│  ├─ Frequency: 3000-8000 Hz
│  ├─ Light droplet sounds
│  ├─ Subtle, organic
│  └─ Natural water effect
│
└─ ADSR Envelope:
   ├─ Attack: 0.3s (gentle start)
   ├─ Decay: 0.4s
   ├─ Sustain: 1.8s (soft, continuous)
   └─ Release: 0.3s (smooth fade)
```

### **SC_Whirlwind_Aqua_Loop Setup**

```
Sound Cue Configuration:
├─ Volume: 0.75 (balanced)
├─ Pitch: 0.95 (slightly lower = calm)
├─ Loop: TRUE
└─ Priority: Normal
```

### **Enhancement Sounds - Aqua**

**SFX_Whirlwind_Aqua_Attract:**

```
Duration: 0.4 seconds
Sound: Water suction/whoosh
├─ Smooth water pull sound
├─ Not aggressive
├─ Natural and gentle
└─ Triggered on attraction
```

**SFX_Whirlwind_Aqua_Launch:**

```
Duration: 0.7 seconds
Sound: Water splash/spray
├─ Splashing water sound
├─ Droplets flying
├─ Satisfying but not violent
└─ Triggered on ejection
```

---

## 💡 LIGHT EFFECTS

### **Whirlwind Light Component Setup**

**Common Properties (All Types):**

```
Basic Settings:
├─ Type: Point Light
├─ Intensity Calculation: Inverse Square
├─ Visible: TRUE
├─ Affects World: TRUE
├─ Affects Translucency: TRUE
└─ Use Inverse Squared Falloff: TRUE

Attenuation:
├─ Intensity: Type-dependent (see below)
├─ Attenuation Radius: 2000-2500 cm
├─ Source Radius: 100-200 cm
│  └─ Soft light edges
│
└─ Source Length: 0 cm (point source)

Shadow Settings:
├─ Cast Shadows: FALSE (performance)
├─ Dynamic Shadows: FALSE
└─ For performance on multiple vortexes
```

### **Type-Specific Light Values**

**Frost Vortex Light:**

```
Color: Cyan (0, 1, 1)
├─ R: 0
├─ G: 255
└─ B: 255

Intensity: 2000 lm (moderate)
Attenuation Radius: 2000 cm

Brightness Curve:
├─ Start: 2000 lm
├─ Middle: 1500 lm
└─ End: 0 lm (fade to black)

Feel: Cool, blue-ish area around vortex
```

**Inferno Tornado Light:**

```
Color: Orange/Red (1, 0.4, 0)
├─ R: 255
├─ G: 100
└─ B: 0

Intensity: 3500 lm (VERY BRIGHT!)
Attenuation Radius: 2500 cm

Brightness Curve:
├─ Start: 3500 lm (very intense!)
├─ Middle: 2000 lm
└─ End: 0 lm

Feel: Warm, bright, intense fire glow
```

**Aqua Vortex Light:**

```
Color: Light Blue (0, 0.8, 1)
├─ R: 0
├─ G: 200
└─ B: 255

Intensity: 2200 lm (moderate-high)
Attenuation Radius: 2000 cm

Brightness Curve:
├─ Start: 2200 lm
├─ Middle: 1400 lm
└─ End: 0 lm

Feel: Gentle blue light, water-like glow
```

---

## 🎬 EFFECTS SPAWNING NODE GRAPH

### **Multicast Spawn Whirlwind Effects**

```
[Multicast Spawn Whirlwind Effects]
    ↓
[Input Parameters]
    ├─ WhirlwindType (EWhirlwindType)
    ├─ Location (Vector)
    ├─ Duration (Float)
    └─ Whirlwind Reference (BP_WhirlwindActor)
    ↓
[Switch on WhirlwindType]
    ↓
[Case: FrostVortex]
    ├─ Particle: PT_Whirlwind_Frost already playing
    │  └─ (Started in BP_WhirlwindActor constructor)
    │
    ├─ Audio: Play SC_Whirlwind_Frost_Loop
    │  ├─ Location: WhirlwindLocation
    │  ├─ Volume: 0.7
    │  ├─ Pitch: 1.0
    │  ├─ Loop: TRUE
    │  └─ Duration: Event duration
    │
    ├─ Light: Point Light already created
    │  └─ Color + Intensity set
    │
    └─ All clients hear + see immediately!
    ↓
[Case: InfernoTornado]
    ├─ Particle: PT_Whirlwind_Inferno
    ├─ Audio: SC_Whirlwind_Inferno_Loop
    │  ├─ Volume: 0.85 (LOUDER!)
    │  └─ Pitch: 1.1 (higher intensity)
    │
    ├─ Light: Bright orange glow
    │  └─ Intensity: 3500 lm
    │
    └─ Intense appearance on all clients!
    ↓
[Case: AquaVortex]
    ├─ Particle: PT_Whirlwind_Aqua
    ├─ Audio: SC_Whirlwind_Aqua_Loop
    │  ├─ Volume: 0.75
    │  └─ Pitch: 0.95 (calm)
    │
    ├─ Light: Soft blue glow
    │  └─ Intensity: 2200 lm
    │
    └─ Balanced appearance on all clients!
    ↓
[Complete Spawn]
```

### **Despawn Effects**

```
[Multicast Despawn Whirlwind]
    ↓
[Fade Out Particles]
    ├─ Deactivate particle system
    ├─ Fade over 1 second
    └─ Particles drift away
    ↓
[Fade Out Audio]
    ├─ Get audio component
    ├─ Fade volume: 1.0 → 0.0
    ├─ Duration: 0.5 seconds
    └─ Stop audio
    ↓
[Fade Out Light]
    ├─ Get light component
    ├─ Fade intensity: Current → 0
    ├─ Duration: 0.5 seconds
    └─ Light disappears
    ↓
[Cleanup]
    ├─ Destroy particle system
    ├─ Destroy audio component
    ├─ Destroy light component
    └─ All cleaned up!
    ↓
[Complete Despawn]
```

---

## 🧪 TESTING VFX/SFX

### **Test Case 1: Particle Visibility**

```
Setup:
├─ Spawn Frost Vortex
├─ Enable detailed viewport rendering
└─ Watch particles

Expected:
├─ Cyan particles spawn continuously
├─ Spiral upward in vortex
├─ Fade out and disappear
├─ No lag/performance drop
└─ Visual matches description

Checklist:
├─ [ ] Particles visible
├─ [ ] Color correct (cyan)
├─ [ ] Movement smooth
├─ [ ] Fade realistic
├─ [ ] No clipping with terrain
└─ [ ] Frame rate stable
```

### **Test Case 2: Audio Playback**

```
Setup:
├─ Spawn all three types
├─ Listen to sounds
├─ Note differences

Expected:
├─ Frost: Eerie howling wind
├─ Inferno: Intense roaring fire
├─ Aqua: Calm swirling water

Verification:
├─ [ ] Sound starts immediately
├─ [ ] Volume appropriate
├─ [ ] Pitch correct for type
├─ [ ] Looping seamless (no clicks)
├─ [ ] 3D spatialization works
├─ [ ] Heard by all players
└─ [ ] No audio clipping/distortion
```

### **Test Case 3: Light Effects**

```
Setup:
├─ Spawn each type
├─ Look at lighting
├─ Check at night + day

Expected:
├─ Frost: Blue glow around vortex
├─ Inferno: Orange bright glow
├─ Aqua: Soft blue light

Visibility:
├─ [ ] Light color correct
├─ [ ] Brightness appropriate
├─ [ ] Attenuation smooth
├─ [ ] Works at day + night
├─ [ ] No artifacts/flickering
└─ [ ] No performance impact
```

### **Test Case 4: Multiplayer Sync**

```
Setup:
├─ 2+ players
├─ Spawn whirlwind
├─ Both observe

Expected:
├─ Particles same on both clients
├─ Audio starts at same time
├─ Light appears simultaneously
├─ No desync visible

Verification:
├─ [ ] Particle sync (< 100ms)
├─ [ ] Audio sync (< 100ms)
├─ [ ] Light sync (< 100ms)
├─ [ ] No flickering on either client
└─ [ ] Works with latency (500ms test)
```

---

## 📋 EFFECTS COMPLETION CHECKLIST

```
PARTICLE SYSTEMS:

Frost Vortex:
├─ [ ] PT_Whirlwind_Frost created
├─ [ ] Emitter properties configured
├─ [ ] Color over lifetime set
├─ [ ] Size over lifetime set
├─ [ ] M_Whirlwind_Frost material created
└─ [ ] Tested and approved

Inferno Tornado:
├─ [ ] PT_Whirlwind_Inferno created
├─ [ ] Emitter properties configured (FASTER!)
├─ [ ] Color over lifetime set
├─ [ ] Size over lifetime set
├─ [ ] M_Whirlwind_Inferno material created
└─ [ ] Tested and approved

Aqua Vortex:
├─ [ ] PT_Whirlwind_Aqua created
├─ [ ] Emitter properties configured
├─ [ ] Color over lifetime set
├─ [ ] Size over lifetime set
├─ [ ] M_Whirlwind_Aqua material created
└─ [ ] Tested and approved

AUDIO CUES:

Frost:
├─ [ ] SFX_Whirlwind_Frost_Loop imported
├─ [ ] SC_Whirlwind_Frost_Loop created
├─ [ ] SFX_Whirlwind_Frost_Attract imported
├─ [ ] SFX_Whirlwind_Frost_Launch imported
└─ [ ] Tested in game

Inferno:
├─ [ ] SFX_Whirlwind_Inferno_Loop imported
├─ [ ] SC_Whirlwind_Inferno_Loop created
├─ [ ] SFX_Whirlwind_Inferno_Attract imported
├─ [ ] SFX_Whirlwind_Inferno_Launch imported
└─ [ ] Tested in game

Aqua:
├─ [ ] SFX_Whirlwind_Aqua_Loop imported
├─ [ ] SC_Whirlwind_Aqua_Loop created
├─ [ ] SFX_Whirlwind_Aqua_Attract imported
├─ [ ] SFX_Whirlwind_Aqua_Launch imported
└─ [ ] Tested in game

LIGHT COMPONENTS:

All Types:
├─ [ ] Light colors configured
├─ [ ] Intensities set (type-specific)
├─ [ ] Attenuation radius correct
├─ [ ] Source radius appropriate
├─ [ ] Shadow casting OFF (perf)
└─ [ ] Tested all types

MULTICAST:

├─ [ ] Spawn effects multicast working
├─ [ ] Despawn effects multicast working
├─ [ ] All clients see effects simultaneously
├─ [ ] Audio synced across clients
├─ [ ] Lighting synced across clients
└─ [ ] Tested with latency
```

---

**Verzió**: 1.0  
**DevKit**: 5.5.4  
**Dátum**: 2026-06-22  
**Status**: 🟡 Implementálható - Effects setup guide
