# Dimensions - The Dashboard Layer

**Dimensions are the answer to: "How do I feel?"**

These are READ-ONLY scores calculated FROM internal state. The user cannot set them directly—they must do activities that affect the underlying variables.

---

## Core Principle

```rust
impl Dimensions {
    /// PURE FUNCTION: Always calculated, never stored
    fn calculate(state: &BodyState) -> Self { ... }
}
```

The UI shows these scores. The simulation stores raw variables. Dimensions are **derived**.

---

## Dimension: Focus

**Description:** Ability to concentrate, do deep work, enter flow state.

### Formula

```rust
fn calc_focus(s: &BodyState) -> f32 {
    // BASE: Catecholamines drive attention
    let base = s.horm.dopamine_baseline * 30.0
             + s.horm.norepinephrine * 25.0
             + (1.0 - s.energy.adenosine) * 20.0;  // Less adenosine = more alert

    // GLUCOSE: Too low or too high hurts focus
    let glucose_penalty = if s.meta.glucose < 70.0 {
        (70.0 - s.meta.glucose) * 0.5  // Hypoglycemia fog
    } else if s.meta.glucose > 140.0 {
        (s.meta.glucose - 140.0) * 0.3  // Hyperglycemia fatigue
    } else {
        0.0
    };

    // SLEEP DEBT: Major focus killer
    let sleep_penalty = s.energy.sleep_debt * 5.0;

    // ULTRADIAN: Cycle matters
    let cycle_mod = match s.ultra.state {
        CycleState::Peak => 1.2,
        CycleState::Plateau => 1.0,
        CycleState::Trough => 0.6,
        CycleState::Recovery => 0.8,
    };

    ((base - glucose_penalty - sleep_penalty) * cycle_mod).clamp(0.0, 100.0)
}
```

### UI States
| Score | Label | Description |
|-------|-------|-------------|
| 0-20 | **Brain Fog** | Cannot concentrate, scattered |
| 20-40 | **Distracted** | Difficulty maintaining attention |
| 40-60 | **Functional** | Can work with effort |
| 60-80 | **Sharp** | Good concentration |
| 80-100 | **Flow State** | Deep focus, time distortion |

### What Improves Focus
- Morning sunlight (cortisol spike)
- Caffeine (after 90min of waking)
- Cold exposure (norepinephrine)
- NSDR (dopamine restoration)
- Adequate sleep (adenosine cleared)
- Stable blood glucose (keto or timed eating)

---

## Dimension: Energy

**Description:** Perceived physical and mental vitality.

### Formula

```rust
fn calc_energy(s: &BodyState) -> f32 {
    // BASE: ATP and fuel availability
    let base = s.energy.atp * 25.0
             + (1.0 - s.energy.adenosine) * 25.0  // Adenosine suppresses energy
             + s.comp.glycogen_mass * 50.0;        // Fuel reserves

    // SLEEP DEBT: Biggest energy drain
    let sleep_penalty = s.energy.sleep_debt * 8.0;

    // CAFFEINE: Masks fatigue temporarily
    let caffeine_boost = s.energy.receptors_blocked * 15.0;

    // BUT: Buildup will crash later
    let crash_pending = s.energy.adenosine_buildup * 10.0;

    // CENTRAL FATIGUE: CNS exhaustion
    let cns_penalty = s.energy.central_fatigue * 20.0;

    // HYDRATION: Dehydration kills energy
    let hydration_mod = if s.hydra.osmolality > 295.0 {
        0.8  // Dehydrated = 20% penalty
    } else {
        1.0
    };

    ((base - sleep_penalty + caffeine_boost - crash_pending - cns_penalty) * hydration_mod)
        .clamp(0.0, 100.0)
}
```

### UI States
| Score | Label | Description |
|-------|-------|-------------|
| 0-20 | **Exhausted** | Cannot function, need rest |
| 20-40 | **Tired** | Low motivation, sluggish |
| 40-60 | **Moderate** | Functional but not optimal |
| 60-80 | **Energized** | Good vitality |
| 80-100 | **Peak** | High energy, ready for anything |

### What Improves Energy
- Quality sleep (clears adenosine)
- Morning light (cortisol, sets rhythm)
- Strategic caffeine (90min after wake)
- Stable blood sugar
- Hydration with electrolytes

---

## Dimension: Mood

**Description:** Emotional state, positivity, resilience.

### Formula

```rust
fn calc_mood(s: &BodyState) -> f32 {
    // SEROTONIN: Core mood regulator
    let serotonin_contrib = s.horm.serotonin * 35.0;

    // DOPAMINE: Motivation and pleasure
    let dopamine_contrib = s.horm.dopamine_baseline * 25.0;

    // GUT: 90% of serotonin made here
    let gut_contrib = s.gut.serotonin * 15.0;

    // SOCIAL: Social support buffers mood
    let social_mod = if s.env.social_context == SocialContext::Intimate {
        1.2
    } else if s.env.social_context == SocialContext::Alone {
        0.9
    } else {
        1.0
    };

    // CORTISOL: High stress crushes mood
    let cortisol_penalty = if s.ans.cortisol > 0.7 {
        (s.ans.cortisol - 0.5) * 40.0
    } else {
        0.0
    };

    // SLEEP: Mood tanks with debt
    let sleep_penalty = s.energy.sleep_debt * 4.0;

    ((serotonin_contrib + dopamine_contrib + gut_contrib) * social_mod
        - cortisol_penalty - sleep_penalty).clamp(0.0, 100.0)
}
```

### UI States
| Score | Label | Description |
|-------|-------|-------------|
| 0-20 | **Depressed** | Low mood, anhedonia |
| 20-40 | **Low** | Negative bias, low motivation |
| 40-60 | **Neutral** | Stable, neither good nor bad |
| 60-80 | **Good** | Positive outlook |
| 80-100 | **Elevated** | Joyful, optimistic |

### What Improves Mood
- Social connection
- Exercise (BDNF, endorphins)
- Sunlight (serotonin)
- Gut health (fiber, fermented foods)
- Quality sleep
- Cold exposure (dopamine boost)

---

## Dimension: Libido

**Description:** Sexual drive and interest.

### Formula

```rust
fn calc_libido(s: &BodyState) -> f32 {
    // TESTOSTERONE: Primary driver
    let t_contrib = (s.horm.testosterone / 10.0).min(40.0);  // ng/dL normalized

    // DOPAMINE: Desire and motivation
    let dopamine_contrib = s.horm.dopamine_baseline * 30.0;

    // RETENTION BONUS
    let retention_bonus = match s.horm.retention_days {
        0..=2 => -20.0,   // Post-ejac crash
        3..=6 => 0.0,     // Recovery
        7..=14 => 15.0,   // Peak T window
        15.. => 10.0,     // Transmutation phase
    };

    // PROLACTIN: Suppresses libido (post-ejac)
    let prolactin_penalty = s.horm.prolactin * 30.0;

    // STRESS: High cortisol kills libido
    let stress_penalty = s.ans.cortisol * 20.0;

    // ENERGY: Need energy for libido
    let energy_mod = if s.energy.perceived < 30.0 {
        0.5
    } else {
        1.0
    };

    ((t_contrib + dopamine_contrib + retention_bonus - prolactin_penalty - stress_penalty)
        * energy_mod).clamp(0.0, 100.0)
}
```

### UI States
| Score | Label | Description |
|-------|-------|-------------|
| 0-20 | **Absent** | No interest |
| 20-40 | **Low** | Minimal drive |
| 40-60 | **Moderate** | Normal range |
| 60-80 | **High** | Strong drive |
| 80-100 | **Peak** | Very high libido |

### What Affects Libido
- Testosterone (zinc, sleep, exercise)
- Semen retention (day 7 peak)
- Dopamine (novelty, achievement)
- Low stress
- Adequate energy

---

## Dimension: Readiness (Physical)

**Description:** Physical preparedness for exercise or exertion.

### Formula

```rust
fn calc_readiness(s: &BodyState) -> f32 {
    // GLYCOGEN: Fuel for performance
    let glycogen_score = s.comp.glycogen_mass * 40.0;  // 0-0.5 kg → 0-20

    // HRV: Recovery indicator
    let hrv_score = (s.ans.hrv / 100.0 * 30.0).min(30.0);

    // MUSCLE FATIGUE: Inverse relationship
    let fatigue_penalty = s.energy.muscle_fatigue * 25.0;

    // SLEEP QUALITY: Critical for recovery
    let sleep_score = (1.0 - s.energy.sleep_debt / 8.0) * 20.0;

    // MPS WINDOW: Good if recovering from workout
    let anabolic_bonus = if s.anab.in_window { 5.0 } else { 0.0 };

    (glycogen_score + hrv_score - fatigue_penalty + sleep_score + anabolic_bonus)
        .clamp(0.0, 100.0)
}
```

### UI States
| Score | Label | Description |
|-------|-------|-------------|
| 0-20 | **Depleted** | Need recovery, avoid hard training |
| 20-40 | **Low** | Light activity only |
| 40-60 | **Moderate** | Can train moderately |
| 60-80 | **Ready** | Good for training |
| 80-100 | **Peak** | Optimal performance day |

---

## Dimension: Hunger

**Description:** Perceived hunger drive.

### Formula

```rust
fn calc_hunger(s: &BodyState) -> f32 {
    // GHRELIN: Hunger hormone
    let ghrelin_contrib = s.hunger.ghrelin * 50.0;

    // LEPTIN: Satiety hormone (inverse)
    let leptin_suppress = s.hunger.leptin * s.hunger.leptin_sensitivity * 30.0;

    // BLOOD GLUCOSE: Low glucose → hunger
    let glucose_factor = if s.meta.glucose < 80.0 {
        (80.0 - s.meta.glucose) * 0.5
    } else {
        -(s.meta.glucose - 80.0) * 0.2  // High glucose suppresses
    };

    // STOMACH FULLNESS: Mechanical satiety
    let fullness_suppress = s.digest.fullness * 40.0;

    // KETONES: Suppress hunger (appetite blunting)
    let keto_suppress = s.meta.ketones * 15.0;

    (ghrelin_contrib - leptin_suppress + glucose_factor - fullness_suppress - keto_suppress)
        .clamp(0.0, 100.0)
}
```

### UI States
| Score | Label | Description |
|-------|-------|-------------|
| 0-20 | **Full** | No interest in food |
| 20-40 | **Satisfied** | Could eat but don't need to |
| 40-60 | **Neutral** | Normal hunger |
| 60-80 | **Hungry** | Want to eat |
| 80-100 | **Ravenous** | Strong hunger, hard to ignore |

---

## Dimension: Stress

**Description:** Perceived stress and tension level.

### Formula

```rust
fn calc_stress(s: &BodyState) -> f32 {
    // CORTISOL: Primary stress marker
    let cortisol_contrib = s.ans.cortisol * 40.0;

    // SYMPATHETIC: Fight/flight activation
    let sympathetic_contrib = s.ans.sympathetic * 30.0;

    // ALLOSTATIC LOAD: Cumulative burden
    let allostatic_contrib = s.allo.load * 0.3;

    // PARASYMPATHETIC: Calming influence (inverse)
    let para_offset = s.ans.parasympathetic * 20.0;

    // HRV: Low HRV = stressed
    let hrv_factor = if s.ans.hrv < 40.0 {
        (40.0 - s.ans.hrv) * 0.5
    } else {
        0.0
    };

    (cortisol_contrib + sympathetic_contrib + allostatic_contrib - para_offset + hrv_factor)
        .clamp(0.0, 100.0)
}
```

### UI States
| Score | Label | Description |
|-------|-------|-------------|
| 0-20 | **Calm** | Relaxed, at peace |
| 20-40 | **Low** | Minimal stress |
| 40-60 | **Moderate** | Manageable stress |
| 60-80 | **High** | Feeling stressed |
| 80-100 | **Overwhelmed** | High stress, burnout risk |

### What Reduces Stress
- Physiological sigh (fastest)
- Panoramic vision (horizon gazing)
- Walking (optic flow)
- NSDR
- Social support
- Cold exposure (rebound calm)

---

## Dimension: Clarity (Mental)

**Description:** Mental sharpness, cognitive clarity.

### Formula

```rust
fn calc_clarity(s: &BodyState) -> f32 {
    // KETONES: Brain fuel, clarity boost
    let ketone_bonus = if s.meta.ketones > 0.5 && s.meta.ketones < 3.0 {
        s.meta.ketones * 15.0  // "Keto clarity"
    } else {
        0.0
    };

    // GLUCOSE STABILITY: Stable is better
    let glucose_factor = if s.meta.glucose > 70.0 && s.meta.glucose < 100.0 {
        20.0  // Optimal range
    } else {
        10.0
    };

    // HYDRATION: Dehydration = brain fog
    let hydration_factor = if s.hydra.osmolality < 295.0 {
        20.0
    } else {
        10.0 - (s.hydra.osmolality - 295.0) * 0.5
    };

    // SLEEP DEBT: Fog accumulates
    let sleep_penalty = s.energy.sleep_debt * 5.0;

    // INFLAMMATION: Brain inflammation = fog
    let inflammation_penalty = s.immune.inflammation * 15.0;

    (ketone_bonus + glucose_factor + hydration_factor - sleep_penalty - inflammation_penalty
        + s.horm.bdnf * 15.0)  // BDNF supports clarity
        .clamp(0.0, 100.0)
}
```

### UI States
| Score | Label | Description |
|-------|-------|-------------|
| 0-20 | **Brain Fog** | Confused, slow thinking |
| 20-40 | **Cloudy** | Difficulty with complex thought |
| 40-60 | **Clear** | Normal mental function |
| 60-80 | **Sharp** | Quick thinking |
| 80-100 | **Crystal** | Exceptional clarity |

---

## UI Display

```
┌─────────────────────────────────────────────────────────────────┐
│                     HOW ARE YOU FEELING?                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Focus     ████████████████░░░░  78/100  [Sharp]              │
│   Energy    ██████████████░░░░░░  65/100  [Energized]          │
│   Mood      ███████████████░░░░░  72/100  [Good]               │
│   Libido    ████████░░░░░░░░░░░░  40/100  [Moderate]           │
│   Readiness █████████████░░░░░░░  62/100  [Moderate]           │
│   Hunger    ████░░░░░░░░░░░░░░░░  22/100  [Satisfied]          │
│   Stress    ██████░░░░░░░░░░░░░░  28/100  [Low]                │
│   Clarity   █████████████████░░░  82/100  [Sharp]              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Implementation

```rust
struct Dimensions {
    focus: f32,
    energy: f32,
    mood: f32,
    libido: f32,
    readiness: f32,
    hunger: f32,
    stress: f32,
    clarity: f32,
}

impl Dimensions {
    pub fn calculate(state: &BodyState) -> Self {
        Self {
            focus: Self::calc_focus(state),
            energy: Self::calc_energy(state),
            mood: Self::calc_mood(state),
            libido: Self::calc_libido(state),
            readiness: Self::calc_readiness(state),
            hunger: Self::calc_hunger(state),
            stress: Self::calc_stress(state),
            clarity: Self::calc_clarity(state),
        }
    }

    pub fn label(&self, dim: &str) -> &'static str {
        let score = match dim {
            "focus" => self.focus,
            "energy" => self.energy,
            // ... etc
            _ => 50.0,
        };

        match score as u8 {
            0..=20 => LABELS[dim][0],
            21..=40 => LABELS[dim][1],
            41..=60 => LABELS[dim][2],
            61..=80 => LABELS[dim][3],
            81..=100 => LABELS[dim][4],
            _ => "Unknown",
        }
    }
}
```
