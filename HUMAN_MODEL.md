# Human Body Simulator - Physiological Model & Architecture

## Overview

Discrete-event simulator with branching timelines (inspired by mcts-simple).
Each "tick" = 1 minute. State is immutable → enables time-travel.

This document defines the core physiological systems and their interactions for simulating how activities affect body state over time.

---

## 1. System Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              SIMULATION STATE                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────┐      ┌─────────────────────┐                      │
│  │   METABOLIC         │      │   AUTONOMIC NS      │                      │
│  │   ─────────         │      │   ────────────      │                      │
│  │   glucose           │◄────►│   sympathetic       │                      │
│  │   ketones           │      │   parasympathetic   │                      │
│  │   glycogen          │      │   HRV               │                      │
│  │   insulin           │      │   cortisol          │                      │
│  │   glucagon          │      │                     │                      │
│  └──────────┬──────────┘      └──────────┬──────────┘                      │
│             │                            │                                  │
│             ▼                            ▼                                  │
│  ┌─────────────────────┐      ┌─────────────────────┐                      │
│  │   HYDRATION         │      │   HORMONAL          │                      │
│  │   ─────────         │      │   ────────          │                      │
│  │   totalWater        │◄────►│   testosterone      │                      │
│  │   intracellular     │      │   dopamine          │                      │
│  │   extracellular     │      │   serotonin         │                      │
│  │   sodium            │      │   adrenaline        │                      │
│  │   potassium         │      │   noradrenaline     │                      │
│  │   magnesium         │      │   BDNF              │                      │
│  └──────────┬──────────┘      └──────────┬──────────┘                      │
│             │                            │                                  │
│             └────────────┬───────────────┘                                  │
│                          ▼                                                  │
│  ┌─────────────────────┐      ┌─────────────────────┐                      │
│  │   ENERGY / FATIGUE  │      │   CIRCADIAN         │                      │
│  │   ───────────────── │      │   ─────────         │                      │
│  │   ATP               │◄────►│   timeOfDay         │                      │
│  │   muscleGlycogen    │      │   melatonin         │                      │
│  │   perceivedEnergy   │      │   coreBodyTemp      │                      │
│  │   sleepDebt         │      │   circadianPhase    │                      │
│  │   adenosine         │      │   lightExposure     │                      │
│  └─────────────────────┘      └─────────────────────┘                      │
│                                                                             │
│  ┌─────────────────────┐      ┌─────────────────────┐                      │
│  │   THERMOREGULATION  │      │   IMMUNE            │                      │
│  │   ─────────────────│      │   ──────            │                      │
│  │   coreTemp          │◄────►│   status            │                      │
│  │   skinTemp          │      │   inflammationLevel │                      │
│  │   sweating          │      │   cytokines         │                      │
│  │   shivering         │      │   whiteBloodCells   │                      │
│  └─────────────────────┘      └─────────────────────┘                      │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Detailed Subsystem Interfaces

### 2.1 Metabolic Subsystem

```typescript
interface MetabolicState {
  bloodGlucose: number;      // mg/dL (normal: 70-100, keto: 65-85)
  bloodKetones: number;      // mmol/L (keto: 0.5-3.0, nutritional ketosis: 1.5-3.0)
  liverGlycogen: number;     // grams (max ~100g)
  muscleGlycogen: number;    // grams (max ~400g)
  insulin: number;           // μU/mL (fasting: 2-25, post-meal: can spike 50-100+)
  insulinSensitivity: number; // 0-1 (higher = better, lower = insulin resistance)
  glucagon: number;          // normalized 0-1
  freefattyAcids: number;    // mmol/L
  metabolicMode: 'glycolytic' | 'ketogenic' | 'hybrid';
  fatAdaptation: number;     // 0-1 (how efficient at burning fat, takes weeks)
}
```

**Insulin - The Master Switch:**
```
HIGH INSULIN (fed/carb state):
├── ANABOLIC mode (building/storing)
├── Glucose → Glycogen (storage)
├── Glucose → Fat (lipogenesis if glycogen full)
├── Fat burning: BLOCKED (lipolysis inhibited)
├── Protein synthesis: ENABLED
└── Ketone production: BLOCKED

LOW INSULIN (fasted/keto state):
├── CATABOLIC mode (burning)
├── Glycogen → Glucose (glycogenolysis)
├── Fat → Ketones (ketogenesis)
├── Fat burning: ENABLED (lipolysis active)
├── Gluconeogenesis: ACTIVE (protein/glycerol → glucose)
└── Growth hormone: ELEVATED
```

**Fuel Systems Comparison:**

| Aspect | Glycolytic (Carb Burning) | Ketogenic (Fat Burning) |
|--------|---------------------------|-------------------------|
| Primary fuel | Glucose | Ketones + FFAs |
| Insulin level | High/fluctuating | Low/stable |
| Energy stability | Peaks and crashes | Steady state |
| Mental clarity | Variable (glucose dependent) | High (once adapted) |
| Hunger signals | Frequent (glucose dips) | Reduced (stable fuel) |
| Workout capacity | High intensity excellent | Endurance excellent |
| Adaptation time | Immediate | 2-4 weeks for full adaptation |
| Glycogen needs | Must replenish constantly | Can run low indefinitely |

**Fat Adaptation Process:**
```
Week 1: Keto flu (glycogen depletion, electrolyte loss, brain adjusting)
Week 2: Mitochondria upregulating fat oxidation enzymes
Week 3: Brain efficiently using ketones (reduces glucose dependency)
Week 4+: Full fat adaptation - metabolic flexibility restored
```

**Transitions:**
- High carb meal → ↑glucose → ↑insulin → ↓ketones → glycolytic mode
- Fasting/keto → ↓glucose → ↓insulin → ↑ketones → ↑FFA → ketogenic mode
- Exercise → ↓glycogen → ↑glucagon
- 12-16h fasting → liver glycogen depleted → ketone production begins
- Chronic high carb → ↓insulin sensitivity → metabolic dysfunction
- Keto adaptation → ↑insulin sensitivity → metabolic flexibility

### 2.2 Autonomic Nervous System

```typescript
interface AutonomicState {
  sympathetic: number;       // 0-1 (fight/flight activation)
  parasympathetic: number;   // 0-1 (rest/digest activation)
  hrv: number;               // ms RMSSD (higher = better recovery, typical 20-100ms)
  cortisol: number;          // normalized, follows circadian (peak ~30min after waking)
}
```

**Transitions:**
- HIIT → ↑sympathetic → ↓HRV → ↑cortisol
- Rest/meditation → ↑parasympathetic → ↑HRV
- Chronic stress → sustained ↑cortisol → ↓HRV baseline
- Cold exposure → acute ↑sympathetic → then ↑parasympathetic rebound

**HRV Calculation:**
```
HRV = baselineHRV * parasympathetic / (sympathetic + 0.1)
```

### 2.3 Hydration & Electrolytes

```typescript
interface HydrationState {
  totalBodyWater: number;        // liters (typically ~42L for 70kg male)
  intracellularWater: number;    // ~28L (2/3 of total)
  extracellularWater: number;    // ~14L (1/3 of total)
  sodium: number;                // mEq/L (normal: 136-145)
  potassium: number;             // mEq/L (normal: 3.5-5.0)
  magnesium: number;             // mg/dL (normal: 1.7-2.2)
  zinc: number;                  // μg/dL (normal: 80-120)
  osmolality: number;            // mOsm/kg (normal: 275-295)
}
```

**Electrolyte Functions:**

| Electrolyte | Primary Functions | Deficiency Symptoms |
|-------------|-------------------|---------------------|
| **Sodium** | Fluid balance, nerve impulses, muscle contraction | Headache, fatigue, cramps, brain fog |
| **Potassium** | Heart rhythm, muscle function, nerve signals | Weakness, cramps, heart palpitations |
| **Magnesium** | 300+ enzyme reactions, muscle/nerve, sleep, mood | Anxiety, insomnia, cramps, irritability |
| **Zinc** | Testosterone, immune, protein synthesis, taste | Low T, slow healing, brain fog, low libido |

**Sodium (Na+):**
```
Normal diet: ~2-3g/day sufficient
Keto diet: 4-5g/day needed (insulin-driven natriuresis)
Heavy sweating: +1-2g/day additional
Symptoms of low sodium: "keto flu", headache, dizziness, fatigue, cramps
```

**Magnesium (Mg2+):**
```
Functions:
├── Sleep quality (activates parasympathetic)
├── Muscle relaxation (prevents cramps)
├── Stress response (buffers cortisol)
├── Blood sugar regulation
├── ATP production (every energy reaction)
└── GABA receptor function (calming)

Forms: Glycinate (sleep), Threonate (cognitive), Citrate (general)
Daily need: 300-400mg elemental, up to 600mg if depleted
Best time: Evening (promotes sleep)
```

**Zinc (Zn):**
```
Functions:
├── Testosterone production (essential cofactor)
├── Immune function (T-cell activation)
├── Protein synthesis (muscle building)
├── Dopamine regulation
├── Wound healing
├── Taste/smell (deficiency = loss)
└── Semen production (high zinc concentration)

Daily need: 15-30mg (higher if sexually active/retained)
Lost in: Ejaculation (~1-5mg per emission), Sweat, Stress
Signs of deficiency: Low libido, slow recovery, frequent illness
Note: Competes with copper - don't mega-dose long-term
```

**Keto-specific effects:**
- ↓insulin → ↑sodium excretion (natriuresis) → need 4-5g sodium/day vs ~2g normally
- Glycogen depletion → water loss (3g water per 1g glycogen)
- Higher protein intake → may need more zinc
- Keto electrolyte needs: ~4-5g sodium, 1-3g potassium, 300-400mg magnesium, 15-30mg zinc daily

**Electrolyte Timing:**
```
Morning: Sodium (with water, activates)
Pre-workout: Sodium + Potassium
Post-workout: Full spectrum (lost in sweat)
Evening: Magnesium (promotes sleep)
Daily: Zinc with food (absorption)
```

### 2.4 Hormonal & Neurochemical

```typescript
interface HormonalState {
  // Catecholamine cascade: L-Tyrosine → L-DOPA → Dopamine → Norepinephrine → Adrenaline
  dopamine: {
    baseline: number;        // tonic level (motivation threshold)
    phasic: number;          // acute spikes (reward response)
  };
  norepinephrine: number;    // normalized 0-1 (alertness, "frame rate")
  adrenaline: number;        // normalized 0-1 (acute stress response)

  // Wakefulness system
  orexin: number;            // normalized 0-1 (wakefulness, appetite, motivation)

  // Sex hormones & retention
  testosterone: number;      // ng/dL (male: 300-1000, female: 15-70)
  DHT: number;               // dihydrotestosterone (potent androgen)
  prolactin: number;         // normalized (rises post-ejaculation, suppresses T)
  retentionDays: number;     // days since last ejaculation

  // Other hormones
  cortisol: number;          // follows circadian (reference from autonomic)
  serotonin: number;         // normalized 0-1 (mood, satiety)
  melatonin: number;         // normalized 0-1 (sleep pressure)

  // Growth factors
  BDNF: number;              // normalized (brain-derived neurotrophic factor)
  IGF1: number;              // ng/mL (growth/recovery)
  growthHormone: number;     // ng/mL (peaks during deep sleep, fasting)
}
```

**Gamma Function for Motivation (ADHD model):**
```
motivation = input^gamma
- Neurotypical: gamma ≈ 1 (linear response)
- High-gamma brain: gamma > 2.5 (low signals crushed to zero)
```

**Transitions:**
- HIIT → ↑testosterone (acute), ↑adrenaline, ↑BDNF, ↑norepinephrine
- Fasting → ↑noradrenaline, ↑BDNF, ↑growth hormone (after 16h)
- Sleep deprivation → ↓testosterone, ↓dopamine sensitivity, ↓orexin
- Cold exposure → ↑noradrenaline (+530%), ↑dopamine (+250%)

**Orexin System (Wakefulness):**
| Factor | Effect on Orexin | Result |
|--------|------------------|--------|
| Light exposure | ↑ | Alertness, wakefulness |
| Fasting/Low glucose | ↑ | Hunger + alertness (survival mode) |
| High carb meal | ↓↓ | Post-meal drowsiness |
| Sleep debt | ↓ baseline | Chronic fatigue |
| Exercise | ↑ (acute) | Energy, motivation |

Orexin neurons are central to:
- Wake/sleep switching (loss = narcolepsy)
- Reward-seeking behavior
- Energy expenditure regulation
- Preventing "food coma" during fasting

**Semen Retention Effects:**

| Retention Period | Physiological Changes | Subjective Effects |
|------------------|----------------------|-------------------|
| Days 1-3 | Prolactin normalizing, androgen receptors upregulating | Recovery baseline |
| Days 4-7 | Testosterone peaks (~145% at day 7), DHT rising | Energy, confidence surge |
| Days 7-14 | Androgen receptor sensitivity ↑, dopamine sensitivity ↑ | Motivation, drive increase |
| Days 14-30 | Stable elevated baseline, zinc/protein retention | Sustained energy, mental clarity |
| Days 30+ | Transmutation potential (energy redirection) | Variable - depends on energy use |

**Mechanism:**
```
Ejaculation → Prolactin spike → Dopamine ↓ → Androgen receptor downregulation
             → Zinc/protein loss → Refractory period (tired, unmotivated)

Retention → Prolactin low → Dopamine baseline ↑ → Androgen receptors sensitize
          → Zinc/nutrients retained → Testosterone effects amplified
```

**Key factors affected by retention:**
- **Dopamine sensitivity**: Higher baseline, rewards feel more meaningful
- **Drive/Motivation**: Task completion, goal pursuit enhanced
- **Social confidence**: Pheromone production, eye contact comfort
- **Physical energy**: Less metabolic "drain"
- **Mental clarity**: Reduced "brain fog"

**Note:** Wet dreams (nocturnal emissions) partially reset the cycle but with less prolactin impact than conscious ejaculation.

### 2.5 Energy & Fatigue

```typescript
interface EnergyState {
  ATP: number;               // normalized cellular energy 0-1
  perceivedEnergy: number;   // 0-100 subjective
  sleepDebt: number;         // hours accumulated
  adenosine: number;         // 0-1 (sleep pressure, cleared by sleep/caffeine)
  muscleFatigue: number;     // 0-1 per muscle group
  centralFatigue: number;    // 0-1 CNS fatigue (limits max effort)
}
```

### 2.6 Circadian System (Expanded)

```typescript
interface CircadianState {
  timeOfDay: number;         // minutes since midnight (0-1440)
  circadianPhase: 'early_morning' | 'morning' | 'afternoon' | 'evening' | 'night' | 'late_night';
  lightExposure: number;     // lux accumulated today
  melatonin: number;         // 0-1 (rises ~2h before sleep)
  coreBodyTemp: number;      // °C (nadir ~4am, peak ~6pm)
  cortisolRhythm: number;    // multiplier for cortisol (peak at wake, low at night)
}
```

**Circadian Rhythm Details:**

| Time | Phase | Cortisol | Melatonin | Core Temp | Optimal For |
|------|-------|----------|-----------|-----------|-------------|
| 04:00-06:00 | Late Night | Rising | High→Low | Nadir (36.0°C) | Deep sleep, GH release |
| 06:00-09:00 | Early Morning | Peak (+50%) | Low | Rising | Light exposure critical |
| 09:00-12:00 | Morning | High | Low | Rising | Focus work, testosterone peak |
| 12:00-15:00 | Afternoon | Declining | Low | Peak (37.5°C) | Physical performance |
| 15:00-18:00 | Late Afternoon | Low-moderate | Low | Peak | Strength/coordination peak |
| 18:00-21:00 | Evening | Low | Rising | Declining | Wind down, avoid bright light |
| 21:00-00:00 | Night | Minimal | High | Declining | Sleep onset |
| 00:00-04:00 | Deep Night | Minimal | High | Lowest | Deep sleep, repair |

**Light Exposure Effects:**
- Morning light (first 2 hours): Sets circadian clock, ↑cortisol, ↑alertness
- Bright light (>10,000 lux): Suppresses melatonin, shifts rhythm earlier
- Evening blue light: Delays melatonin onset, disrupts sleep
- Darkness at night: Enables melatonin rise, supports sleep architecture

### 2.7 Thermoregulation

```typescript
interface ThermoState {
  coreTemp: number;          // °C (normal: 36.5-37.5, follows circadian)
  skinTemp: number;          // °C (varies by location)
  sweating: number;          // 0-1 activation level
  shivering: number;         // 0-1 activation level
  vasodilation: number;      // 0-1 (heat dissipation)
  vasoconstriction: number;  // 0-1 (heat conservation)
  yawning: number;           // 0-1 (brain cooling mechanism)
}
```

**Thermoregulation Mechanisms:**
| Mechanism | Trigger | Function | Notes |
|-----------|---------|----------|-------|
| Yawning | Brain temp ↑ | Cooling via blood flow to head | Also for state transitions |
| Sweating | Core temp > 37.5°C | Evaporative cooling | Loses electrolytes |
| Shivering | Core temp < 36°C | Heat generation via muscle | Energy expensive |
| Vasodilation | Heat stress | Blood to skin surface | Drops BP |
| Vasoconstriction | Cold stress | Blood to core | Preserves heat |

### 2.8 Immune System

```typescript
interface ImmuneState {
  status: 'healthy' | 'fighting_bacterial' | 'fighting_viral' | 'inflamed' | 'recovering';
  inflammationLevel: number;  // 0-1
  whiteBloodCells: number;    // normalized
  cytokines: {
    proInflammatory: number;  // IL-6, TNF-alpha
    antiInflammatory: number; // IL-10
  };
  feverResponse: number;      // 0-1 (intentional temp elevation)
}
```

**Immune-Metabolic Interactions (Wang Study):**
| Infection Type | Optimal Strategy | Mechanism |
|----------------|------------------|-----------|
| Bacterial | Fasting/Ketosis | Glucose restriction starves bacteria |
| Viral | Glucose intake | Glucose protects neurons from inflammation |
| Post-illness | Hybrid (50-70g carbs) | Support recovery without inflammation spike |

---

## 3. State Machine: Operating Modes

### 3.1 Quantization Model (1-bit vs Gradient)

Some individuals operate in binary states rather than gradients:

| State | Characteristics | Neurochemical Profile | Gamma |
|-------|-----------------|----------------------|-------|
| **OFF** | Asleep, Bored, Paralyzed | Low dopamine, Low norepinephrine | - |
| **ON** | "God Mode", Flow, "Beast Mode" | High dopamine, High norepinephrine, Low cortisol | < 1 |
| **Gradient** | Normal functioning | Moderate, balanced | ≈ 1 |

### 3.2 Flow Channel (Challenge vs Skill)

```
                    HIGH CHALLENGE
                          │
              [Anxiety]   │   [FLOW]
                          │   (High dopamine + High norepinephrine
                          │    + Low cortisol + High skill match)
    LOW SKILL ────────────┼──────────── HIGH SKILL
                          │
             [Apathy]     │   [Boredom]
                          │   (Need higher challenge to activate)
                          │
                    LOW CHALLENGE
```

### 3.3 Activation Energy Model

Like chemical reactions, state transitions require **activation energy**:

```
Energy Required = baseActivation * (1 / dopamine_baseline) * gamma_factor

Where:
- baseActivation: inherent task difficulty
- dopamine_baseline: current tonic dopamine (higher = easier to start)
- gamma_factor: individual's gamma coefficient (higher = needs bigger "mountains")
```

### 3.4 Tachypsychia ("Frame Rate" Model)

Peak state = increased perceptual sampling rate:

| State | Norepinephrine | Perceived "Frame Rate" | Experience |
|-------|----------------|------------------------|------------|
| Low alert | Low | ~30 fps | Normal time perception |
| High alert | High | ~60-120 fps | Time seems slower |
| Peak/Flow | Very high | ~120+ fps | "Slow motion", intuitive action |

---

## 4. Activities (Inputs)

```typescript
type Activity =
  | { type: 'meal'; carbs: number; protein: number; fat: number; fiber: number; glycemicIndex?: number }
  | { type: 'water'; liters: number; sodium?: number; potassium?: number; magnesium?: number }
  | { type: 'supplement'; name: SupplementType; amount: number }
  | { type: 'exercise'; mode: ExerciseMode; duration: number; intensity: number }
  | { type: 'sleep'; quality: number; duration: number; stages?: SleepStages }
  | { type: 'stress'; level: number; duration: number; source?: string }
  | { type: 'cold_exposure'; temp: number; duration: number }
  | { type: 'heat_exposure'; temp: number; duration: number }
  | { type: 'light_exposure'; lux: number; duration: number; timeOfDay: number }
  | { type: 'meditation'; duration: number; technique?: 'breath' | 'nsdr' | 'yoga_nidra' }
  | { type: 'caffeine'; amount: number; timeSinceWaking: number }
  | { type: 'fasting' }  // no-op, just no food
  | { type: 'breathing'; technique: 'box' | 'wim_hof' | 'physiological_sigh'; duration: number }
  | { type: 'ejaculation' }  // resets retention counter, triggers prolactin
  | { type: 'nocturnal_emission' };  // partial reset, less prolactin impact

type SupplementType = 'sodium' | 'potassium' | 'magnesium' | 'caffeine' | 'l_tyrosine' |
                      'alpha_gpc' | 'creatine' | 'omega3' | 'vitamin_d' | 'zinc';

type ExerciseMode = 'HIIT' | 'zone2' | 'strength' | 'walk' | 'sprint' | 'yoga';

interface SleepStages {
  light: number;      // minutes
  deep: number;       // minutes (SWS)
  rem: number;        // minutes
  awakenings: number;
}
```

---

## 5. Effects Matrix

| Activity | Glucose | Ketones | Sympathetic | HRV | Dopamine | Norepi | Cortisol | Orexin | Notes |
|----------|---------|---------|-------------|-----|----------|--------|----------|--------|-------|
| High-carb meal | ↑↑ | ↓↓ | ↓ | ↑ | ↑ (temp) | → | → | ↓↓ | Insulin spike, drowsy |
| Keto meal | → | ↑ | → | → | → | → | → | → | Stable energy |
| Protein meal | ↑ | → | → | → | ↑ | → | → | → | Gluconeogenesis |
| HIIT | ↓↓ | ↑ | ↑↑↑ | ↓↓ | ↑↑ | ↑↑↑ | ↑↑ | ↑↑ | BDNF spike |
| Zone 2 | ↓ | ↑ | ↑ | → | ↑ | ↑ | → | ↑ | Fat oxidation |
| Strength | ↓ | → | ↑↑ | ↓ | ↑ | ↑↑ | ↑ | ↑ | Testosterone ↑ |
| Fasting | ↓ | ↑↑ | ↑ | varies | ↑ | ↑ | ↑ | ↑↑ | GH after 16h, alert |
| Sleep | → | varies | ↓↓ | ↑↑ | reset | ↓ | rhythm | ↓↓ | Adenosine cleared |
| Cold (brief) | → | → | ↑↑ | ↓→↑ | ↑↑↑ | ↑↑↑ | ↑ | ↑↑ | 250% DA, 530% NE |
| Heat/Sauna | → | → | ↑ | ↓ | ↑ | ↑ | ↑ | → | GH spike |
| Caffeine | ↑ | → | ↑ | ↓ | ↑ | ↑ | ↑ | ↑ | Blocks adenosine |
| NSDR | → | → | ↓ | ↑↑ | ↑↑ | ↓ | ↓ | → | 65% DA restoration |
| Morning light | → | → | ↑ | → | ↑ | ↑ | ↑↑ | ↑↑ | Circadian anchor |
| Wim Hof breathing | → | → | ↑↑ | varies | ↑ | ↑↑ | ↑ | ↑ | Alkalosis, adrenaline |
| Ejaculation | → | → | ↑→↓ | ↓ | ↓↓ | ↓ | → | ↓ | Prolactin ↑↑, refractory |
| Retention (7+ days) | → | → | → | → | ↑ sens | ↑ | → | ↑ | T receptors ↑ |

**Retention Effects Over Time:**

| Day | Testosterone | Prolactin | Dopamine Sensitivity | Drive/Motivation | Zinc Status |
|-----|--------------|-----------|---------------------|------------------|-------------|
| 0 (ejac) | baseline | ↑↑↑ spike | ↓↓ | ↓↓ refractory | ↓ lost |
| 1-3 | recovering | normalizing | recovering | returning | stabilizing |
| 4-6 | ↑ rising | low | ↑ increasing | ↑ noticeable | ↑ retained |
| 7 | ↑↑ peak (145%) | low | ↑↑ high | ↑↑ strong | ↑ good |
| 8-14 | stable elevated | low | ↑↑ high | ↑↑ sustained | ↑ accumulated |
| 14-30 | ↑ stable | low | ↑↑↑ very high | ↑↑↑ strong | ↑ optimal |
| 30+ | stable | low | transmutation phase | depends on direction | ↑ stable |

---

## 6. State Transitions (Tick Function)

Each tick (1 minute):

```typescript
function tick(state: BodyState, events: Activity[]): BodyState {
  // 1. Apply circadian rhythms (cortisol, melatonin, temperature)
  state = applyCircadian(state, state.timeOfDay);

  // 2. Process queued events
  for (const event of events) {
    state = applyActivity(state, event);
  }

  // 3. Run subsystem updates (order matters!)
  state = updateMetabolic(state);      // glucose, ketones, insulin
  state = updateAutonomic(state);      // sympathetic/para balance
  state = updateHydration(state);      // water, electrolytes
  state = updateHormonal(state);       // testosterone, dopamine, etc.
  state = updateThermoregulation(state); // temperature regulation
  state = updateImmune(state);         // if fighting infection
  state = updateEnergy(state);         // fatigue, perceived energy

  // 4. Apply cross-system effects
  state = applyCrossEffects(state);

  // 5. Update time
  state.circadian.timeOfDay = (state.circadian.timeOfDay + 1) % 1440;

  return state;
}

// Cross-system effects examples:
function applyCrossEffects(state: BodyState): BodyState {
  // Low glucose affects mood
  if (state.metabolic.bloodGlucose < 60) {
    state.hormonal.dopamine.baseline *= 0.9;
    state.energy.perceivedEnergy -= 10;
  }

  // High cortisol suppresses immune function
  if (state.autonomic.cortisol > 0.8) {
    state.immune.whiteBloodCells *= 0.95;
  }

  // Sleep debt accumulates adenosine
  if (state.energy.sleepDebt > 0) {
    state.energy.adenosine += 0.001 * state.energy.sleepDebt;
  }

  // Dehydration affects cognition
  if (state.hydration.totalBodyWater < 40) {
    state.energy.perceivedEnergy -= 5;
    state.hormonal.dopamine.phasic *= 0.9;
  }

  return state;
}
```

---

## 7. Timeline / Branching (Time Travel)

```typescript
interface Timeline {
  id: string;
  parentId?: string;
  branchPoint: number;        // tick where branched
  states: BodyState[];        // state at each tick
  events: Map<number, Activity[]>;  // scheduled events
}

// Time travel: branch from any point
function branch(timeline: Timeline, fromTick: number): Timeline {
  return {
    id: generateId(),
    parentId: timeline.id,
    branchPoint: fromTick,
    states: timeline.states.slice(0, fromTick + 1),
    events: new Map()
  };
}

// Compare timelines
function compareTimelines(t1: Timeline, t2: Timeline, metric: keyof BodyState): Comparison {
  // Show divergence from branch point
}
```

---

## 8. Protocols & Interventions

### 8.1 "The Manual" - Three Pillars
```
1. FUEL IT    → Keto/Hydration, Electrolytes
2. TRAIN IT   → HIIT + Zone 2
3. CHALLENGE IT → Deep Strategy, Hard Problems (Level 10 targets)
```

### 8.2 Morning Protocol (Huberman-style)
```yaml
wake_up:
  - delay_caffeine: 90-120 min (allows natural cortisol spike + adenosine clearing)
  - light_exposure:
      clear_day: 5 min outdoor
      cloudy_day: 10 min outdoor
      overcast: 20-30 min outdoor
      target: 100,000 lux before 9am (indoor lights only ~4-5k lux)
      note: No sunglasses/blue blockers; contacts/glasses OK
  - cold_exposure: 1-3 min (optional, for dopamine/norepinephrine)
  - hydration: 16-32 oz water with sodium

supplements_optional:
  - l_tyrosine: 500mg (dopamine precursor, 4x/week max)
  - alpha_gpc: 300mg (acetylcholine, focus)
  - phenylethylamine: 500mg (alt to tyrosine, 30-45 min dopamine spike)
```

**Why Delay Caffeine:**
- Caffeine blocks adenosine receptors (doesn't clear adenosine)
- Morning cortisol spike naturally clears adenosine in first 90 min
- Immediate caffeine = adenosine builds up behind the block → afternoon crash
- Delayed caffeine = cortisol clears adenosine first → sustained energy

**Morning Light Science:**
- Triggers 50% increase in cortisol pulse amplitude
- Sets circadian timer for sleep ~16 hours later
- Enhances immune function, metabolism, and focus all day
- 2 days of proper light exposure can reset drifted circadian rhythms

### 8.3 Illness Recovery (Viral)
```yaml
strategy: "Hybrid"
nutrition:
  - bone_broth: liters (electrolytes, soothing)
  - berries: moderate (low glycemic glucose)
  - honey: 1 tsp raw (antiviral, 5-6g carbs)
  - light_proteins: [eggs, chicken_breast, white_fish]
avoid:
  - heavy_fats (digestion impaired)
  - fasting (increases cortisol)
  - sudden_high_carb (inflammation if keto-adapted)
target_carbs: 50-70g ("medicinal carbs")
```

### 8.4 Sleep Optimization
```yaml
evening:
  - dim_lights: 2h before bed
  - cool_room: 65-68°F (18-20°C) # Body needs to drop 1-3°C to sleep
  - no_caffeine: after 2pm (caffeine has 8h half-life, 25% still active at midnight if noon dose)
  - no_late_exercise: 3h before bed
  - warm_bath: Optional, facilitates body heat loss after

sleep_architecture_targets:
  - deep_sleep: 90-120 min (GH release, physical repair, memory consolidation)
  - rem_sleep: 90-120 min (emotional processing, creative problem-solving)
  - total: 7-9 hours
  - note: Deep sleep dominates first half of night; REM dominates second half

5_pillars_of_sleep_hygiene:
  1. Regularity (consistent wake/sleep times)
  2. Darkness (melatonin production)
  3. Temperature (cool room, body temp drop)
  4. Get out of bed if can't sleep after 20 min
  5. Mindful of alcohol and caffeine

if_disrupted:
  - nsdr: 10-30 min next day (restores dopamine 65%)
  - extended_nsdr: 30-60 min if significant sleep loss
  - no_naps: after 3pm (preserves sleep drive)
```

### 8.5 Cold Exposure Protocol (Huberman)
```yaml
weekly_target: 11 minutes total
session_structure:
  - frequency: 2-4 sessions per week
  - duration: 1-5 minutes per session
  - temperature: 37-55°F (3-13°C) # Cold enough to want to get out

temperature_duration_tradeoff:
  very_cold: # 35-45°F (2-7°C)
    duration: 1-3 min
    effect: Rapid catecholamine release
  moderate: # 55-60°F (13-15°C)
    duration: 30-60 min
    effect: Sustained dopamine/norepinephrine elevation

neurochemical_effects:
  dopamine: +250% (sustained for hours)
  norepinephrine: +530% (sustained for hours)
  adrenaline: Acute spike

timing:
  optimal: Morning (enhances alertness)
  avoid: Within 6h after strength training (suppresses adaptation)
  avoid: Close to bedtime

progression:
  beginner: Start 55-60°F, 30 sec - 1 min
  intermediate: Progress to 45-50°F, 2-3 min
  advanced: 37-45°F, 3-5 min
```

### 8.6 NSDR / Yoga Nidra Protocol
```yaml
what_it_is:
  - Guided relaxation without falling asleep
  - Body scan + intentional deep breathing
  - Shifts to parasympathetic dominance

effects:
  dopamine: +65% baseline restoration
  cortisol: Reduced
  attention: Improved working memory
  anxiety: Reduced
  brain_waves: Shift to theta/delta (like deep sleep)

protocols:
  daily_maintenance:
    duration: 10-30 min
    timing: Afternoon (after lunch, before work)
  sleep_debt_recovery:
    duration: 30-60 min
    timing: Early afternoon
  focus_restoration:
    duration: 10 min
    timing: After demanding cognitive work

huberman_personal: 10-30 min daily, extended when sleep-deprived
```

### 8.7 Breathing Techniques
```yaml
physiological_sigh: # Fastest real-time stress relief
  technique:
    - Double inhale through nose (big + small with no exhale between)
    - Long exhale through mouth until lungs empty
  duration: 1-3 breaths for acute stress; 5 min daily for chronic stress reduction
  effects:
    - Rapid shift to parasympathetic
    - Lower heart rate
    - Improved sleep
  research: Stanford study showed 5 min daily reduces overall stress

box_breathing: # Balanced, calming
  technique:
    - Inhale 4 sec
    - Hold 4 sec
    - Exhale 4 sec
    - Hold 4 sec
  duration: 5-10 min
  effects: Even heart rate, calm focus
  note: Duration based on CO2 tolerance

cyclic_hyperventilation: # Wim Hof style - arousing
  technique:
    - 25-30 deep breaths (vigorous inhale, passive exhale)
    - Hold after exhale (15-60 sec)
    - Recovery breath (deep inhale, hold 15 sec)
    - Repeat 3-4 rounds
  effects:
    - Adrenaline release
    - Increased body temperature
    - Alkalosis (tingling, light-headedness)
    - Enhanced stress resilience
  timing: Morning, before cold exposure
  avoid: Before sleep, in water, while driving
```

### 8.8 Focus & Attention Protocol
```yaml
visual_focus_training:
  technique:
    - Pick fixed point
    - Focus 1-3 min (blinking OK, relaxed breathing)
    - Refocus when attention wanders
  frequency: Daily
  effect: Trains focus circuits, improves attention duration

closed_eye_meditation:
  technique:
    - Close eyes
    - Direct attention to point behind forehead
    - 8-13 min
  effect: Enhances cognitive focus (even for novices)

supplement_stack_for_focus:
  pre_work:
    - l_tyrosine: 500mg (30 min before)
    - alpha_gpc: 300mg
    - caffeine: 100-200mg (if past 90 min since waking)
  note: Use 4x/week max to avoid crash/tolerance
  alternative:
    - phenylethylamine: 500mg (shorter 30-45 min boost, less crash)

omega_3_for_attention:
  epa: 1-3g/day (mood, inflammation, focus)
  dha: 300mg/day (structural, memory)
  mechanism: EPA reduces inflammatory cytokines that inhibit serotonin/dopamine
```

### 8.9 Testosterone Optimization Protocol
```yaml
exercise:
  resistance_training:
    - Compound movements: squats, deadlifts, chin-ups
    - Protocol: 6 sets x 10 reps, 2 min rest between sets
    - Frequency: 2x/week max for T optimization
    - Order: Weights FIRST, then cardio (cardio first = decreased T)

lifestyle:
  sleep: 7-9 hours (T produced during deep sleep)
  light: Morning sunlight → dopamine → melanocytes → indirectly ↑T
  cold: Cold exposure increases T receptors
  apnea: Control sleep apnea (major T disruptor)
  light_at_night: Avoid (disrupts dopamine → T pathway)

supplements:
  tier_1_behavioral: # Do these first
    - sleep_optimization
    - morning_light
    - exercise_protocol
  tier_2_supplements:
    - tongkat_ali: 300-600mg (increased T in studies)
    - fadogia_agrestis: 300-600mg (cycle 8-12 weeks on, few weeks off)
    - zinc: 15-30mg (essential cofactor, often depleted)
    - boron: 3-6mg (may increase free T)
    - creatine: 5g/day (slight T benefit, proven for performance)

huberman_personal_result:
  before: 600 ng/dL
  after: ~800 ng/dL (tongkat ali + fadogia stack)
```

---

## 9. UI Concept

```
┌─────────────────────────────────────────────────────────────────┐
│  Timeline: [■■■■■■■■■■■■■■■░░░░░░░░░░░░░░░░░░░░░]  Day 1, 14:30 │
│            ↑ branch here                                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ⚡ Energy    ████████████████████  89/100                     │
│   🧠 Dopamine  ██████████████░░░░░░  0.72 (baseline)           │
│   ⚡ Alertness ████████████████░░░░  0.81 (norepinephrine)     │
│                                                                 │
│   🩸 Glucose   ████████░░░░░░░░░░░░  85 mg/dL                  │
│   🔥 Ketones   ██████████████░░░░░░  1.8 mmol/L  ← in ketosis  │
│   💓 HRV       ████████████░░░░░░░░  62 ms                     │
│                                                                 │
│   🧂 Sodium    ██████░░░░░░░░░░░░░░  LOW - need supplement     │
│   💧 Water     ████████████░░░░░░░░  OK                        │
│   🌡️ Temp      ████████████████░░░░  37.2°C                    │
│                                                                 │
│   Mode: [KETOGENIC] [FLOW] [AFTERNOON]                         │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│  [+ Add Activity]                                               │
│                                                                 │
│  Scheduled:                                                     │
│    15:00  HIIT workout (20 min)                                │
│    16:00  Keto meal (30g protein, 40g fat)                     │
│    18:00  Magnesium supplement                                  │
│    22:00  Begin sleep                                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 10. Implementation Phases

### Phase 1: Core Loop
- [ ] State types and tick function
- [ ] Basic metabolic (glucose, ketones)
- [ ] Event queue
- [ ] Simple CLI output

### Phase 2: Expand Subsystems
- [ ] Full metabolic (glycogen, insulin)
- [ ] Hydration + electrolytes
- [ ] Autonomic (sympathetic/para, HRV)
- [ ] Circadian rhythms

### Phase 3: Hormonal & Energy
- [ ] Dopamine (baseline + phasic), norepinephrine
- [ ] Fatigue model (adenosine, sleep debt)
- [ ] Thermoregulation
- [ ] Immune interactions

### Phase 4: Timeline & UI
- [ ] Branching/time-travel
- [ ] React UI with visualizations
- [ ] Activity scheduler
- [ ] Compare timelines

### Phase 5: Advanced
- [ ] Gamma function for individual calibration
- [ ] Long-term adaptations (keto adaptation, training effects)
- [ ] Presets/scenarios

---

## 11. Key Hypotheses & Models

### 11.1 Carbohydrate-Insulin Model of Obesity (What I've Learned)

**Core Thesis (Joseph Everett / WIL):**
```
Obesity trends began rising when dietary guidelines shifted to low-fat, high-carb recommendations.
We replaced steak with pasta, butter with margarine, eggs with cereal.
Result: We became fatter than ever despite eating "healthier."
```

**The Insulin Hypothesis:**
```
1. High carb intake → High insulin → Fat storage mode
2. Fat can only be released when insulin is low
3. Low-fat diets replaced fat with carbs → Chronically elevated insulin
4. Solution: Reduce carbs, allow insulin to fall, burn stored fat
```

**Intermittent Fasting Mechanism:**
```
Hours 0-10: Burning glucose from food
Hours 10-12: Glycogen stores depleting
Hours 12-16: Ketone production begins
Hours 16+: Fat burning in full effect

Key insight: Give body TIME to deplete glycogen and enter ketosis.
16-hour fasting window allows this transition daily.
```

**WIL Diet Approach:**
- ~90% low-carb over month
- ~70% keto (very low carb)
- ~80% of eating within 7-hour window
- Allows flexibility for social eating

### 11.2 Sugar Addiction Model

**Neurological Comparison:**
```
Sugar triggers same reward pathways as drugs of abuse:
- Dopamine release in nucleus accumbens
- Opioid release
- Binge patterns with intermittent access

Study: 94% of rats chose saccharin-sweetened water over IV cocaine
(even when cocaine dose increased)
```

**Sugar → Insulin → Dopamine Connection:**
```
Sugar spike → Insulin spike → Blood sugar crash
→ Brain seeks more sugar (dopamine hit)
→ Cycle repeats → Tolerance builds → Need more for same effect
```

### 11.3 Autophagy Window

**Timing for Cellular Cleanup:**
```
Hours 12-16: Ketones rising, autophagy beginning
Hours 24-48: Significant autophagy activation
Hours 36-72: Peak autophagy (cellular recycling)
72+: Levels off but remains elevated vs fed state
```

**Synergy with Ketosis:**
- Ketones signal nutrient scarcity
- Cells enter cleanup/repair mode
- Damaged proteins and organelles recycled
- Particularly beneficial for brain (neurodegeneration protection)

---

## 12. Sources & References

### Research
- Wang Study (Yale/Medzhitov Lab): Differential metabolic requirements for bacterial vs viral immunity
- Cold water immersion: 250% dopamine, 530% norepinephrine increase at 14°C
- Yoga Nidra PET study (2002): 65% dopamine increase
- Stanford Cyclic Sighing Study: 5 min daily physiological sighs reduce stress
- Sugar Addiction Study (2007): 94% of rats chose saccharin over cocaine
- CarbMetSim glucose model: https://github.com/mukulgoyalmke/CarbMetSim

### Huberman Lab Episodes & Resources
- [Controlling Your Dopamine](https://www.hubermanlab.com/episode/controlling-your-dopamine-for-motivation-focus-and-satisfaction)
- [Using Cortisol & Adrenaline](https://www.hubermanlab.com/episode/using-cortisol-and-adrenaline-to-boost-our-energy-and-immune-system)
- [Optimize Brain Chemistry](https://www.hubermanlab.com/episode/optimize-and-control-your-brain-chemistry-to-improve-health-and-performance)
- [Sleep Toolkit](https://www.hubermanlab.com/episode/sleep-toolkit-tools-for-optimizing-sleep-and-sleep-wake-timing)
- [Using Caffeine to Optimize Performance](https://www.hubermanlab.com/episode/using-caffeine-to-optimize-mental-and-physical-performance)
- [Cold Exposure for Health & Performance](https://www.hubermanlab.com/episode/using-deliberate-cold-exposure-for-health-and-performance)
- [ADHD & Focus](https://www.hubermanlab.com/episode/adhd-and-how-anyone-can-improve-their-focus)
- [Optimize Testosterone & Estrogen](https://www.hubermanlab.com/episode/the-science-of-how-to-optimize-testosterone-and-estrogen)
- [Breathing for Health & Performance](https://www.hubermanlab.com/episode/how-to-breathe-correctly-for-optimal-health-mood-learning-and-performance)
- [Light for Health](https://www.hubermanlab.com/newsletter/using-light-for-health)
- [NSDR Portal](https://www.hubermanlab.com/nsdr)
- Guest Series: Dr. Matt Walker on Sleep Protocols
- Guest Series: Dr. Susanna Søberg on Cold & Heat Exposure

### What I've Learned (Joseph Everett)
- YouTube Channel: https://www.youtube.com/@WhatIveLearned
- Substack: https://substack.com/@josepheverettwil
- Key topics: Carbohydrate-insulin model, keto, intermittent fasting, saturated fat/cholesterol myths
- Personal protocol: ~90% low-carb, ~70% keto, 7-hour eating window

### Personal Research (Gemini Conversations)
- Hydration and Dehydration Effects (ADHD model, gamma function, flow states)
- Why We Yawn: Science Explains (thermoregulation, state transitions)
- Cold/Flu Recovery (metabolic strategy for illness)

### Electrolyte Guidelines (Keto)
- Sodium: 4-5g/day (vs 2g normally)
- Potassium: 1-3g/day
- Magnesium: 300-400mg/day
- Zinc: 15-30mg/day
