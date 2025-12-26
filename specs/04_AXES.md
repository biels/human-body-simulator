# Axes - Primary Physiological Pathways

**Axes are the fundamental dimensions you can FEEL.** They map directly to biological pathways. The simulation visualizes these axes, and "state" is a classification of the current snapshot.

---

## Philosophy

```
110+ Internal Variables
        ↓
   (dimensionality reduction)
        ↓
   7 AXES (pathway-based)
        ↓
   State Classification ("How am I right now?")
```

Each axis represents a real physiological pathway with:
- Clear subjective experience ("I can feel this")
- Known modulators (what changes it)
- Measurable correlates (if you had sensors)

---

## The 7 Axes

### 1. ANDROGENIC (Retention/Testosterone Pathway)

**What you feel:** Drive, assertiveness, confidence, libido, aggression, "fire in the belly"

**Pathway:** Testosterone → DHT → Androgen receptors

```rust
struct Androgenic {
    level: f32,           // 0.0 (depleted) to 1.0 (peak)
    receptor_sensitivity: f32,  // Upregulates with retention
}
```

**Key variables:**
- `horm.testosterone`
- `horm.dht`
- `horm.prolactin` (inverse - suppresses)
- `horm.retention_days`

**Modulators:**
| Raises | Lowers |
|--------|--------|
| Retention (day 7 peak) | Ejaculation |
| Sleep (8h+) | Sleep deprivation |
| Heavy lifting | Chronic stress |
| Zinc, Vitamin D | Alcohol |
| Morning sunlight | High prolactin |

**Subjective states:**
| Level | You Feel |
|-------|----------|
| 0.0-0.3 | Passive, low drive, avoidant |
| 0.3-0.5 | Normal baseline |
| 0.5-0.7 | Motivated, assertive |
| 0.7-1.0 | Driven, confident, high libido |

---

### 2. DOPAMINERGIC (Baseline Tone / Seeking Pathway)

**What you feel:** Content vs seeking, patient vs impulsive, exploiting vs exploring

**Pathway:** Dopamine baseline (tonic) → receptor binding → affects reward threshold

```rust
struct Dopaminergic {
    effective_level: f32, // What you actually experience (0.0-1.0)
    baseline: f32,        // Tonic production level (0.0-1.0)
    receptor_density: f32, // Number of D2 receptors (downregulates with overuse)
    receptor_sensitivity: f32, // How responsive each receptor is
    reuptake_rate: f32,   // DAT activity - ADHD: often high (clears too fast)
    seeking: f32,         // Inverse of effective - how much you're hunting
    time_preference: f32, // -1.0 (present bias) to 1.0 (future bias)
}
```

**The ADHD insight (Reuptake/DAT):**
```
EFFECTIVE DOPAMINE = baseline × receptor_density × sensitivity / reuptake_rate

ADHD typically has:
- Normal or low baseline
- Normal receptor density
- HIGH REUPTAKE RATE (DAT hyperactivity)
  → Dopamine clears synapse too fast
  → Need constant hits to maintain any level
  → Explains why stimulants (DAT blockers) help

This is why:
- Same dopamine input → ADHD feels less
- Need "Level 10" stimulation to feel "Level 5" effect
- Boredom is PAINFUL (dopamine crashes to zero)
- Hyperfocus on interesting = finally enough dopamine
```

**The key insight:**
- **LOW effective** = hungry for dopamine, seeking, distractible, present-biased
- **HIGH effective** = content, patient, can exploit current task, future-oriented

**Key variables:**
- `horm.dopamine_baseline` (production)
- `horm.receptor_density` (how many receptors)
- `horm.receptor_sensitivity` (how responsive)
- `horm.dopamine_reuptake` (DAT - how fast it clears)
- `horm.prolactin` (antagonizes dopamine)
- `attn.gamma` (individual's motivation curve steepness)

**Receptor dynamics:**
| Mechanism | Speed | Effect |
|-----------|-------|--------|
| Sensitivity | Days | Upregulates with abstinence, peaks day 4-6 |
| Density | Weeks | Downregulates with chronic overstimulation |
| Reuptake | Trait | Mostly genetic, modifiable with stimulants |

**Modulators:**
| Raises Effective | Lowers Effective |
|------------------|------------------|
| Retention (sustained) | Ejaculation (prolactin surge) |
| Cold exposure (+250%) | Porn/superstimuli (density crash) |
| NSDR (+65%) | Social media binges |
| Achieving goals | Chronic novelty-seeking |
| Sunlight | Sleep deprivation |
| Abstinence 4-6 days (sensitivity ↑) | Frequent rewards (density ↓) |

**Subjective states:**
| Effective | Seeking | You Feel |
|-----------|---------|----------|
| Low | High | "Need stimulation", scrolling, restless, impulsive |
| Medium | Medium | Normal, can work with effort, some distractibility |
| High | Low | Content, focused, patient, "in the zone", future-oriented |

**Exploration vs Exploitation:**
```
Low dopamine  → HIGH exploration (seeking novelty)
              → LOW exploitation (can't stick with known path)
              → "Puer Aeternus" pattern (can't commit)

High dopamine → LOW exploration (content)
              → HIGH exploitation (can extract value from current path)
              → "Completion mode" activated
```

**Time preference:**
```
Low dopamine → High discount rate → "I want it NOW"
High dopamine → Low discount rate → "I can wait for better"
```

---

### 3. METABOLIC (Fuel Mode / Keto-Glucose Pathway)

**What you feel:** Hunger, energy stability, mental clarity, cravings

**Pathway:** Insulin ↔ Glucagon → Glucose vs Ketone burning

```rust
struct Metabolic {
    mode: FuelMode,       // Glycolytic, Ketogenic, Hybrid
    stability: f32,       // How stable is energy? (0.0-1.0)
    clarity: f32,         // "Keto clarity" effect (0.0-1.0)
    hunger: f32,          // Appetite drive (0.0-1.0)
}

enum FuelMode {
    Glycolytic,   // Burning glucose, insulin high
    Hybrid,       // Transitioning
    Ketogenic,    // Burning ketones, insulin low
}
```

**Key variables:**
- `meta.insulin`
- `meta.ketones`
- `meta.glucose`
- `hunger.ghrelin`
- `hunger.leptin`

**Modulators:**
| Towards Ketogenic | Towards Glycolytic |
|-------------------|-------------------|
| Fasting 12h+ | Carb meal |
| Low carb diet | High carb diet |
| Morning (fasted) | Post-meal |
| Exercise (depletes glycogen) | Frequent eating |

**Subjective states:**
| Mode | Stability | Hunger | Clarity | You Feel |
|------|-----------|--------|---------|----------|
| Glycolytic (fed) | Low | Low→High (rollercoaster) | Medium | Energy swings, food coma, then hungry |
| Glycolytic (crashed) | Low | High | Low | Brain fog, hangry, need carbs |
| Ketogenic | High | Low | High | Stable energy, clear mind, not hungry |
| Hybrid | Medium | Medium | Medium | Transitioning, "keto flu" possible |

---

### 4. ADENOSINERGIC (Sleep Pressure / Alertness Pathway)

**What you feel:** Sleepy vs alert, able to focus vs foggy, tired vs awake

**Pathway:** Adenosine accumulation → receptor binding → sleepiness

```rust
struct Adenosinergic {
    pressure: f32,        // Sleep pressure (0.0-1.0)
    alertness: f32,       // Inverse - how awake (0.0-1.0)
    debt: f32,            // Accumulated sleep debt (hours)
    caffeine_mask: f32,   // How much caffeine is hiding the pressure
}
```

**Key variables:**
- `energy.adenosine`
- `energy.receptors_blocked` (caffeine)
- `energy.adenosine_buildup` (behind caffeine)
- `energy.sleep_debt`

**Modulators:**
| Lowers Pressure (↑Alert) | Raises Pressure (↓Alert) |
|--------------------------|--------------------------|
| Sleep (clears adenosine) | Time awake |
| Caffeine (masks, doesn't clear) | Sleep deprivation |
| Morning cortisol spike | Caffeine crash |
| Nap / NSDR | Debt accumulation |

**Subjective states:**
| Pressure | Caffeine | You Feel |
|----------|----------|----------|
| Low | None | Naturally alert, sharp |
| Low | Some | Wired, maybe jittery |
| High | Masked | "Running on coffee", functional |
| High | Wearing off | CRASH, exhausted, brain fog |
| High | None | Sleepy, can't focus, need rest |

**The caffeine trap:**
```
Caffeine blocks receptors
    ↓
Adenosine builds up BEHIND the block
    ↓
Caffeine wears off (5-6h half-life)
    ↓
ALL the adenosine floods in
    ↓
Crash worse than baseline
```

---

### 5. AUTONOMIC (Stress / Calm Pathway)

**What you feel:** Stressed vs calm, anxious vs relaxed, fight-flight vs rest-digest

**Pathway:** Sympathetic ↔ Parasympathetic balance

```rust
struct Autonomic {
    activation: f32,      // -1.0 (para) to 1.0 (symp)
    stress: f32,          // Perceived stress (0.0-1.0)
    calm: f32,            // Inverse
    hrv: f32,             // Heart rate variability (recovery indicator)
}
```

**Key variables:**
- `ans.sympathetic`
- `ans.parasympathetic`
- `ans.cortisol`
- `ans.hrv`

**Modulators:**
| Towards Calm (Para) | Towards Stress (Symp) |
|---------------------|----------------------|
| Physiological sigh | Threat/danger |
| Panoramic vision | Focal vision (screens) |
| Walking (optic flow) | Sitting still, scanning |
| Box breathing | Shallow breathing |
| Cold (after rebound) | Cold (acute) |
| Social safety | Social threat |

**Subjective states:**
| Activation | HRV | You Feel |
|------------|-----|----------|
| High symp, low para | Low | Anxious, tense, racing thoughts |
| Balanced | Medium | Alert but not stressed |
| Low symp, high para | High | Calm, relaxed, digesting well |
| Stuck high symp | Very low | Burnout, can't relax, exhausted but wired |

---

### 6. IMMUNE (Inflammation / Resilience Pathway)

**What you feel:** Healthy vs run down, resilient vs fragile, fighting something

**Pathway:** Immune activation, cytokines, inflammation

```rust
struct Immune {
    resilience: f32,      // How robust (0.0-1.0)
    inflammation: f32,    // Systemic inflammation (0.0-1.0)
    fighting: bool,       // Currently fighting infection
}
```

**Key variables:**
- `immune.status`
- `immune.inflammation`
- `immune.wbc`
- `allo.load` (allostatic load)

**Modulators:**
| Raises Resilience | Lowers Resilience |
|-------------------|-------------------|
| Sleep 7-9h | Sleep deprivation |
| Low chronic stress | Chronic high cortisol |
| Diverse diet, fiber | Processed food |
| Exercise (moderate) | Overtraining |
| Cold exposure | Chronic inflammation |
| Social connection | Isolation |

**Subjective states:**
| Resilience | Inflammation | You Feel |
|------------|--------------|----------|
| High | Low | Robust, healthy, resilient |
| Medium | Low | Normal, fine |
| Medium | High | Something's off, brain fog |
| Low | High | Run down, getting sick, fragile |
| Any | Fighting | Actively ill, fatigue, fever |

---

### 7. THERMOREGULATORY (Cold/Heat Adaptation Pathway)

**What you feel:** Cold tolerance, heat tolerance, comfortable temperature range

**Pathway:** BAT (brown adipose tissue), vasoconstriction/dilation, thermogenesis

```rust
struct Thermoregulatory {
    cold_tolerance: f32,  // How well you handle cold (0.0-1.0)
    heat_tolerance: f32,  // How well you handle heat (0.0-1.0)
    comfort_temp: f32,    // Preferred ambient temp (°C)
    bat_mass: f32,        // Brown adipose tissue (trainable)
}
```

**Key variables:**
- `thermo.bat_mass`
- `thermo.bat_activation`
- `thermo.core_temp`
- `thermo.cold_tolerance`

**Modulators:**
| Raises Cold Tolerance | Lowers Cold Tolerance |
|-----------------------|----------------------|
| Regular cold exposure | Avoiding cold |
| BAT development | Low BAT mass |
| Fat adaptation (keto) | Glycolytic metabolism |
| Higher norepinephrine | Low catecholamines |

**Subjective states:**
| Cold Tolerance | BAT Mass | You Feel |
|----------------|----------|----------|
| Low | Low | Always cold, hate cold showers |
| Medium | Medium | Can tolerate cold with effort |
| High | High | Cold feels invigorating, good tolerance |

---

## State Classification

Given the 7 axes, classify current state:

```rust
struct AxisSnapshot {
    androgenic: f32,      // 0.0-1.0
    dopaminergic: f32,    // 0.0-1.0 (baseline, not seeking)
    metabolic: FuelMode,
    adenosinergic: f32,   // 0.0-1.0 (alertness, inverse of pressure)
    autonomic: f32,       // -1.0 (stressed) to 1.0 (calm)
    immune: f32,          // 0.0-1.0 (resilience)
    thermoregulatory: f32, // 0.0-1.0 (cold tolerance)
}

fn classify_state(axes: &AxisSnapshot) -> StateClassification {
    // Example classifications:

    if axes.androgenic > 0.7 && axes.dopaminergic > 0.7 && axes.adenosinergic > 0.7 {
        return StateClassification::Peak; // "In the zone"
    }

    if axes.dopaminergic < 0.3 && axes.adenosinergic < 0.4 {
        return StateClassification::Depleted; // "Need recovery"
    }

    if axes.autonomic < -0.5 && axes.immune < 0.4 {
        return StateClassification::Burnout; // "Running on empty"
    }

    // etc.
}
```

**Example state labels:**
| State | Axes Pattern |
|-------|--------------|
| **Peak** | High androgenic, high dopaminergic, high alertness, calm |
| **Flow** | High dopaminergic (content), high alertness, calm, ketogenic |
| **Depleted** | Low dopaminergic (seeking), low alertness, any |
| **Wired** | High alertness (caffeine), high stress, low HRV |
| **Crashed** | Low everything, post-ejac or post-binge |
| **Recovering** | Medium axes, trending upward |
| **Sick** | Low immune, high inflammation |
| **Adapted** | High cold tolerance, ketogenic, high resilience |

---

## Visualization

```
AXES DASHBOARD
═══════════════════════════════════════════════════════════════

Androgenic    [██████████████░░░░░░]  72%  "Driven"
Dopaminergic  [████████████████░░░░]  81%  "Content"
Metabolic     [═══ KETOGENIC ═══════]       "Stable"
Adenosinergic [██████████████████░░]  88%  "Alert"
Autonomic     [←──────────●────────→]  +0.4  "Calm"
Immune        [█████████████████░░░]  85%  "Resilient"
Thermo        [████████████░░░░░░░░]  58%  "Adapting"

───────────────────────────────────────────────────────────────
STATE: Peak Performance
───────────────────────────────────────────────────────────────
```

---

## Cross-Axis Interactions

| If... | Then... |
|-------|---------|
| Androgenic ↑ + Dopaminergic ↑ | Maximum drive, can tackle hard things |
| Androgenic ↑ + Dopaminergic ↓ | Aggressive seeking, impulsive |
| Dopaminergic ↓ + Adenosinergic ↓ | Tired AND seeking = scrolling zombie |
| Metabolic = Keto + Adenosinergic ↑ | Clear, stable, productive |
| Autonomic = Stressed + Immune ↓ | Heading toward burnout/sickness |
| All axes high | Peak state, rare, leverage it |

---

## Implementation

```rust
impl AxisSnapshot {
    pub fn from_state(state: &BodyState) -> Self {
        Self {
            androgenic: Self::calc_androgenic(state),
            dopaminergic: Self::calc_dopaminergic(state),
            metabolic: Self::calc_metabolic(state),
            adenosinergic: Self::calc_adenosinergic(state),
            autonomic: Self::calc_autonomic(state),
            immune: Self::calc_immune(state),
            thermoregulatory: Self::calc_thermoregulatory(state),
        }
    }

    fn calc_androgenic(s: &BodyState) -> f32 {
        let t_factor = (s.horm.testosterone / 800.0).clamp(0.0, 1.0);
        let retention_bonus = match s.horm.retention_days as u16 {
            0..=2 => -0.3,
            3..=6 => 0.0,
            7..=14 => 0.2,
            _ => 0.15,
        };
        let prolactin_penalty = s.horm.prolactin * 0.4;

        (t_factor + retention_bonus - prolactin_penalty).clamp(0.0, 1.0)
    }

    fn calc_dopaminergic(s: &BodyState) -> f32 {
        // EFFECTIVE dopamine = what you actually experience
        // This is the key formula for ADHD modeling
        let effective = s.horm.dopamine_baseline
            * s.horm.receptor_density
            * s.horm.receptor_sensitivity
            / s.horm.dopamine_reuptake.max(0.5);

        // Prolactin antagonizes dopamine signaling
        let prolactin_penalty = (s.horm.prolactin - 1.0).max(0.0) * 0.3;

        (effective - prolactin_penalty).clamp(0.0, 1.0)
    }

    fn calc_adenosinergic(s: &BodyState) -> f32 {
        // Higher = more alert
        let pressure = s.energy.adenosine;
        let masked = s.energy.receptors_blocked * 0.5;
        let debt_penalty = (s.energy.sleep_debt / 16.0).min(0.5);

        (1.0 - pressure + masked - debt_penalty).clamp(0.0, 1.0)
    }

    fn calc_autonomic(s: &BodyState) -> f32 {
        // Negative = stressed, Positive = calm
        s.ans.parasympathetic - s.ans.sympathetic
    }

    fn calc_immune(s: &BodyState) -> f32 {
        let inflammation_penalty = s.immune.inflammation * 0.5;
        let load_penalty = s.allo.load / 100.0 * 0.3;

        (1.0 - inflammation_penalty - load_penalty).clamp(0.0, 1.0)
    }

    fn calc_thermoregulatory(s: &BodyState) -> f32 {
        let bat_factor = (s.thermo.bat_mass / 200.0).clamp(0.0, 0.5);
        let adaptation = s.thermo.cold_tolerance;

        (bat_factor + adaptation * 0.5).clamp(0.0, 1.0)
    }
}
```
