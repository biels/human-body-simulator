# Human Body Simulator - Systems Reference

Complete list of all systems, subsystems, variables, and interactions to simulate.

---

# PART A: FOUNDATIONAL HOMEOSTATIC SYSTEMS ("The Engine Block")

These systems provide the **machinery** that drives all other effects. Without these, variables are static rather than dynamically regulated.

---

## 0A. RENAL SYSTEM (Kidneys)

The machine that regulates water and electrolytes. Critical for Keto simulation (the "Whoosh" effect).

### Variables
| Variable | Unit | Normal Range | Notes |
|----------|------|--------------|-------|
| `GFR` | mL/min | 90-120 | Glomerular filtration rate |
| `urineProduction` | mL/hour | 30-80 | Output rate |
| `urineConcentration` | mOsm/kg | 300-1200 | Dilute vs concentrated |
| `aldosterone` | normalized 0-1 | - | Controls sodium RETENTION |
| `ADH` (vasopressin) | normalized 0-1 | - | Controls water RETENTION |
| `renin` | normalized 0-1 | - | Upstream of aldosterone |
| `angiotensinII` | normalized 0-1 | - | Vasoconstriction + aldosterone |

### The RAAS Cascade (Renin-Angiotensin-Aldosterone System)
```
Low blood pressure / Low sodium detected by kidneys
    ↓
Kidneys release RENIN
    ↓
Renin converts Angiotensinogen → Angiotensin I
    ↓
ACE (in lungs) converts Angiotensin I → Angiotensin II
    ↓
Angiotensin II:
├── Vasoconstriction → ↑bloodPressure
├── Stimulates ALDOSTERONE release (adrenal glands)
│   └── Aldosterone → Kidneys retain Na+ (and water follows)
└── Stimulates ADH release (pituitary)
    └── ADH → Kidneys retain water directly
```

### Keto "Whoosh" Effect Mechanism
```
Carb restriction
    ↓
↓ Insulin
    ↓
↓ Aldosterone (insulin normally stimulates aldosterone)
    ↓
↓ Sodium retention → Kidneys DUMP sodium
    ↓
Water follows sodium → Rapid water weight loss
    ↓
↓ Blood volume → ↑Heart rate (compensation)
    ↓
If not supplementing: "Keto Flu" (headache, fatigue, cramps)
```

### Homeostatic Loops
| Trigger | Response | Mechanism |
|---------|----------|-----------|
| ↑Osmolality (dehydrated) | ↑ADH | Retain water, concentrate urine |
| ↓Osmolality (overhydrated) | ↓ADH | Excrete water, dilute urine |
| ↓Blood pressure | ↑Renin→↑Aldosterone | Retain Na+, ↑blood volume |
| ↑Blood pressure | ↓Renin | Excrete Na+, ↓blood volume |
| ↓Insulin (keto/fasting) | ↓Aldosterone | Excrete Na+ (must supplement!) |
| ↑Potassium | ↑Aldosterone | Excrete K+ (protective) |

---

## 0B. RESPIRATORY & pH SYSTEM (The Exhaust)

Controls CO2 elimination, oxygen delivery, and blood pH. **Critical distinction: Ketosis (good) vs Ketoacidosis (deadly) is pH.**

### Variables
| Variable | Unit | Normal Range | Notes |
|----------|------|--------------|-------|
| `bloodpH` | pH units | 7.35-7.45 | TIGHT regulation, death outside 6.8-7.8 |
| `pCO2` | mmHg | 35-45 | Partial pressure of CO2 |
| `pO2` | mmHg | 80-100 | Partial pressure of O2 |
| `O2saturation` | % | 95-100 | Hemoglobin oxygen binding |
| `bicarbonate` | mEq/L | 22-26 | Buffer system |
| `respiratoryRate` | breaths/min | 12-20 | Primary CO2 control |
| `tidalVolume` | mL | 500 | Air per breath |
| `CO2tolerance` | normalized 0-1 | - | Anxiety threshold |

### Blood pH Buffer System
```
CO2 + H2O ⇌ H2CO3 ⇌ H+ + HCO3-
(carbon dioxide) (carbonic acid) (hydrogen ion + bicarbonate)

To LOWER pH (more acidic): ↑CO2 or ↑metabolic acids
To RAISE pH (more alkaline): ↓CO2 (breathe faster) or ↑bicarbonate
```

### Respiratory Control of pH
| State | pH | pCO2 | Respiratory Response |
|-------|----|----|---------------------|
| Normal | 7.40 | 40 | Normal rate |
| Respiratory Acidosis | <7.35 | >45 | Hyperventilate to blow off CO2 |
| Respiratory Alkalosis | >7.45 | <35 | Slow breathing (or it self-corrects) |
| Metabolic Acidosis | <7.35 | <35 | Hyperventilate to compensate (Kussmaul) |
| Metabolic Alkalosis | >7.45 | >45 | Hypoventilate to compensate |

### Ketosis vs Ketoacidosis
| State | Ketones | pH | Insulin | Danger |
|-------|---------|-----|---------|--------|
| Nutritional Ketosis | 0.5-3.0 mmol/L | 7.35-7.45 | Low but present | None |
| Starvation Ketosis | 3-5 mmol/L | 7.30-7.40 | Very low | Minimal |
| Diabetic Ketoacidosis | >10 mmol/L | <7.30 | ABSENT | Life-threatening |

**Key insight:** Even trace insulin prevents runaway ketone production. Type 1 diabetics have NO insulin → ketones spiral → acidosis.

### Breathing Techniques & pH
| Technique | Effect on CO2 | Effect on pH | Result |
|-----------|---------------|--------------|--------|
| Physiological Sigh | ↓↓CO2 quickly | ↑pH (alkaline shift) | Vagal activation, calm |
| Box Breathing | Balanced | Stable | Equilibrium |
| Wim Hof (hyperventilation) | ↓↓↓CO2 | ↑↑pH (alkalosis) | Tingling, adrenaline, cold tolerance |
| Breath holds | ↑CO2 | ↓pH | CO2 tolerance training |

### CO2 Tolerance & Anxiety
```
Low CO2 tolerance:
- Small ↑CO2 → Panic, need to breathe
- Associated with anxiety disorders
- Treated with breath hold training

High CO2 tolerance:
- Can hold breath longer
- Less reactive to stress
- "Cool under pressure"
```

---

## 0C. BODY COMPOSITION (The Fuel Tanks)

Actual mass storage - where energy comes from and goes to. Required for conservation of mass/energy.

### Variables
| Variable | Unit | Notes |
|----------|------|-------|
| `adiposeTissueMass` | kg | Total fat storage (subcutaneous + visceral) |
| `visceralFat` | kg | Inflammatory, metabolically active |
| `subcutaneousFat` | kg | Less harmful storage |
| `leanMuscleMass` | kg | Metabolically active tissue |
| `glycogenMass` | kg | ~0.5 kg max (liver + muscle) |
| `bodyWeight` | kg | Sum of all compartments |
| `BMR` | kcal/day | Basal metabolic rate (scales with lean mass) |
| `TDEE` | kcal/day | Total daily energy expenditure |

### Energy Storage Capacities
| Compartment | Capacity | Energy Density | Total Energy |
|-------------|----------|----------------|--------------|
| Blood glucose | ~5g | 4 kcal/g | ~20 kcal |
| Liver glycogen | ~100g | 4 kcal/g | ~400 kcal |
| Muscle glycogen | ~400g | 4 kcal/g | ~1,600 kcal |
| Adipose tissue | ~15kg (avg) | 9 kcal/g | ~135,000 kcal |
| Muscle protein | ~10kg | 4 kcal/g | ~40,000 kcal (emergency only) |

### Energy Balance
```typescript
// Per tick energy accounting
energyIn = caloriesConsumed;
energyOut = BMR + activityExpenditure + thermogenesis;
energyBalance = energyIn - energyOut;

if (energyBalance > 0) {
  // Surplus: Store energy
  if (insulinHigh) {
    glycogenMass += min(surplus, glycogenCapacity - glycogenMass);
    adiposeTissueMass += remainder;
  }
}

if (energyBalance < 0) {
  // Deficit: Mobilize energy
  if (glycogenMass > 0) {
    glycogenMass -= deficit; // Use glycogen first
  } else if (ketonesAvailable) {
    adiposeTissueMass -= deficit / 9; // Burn fat
  } else {
    leanMuscleMass -= deficit / 4; // Muscle catabolism (bad!)
  }
}
```

### Body Composition Effects
| More Muscle | Effect |
|-------------|--------|
| ↑leanMuscleMass | ↑BMR, ↑insulin sensitivity, ↑glycogen capacity |
| ↑adiposeTissueMass | ↑leptin, ↓insulin sensitivity, ↑inflammation |
| ↑visceralFat | ↑↑inflammation, ↑cortisol, ↑disease risk |

---

## 0D. VISUAL & VESTIBULAR SYSTEM (Huberman's Primary Lever)

The sensory gateway to autonomic state. Focal vs panoramic vision is one of Huberman's core tools.

### Variables
| Variable | Unit | Notes |
|----------|------|-------|
| `visualMode` | enum | focal/panoramic |
| `gazeTarget` | enum | near/mid/far/horizon |
| `opticFlow` | 0-1 | Forward motion perception |
| `lightInput` | lux | Current light exposure |
| `vestibularActivation` | 0-1 | Balance system engagement |

### Visual Mode Effects
| Mode | Description | Effect on ANS | Use Case |
|------|-------------|---------------|----------|
| **Focal** | Narrow attention, near object | ↑Sympathetic, ↑alertness | Reading, phone, hunting |
| **Panoramic** | Wide peripheral awareness | ↑Parasympathetic, ↓stress | Horizon gazing, relaxation |

### Optic Flow
```
Forward movement (walking, running, driving):
- Visual field flows past periphery
- Quiets the Amygdala
- ↓Anxiety, ↓rumination
- Why: Evolutionarily, if you're moving forward safely, threats are behind you

Static visual input:
- No flow = scanning for threats
- ↑Amygdala activation
- ↑Vigilance, potential anxiety
```

### Gaze & Alertness
| Gaze Direction | Effect |
|----------------|--------|
| Upward | ↑Alertness (why we look up when thinking) |
| Downward | ↓Alertness, ↑sleepiness |
| Horizon | Neutral, calming |

### Transitions
- Screen work (focal, near) → ↑sympathetic, ↑eye strain, ↑cortisol
- Horizon gazing (panoramic, far) → ↑parasympathetic, ↓stress
- Walking outside → optic flow + panoramic → ↓↓anxiety
- Closed eyes (NSDR) → removes visual input → enables deep rest

---

# PART B: METABOLIC & ENERGY SYSTEMS

---

## 1. METABOLIC SYSTEM

### Variables
| Variable | Unit | Normal Range | Notes |
|----------|------|--------------|-------|
| `bloodGlucose` | mg/dL | 70-100 (keto: 65-85) | Primary fuel indicator |
| `bloodKetones` | mmol/L | 0-0.5 (keto: 0.5-3.0) | Alternative fuel |
| `liverGlycogen` | grams | 0-100 | Depletes in 10-12h fasting |
| `muscleGlycogen` | grams | 0-400 | Local muscle fuel |
| `insulin` | μU/mL | 2-25 fasting, 50-100+ post-meal | Master metabolic switch |
| `insulinSensitivity` | 0-1 | Higher = better | Degrades with chronic high carb |
| `glucagon` | normalized 0-1 | - | Counter-regulatory to insulin |
| `freefattyAcids` | mmol/L | 0.1-0.7 | Released when insulin low |
| `metabolicMode` | enum | glycolytic/ketogenic/hybrid | Current fuel preference |
| `fatAdaptation` | 0-1 | 0 = none, 1 = full | Takes 2-4 weeks to build |
| `lactate` | mmol/L | 0.5-2.0 rest, up to 20+ max effort | Bridge fuel, neuroprotective |

### Lactate System
```
Lactate is NOT a waste product - it's a valuable fuel!

Production:
- High-intensity exercise → glycolysis → pyruvate → lactate
- Can be used by: Heart, brain, slow-twitch muscle

Lactate Shuttle:
Fast-twitch muscle (produces) → Blood → Slow-twitch/Heart/Brain (consumes)

Benefits:
├── Neuroprotective (brain fuel during hypoglycemia)
├── Signals for mitochondrial biogenesis
├── Precursor for gluconeogenesis (liver)
└── Indicates training intensity (lactate threshold)

Lactate Threshold: ~4 mmol/L (above = unsustainable intensity)
```

### Transitions
- High carb → ↑glucose → ↑insulin → ↓ketones → glycolytic mode
- Fasting 12-16h → ↓glucose → ↓insulin → ↑ketones → ketogenic mode
- Exercise → ↓glycogen → ↑glucagon
- High intensity exercise → ↑↑lactate → fuel for heart/brain
- Chronic high carb → ↓insulinSensitivity
- Weeks on keto → ↑fatAdaptation

### Key Mechanisms
- **Insulin high**: Anabolic (store glucose, block fat burning, block ketones)
- **Insulin low**: Catabolic (burn fat, produce ketones, gluconeogenesis)
- **Glycogen depletion**: Triggers ketone production (liver)
- **Fat adaptation**: Mitochondria upregulate fat oxidation enzymes

---

## 2. AUTONOMIC NERVOUS SYSTEM

### Variables
| Variable | Unit | Normal Range | Notes |
|----------|------|--------------|-------|
| `sympathetic` | 0-1 | - | Fight/flight activation |
| `parasympathetic` | 0-1 | - | Rest/digest activation |
| `hrv` | ms (RMSSD) | 20-100 | Higher = better recovery |
| `cortisol` | normalized 0-1 | Peaks 30min after waking | Follows circadian rhythm |

### Transitions
- HIIT/stress → ↑sympathetic → ↓HRV → ↑cortisol
- Rest/meditation → ↑parasympathetic → ↑HRV
- Cold exposure → acute ↑sympathetic → rebound ↑parasympathetic
- Chronic stress → sustained ↑cortisol → ↓HRV baseline
- Sleep → ↓sympathetic → ↑parasympathetic

### Key Mechanisms
- **HRV Calculation**: `HRV = baselineHRV * parasympathetic / (sympathetic + 0.1)`
- **Sympathetic/Parasympathetic balance**: Usually reciprocal but can co-activate

---

## 3. HYDRATION & ELECTROLYTE SYSTEM

### Variables
| Variable | Unit | Normal Range | Notes |
|----------|------|--------------|-------|
| `totalBodyWater` | liters | ~42L (70kg male) | 60% body weight |
| `intracellularWater` | liters | ~28L | 2/3 of total |
| `extracellularWater` | liters | ~14L | 1/3 of total |
| `sodium` | mEq/L | 136-145 | Critical for nerve/muscle |
| `potassium` | mEq/L | 3.5-5.0 | Heart rhythm, muscle |
| `magnesium` | mg/dL | 1.7-2.2 | 300+ enzyme reactions |
| `zinc` | μg/dL | 80-120 | Testosterone, immune |
| `osmolality` | mOsm/kg | 275-295 | Fluid balance indicator |

### Transitions
- Sweating → ↓water, ↓sodium, ↓potassium, ↓magnesium
- Keto/low insulin → ↑sodium excretion (natriuresis)
- Glycogen depletion → water loss (3g water per 1g glycogen)
- Ejaculation → ↓zinc (1-5mg per emission)
- Dehydration → ↑osmolality → thirst signal

### Key Mechanisms
- **Keto electrolyte needs**: 4-5g sodium, 1-3g potassium, 300-400mg magnesium, 15-30mg zinc
- **Water follows sodium**: Sodium retention = water retention
- **Magnesium timing**: Evening (promotes sleep via GABA)

---

## 4. HORMONAL SYSTEM

### 4.1 Catecholamine Cascade
| Variable | Unit | Notes |
|----------|------|-------|
| `dopamine.baseline` | 0-1 | Tonic level (motivation threshold) |
| `dopamine.phasic` | 0-1 | Acute spikes (reward response) |
| `norepinephrine` | 0-1 | Alertness, "frame rate" |
| `adrenaline` | 0-1 | Acute stress response |

**Synthesis pathway**: L-Tyrosine → L-DOPA → Dopamine → Norepinephrine → Adrenaline

### 4.2 Wakefulness System
| Variable | Unit | Notes |
|----------|------|-------|
| `orexin` | 0-1 | Wakefulness, appetite, motivation |

**Orexin triggers**:
- ↑ Light exposure, fasting, exercise
- ↓ High carb meal, sleep debt

### 4.3 Sex Hormones & Retention
| Variable | Unit | Normal Range | Notes |
|----------|------|--------------|-------|
| `testosterone` | ng/dL | 300-1000 (male) | Peaks day 7 of retention |
| `DHT` | normalized | - | Potent androgen |
| `prolactin` | normalized | - | Rises post-ejaculation |
| `retentionDays` | days | 0+ | Days since ejaculation |

**Retention timeline**:
- Day 0: Prolactin spike, dopamine crash
- Days 1-3: Recovery
- Days 4-6: T rising, receptors sensitizing
- Day 7: T peak (~145% baseline)
- Days 7-30+: Stable elevated, transmutation phase

### 4.4 Mood & Sleep Hormones
| Variable | Unit | Notes |
|----------|------|-------|
| `serotonin` | 0-1 | Mood, satiety |
| `melatonin` | 0-1 | Sleep pressure (rises 2h before sleep) |

### 4.5 Growth Factors
| Variable | Unit | Notes |
|----------|------|-------|
| `BDNF` | normalized | Brain plasticity (↑ with exercise, fasting) |
| `IGF1` | ng/mL | Growth/recovery |
| `growthHormone` | ng/mL | Peaks during deep sleep, fasting 16h+ |

### Transitions
- HIIT → ↑testosterone, ↑adrenaline, ↑BDNF, ↑norepinephrine
- Fasting 16h+ → ↑norepinephrine, ↑BDNF, ↑growthHormone
- Cold exposure → ↑norepinephrine (+530%), ↑dopamine (+250%)
- Sleep deprivation → ↓testosterone, ↓dopamine sensitivity, ↓orexin
- Ejaculation → ↑prolactin, ↓dopamine, reset retentionDays
- NSDR → ↑dopamine (+65% restoration)

---

## 5. ENERGY & FATIGUE SYSTEM

### Variables
| Variable | Unit | Notes |
|----------|------|-------|
| `ATP` | 0-1 | Cellular energy availability |
| `perceivedEnergy` | 0-100 | Subjective energy level |
| `sleepDebt` | hours | Accumulated sleep deficit |
| `adenosine` | 0-1 | Sleep pressure molecule |
| `muscleFatigue` | 0-1 | Per muscle group |
| `centralFatigue` | 0-1 | CNS fatigue (limits max effort) |
| `adenosineReceptors.available` | 0-1 | Free receptors (not blocked) |
| `adenosineReceptors.blocked` | 0-1 | Blocked by caffeine |
| `adenosineBuildup` | normalized | Adenosine waiting behind caffeine block |

### Adenosine Receptor Dynamics (Caffeine Crash Mechanism)
```
Normal state:
- Adenosine binds to available receptors → Sleepiness

Caffeine consumed:
- Caffeine blocks receptors (adenosineReceptors.blocked ↑)
- Adenosine can't bind → No sleepiness signal
- BUT adenosine keeps accumulating (adenosineBuildup ↑)

Caffeine wears off (half-life ~5-6 hours):
- Receptors become available again
- ALL the built-up adenosine floods in at once
- Result: "Caffeine Crash" - worse than baseline tiredness
```

### Transitions
- Waking → adenosine accumulates over day
- Sleep → adenosine cleared, receptors reset
- Caffeine → blocks receptors, adenosine builds up behind
- Caffeine wears off → adenosineBuildup floods available receptors → crash
- Morning cortisol → clears residual adenosine (why delay caffeine 90min)
- Exercise → ↑muscleFatigue, ↑centralFatigue
- Rest → recovery of fatigue
- Sleep debt → ↓perceivedEnergy, ↑adenosine baseline

### Key Mechanisms
- **Adenosine half-life**: ~6 hours
- **Caffeine**: Blocks receptors, adenosine builds behind block
- **Sleep clears adenosine**: Deep sleep most effective

---

## 6. CIRCADIAN SYSTEM

### Variables
| Variable | Unit | Notes |
|----------|------|-------|
| `timeOfDay` | 0-1440 | Minutes since midnight |
| `circadianPhase` | enum | early_morning/morning/afternoon/evening/night/late_night |
| `lightExposure` | lux accumulated | Today's exposure |
| `melatonin` | 0-1 | Rises ~2h before sleep |
| `coreBodyTemp` | °C | Nadir ~4am, peak ~6pm |
| `cortisolRhythm` | multiplier | Peak at wake (+50%), low at night |

### Phase Definitions
| Time | Phase | Cortisol | Melatonin | Temp | Optimal For |
|------|-------|----------|-----------|------|-------------|
| 04:00-06:00 | late_night | Rising | High→Low | Nadir (36.0°C) | Deep sleep |
| 06:00-09:00 | early_morning | Peak | Low | Rising | Light exposure |
| 09:00-12:00 | morning | High | Low | Rising | Focus work |
| 12:00-15:00 | afternoon | Declining | Low | Peak | Physical performance |
| 15:00-18:00 | late_afternoon | Low-mod | Low | Peak | Strength peak |
| 18:00-21:00 | evening | Low | Rising | Declining | Wind down |
| 21:00-00:00 | night | Minimal | High | Declining | Sleep onset |
| 00:00-04:00 | deep_night | Minimal | High | Lowest | Deep sleep/repair |

### Transitions
- Morning light (100k lux) → ↑cortisol spike (+50%), sets sleep timer for +16h
- Evening blue light → delays melatonin onset
- Darkness → enables melatonin rise
- 2 days proper light → resets drifted rhythm

---

## 7. THERMOREGULATION SYSTEM

### Variables
| Variable | Unit | Notes |
|----------|------|-------|
| `coreTemp` | °C | Normal 36.5-37.5, follows circadian |
| `skinTemp` | °C | Varies by location |
| `sweating` | 0-1 | Activation level |
| `shivering` | 0-1 | Activation level |
| `vasodilation` | 0-1 | Heat dissipation |
| `vasoconstriction` | 0-1 | Heat conservation |
| `yawning` | 0-1 | Brain cooling mechanism |
| `BAT.activation` | 0-1 | Brown adipose tissue thermogenesis |
| `BAT.mass` | grams | Amount of brown fat (trainable) |

### Brown Adipose Tissue (BAT)
```
BAT is "calorie-burning" fat (vs white fat which stores calories)

Location: Neck, upper back, clavicle area

Mechanism:
- Contains many mitochondria
- Burns calories directly to produce HEAT (non-shivering thermogenesis)
- Activated by cold exposure and norepinephrine

Cold Exposure → Norepinephrine → BAT activation → Heat + Calorie burn

Trainable:
- Regular cold exposure → ↑BAT mass over weeks
- More BAT = better cold tolerance + higher metabolism
```

### Transitions
- Core temp > 37.5°C → ↑sweating, ↑vasodilation
- Core temp < 36°C → ↑shivering, ↑vasoconstriction
- Cold exposure → ↑norepinephrine → ↑BAT.activation → heat production
- Chronic cold exposure → ↑BAT.mass (adaptation)
- Brain temp ↑ → ↑yawning
- Cold exposure → vasoconstriction → then vasodilation rebound
- Pre-sleep → need temp drop 1-3°C

### Key Mechanisms
- **Yawning**: Brain cooling + state transition marker
- **Sweating**: Evaporative cooling, loses electrolytes
- **Sleep requirement**: Body temp must drop 1-3°C to initiate
- **BAT thermogenesis**: Non-shivering heat + metabolic boost

---

## 8. IMMUNE SYSTEM

### Variables
| Variable | Unit | Notes |
|----------|------|-------|
| `status` | enum | healthy/fighting_bacterial/fighting_viral/inflamed/recovering |
| `inflammationLevel` | 0-1 | General inflammation |
| `whiteBloodCells` | normalized | Immune cell count |
| `cytokines.proInflammatory` | normalized | IL-6, TNF-alpha |
| `cytokines.antiInflammatory` | normalized | IL-10 |
| `feverResponse` | 0-1 | Intentional temp elevation |

### Transitions
- Infection detected → ↑whiteBloodCells, ↑cytokines.proInflammatory
- Chronic stress (high cortisol) → ↓whiteBloodCells
- Recovery → ↑cytokines.antiInflammatory
- Fever → ↑coreTemp (intentional)

### Key Mechanisms (Wang Study)
| Infection | Optimal Strategy | Why |
|-----------|------------------|-----|
| Bacterial | Fasting/Ketosis | Glucose restriction starves bacteria |
| Viral | Glucose intake | Glucose protects neurons from inflammation |
| Post-illness | Hybrid (50-70g carbs) | Support recovery |

---

## 9. ATTENTION & COGNITIVE SYSTEM

### Variables
| Variable | Unit | Notes |
|----------|------|-------|
| `gamma` | 1-5 | Individual's motivation curve steepness |
| `flowState` | boolean | In flow or not |
| `activationEnergy` | normalized | Current threshold to start tasks |
| `focusDuration` | minutes | Current sustained attention capacity |

### 9.1 Gamma Function (ADHD Model)
```
motivation = input^gamma

gamma ≈ 1.0: Neurotypical (linear response)
gamma > 2.5: High-gamma brain (low signals crushed to zero)
```

High-gamma individuals need "Level 10" challenges to activate.

### 9.2 Flow State Detection
```
                HIGH CHALLENGE
                      │
          [Anxiety]   │   [FLOW]
                      │   (High DA + High NE + Low cortisol + skill match)
LOW SKILL ────────────┼──────────── HIGH SKILL
                      │
         [Apathy]     │   [Boredom]
                      │
                LOW CHALLENGE
```

### 9.3 Activation Energy
```
activationEnergy = baseTaskDifficulty * (1 / dopamine.baseline) * gamma
```

### Transitions
- ↑dopamine.baseline → ↓activationEnergy (easier to start)
- Challenge matches skill → flowState = true
- Sustained focus training → ↑focusDuration capacity

---

## 10. PERCEPTUAL SYSTEM

### Variables
| Variable | Unit | Notes |
|----------|------|-------|
| `frameRate` | fps equivalent | Perceptual sampling rate |
| `quantizationMode` | enum | binary/gradient | 1-bit vs smooth states |

### 10.1 Tachypsychia (Frame Rate Model)
| State | Norepinephrine | Frame Rate | Experience |
|-------|----------------|------------|------------|
| Low alert | Low | ~30 fps | Normal time |
| High alert | High | ~60-120 fps | Time slower |
| Peak/Flow | Very high | ~120+ fps | "Slow motion" |

### 10.2 Quantization Mode
Some individuals operate binary (ON/OFF) rather than gradient:

| State | Description | Neurochemistry |
|-------|-------------|----------------|
| OFF | Asleep, bored, paralyzed | Low DA, low NE |
| ON | "God mode", flow, "beast mode" | High DA, high NE, low cortisol |

---

## 11. ULTRADIAN RHYTHM SYSTEM

### Variables
| Variable | Unit | Notes |
|----------|------|-------|
| `ultradianPhase` | 0-90 | Minutes into current 90-min cycle |
| `cycleState` | enum | peak/plateau/trough/recovery |
| `focusCapacity` | 0-1 | Current cognitive capacity in cycle |
| `restNeed` | 0-1 | Accumulated need for micro-recovery |

### The Basic Rest-Activity Cycle (BRAC)
Discovered by Nathaniel Kleitman (1950s): 90-120 minute cycles during both sleep and wakefulness.

| Phase | Minutes | Brain State | Optimal For |
|-------|---------|-------------|-------------|
| **Peak** | 0-20 | High alertness, fast waves | Complex problem-solving |
| **Plateau** | 20-50 | Sustained focus | Deep work, flow state |
| **Trough** | 50-70 | Slowing waves, dreamy | Creative insight, breaks |
| **Recovery** | 70-90 | Transition, preparing | Light tasks, rest |

### Cycle Characteristics
- **During sleep**: Corresponds to REM/NREM cycles
- **During wake**: Corresponds to alertness/fatigue oscillations
- **Cycle length**: ~60 min in infants → ~90 min in adults → longer with age
- **Hormonal sync**: Cortisol pulses align with ultradian alertness

### Neurological Control
- Noradrenergic (LC) system: Active during wake portions
- Cholinergic system: Active during REM portions
- Systems alternate reciprocally

### Transitions
- Peak phase → ↑focusCapacity, ↑norepinephrine
- Trough phase → ↓focusCapacity, ↑creativity, ↑restNeed
- Ignoring rest signals → ↑stress, ↓next cycle performance
- Honoring cycles → sustained productivity

---

## 12. HUNGER & SATIETY SYSTEM

### Variables
| Variable | Unit | Normal Range | Notes |
|----------|------|--------------|-------|
| `ghrelin` | normalized 0-1 | - | "Hunger hormone" from stomach |
| `leptin` | normalized 0-1 | - | "Satiety hormone" from fat cells |
| `hunger` | 0-100 | - | Perceived hunger signal |
| `satiety` | 0-100 | - | Perceived fullness |
| `leptinSensitivity` | 0-1 | - | Can develop resistance |

### Ghrelin (Short-term hunger)
```
Functions:
├── Signals brain to eat (meal initiation)
├── Rises before meals (anticipatory)
├── Falls after meals
├── Triggers growth hormone release
├── Increases food-seeking behavior
└── Stimulates reward pathways (dopamine)
```

### Leptin (Long-term energy status)
```
Functions:
├── Signals energy sufficiency (fat stores)
├── Suppresses appetite when adequate
├── Proportional to fat mass
├── Regulates metabolic rate
└── Chronic high = resistance develops
```

### Transitions
- Empty stomach → ↑ghrelin → ↑hunger
- Meal consumed → ↓ghrelin, ↑satiety (20 min delay)
- Protein/fiber → stronger satiety signal
- High carb → fast satiety → fast hunger return
- Poor sleep → ↑ghrelin (+28%), ↓leptin (-18%)
- Chronic stress → ↑ghrelin, ↑cortisol → emotional eating
- Obesity → ↑leptin but ↓leptinSensitivity (resistance)
- Fasting → ↑ghrelin (hours), then ↓ as ketones rise

### Cross-system Effects
- Ghrelin → ↑dopamine (reward anticipation)
- Leptin resistance → metabolic dysfunction
- Sleep debt → hunger hormone dysregulation
- Orexin ↑ during fasting → hunger + alertness

---

## 13. ANABOLIC / MUSCLE PROTEIN SYNTHESIS SYSTEM

### Variables
| Variable | Unit | Notes |
|----------|------|-------|
| `mTOR` | 0-1 | Anabolic signaling pathway activation |
| `MPS` | normalized | Muscle protein synthesis rate |
| `MPB` | normalized | Muscle protein breakdown rate |
| `netProteinBalance` | signed | MPS - MPB |
| `anabolicWindow` | boolean | Currently in elevated MPS state |
| `leucineLevel` | normalized | Key amino acid for mTOR |

### mTOR Signaling (Master Anabolic Switch)
```
Activators:
├── Resistance exercise
├── Leucine (amino acid)
├── Insulin
├── Growth factors (IGF-1)
└── Mechanical tension

Inhibitors:
├── AMPK (energy deficit sensor)
├── Fasting (prolonged)
├── Endurance exercise (acute)
└── Caloric restriction
```

### The Anabolic Window
| Time Post-Exercise | mTOR Activity | MPS Rate | Notes |
|--------------------|---------------|----------|-------|
| 0-1h | Rising | Increasing | Pathway activating |
| 1-2h | Peak | Elevated | Optimal protein timing |
| 3-5h | Declining | Peak | MPS peaks after mTOR |
| 5-24h | Baseline | Elevated | Sustained synthesis |
| 24h+ | Baseline | Baseline | Returns to normal |

### Protein Synthesis Triggers
- Resistance exercise → ↑mTOR for 24h
- Leucine (2-3g) → ↑mTOR activation
- 20-40g protein → ↑MPS
- Whey protein → faster absorption → faster mTOR

### Transitions
- Resistance exercise → ↑mTOR → ↑MPS (3-5h delay)
- Fasting → ↓mTOR → ↓MPS, ↑autophagy
- Protein intake → ↑leucine → ↑mTOR
- Cold exposure post-workout → blunts mTOR (avoid within 6h)
- Sleep → ↑GH → supports MPS
- Endurance + resistance same session → AMPK competes with mTOR

---

## 14. ALLOSTATIC LOAD SYSTEM

### Variables
| Variable | Unit | Notes |
|----------|------|-------|
| `allostaticLoad` | 0-100 | Cumulative stress burden |
| `adaptiveCapacity` | 0-1 | Ability to handle stress |
| `recoveryDebt` | normalized | Accumulated recovery deficit |
| `systemWear` | per-system | Degradation per organ system |

### What is Allostatic Load?
"Wear and tear on the body from repeated activation of stress response systems."
- **Allostasis**: Stability through change (healthy adaptation)
- **Allostatic load**: Cost of maintaining allostasis
- **Allostatic overload**: When burden exceeds capacity → disease

### Contributing Factors
```
Increases allostatic load:
├── Chronic psychological stress
├── Repeated acute stressors
├── Inadequate recovery
├── Sleep deprivation
├── Poor nutrition
├── Social isolation
├── Environmental toxins
└── Lifestyle factors (smoking, sedentary)

Decreases allostatic load:
├── Adequate sleep
├── Social support
├── Exercise (moderate)
├── Meditation/stress management
├── Proper nutrition
└── Recovery practices
```

### Biomarkers (Measurable Indicators)
| System | Markers |
|--------|---------|
| Neuroendocrine | Cortisol, DHEA, epinephrine, norepinephrine |
| Cardiovascular | Blood pressure, heart rate |
| Metabolic | HbA1c, cholesterol, BMI, waist-hip ratio |
| Immune | C-reactive protein (CRP), IL-6 |

### Transitions
- Chronic high cortisol → ↑allostaticLoad
- Inadequate sleep → ↑recoveryDebt → ↑allostaticLoad
- Good recovery practices → ↓allostaticLoad
- Sustained overload → ↓adaptiveCapacity → accelerated aging
- Childhood adversity → higher baseline allostaticLoad

### Long-term Consequences
- ↑ Cardiovascular disease risk
- ↑ Metabolic syndrome
- ↑ Cognitive decline
- ↑ Immune dysfunction
- ↓ Longevity
- Accelerated biological aging

---

## 15. GUT-BRAIN AXIS / MICROBIOME SYSTEM

### Variables
| Variable | Unit | Notes |
|----------|------|-------|
| `microbiomeDiversity` | 0-1 | Species diversity (higher = healthier) |
| `microbiomeBalance` | signed | Beneficial vs harmful ratio |
| `gutPermeability` | 0-1 | "Leaky gut" indicator |
| `gutSerotonin` | normalized | 90% of body's serotonin made here |
| `vagalTone` | 0-1 | Gut-brain communication strength |

### Key Functions
```
Microbiome produces:
├── Serotonin (~90% of body total)
├── GABA
├── Dopamine (some)
├── Noradrenaline (some)
├── Short-chain fatty acids (SCFAs)
└── Vitamins (B, K)
```

### Communication Pathways
| Pathway | Direction | Mechanism |
|---------|-----------|-----------|
| Vagus nerve | Bidirectional | Neural signaling |
| Immune | Gut → Brain | Cytokines |
| Endocrine | Gut → Brain | Hormones, neurotransmitters |
| Metabolic | Gut → Brain | SCFAs, metabolites |

### Microbiome Modulators
```
Improves microbiome:
├── Fiber (prebiotics)
├── Fermented foods (probiotics)
├── Polyphenols
├── Diverse whole foods
├── Stress reduction
└── Exercise

Harms microbiome:
├── Processed foods
├── Antibiotics
├── Chronic stress
├── Poor sleep
├── Artificial sweeteners
└── Alcohol excess
```

### Transitions
- High fiber diet → ↑microbiomeDiversity → ↑gutSerotonin
- Chronic stress → ↓microbiomeBalance → ↑gutPermeability
- Dysbiosis → ↓serotonin → ↓mood, ↑anxiety
- Probiotics → ↑beneficial bacteria → ↑GABA → ↓anxiety
- Antibiotics → temporary ↓↓microbiomeDiversity

### Cross-system Effects
- Gut serotonin → affects mood, sleep, appetite
- Gut inflammation → systemic inflammation → brain inflammation
- Microbiome → immune training → allergy/autoimmune risk
- Vagal tone → HRV → stress resilience

---

## 16. CARDIOVASCULAR SYSTEM

### Variables
| Variable | Unit | Normal Range | Notes |
|----------|------|--------------|-------|
| `heartRate` | bpm | 60-100 resting | Varies with activity |
| `bloodPressure.systolic` | mmHg | 90-120 | During contraction |
| `bloodPressure.diastolic` | mmHg | 60-80 | During relaxation |
| `strokeVolume` | mL | 70-100 | Blood per beat |
| `cardiacOutput` | L/min | 4-8 | HR × stroke volume |
| `bloodVolume` | liters | 4.5-5.5 | Total blood |

### Transitions
- Exercise → ↑heartRate, ↑cardiacOutput
- Sympathetic activation → ↑heartRate, ↑bloodPressure
- Parasympathetic activation → ↓heartRate
- Dehydration → ↓bloodVolume → ↑heartRate (compensation)
- Cold exposure → ↑bloodPressure (vasoconstriction)
- Heat → ↓bloodPressure (vasodilation)

---

## 17. DIGESTIVE SYSTEM

### Variables
| Variable | Unit | Notes |
|----------|------|-------|
| `stomachFullness` | 0-1 | Mechanical stretch |
| `gastricEmptying` | hours | Time to empty stomach |
| `intestinalMotility` | 0-1 | Gut movement rate |
| `digestiveState` | enum | fasted/digesting/absorbing/fasted_deep |
| `bloodFlowToGut` | 0-1 | Parasympathetic = more |

### Gastric Emptying Rates
| Food Type | Emptying Time | Notes |
|-----------|---------------|-------|
| Water/liquids | 10-20 min | Fastest |
| Simple carbs | 1-2 hours | Quick absorption |
| Protein | 2-3 hours | Moderate |
| Fat | 3-5 hours | Slowest, most satiating |
| Fiber | Extends all | Slows everything |

### Transitions
- Eating → ↑stomachFullness, ↑bloodFlowToGut, ↓bloodFlowToMuscles
- Sympathetic activation → ↓intestinalMotility (fight/flight)
- Parasympathetic → ↑intestinalMotility (rest/digest)
- Large meal → "food coma" (blood diverted to gut)

---

## 18. MEMORY & LEARNING SYSTEM

### Variables
| Variable | Unit | Notes |
|----------|------|-------|
| `workingMemory` | 0-1 | Active cognitive capacity |
| `consolidationState` | enum | encoding/consolidating/retrieving |
| `neuroplasticityWindow` | boolean | Heightened learning capacity |
| `memoryStrength` | per-memory | Trace strength |

### Neuroplasticity Windows
| Trigger | Duration | Mechanism |
|---------|----------|-----------|
| Focused attention | During focus | Acetylcholine release |
| BDNF elevation (exercise) | 1-2 hours | Growth factor |
| Sleep (SWS) | During deep sleep | Memory consolidation |
| Sleep (REM) | During REM | Emotional memory processing |
| Novelty/challenge | Variable | Dopamine-enhanced encoding |

### Transitions
- Focused attention → ↑acetylcholine → neuroplasticityWindow = true
- Exercise → ↑BDNF → enhanced learning for 1-2h
- Deep sleep → consolidation of declarative memories
- REM sleep → consolidation of procedural/emotional memories
- Sleep deprivation → ↓consolidation, ↓workingMemory

---

## 19. ACTIVITY INPUTS (EVENTS)

### 19.1 Nutrition
| Activity | Parameters |
|----------|------------|
| `meal` | carbs, protein, fat, fiber, glycemicIndex |
| `water` | liters, sodium?, potassium?, magnesium? |
| `fasting` | (no-op, just absence of food) |

### 19.2 Supplements
| Supplement | Primary Effect |
|------------|----------------|
| `sodium` | Electrolyte, hydration |
| `potassium` | Electrolyte, heart |
| `magnesium` | Sleep, muscle, GABA |
| `zinc` | Testosterone, immune |
| `caffeine` | Adenosine block, alertness |
| `l_tyrosine` | Dopamine precursor |
| `alpha_gpc` | Acetylcholine, focus |
| `phenylethylamine` | Short dopamine spike |
| `creatine` | ATP, slight T boost |
| `omega3` | EPA (inflammation), DHA (brain structure) |
| `vitamin_d` | Hormone precursor |
| `tongkat_ali` | Testosterone support |
| `fadogia_agrestis` | Testosterone support |

### 19.3 Exercise
| Mode | Primary Effects |
|------|-----------------|
| `HIIT` | ↑↑sympathetic, ↑↑dopamine, ↑↑BDNF, ↓↓glycogen |
| `zone2` | ↑fat oxidation, ↑mitochondria, moderate stress |
| `strength` | ↑testosterone, ↑muscle fatigue, ↓glycogen |
| `walk` | ↑parasympathetic, ↓cortisol, low impact |
| `sprint` | ↑↑↑adrenaline, ↑↑growth hormone |
| `yoga` | ↑parasympathetic, ↑flexibility, ↓cortisol |

### 19.4 Recovery & Rest
| Activity | Parameters | Effects |
|----------|------------|---------|
| `sleep` | quality, duration, stages (light/deep/REM) | Clears adenosine, ↑GH, ↑testosterone |
| `meditation` | duration, technique | ↑parasympathetic, ↓cortisol |
| `nsdr` | duration | ↑dopamine (+65%), ↓cortisol |
| `nap` | duration | Partial adenosine clearing |

### 19.5 Environmental
| Activity | Parameters | Effects |
|----------|------------|---------|
| `cold_exposure` | temp, duration | ↑↑dopamine, ↑↑↑norepinephrine |
| `heat_exposure` | temp, duration | ↑growth hormone, ↑heat shock proteins |
| `light_exposure` | lux, duration, timeOfDay | Sets circadian, ↑cortisol (morning) |

### 19.6 Breathing
| Technique | Parameters | Effects |
|-----------|------------|---------|
| `physiological_sigh` | duration | Fastest ↓sympathetic, ↓stress |
| `box_breathing` | duration | Balanced calm |
| `wim_hof` | rounds | ↑↑adrenaline, ↑body temp |

### 19.7 Other
| Activity | Parameters | Effects |
|----------|------------|---------|
| `stress` | level, duration, source | ↑cortisol, ↑sympathetic |
| `ejaculation` | - | ↑↑prolactin, ↓↓dopamine, reset retention |
| `nocturnal_emission` | - | Partial reset, less prolactin |

---

## 20. CROSS-SYSTEM INTERACTIONS

### Primary Interactions Matrix

| Trigger | → | Affected Systems | Effect |
|---------|---|------------------|--------|
| Low glucose (<60) | → | Hormonal, Energy | ↓dopamine.baseline, ↓perceivedEnergy |
| High cortisol (>0.8) | → | Immune | ↓whiteBloodCells |
| Sleep debt (>0) | → | Energy, Hunger | ↑adenosine, ↑ghrelin, ↓leptin |
| Dehydration (<40L) | → | Energy, Hormonal | ↓perceivedEnergy, ↓dopamine.phasic |
| Low insulin | → | Metabolic | ↑ketones, ↑FFA, ↑sodium excretion |
| High insulin | → | Metabolic, Anabolic | ↓ketones, ↓fat burning, ↑mTOR |
| Retention (7+ days) | → | Hormonal | ↑androgen receptor sensitivity, ↑dopamine.baseline |
| High norepinephrine | → | Perceptual | ↑frameRate |
| Flow state | → | Perceptual, Hormonal | High DA + NE, low cortisol |
| Cold exposure | → | Hormonal, Autonomic | ↑dopamine, ↑norepinephrine, acute ↑sympathetic |
| Fasting 16h+ | → | Metabolic, Hormonal | ↑ketones, ↑GH, ↑BDNF, ↓mTOR, ↑autophagy |
| High carb meal | → | Metabolic, Hormonal, Hunger | ↑glucose, ↑insulin, ↓orexin, ↓ghrelin |
| Morning light | → | Circadian, Autonomic | ↑cortisol (+50%), sets sleep timer |
| NSDR | → | Hormonal, Energy | ↑dopamine.baseline (+65%), ↓cortisol |
| Exercise (HIIT) | → | Multiple | ↑sympathetic, ↑BDNF, ↑testosterone, ↓glycogen, ↑mTOR |
| Ultradian trough | → | Cognitive, Energy | ↓focusCapacity, ↑creativity, ↑restNeed |
| Chronic stress | → | Allostatic, Gut, Immune | ↑allostaticLoad, ↓microbiomeBalance, ↑inflammation |
| Resistance exercise | → | Anabolic, Hormonal | ↑mTOR (24h), ↑testosterone, ↑GH |
| Protein intake | → | Anabolic, Hunger | ↑leucine, ↑mTOR, ↑satiety |
| Fiber intake | → | Digestive, Gut, Hunger | ↓gastricEmptying, ↑microbiomeDiversity, ↑satiety |
| Empty stomach | → | Hunger, Hormonal | ↑ghrelin, ↑orexin (alert hunger) |
| Deep sleep | → | Memory, Hormonal, Energy | Memory consolidation, ↑GH, ↓adenosine |
| Gut dysbiosis | → | Hormonal, Immune | ↓serotonin, ↑inflammation, ↓mood |

### Circadian Modifiers
All systems are modulated by circadian phase:
- Cortisol: Peak morning, trough night
- Testosterone: Peak morning
- Growth hormone: Peak deep sleep
- Melatonin: Rising evening, peak night
- Core temp: Nadir 4am, peak 6pm
- Insulin sensitivity: Better morning than evening

---

## 21. STATE UPDATE ORDER

Each tick (1 minute), systems update in this order:

```typescript
function tick(state: BodyState, events: Activity[]): BodyState {
  // ═══════════════════════════════════════════════════════════
  // PASS 1: RHYTHM SYSTEMS (Time-based modulation)
  // ═══════════════════════════════════════════════════════════
  state = applyCircadian(state);         // 24-hour rhythms
  state = applyUltradian(state);         // 90-min cycles

  // ═══════════════════════════════════════════════════════════
  // PASS 2: INPUT PROCESSING (External events)
  // ═══════════════════════════════════════════════════════════
  for (const event of events) {
    state = applyActivity(state, event);
  }

  // ═══════════════════════════════════════════════════════════
  // PASS 3: REACTION SYSTEMS (Immediate physiological responses)
  // ═══════════════════════════════════════════════════════════
  state = updateDigestive(state);        // gastric emptying, motility
  state = updateMetabolic(state);        // glucose, ketones, insulin, lactate
  state = updateRespiratory(state);      // CO2, O2, pH (produces waste)
  state = updateHungerSatiety(state);    // ghrelin, leptin
  state = updateHormonal(state);         // all hormones spike/decay
  state = updateAnabolic(state);         // mTOR, MPS
  state = updateEnergy(state);           // adenosine accumulation

  // ═══════════════════════════════════════════════════════════
  // PASS 4: HOMEOSTASIS (The body fights to restore balance)
  // ═══════════════════════════════════════════════════════════
  state = homeostasisPass(state);

  // ═══════════════════════════════════════════════════════════
  // PASS 5: EFFECTOR SYSTEMS (Output responses)
  // ═══════════════════════════════════════════════════════════
  state = updateAutonomic(state);        // sympathetic/para, HRV
  state = updateThermoregulation(state); // temperature, BAT, sweating
  state = updateCardiovascular(state);   // heart rate, BP
  state = updateRenal(state);            // urine production, electrolyte excretion
  state = updateImmune(state);           // if fighting infection
  state = updateVisual(state);           // focal/panoramic mode effects

  // ═══════════════════════════════════════════════════════════
  // PASS 6: COGNITIVE & PERCEPTUAL (Emergent states)
  // ═══════════════════════════════════════════════════════════
  state = updateGutBrain(state);         // microbiome effects
  state = updateCognitive(state);        // attention, flow, gamma
  state = updatePerceptual(state);       // frame rate, quantization
  state = updateMemory(state);           // consolidation, plasticity

  // ═══════════════════════════════════════════════════════════
  // PASS 7: INTEGRATION (Cross-system effects, long-term)
  // ═══════════════════════════════════════════════════════════
  state = applyCrossEffects(state);
  state = updateAllostaticLoad(state);   // cumulative stress burden
  state = updateBodyComposition(state);  // fat/muscle mass changes

  // ═══════════════════════════════════════════════════════════
  // PASS 8: ADVANCE TIME
  // ═══════════════════════════════════════════════════════════
  state.circadian.timeOfDay = (state.circadian.timeOfDay + 1) % 1440;
  state.ultradian.phase = (state.ultradian.phase + 1) % 90;

  return state;
}

// ═══════════════════════════════════════════════════════════════
// THE HOMEOSTASIS PASS - Where the body "fights" to maintain balance
// ═══════════════════════════════════════════════════════════════
function homeostasisPass(state: BodyState): BodyState {

  // pH REGULATION (Respiratory compensation)
  if (state.respiratory.bloodpH < 7.35) {
    // Acidosis → Hyperventilate to blow off CO2
    state.respiratory.respiratoryRate += 2;
  } else if (state.respiratory.bloodpH > 7.45) {
    // Alkalosis → Slow breathing to retain CO2
    state.respiratory.respiratoryRate -= 1;
  }

  // FLUID BALANCE (Renal compensation)
  if (state.hydration.osmolality > 295) {
    // Dehydrated → Retain water
    state.renal.ADH += 0.1;
    state.renal.urineConcentration += 50;
  } else if (state.hydration.osmolality < 275) {
    // Overhydrated → Excrete water
    state.renal.ADH -= 0.1;
    state.renal.urineProduction += 10;
  }

  // BLOOD PRESSURE (RAAS compensation)
  if (state.cardiovascular.bloodPressure.systolic < 90) {
    // Low BP → Activate RAAS
    state.renal.renin += 0.1;
    state.renal.aldosterone += 0.05;
  } else if (state.cardiovascular.bloodPressure.systolic > 140) {
    // High BP → Suppress RAAS
    state.renal.renin -= 0.05;
  }

  // GLUCOSE REGULATION (Pancreatic compensation)
  if (state.metabolic.bloodGlucose > 140) {
    // High glucose → Release insulin
    state.metabolic.insulin += 5;
  } else if (state.metabolic.bloodGlucose < 70) {
    // Low glucose → Release glucagon
    state.metabolic.glucagon += 0.1;
    // Emergency: Release cortisol + adrenaline
    if (state.metabolic.bloodGlucose < 55) {
      state.autonomic.cortisol += 0.2;
      state.hormonal.adrenaline += 0.3;
    }
  }

  // TEMPERATURE REGULATION
  if (state.thermo.coreTemp > 37.5) {
    // Overheating → Cool down
    state.thermo.sweating += 0.2;
    state.thermo.vasodilation += 0.1;
  } else if (state.thermo.coreTemp < 36.5) {
    // Cold → Warm up
    state.thermo.shivering += 0.1;
    state.thermo.BAT.activation += 0.2;
    state.thermo.vasoconstriction += 0.1;
  }

  // ELECTROLYTE BALANCE (via Aldosterone)
  if (state.hydration.potassium > 5.0) {
    // Dangerous high K+ → Excrete it
    state.renal.aldosterone += 0.1;
  }

  return state;
}
```

---

## 22. SUMMARY COUNTS

| Category | Count |
|----------|-------|
| **Foundational Homeostatic Systems** | 4 |
| **Metabolic & Energy Systems** | 5 |
| **Regulatory Systems** | 4 |
| **Cognitive & Behavioral Systems** | 5 |
| **Rhythm Systems** | 2 |
| **Long-term Adaptation Systems** | 2 |
| **TOTAL SYSTEMS** | **22** |
| **Total Variables** | **110+** |
| **Activity Types** | 25+ |
| **Homeostatic Loops** | 6 major |
| **Tick Passes** | 8 |

### Complete System List

#### Part A: Foundational Homeostatic Systems ("Engine Block")
| # | System | Key Variables |
|---|--------|---------------|
| 0A | **Renal (Kidneys)** | GFR, aldosterone, ADH, renin, urineProduction |
| 0B | **Respiratory & pH** | bloodpH, pCO2, pO2, bicarbonate, CO2tolerance |
| 0C | **Body Composition** | adiposeTissueMass, leanMuscleMass, BMR, TDEE |
| 0D | **Visual & Vestibular** | visualMode, gazeTarget, opticFlow, vestibular |

#### Part B: Metabolic & Energy Systems
| # | System | Key Variables |
|---|--------|---------------|
| 1 | Metabolic | glucose, ketones, insulin, lactate, fatAdaptation |
| 2 | Energy & Fatigue | ATP, adenosine, adenosineReceptors, sleepDebt |
| 3 | Hunger & Satiety | ghrelin, leptin, leptinSensitivity |
| 4 | Anabolic / MPS | mTOR, MPS, MPB, leucineLevel |
| 5 | Digestive | stomachFullness, gastricEmptying, motility |

#### Part C: Regulatory Systems
| # | System | Key Variables |
|---|--------|---------------|
| 6 | Autonomic NS | sympathetic, parasympathetic, HRV, cortisol |
| 7 | Hydration & Electrolytes | water, Na, K, Mg, Zn, osmolality |
| 8 | Thermoregulation | coreTemp, sweating, shivering, BAT |
| 9 | Cardiovascular | heartRate, bloodPressure, cardiacOutput |

#### Part D: Hormonal & Immune
| # | System | Key Variables |
|---|--------|---------------|
| 10 | Hormonal | dopamine, norepinephrine, testosterone, orexin |
| 11 | Immune | status, inflammation, cytokines |
| 12 | Gut-Brain Axis | microbiomeDiversity, gutSerotonin, vagalTone |

#### Part E: Cognitive & Behavioral
| # | System | Key Variables |
|---|--------|---------------|
| 13 | Attention & Cognitive | gamma, flowState, activationEnergy |
| 14 | Perceptual | frameRate, quantizationMode |
| 15 | Memory & Learning | workingMemory, neuroplasticityWindow |

#### Part F: Rhythm Systems
| # | System | Key Variables |
|---|--------|---------------|
| 16 | Circadian | timeOfDay, melatonin, cortisolRhythm |
| 17 | Ultradian | ultradianPhase, cycleState, focusCapacity |

#### Part G: Long-term Adaptation
| # | System | Key Variables |
|---|--------|---------------|
| 18 | Allostatic Load | allostaticLoad, adaptiveCapacity, systemWear |

### Completeness Assessment

| Domain | Grade | Notes |
|--------|-------|-------|
| **Biohacking/Protocols** | **A+** | Huberman, Keto, Retention, Dopamine |
| **Core Physiology** | **A** | Kidneys, pH, Composition added |
| **Homeostatic Loops** | **A** | 6 major feedback systems |
| **Physics/Conservation** | **B+** | Mass/energy accounting added |
| **Sensory/Visual** | **A** | Focal/Panoramic, Optic Flow |

---

## 23. IMPLEMENTATION PRIORITY

### Phase 1: Core Loop
1. Metabolic (glucose, ketones, insulin)
2. Energy (adenosine, perceived energy)
3. Circadian (time, phase)
4. Basic activities (meal, sleep, fasting)

### Phase 2: Autonomic & Hydration
5. Autonomic (sympathetic/para, HRV, cortisol)
6. Hydration (water, sodium, potassium, magnesium)
7. Exercise activities
8. Caffeine/supplement effects

### Phase 3: Hunger & Digestion
9. Hunger/Satiety (ghrelin, leptin)
10. Digestive (gastric emptying, motility)
11. Ultradian rhythms (90-min cycles)

### Phase 4: Hormonal
12. Catecholamines (dopamine, norepinephrine, adrenaline)
13. Sex hormones (testosterone, prolactin, retention)
14. Orexin, serotonin, melatonin
15. Growth factors (BDNF, GH, IGF1)

### Phase 5: Anabolic & Recovery
16. Anabolic/MPS (mTOR, protein synthesis)
17. Allostatic load (stress accumulation)
18. Thermoregulation

### Phase 6: Advanced Systems
19. Immune system
20. Gut-brain axis / microbiome
21. Cardiovascular
22. Cognitive system (gamma, flow, activation)
23. Perceptual system (frame rate, quantization)
24. Memory & learning

### Phase 7: Full Integration
25. Full cross-system interactions
26. Long-term adaptations

### Phase 8: Timeline & UI
27. Branching/time-travel
28. React UI with visualizations
29. Scenario comparisons
