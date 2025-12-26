# Unclassified Insights

> **Purpose:** Orphaned valuable insights that haven't been integrated into the main specs yet. Review periodically and migrate to appropriate spec files.

---

## Protocols (from Huberman Lab, personal experimentation)

### Morning Protocol

```yaml
wake_up:
  - delay_caffeine: 90-120 min  # allows natural cortisol spike + adenosine clearing
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

why_delay_caffeine:
  - Caffeine blocks adenosine receptors (doesn't clear adenosine)
  - Morning cortisol spike naturally clears adenosine in first 90 min
  - Immediate caffeine = adenosine builds up behind the block → afternoon crash
  - Delayed caffeine = cortisol clears adenosine first → sustained energy

why_morning_light:
  - Triggers 50% increase in cortisol pulse amplitude
  - Sets circadian timer for sleep ~16 hours later
  - Enhances immune function, metabolism, and focus all day
  - 2 days of proper light exposure can reset drifted circadian rhythms
```

### Illness Recovery (Viral vs Bacterial)

```yaml
strategy: "Hybrid" for viral
nutrition:
  - bone_broth: liters (electrolytes, soothing)
  - berries: moderate (low glycemic glucose)
  - honey: 1 tsp raw (antiviral, 5-6g carbs)
  - light_proteins: [eggs, chicken_breast, white_fish]
avoid:
  - heavy_fats (digestion impaired)
  - fasting (increases cortisol during viral - bad)
  - sudden_high_carb (inflammation if keto-adapted)
target_carbs: 50-70g ("medicinal carbs")

# Wang Study (Yale/Medzhitov Lab):
# - Bacterial infection: Fasting/Ketosis better (glucose restriction starves bacteria)
# - Viral infection: Glucose intake better (protects neurons from inflammation)
# - Post-illness: Hybrid 50-70g carbs (support recovery)
```

### Sleep Optimization

```yaml
evening:
  - dim_lights: 2h before bed
  - cool_room: 65-68°F (18-20°C)  # Body needs to drop 1-3°C to sleep
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

### Cold Exposure Protocol

```yaml
weekly_target: 11 minutes total
session_structure:
  - frequency: 2-4 sessions per week
  - duration: 1-5 minutes per session
  - temperature: 37-55°F (3-13°C)  # Cold enough to want to get out

temperature_duration_tradeoff:
  very_cold:  # 35-45°F (2-7°C)
    duration: 1-3 min
    effect: Rapid catecholamine release
  moderate:  # 55-60°F (13-15°C)
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

### NSDR / Yoga Nidra Protocol

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

### Breathing Techniques

```yaml
physiological_sigh:  # Fastest real-time stress relief
  technique:
    - Double inhale through nose (big + small with no exhale between)
    - Long exhale through mouth until lungs empty
  duration: 1-3 breaths for acute stress; 5 min daily for chronic stress reduction
  effects:
    - Rapid shift to parasympathetic
    - Lower heart rate
    - Improved sleep
  research: Stanford study showed 5 min daily reduces overall stress

box_breathing:  # Balanced, calming
  technique:
    - Inhale 4 sec
    - Hold 4 sec
    - Exhale 4 sec
    - Hold 4 sec
  duration: 5-10 min
  effects: Even heart rate, calm focus
  note: Duration based on CO2 tolerance

cyclic_hyperventilation:  # Wim Hof style - arousing
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

### Focus & Attention Protocol

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

### Testosterone Optimization Protocol

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
  tier_1_behavioral:  # Do these first
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

## Hypotheses & Models

### Carbohydrate-Insulin Model of Obesity (WIL)

```
Core Thesis (Joseph Everett / What I've Learned):

Obesity trends began rising when dietary guidelines shifted to low-fat, high-carb recommendations.
We replaced steak with pasta, butter with margarine, eggs with cereal.
Result: We became fatter than ever despite eating "healthier."

The Insulin Hypothesis:
1. High carb intake → High insulin → Fat storage mode
2. Fat can only be released when insulin is low
3. Low-fat diets replaced fat with carbs → Chronically elevated insulin
4. Solution: Reduce carbs, allow insulin to fall, burn stored fat
```

### Intermittent Fasting Mechanism

```
Hours 0-10: Burning glucose from food
Hours 10-12: Glycogen stores depleting
Hours 12-16: Ketone production begins
Hours 16+: Fat burning in full effect

Key insight: Give body TIME to deplete glycogen and enter ketosis.
16-hour fasting window allows this transition daily.

WIL Diet Approach:
- ~90% low-carb over month
- ~70% keto (very low carb)
- ~80% of eating within 7-hour window
- Allows flexibility for social eating
```

### Sugar Addiction Model

```
Neurological Comparison:
Sugar triggers same reward pathways as drugs of abuse:
- Dopamine release in nucleus accumbens
- Opioid release
- Binge patterns with intermittent access

Study: 94% of rats chose saccharin-sweetened water over IV cocaine
(even when cocaine dose increased)

Sugar → Insulin → Dopamine Connection:
Sugar spike → Insulin spike → Blood sugar crash
→ Brain seeks more sugar (dopamine hit)
→ Cycle repeats → Tolerance builds → Need more for same effect
```

### Autophagy Window

```
Timing for Cellular Cleanup:
Hours 12-16: Ketones rising, autophagy beginning
Hours 24-48: Significant autophagy activation
Hours 36-72: Peak autophagy (cellular recycling)
72+: Levels off but remains elevated vs fed state

Synergy with Ketosis:
- Ketones signal nutrient scarcity
- Cells enter cleanup/repair mode
- Damaged proteins and organelles recycled
- Particularly beneficial for brain (neurodegeneration protection)
```

---

## Specific Mechanism Details

### RAAS Cascade (Renin-Angiotensin-Aldosterone System)

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

### Brown Adipose Tissue (BAT) Thermogenesis

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

### Adenosine Receptor Dynamics (Caffeine Crash)

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

---

## Visual System Effects (Huberman)

### Focal vs Panoramic Vision

```
FOCAL (narrow attention, near object):
- ↑Sympathetic activation
- ↑Alertness
- Used for: Reading, phone, hunting

PANORAMIC (wide peripheral awareness):
- ↑Parasympathetic activation
- ↓Stress
- Used for: Horizon gazing, relaxation
```

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

Transitions:
- Screen work (focal, near) → ↑sympathetic, ↑eye strain, ↑cortisol
- Horizon gazing (panoramic, far) → ↑parasympathetic, ↓stress
- Walking outside → optic flow + panoramic → ↓↓anxiety
```

### Gaze & Alertness

```
Upward gaze → ↑Alertness (why we look up when thinking)
Downward gaze → ↓Alertness, ↑sleepiness
Horizon gaze → Neutral, calming
```

---

## Cognitive Models

### Gamma Function (ADHD Model)

```
motivation = input^gamma

gamma ≈ 1.0: Neurotypical (linear response)
gamma > 2.5: High-gamma brain (low signals crushed to zero)

High-gamma individuals need "Level 10" challenges to activate.

Activation Energy:
activationEnergy = baseTaskDifficulty * (1 / dopamine.baseline) * gamma
```

### Tachypsychia (Frame Rate Model)

```
Peak state = increased perceptual sampling rate:

| State      | Norepinephrine | Frame Rate   | Experience           |
|------------|----------------|--------------|----------------------|
| Low alert  | Low            | ~30 fps      | Normal time          |
| High alert | High           | ~60-120 fps  | Time seems slower    |
| Peak/Flow  | Very high      | ~120+ fps    | "Slow motion"        |
```

### Quantization Mode (1-bit vs Gradient)

```
Some individuals operate binary rather than gradient:

| State      | Characteristics               | Neurochemistry                    |
|------------|-------------------------------|-----------------------------------|
| OFF        | Asleep, Bored, Paralyzed      | Low dopamine, Low norepinephrine  |
| ON         | "God Mode", Flow, "Beast Mode"| High dopamine, High NE, Low cort  |
| Gradient   | Normal functioning            | Moderate, balanced                |
```

---

## References

### Research Studies

- **Wang Study (Yale/Medzhitov Lab):** Differential metabolic requirements for bacterial vs viral immunity
- **Cold water immersion:** 250% dopamine, 530% norepinephrine increase at 14°C
- **Yoga Nidra PET study (2002):** 65% dopamine increase
- **Stanford Cyclic Sighing Study:** 5 min daily physiological sighs reduce stress
- **Sugar Addiction Study (2007):** 94% of rats chose saccharin over cocaine
- **CarbMetSim glucose model:** https://github.com/mukulgoyalmke/CarbMetSim

### Huberman Lab Episodes

- Controlling Your Dopamine
- Using Cortisol & Adrenaline
- Optimize Brain Chemistry
- Sleep Toolkit
- Using Caffeine to Optimize Performance
- Cold Exposure for Health & Performance
- ADHD & Focus
- Optimize Testosterone & Estrogen
- Breathing for Health & Performance
- Light for Health
- NSDR Portal
- Guest Series: Dr. Matt Walker on Sleep Protocols
- Guest Series: Dr. Susanna Soberg on Cold & Heat Exposure

### What I've Learned (Joseph Everett)

- YouTube: https://www.youtube.com/@WhatIveLearned
- Topics: Carbohydrate-insulin model, keto, intermittent fasting, saturated fat/cholesterol myths
- Personal protocol: ~90% low-carb, ~70% keto, 7-hour eating window

### Electrolyte Guidelines (Keto)

- Sodium: 4-5g/day (vs 2g normally)
- Potassium: 1-3g/day
- Magnesium: 300-400mg/day
- Zinc: 15-30mg/day

---

## UI Concept (for future reference)

```
┌─────────────────────────────────────────────────────────────────┐
│  Timeline: [■■■■■■■■■■■■■■■░░░░░░░░░░░░░░░░░░░░░]  Day 1, 14:30 │
│            ↑ branch here                                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Energy    ████████████████████  89/100                       │
│   Dopamine  ██████████████░░░░░░  0.72 (baseline)              │
│   Alertness ████████████████░░░░  0.81 (norepinephrine)        │
│                                                                 │
│   Glucose   ████████░░░░░░░░░░░░  85 mg/dL                     │
│   Ketones   ██████████████░░░░░░  1.8 mmol/L  ← in ketosis     │
│   HRV       ████████████░░░░░░░░  62 ms                        │
│                                                                 │
│   Sodium    ██████░░░░░░░░░░░░░░  LOW - need supplement        │
│   Water     ████████████░░░░░░░░  OK                           │
│   Temp      ████████████████░░░░  37.2°C                       │
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
