# Controllers - Regulatory Systems

The "software" layer that modulates the engine. These systems don't maintain homeostasis directly—they regulate timing, signaling, and state transitions.

---

## System: Circadian

The master 24-hour clock that modulates nearly every other system.

### State Variables
| Variable | Rust Path | Notes |
|----------|-----------|-------|
| `time_of_day` | `circ.time_of_day` | Minutes since midnight (0-1439) |
| `phase` | `circ.phase` | Enum: 8 phases |
| `light_exposure` | `circ.light_exposure` | Today's lux-hours |
| `core_temp` | `circ.core_temp` | Follows circadian rhythm |
| `cortisol_mult` | `circ.cortisol_mult` | Circadian modifier |
| `hours_awake` | `circ.hours_awake` | For caffeine timing |

### Inputs (Dependencies)
| Reads From | Variable | Purpose |
|------------|----------|---------|
| `env` | `light_lux` | Light exposure input |
| `env` | `clock_time` | External time reference |
| Activities | `wake`, `sleep` | State transitions |

### Phase Definitions

```rust
enum CircadianPhase {
    LateNight,      // 00:00-04:00 - Deep sleep/repair
    DeepNight,      // 04:00-06:00 - Cortisol rising, temp nadir
    EarlyMorning,   // 06:00-09:00 - Wake window, light exposure critical
    Morning,        // 09:00-12:00 - Peak focus, high cortisol
    Afternoon,      // 12:00-15:00 - Physical performance
    LateAfternoon,  // 15:00-18:00 - Strength peak
    Evening,        // 18:00-21:00 - Wind down, temp dropping
    Night,          // 21:00-00:00 - Sleep onset, melatonin rising
}
```

### Circadian Logic

```rust
fn circadian_tick(state: &mut BodyState, env: &Environment) {
    // PHASE DETERMINATION
    state.circ.phase = match env.clock_time {
        0..=239 => CircadianPhase::LateNight,
        240..=359 => CircadianPhase::DeepNight,
        360..=539 => CircadianPhase::EarlyMorning,
        540..=719 => CircadianPhase::Morning,
        720..=899 => CircadianPhase::Afternoon,
        900..=1079 => CircadianPhase::LateAfternoon,
        1080..=1259 => CircadianPhase::Evening,
        1260..=1439 => CircadianPhase::Night,
        _ => CircadianPhase::LateNight,
    };

    // CORTISOL RHYTHM
    state.circ.cortisol_mult = match state.circ.phase {
        CircadianPhase::EarlyMorning => 1.5,  // Peak (+50%)
        CircadianPhase::Morning => 1.3,
        CircadianPhase::Afternoon => 1.0,
        CircadianPhase::LateAfternoon => 0.8,
        CircadianPhase::Evening => 0.6,
        CircadianPhase::Night => 0.4,
        CircadianPhase::LateNight => 0.3,
        CircadianPhase::DeepNight => 0.5,  // Rising
    };

    // CORE TEMPERATURE RHYTHM (nadir ~4am, peak ~6pm)
    let temp_phase = (env.clock_time as f32 - 240.0) / 1440.0 * 2.0 * PI;
    state.circ.core_temp = 36.8 + 0.5 * temp_phase.sin();

    // LIGHT EXPOSURE EFFECTS
    if env.light_lux > 10_000.0 {
        state.circ.light_exposure += env.light_lux / 60.0;  // Lux-minutes

        // Morning light within 2h of wake → cortisol spike
        if state.circ.hours_awake < 2.0 {
            state.ans.cortisol += 0.3;  // Morning spike
            state.horm.melatonin *= 0.5;  // Suppress melatonin
        }
    }

    // MELATONIN ONSET (dim light in evening)
    if state.circ.phase == CircadianPhase::Evening && env.light_lux < 100.0 {
        state.horm.melatonin += 0.1;
    }

    // Blue light in evening delays melatonin
    if state.circ.phase == CircadianPhase::Evening && env.light_lux > 500.0 {
        state.horm.melatonin *= 0.9;
    }
}
```

### Outputs (Modifies)
| Writes To | Variable | Effect |
|-----------|----------|--------|
| `ans` | `cortisol` | Circadian modulation |
| `horm` | `melatonin` | Sleep pressure |
| `thermo` | `core_temp` | Temperature rhythm |
| `meta` | `insulin_sensitivity` | Better morning than evening |

### Key Protocols
- **Morning light (100k lux within 30min of wake)**: Sets clock, cortisol spike
- **Dim evening lights**: Allows melatonin onset
- **Consistent wake time**: Anchors rhythm
- **2 days proper light**: Resets drifted rhythm

---

## System: Ultradian

90-minute cycles throughout the day (BRAC - Basic Rest-Activity Cycle).

### State Variables
| Variable | Rust Path | Notes |
|----------|-----------|-------|
| `phase` | `ultra.phase` | Minutes into current cycle (0-90) |
| `state` | `ultra.state` | peak/plateau/trough/recovery |
| `focus_capacity` | `ultra.focus_capacity` | Current cognitive capacity |
| `rest_need` | `ultra.rest_need` | Accumulated need for break |

### Ultradian Logic

```rust
fn ultradian_tick(state: &mut BodyState) {
    // Advance phase
    state.ultra.phase = (state.ultra.phase + 1) % 90;

    // Determine cycle state
    state.ultra.state = match state.ultra.phase {
        0..=19 => UltradianState::Peak,      // High alertness
        20..=49 => UltradianState::Plateau,   // Sustained focus
        50..=69 => UltradianState::Trough,    // Dreamy, creative
        70..=89 => UltradianState::Recovery,  // Transition
        _ => UltradianState::Peak,
    };

    // Focus capacity follows cycle
    state.ultra.focus_capacity = match state.ultra.state {
        UltradianState::Peak => 1.0,
        UltradianState::Plateau => 0.9,
        UltradianState::Trough => 0.5,
        UltradianState::Recovery => 0.7,
    };

    // Rest need accumulates in trough if ignored
    if state.ultra.state == UltradianState::Trough {
        state.ultra.rest_need += 0.02;
    }

    // Taking breaks clears rest need
    // (handled in activity processing)
}
```

### Outputs (Modifies)
| Writes To | Variable | Effect |
|-----------|----------|--------|
| `attn` | `focus_duration` | Modulated by cycle |
| `dim` | `focus` | Cycle affects focus score |

### Best Practices
- Work WITH cycles, not against them
- Take breaks during trough phase
- Schedule deep work for peak/plateau
- Creative work during trough

---

## System: Autonomic Nervous System

The sympathetic/parasympathetic balance that governs stress response.

### State Variables
| Variable | Rust Path | Notes |
|----------|-----------|-------|
| `sympathetic` | `ans.sympathetic` | Fight/flight (0-1) |
| `parasympathetic` | `ans.parasympathetic` | Rest/digest (0-1) |
| `hrv` | `ans.hrv` | Heart rate variability (ms) |
| `cortisol` | `ans.cortisol` | Stress hormone (0-1) |

### Inputs (Dependencies)
| Reads From | Variable | Purpose |
|------------|----------|---------|
| `visual` | `mode` | Focal → sympathetic |
| `meta` | `glucose` | Low glucose → stress |
| `circ` | `cortisol_mult` | Circadian modulation |
| Activities | Exercise, stress, meditation | Direct effects |

### Autonomic Logic

```rust
fn autonomic_tick(state: &mut BodyState) {
    // VISUAL MODE EFFECTS (Huberman lever)
    match state.visual.mode {
        VisualMode::Focal => {
            state.ans.sympathetic += 0.02;
            state.ans.parasympathetic -= 0.01;
        }
        VisualMode::Panoramic => {
            state.ans.parasympathetic += 0.02;
            state.ans.sympathetic -= 0.01;
        }
    }

    // OPTIC FLOW (forward movement calms)
    if state.visual.optic_flow > 0.5 {
        state.ans.sympathetic -= 0.05;
    }

    // GLUCOSE STRESS
    if state.meta.glucose < 60.0 {
        state.ans.sympathetic += 0.1;
        state.ans.cortisol += 0.1;
    }

    // HRV CALCULATION
    // Higher para, lower symp → higher HRV
    let baseline_hrv = 60.0;  // Individual baseline
    state.ans.hrv = baseline_hrv
        * state.ans.parasympathetic
        / (state.ans.sympathetic + 0.1);
    state.ans.hrv = state.ans.hrv.clamp(10.0, 150.0);

    // CORTISOL CIRCADIAN MODULATION
    state.ans.cortisol *= state.circ.cortisol_mult;

    // NATURAL DECAY toward balance
    state.ans.sympathetic *= 0.98;
    state.ans.parasympathetic *= 0.98;

    // Ensure some baseline activation
    state.ans.sympathetic = state.ans.sympathetic.max(0.1);
    state.ans.parasympathetic = state.ans.parasympathetic.max(0.1);
}
```

### Outputs (Modifies)
| Writes To | Variable | Effect |
|-----------|----------|--------|
| `cardio` | `hr` | Sympathetic ↑ HR |
| `digest` | `motility` | Para ↑ digestion |
| `immune` | `wbc` | Chronic cortisol ↓ immune |
| `dim` | `stress` | Primary stress indicator |

### Key Interventions
| Intervention | Effect | Speed |
|--------------|--------|-------|
| Physiological sigh | ↓sympathetic, ↑para | Seconds |
| Box breathing | Balance | Minutes |
| Panoramic vision | ↑para | Immediate |
| Walking (optic flow) | ↓sympathetic | Minutes |
| Cold exposure | Acute ↑symp → rebound ↑para | Minutes-hours |

---

## System: Hormonal

Signaling molecules that coordinate systems across the body.

### State Variables
| Variable | Rust Path | Notes |
|----------|-----------|-------|
| `dopamine_baseline` | `horm.dopamine_baseline` | Tonic level |
| `dopamine_phasic` | `horm.dopamine_phasic` | Acute spikes |
| `norepinephrine` | `horm.norepinephrine` | Alertness |
| `adrenaline` | `horm.adrenaline` | Acute stress |
| `orexin` | `horm.orexin` | Wakefulness |
| `testosterone` | `horm.testosterone` | ng/dL |
| `prolactin` | `horm.prolactin` | Post-ejac spike |
| `retention_days` | `horm.retention_days` | Days since ejaculation |
| `serotonin` | `horm.serotonin` | Mood |
| `melatonin` | `horm.melatonin` | Sleep |
| `bdnf` | `horm.bdnf` | Brain plasticity |
| `gh` | `horm.gh` | Growth hormone |

### Catecholamine Synthesis

```
L-Tyrosine → L-DOPA → Dopamine → Norepinephrine → Adrenaline
```

### Dopamine Dynamics (Critical for ADHD)

```
The dopamine system has THREE key components:

1. BASELINE (tonic): The steady-state level
   - Sets reward threshold
   - High baseline = content, patient
   - Low baseline = seeking, impulsive

2. RECEPTOR DENSITY: Number of D2 receptors
   - Downregulates with frequent stimulation (porn, social media, etc.)
   - Recovery: weeks to months of abstinence
   - ADHD: often lower baseline density

3. REUPTAKE RATE (DAT): How fast dopamine clears the synapse
   - High DAT = dopamine clears quickly = need more hits
   - ADHD: Often higher DAT activity = chronically low effective dopamine
   - This is why stimulants BLOCK reuptake → dopamine stays longer

The "effective dopamine" = baseline × receptor_density × receptor_sensitivity / reuptake_rate
```

### Hormonal Logic

```rust
fn hormonal_tick(state: &mut BodyState) {
    // DOPAMINE DYNAMICS
    // Phasic spikes decay quickly
    state.horm.dopamine_phasic *= 0.9;

    // REUPTAKE: Higher rate = faster clearance of phasic dopamine
    // ADHD typically has higher reuptake (DAT hyperactivity)
    let reuptake_factor = state.horm.dopamine_reuptake;
    state.horm.dopamine_phasic *= 1.0 - (0.1 * reuptake_factor);

    // RECEPTOR DENSITY: Downregulates with frequent high-reward
    // Recovery is SLOW (weeks)
    let time_since_reward = state.horm.last_reward_time as f32;
    if time_since_reward < 60.0 {
        // Recent reward → slight downregulation
        state.horm.receptor_density *= 0.9999;
    } else if time_since_reward > 4320.0 {
        // 3+ days abstinence → slow recovery (peaks day 4-6)
        state.horm.receptor_density += 0.0001;
    }
    state.horm.receptor_density = state.horm.receptor_density.clamp(0.3, 1.0);

    // RECEPTOR SENSITIVITY: Faster recovery than density
    // Upregulates with abstinence from high-reward activities
    if time_since_reward > 1440.0 {
        // 1+ days → sensitivity starts recovering
        state.horm.receptor_sensitivity += 0.001;
    }
    state.horm.receptor_sensitivity = state.horm.receptor_sensitivity.clamp(0.2, 1.0);

    // SEEKING BEHAVIOR: Inverse of effective dopamine
    // High seeking = restless, distractible, scrolling, craving
    let effective_dopamine = state.horm.dopamine_baseline
        * state.horm.receptor_density
        * state.horm.receptor_sensitivity
        / state.horm.dopamine_reuptake;
    state.horm.seeking = (1.0 - effective_dopamine).clamp(0.0, 1.0);

    // Baseline affected by sleep, retention, activities
    // (mostly set by activities)

    // NOREPINEPHRINE
    // Decays toward baseline
    state.horm.norepinephrine *= 0.95;

    // OREXIN (wakefulness)
    // ↑ with fasting, light, exercise
    // ↓ with high carb meal, sleep debt
    if state.meta.glucose > 120.0 {
        state.horm.orexin *= 0.95;  // Post-meal drowsy
    }
    if state.energy.sleep_debt > 4.0 {
        state.horm.orexin *= 0.9;
    }

    // TESTOSTERONE RHYTHM (peaks morning)
    if state.circ.phase == CircadianPhase::EarlyMorning {
        state.horm.testosterone *= 1.1;
    }

    // RETENTION EFFECTS
    state.horm.retention_days += 1.0 / 1440.0;  // Per minute
    let days = state.horm.retention_days as u16;

    // PROLACTIN DYNAMICS (see also: 03_ACTIVITIES.md for spike on ejaculation)
    // Spike: 4.0 (400% of baseline) immediately post-ejac
    // Decay: ~2 weeks to return to baseline
    // Half-life: ~3-4 days
    // Effect: Antagonizes dopamine, creates "satisfied/sleepy/dad mode"
    let prolactin_half_life_mins = 5760.0;  // ~4 days in minutes
    let prolactin_decay = 0.5_f32.powf(1.0 / prolactin_half_life_mins);
    state.horm.prolactin = 1.0 + (state.horm.prolactin - 1.0) * prolactin_decay;

    // TESTOSTERONE DYNAMICS with retention
    match days {
        0..=2 => {
            // Post-ejac: testosterone suppressed slightly
            // Prolactin is antagonizing it
        }
        3..=6 => {
            // Building toward peak
            state.horm.testosterone *= 1.001;
        }
        7 => {
            // Day 7 peak: +45% from baseline
            state.horm.testosterone *= 1.003;
        }
        8..=14 => {
            // Stabilizes at elevated level (~15% above baseline)
        }
        _ => {
            // Long-term retention: sustained elevated baseline
        }
    }

    // ANDROGEN RECEPTOR SENSITIVITY increases with retention
    // This is why effects compound - same T hits harder
    if days > 7 {
        state.horm.receptor_sensitivity += 0.0001;
    }

    // MELATONIN (handled in circadian)

    // BDNF decay (boosted by exercise, fasting)
    state.horm.bdnf *= 0.999;

    // GROWTH HORMONE (peaks in deep sleep, fasting)
    if state.digest.state == DigestiveState::FastedDeep {
        state.horm.gh += 0.01;
    }
    state.horm.gh *= 0.99;
}
```

### Outputs (Modifies)
| Writes To | Variable | Effect |
|-----------|----------|--------|
| `dim` | `focus` | Dopamine, norepinephrine |
| `dim` | `libido` | Testosterone, prolactin |
| `dim` | `mood` | Serotonin, dopamine |
| `energy` | `perceived` | Orexin |
| `memory` | `plasticity_window` | BDNF |

### Key Hormone Triggers

| Trigger | Hormones Affected |
|---------|-------------------|
| Cold exposure | ↑NE (+530%), ↑DA (+250%) |
| NSDR | ↑DA baseline (+65%) |
| HIIT | ↑T, ↑adrenaline, ↑BDNF |
| Fasting 16h+ | ↑NE, ↑GH, ↑BDNF |
| Ejaculation | ↑↑prolactin, ↓↓dopamine |
| Morning light | ↑cortisol (+50%) |
| High carb meal | ↓orexin (drowsy) |

---

## System: Thermoregulation

Temperature control including BAT (brown adipose tissue).

### State Variables
| Variable | Rust Path | Notes |
|----------|-----------|-------|
| `core_temp` | `thermo.core_temp` | 36.5-37.5°C normal |
| `skin_temp` | `thermo.skin_temp` | Surface temp |
| `sweating` | `thermo.sweating` | Activation 0-1 |
| `shivering` | `thermo.shivering` | Activation 0-1 |
| `vasodilation` | `thermo.vasodilation` | Heat dissipation |
| `vasoconstriction` | `thermo.vasoconstriction` | Heat conservation |
| `bat_activation` | `thermo.bat_activation` | Brown fat activity |
| `bat_mass` | `thermo.bat_mass` | Trainable (grams) |

### Inputs (Dependencies)
| Reads From | Variable | Purpose |
|------------|----------|---------|
| `env` | `temperature_c` | Ambient temp |
| `circ` | `core_temp` | Circadian baseline |
| Activities | Cold exposure, exercise | Direct effects |

### Thermoregulation Logic

```rust
fn thermo_tick(state: &mut BodyState, env: &Environment) {
    // COLD EXPOSURE
    if env.temperature_c < 15.0 {
        let cold_stress = (15.0 - env.temperature_c) / 15.0;

        // Vasoconstriction (immediate)
        state.thermo.vasoconstriction += cold_stress * 0.2;

        // BAT activation (norepinephrine-mediated)
        state.thermo.bat_activation += cold_stress * 0.1;

        // Shivering (if BAT insufficient)
        if state.thermo.bat_activation < 0.5 {
            state.thermo.shivering += cold_stress * 0.1;
        }

        // Hormonal response
        state.horm.norepinephrine += cold_stress * 0.3;
        state.horm.dopamine_baseline += cold_stress * 0.15;
    }

    // HEAT STRESS
    if env.temperature_c > 30.0 || state.thermo.core_temp > 37.5 {
        let heat_stress = ((env.temperature_c - 25.0) / 15.0).max(0.0);

        state.thermo.sweating += heat_stress * 0.2;
        state.thermo.vasodilation += heat_stress * 0.15;
    }

    // BAT THERMOGENESIS (burns calories for heat)
    if state.thermo.bat_activation > 0.3 {
        let heat_production = state.thermo.bat_activation * state.thermo.bat_mass / 100.0;
        state.thermo.core_temp += heat_production * 0.01;
        // Also burns calories (handled in body composition)
    }

    // BAT MASS ADAPTATION (long-term)
    if state.thermo.bat_activation > 0.5 {
        state.thermo.bat_mass += 0.01;  // Slow growth with repeated cold
    }

    // DECAY toward neutral
    state.thermo.sweating *= 0.95;
    state.thermo.shivering *= 0.95;
    state.thermo.vasodilation *= 0.95;
    state.thermo.vasoconstriction *= 0.95;
    state.thermo.bat_activation *= 0.98;

    // CORE TEMP regulation (homeostasis)
    // Handled in homeostasis pass
}
```

### Outputs (Modifies)
| Writes To | Variable | Effect |
|-----------|----------|--------|
| `horm` | `norepinephrine` | Cold → ↑NE |
| `horm` | `dopamine_baseline` | Cold → ↑DA |
| `hydra` | (electrolytes) | Sweating loses Na, K |
| `comp` | (calories) | BAT burns energy |

### Cold Exposure Protocol
```
Target: 11 minutes/week total, uncomfortable but safe

Temperature/Duration tradeoff:
- Colder water = shorter time needed
- 10°C for 2-3 min ≈ 15°C for 5-6 min

Hormonal effects:
- Norepinephrine: +530%
- Dopamine: +250% (lasts 3-5 hours)

Long-term adaptation:
- ↑BAT mass
- ↑Cold tolerance
- ↑Baseline metabolism
```

---

## System: Energy & Fatigue

Adenosine dynamics, sleep pressure, and perceived energy.

### State Variables
| Variable | Rust Path | Notes |
|----------|-----------|-------|
| `atp` | `energy.atp` | Cellular energy |
| `perceived` | `energy.perceived` | Subjective 0-100 |
| `sleep_debt` | `energy.sleep_debt` | Accumulated hours |
| `adenosine` | `energy.adenosine` | Sleep pressure |
| `receptors_available` | `energy.receptors_available` | Free adenosine receptors |
| `receptors_blocked` | `energy.receptors_blocked` | Blocked by caffeine |
| `adenosine_buildup` | `energy.adenosine_buildup` | Behind caffeine block |
| `muscle_fatigue` | `energy.muscle_fatigue` | Physical fatigue |
| `central_fatigue` | `energy.central_fatigue` | CNS fatigue |

### Energy Logic

```rust
fn energy_tick(state: &mut BodyState) {
    // ADENOSINE ACCUMULATES while awake
    if !state.is_sleeping {
        state.energy.adenosine += 0.001;  // ~0.06/hour
    }

    // ADENOSINE BINDS to available receptors → sleepiness
    let binding = state.energy.adenosine * state.energy.receptors_available;
    // This binding creates sleepiness (reduces perceived energy)

    // CAFFEINE blocks receptors
    // (handled in activity processing)
    // When blocked: adenosine can't bind but keeps building up

    // ADENOSINE BUILDUP (behind caffeine block)
    if state.energy.receptors_blocked > 0.3 {
        state.energy.adenosine_buildup += state.energy.adenosine * 0.1;
    }

    // CAFFEINE WEARS OFF (half-life ~5-6 hours)
    state.energy.receptors_blocked *= 0.998;  // ~6h half-life

    // When caffeine wears off, buildup floods in
    if state.energy.receptors_blocked < 0.2 && state.energy.adenosine_buildup > 0.3 {
        state.energy.adenosine += state.energy.adenosine_buildup * 0.3;
        state.energy.adenosine_buildup *= 0.7;
        // This is the "caffeine crash"
    }

    // SLEEP clears adenosine
    if state.is_sleeping {
        state.energy.adenosine *= 0.95;  // Rapid clearance
        state.energy.adenosine_buildup *= 0.9;
        state.energy.sleep_debt -= 0.01;  // ~1h per 100 min sleep
    }

    // PERCEIVED ENERGY calculation
    let adenosine_effect = 1.0 - (state.energy.adenosine * state.energy.receptors_available);
    let sleep_effect = 1.0 - (state.energy.sleep_debt / 16.0);
    let caffeine_boost = state.energy.receptors_blocked * 0.3;

    state.energy.perceived = (
        adenosine_effect * 40.0
        + sleep_effect * 30.0
        + caffeine_boost * 20.0
        + state.energy.atp * 10.0
    ).clamp(0.0, 100.0);

    // FATIGUE RECOVERY
    state.energy.muscle_fatigue *= 0.99;
    state.energy.central_fatigue *= 0.995;
}
```

### Outputs (Modifies)
| Writes To | Variable | Effect |
|-----------|----------|--------|
| `dim` | `energy` | Primary energy score |
| `dim` | `focus` | Adenosine affects focus |

### Caffeine Protocol
```
Optimal timing:
1. Wait 90-120 min after waking
2. Morning cortisol clears residual adenosine naturally
3. Caffeine then blocks empty receptors
4. No adenosine buildup → no crash

Half-life: ~5-6 hours
- 100mg at 8am → 50mg at 2pm → 25mg at 8pm
- Avoid caffeine within 8-10h of sleep
```

---

## System: Hunger & Satiety

Ghrelin/leptin dynamics and appetite regulation.

### State Variables
| Variable | Rust Path | Notes |
|----------|-----------|-------|
| `ghrelin` | `hunger.ghrelin` | Hunger hormone |
| `leptin` | `hunger.leptin` | Satiety hormone |
| `hunger_score` | `hunger.hunger_score` | Perceived 0-100 |
| `satiety_score` | `hunger.satiety_score` | Perceived 0-100 |
| `leptin_sensitivity` | `hunger.leptin_sensitivity` | Can develop resistance |

### Hunger Logic

```rust
fn hunger_tick(state: &mut BodyState) {
    // GHRELIN rises with empty stomach
    if state.digest.fullness < 0.2 {
        state.hunger.ghrelin += 0.01;
    }

    // GHRELIN falls after eating (20 min delay simulated)
    if state.digest.fullness > 0.5 {
        state.hunger.ghrelin *= 0.95;
    }

    // LEPTIN proportional to fat mass
    state.hunger.leptin = (state.comp.adipose_mass / 20.0).min(1.0);

    // LEPTIN RESISTANCE develops with chronic high leptin
    if state.hunger.leptin > 0.8 {
        state.hunger.leptin_sensitivity *= 0.9999;  // Slow degradation
    }

    // KETONES suppress appetite
    let keto_suppression = state.meta.ketones * 0.2;

    // SLEEP affects hunger hormones
    if state.energy.sleep_debt > 4.0 {
        state.hunger.ghrelin *= 1.28;   // +28% with poor sleep
        state.hunger.leptin *= 0.82;     // -18%
    }

    // Calculate scores
    state.hunger.hunger_score = (
        state.hunger.ghrelin * 60.0
        - state.hunger.leptin * state.hunger.leptin_sensitivity * 30.0
        - state.digest.fullness * 40.0
        - keto_suppression * 20.0
    ).clamp(0.0, 100.0);

    state.hunger.satiety_score = 100.0 - state.hunger.hunger_score;
}
```

### Outputs (Modifies)
| Writes To | Variable | Effect |
|-----------|----------|--------|
| `dim` | `hunger` | Primary hunger score |
| `horm` | `dopamine_phasic` | Ghrelin → anticipatory reward |

---

## System: Attention & Cognitive

The gamma function, flow states, and activation energy.

### State Variables
| Variable | Rust Path | Notes |
|----------|-----------|-------|
| `gamma` | `attn.gamma` | Motivation curve steepness |
| `in_flow` | `attn.in_flow` | Currently in flow |
| `activation_energy` | `attn.activation_energy` | Task start threshold |
| `focus_duration` | `attn.focus_duration` | Sustained attention |

### Gamma Function (ADHD Model)

```rust
// motivation = input^gamma
//
// gamma ≈ 1.0: Neurotypical (linear response)
// gamma > 2.5: High-gamma brain (needs "Level 10" to activate)
//
// High-gamma individuals:
// - Low signals crushed to zero
// - Only extreme stimuli trigger action
// - Explains ADHD hyperfocus on interesting tasks

fn motivation(input: f32, gamma: f32) -> f32 {
    input.powf(gamma)
}
```

### Cognitive Degradation Hierarchy

```
ATP is LIMITED. When depleted, brain triages functions:

FIRST TO DEGRADE (most expensive):
1. Social cognition (~20% of brain ATP when active)
   - "Can't greet people" when tired
   - Requires BOTH systems online

SECOND TO DEGRADE:
2. Executive function (~15%)
   - System 2 thinking
   - Complex planning
   - "Can't hold complex thoughts"

THIRD TO DEGRADE:
3. Language fluency (~12%)
   - Word-finding difficulty
   - Slower speech

FOURTH TO DEGRADE:
4. Memory formation (~10%)
   - Can still retrieve
   - Can't form new memories well

LAST TO DEGRADE (protected):
5. Motor control (~8%)
   - "Can sprint but can't talk" when exhausted
   - Crisis response actually ENHANCED

This explains why tired feels like "can still do physical stuff, can't think"
```

### Attention Logic

```rust
fn attention_tick(state: &mut BodyState) {
    // COGNITIVE CAPACITY DEGRADATION
    // Based on sleep debt, adenosine, ATP
    let depletion = (state.energy.sleep_debt / 8.0
        + state.energy.adenosine
        + (1.0 - state.energy.atp))
        / 3.0;

    // DEGRADATION HIERARCHY (first to fail → last)
    state.attn.social_capacity = (1.0 - depletion * 2.0).clamp(0.0, 1.0);
    state.attn.executive_function = (1.0 - depletion * 1.5).clamp(0.0, 1.0);
    state.attn.motor_precision = (1.0 - depletion * 0.3).clamp(0.5, 1.0);

    // MOTOR PRECISION also boosted by retention (dopamine → fine motor control)
    if state.horm.retention_days > 7.0 {
        state.attn.motor_precision += 0.1;
    }
    state.attn.motor_precision = state.attn.motor_precision.clamp(0.0, 1.0);

    // ACTIVATION ENERGY
    // Higher dopamine → lower barrier to start
    // Higher gamma (ADHD) → needs more activation to start
    let effective_dopamine = state.horm.dopamine_baseline
        * state.horm.receptor_density
        * state.horm.receptor_sensitivity;
    state.attn.activation_energy =
        0.5 / effective_dopamine.max(0.1) * state.attn.gamma;

    // EXPLORATION vs EXPLOITATION
    // ADHD: naturally high exploration, low exploitation capacity
    // Dopamine baseline affects the balance:
    // - Low dopamine = high exploration (seeking novelty)
    // - High dopamine = can exploit (content with current path)
    state.attn.exploration = state.horm.seeking;  // Seeking = exploring
    state.attn.exploitation = effective_dopamine;

    // TIME PREFERENCE / DISCOUNT RATE
    // Low dopamine → high discount rate → "I want it NOW"
    // High dopamine → low discount rate → "I can wait for better"
    // Also affected by testosterone (high T → more present bias)
    state.attn.discount_rate = (1.0 - effective_dopamine
        + state.horm.testosterone / 1000.0 * 0.1)
        .clamp(0.0, 1.0);
    state.attn.time_horizon = 24.0 / (state.attn.discount_rate + 0.1);

    // FLOW STATE DETECTION
    // Requires: high dopamine + high norepinephrine + low cortisol + skill-challenge match
    let flow_conditions =
        effective_dopamine > 0.5
        && state.horm.norepinephrine > 0.5
        && state.ans.cortisol < 0.6
        && state.ultra.state != UltradianState::Trough;

    state.attn.in_flow = flow_conditions;

    // FOCUS DURATION modulated by ultradian and executive function
    state.attn.focus_duration = 60.0
        * state.ultra.focus_capacity
        * state.attn.executive_function;
}
```

### Flow State Quadrant
```
                HIGH CHALLENGE
                      │
          [Anxiety]   │   [FLOW]
                      │
LOW SKILL ────────────┼──────────── HIGH SKILL
                      │
         [Apathy]     │   [Boredom]
                      │
                LOW CHALLENGE
```

### Outputs (Modifies)
| Writes To | Variable | Effect |
|-----------|----------|--------|
| `dim` | `focus` | Flow state, activation |
| `percept` | `frame_rate` | Flow → higher frame rate |

---

## System: Perceptual

Time perception and cognitive sampling rate.

### State Variables
| Variable | Rust Path | Notes |
|----------|-----------|-------|
| `frame_rate` | `percept.frame_rate` | Perceptual sampling (fps equiv) |
| `quantization` | `percept.quantization` | binary/gradient mode |

### Perceptual Logic

```rust
fn perceptual_tick(state: &mut BodyState) {
    // FRAME RATE scales with norepinephrine (tachypsychia)
    state.percept.frame_rate = 30.0 + state.horm.norepinephrine * 90.0;

    // In flow/high alert: time seems slower (higher sampling)
    if state.attn.in_flow {
        state.percept.frame_rate *= 1.5;
    }

    state.percept.frame_rate = state.percept.frame_rate.clamp(20.0, 200.0);
}
```

### Tachypsychia
| State | Norepinephrine | Frame Rate | Experience |
|-------|----------------|------------|------------|
| Low alert | Low | ~30 fps | Normal time |
| High alert | High | ~60-120 fps | Time slows |
| Flow/Peak | Very high | ~120+ fps | "Slow motion" |

---

## System: Memory & Learning

Neuroplasticity, encoding strength, and the "commit during sleep" model.

### The Two-Phase Model

```
LEARNING = ENCODING (awake) + CONSOLIDATION (sleep)
════════════════════════════════════════════════════════════════════

1. ENCODING (while focused)
   ─────────────────────────
   - Creates "uncommitted" memory traces
   - Strength depends on: attention, BDNF, testing vs passive, spacing
   - Uncommitted traces DECAY if not consolidated

2. CONSOLIDATION (during sleep)
   ────────────────────────────
   - SWS (deep sleep): Declarative memories → cortex
   - REM: Procedural/emotional integration
   - Clears uncommitted_load, makes memories permanent

   IF sleep skipped → uncommitted traces lost
   This is why "sleep on it" works

THE FORGETTING CURVE:
   - Lose ~50% in first hour if not consolidated
   - Spaced retrieval resets the curve
   - Testing yourself = 4x stronger encoding than passive reading
```

### State Variables
| Variable | Rust Path | Notes |
|----------|-----------|-------|
| `working` | `memory.working` | Active capacity (RAM) |
| `state` | `memory.state` | encoding/consolidating/retrieving |
| `plasticity_window` | `memory.plasticity_window` | Enhanced learning possible |
| `encoding_strength` | `memory.encoding_strength` | The effective K - learning rate NOW |
| `uncommitted_load` | `memory.uncommitted_load` | Pending consolidation |
| `consolidation_rate` | `memory.consolidation_rate` | How fast sleep commits |
| `testing_mult` | `memory.testing_mult` | Active recall multiplier |
| `spacing_factor` | `memory.spacing_factor` | Distributed practice bonus |
| `forgetting_rate` | `memory.forgetting_rate` | Uncommitted decay rate |

### Memory Logic

```rust
fn memory_tick(state: &mut BodyState) {
    // WORKING MEMORY (RAM) - affected by sleep debt
    state.memory.working = (1.0 - state.energy.sleep_debt / 12.0).clamp(0.3, 1.0);

    // NEUROPLASTICITY WINDOW
    let plasticity_active =
        state.horm.bdnf > 0.6      // Exercise/fasting boost
        || state.attn.in_flow      // Focus opens window
        || state.is_sleeping;       // Sleep consolidates

    state.memory.plasticity_window = plasticity_active;

    // ENCODING STRENGTH (The effective K)
    // This is how well you learn RIGHT NOW
    if !state.is_sleeping {
        let attention_factor = state.attn.executive_function;
        let alertness_factor = state.horm.norepinephrine.min(1.0);
        let plasticity_factor = if state.horm.bdnf > 0.5 { 1.0 + state.horm.bdnf } else { 1.0 };
        let sleep_penalty = 1.0 - (state.energy.sleep_debt / 16.0).min(0.5);

        state.memory.encoding_strength = (
            attention_factor
            * alertness_factor
            * plasticity_factor
            * sleep_penalty
            * state.memory.testing_mult
            * state.memory.spacing_factor
        ).clamp(0.0, 1.0);
    } else {
        state.memory.encoding_strength = 0.0;  // Not encoding while asleep
    }

    // SPACING FACTOR - time since last encoding session
    let hours_since_encoding = (state.time - state.memory.last_encoding) as f32 / 60.0;
    state.memory.spacing_factor = match hours_since_encoding {
        0.0..=0.5 => 0.5,    // Massed practice - diminishing returns
        0.5..=2.0 => 0.8,    // Too close
        2.0..=8.0 => 1.2,    // Good spacing
        8.0..=24.0 => 1.5,   // Optimal
        24.0..=72.0 => 2.0,  // Spaced repetition sweet spot
        _ => 1.0,            // Very long gap - reset
    };

    // FORGETTING - uncommitted traces decay
    if !state.is_sleeping && state.memory.uncommitted_load > 0.0 {
        // Decay rate: ~50% in first hour without consolidation
        state.memory.forgetting_rate = 0.01 * (1.0 + state.energy.sleep_debt / 8.0);
        state.memory.uncommitted_load *= 1.0 - state.memory.forgetting_rate;
    }

    // CONSOLIDATION STATE
    if state.is_sleeping {
        state.memory.state = MemoryState::Consolidating;

        // Consolidation rate depends on sleep quality
        let sleep_quality = match state.sleep_stage {
            SleepStage::Deep => 1.0,    // SWS: Best for declarative
            SleepStage::REM => 0.7,     // REM: Procedural/emotional
            SleepStage::Light => 0.3,   // N1/N2: Some consolidation
            _ => 0.0,
        };

        state.memory.consolidation_rate = sleep_quality * (1.0 - state.energy.sleep_debt / 24.0);

        // Actually consolidate - reduce uncommitted load
        let consolidated = state.memory.uncommitted_load * state.memory.consolidation_rate * 0.1;
        state.memory.uncommitted_load -= consolidated;
        state.memory.uncommitted_load = state.memory.uncommitted_load.max(0.0);

    } else if state.attn.in_flow || state.attn.executive_function > 0.7 {
        state.memory.state = MemoryState::Encoding;

        // Encoding adds to uncommitted load
        // (actual amount added depends on activity - see 03_ACTIVITIES.md)

    } else {
        state.memory.state = MemoryState::Retrieving;
    }
}
```

### Testing Effect

```
The Testing Revolution (Huberman/research):

PASSIVE (reading, watching):     retention = 20%
ACTIVE (self-testing):           retention = 80%

WHY: Testing creates retrieval practice, which:
1. Strengthens memory traces (more connections)
2. Identifies gaps (know what you don't know)
3. Resets forgetting curve
4. Activates deeper encoding

testing_mult values:
- Passive consumption: 1.0
- Note-taking: 1.5
- Teaching others: 2.5
- Self-testing: 3.0-4.0
```

### Gap Effects

```
MICRO-BREAKS DURING LEARNING:

Continuous focus:  ████████████████████████
                   Learning flattens, fatigue builds

With gaps:         ████░░████░░████░░████░░
                   Brain replays during gaps (10-20x speed)
                   Actually ACCELERATES learning

Optimal: 10-second breaks every few minutes during intense learning
The brain uses gaps to rehearse and consolidate in real-time
```

### Plasticity Triggers
| Trigger | Duration | Mechanism | K Multiplier |
|---------|----------|-----------|--------------|
| Focused attention | During focus | Acetylcholine | 1.3x |
| Exercise | 1-2 hours | BDNF ↑ | 1.5x |
| Sleep (SWS) | During deep sleep | Consolidation | N/A (commits) |
| Sleep (REM) | During REM | Pattern integration | N/A (commits) |
| Novelty | Variable | Dopamine-enhanced | 1.4x |
| Testing/recall | During test | Retrieval practice | 3-4x |
| Fasting | While fasted | BDNF ↑ | 1.3x |
| Cold exposure | 1-2 hours | NE + BDNF | 1.4x |

### The "Sleep On It" Effect

```rust
// Why you MUST sleep to learn:

fn learning_outcome(encoded: f32, sleep_quality: f32, sleep_hours: f32) -> f32 {
    let consolidation = (sleep_quality * sleep_hours / 8.0).min(1.0);

    // Without consolidation, most is lost
    let retained = encoded * (0.2 + 0.8 * consolidation);

    // This is why all-nighters before exams FAIL
    // You encoded it but never committed it

    retained
}
```

### Outputs (Modifies)
| Writes To | Variable | Effect |
|-----------|----------|--------|
| `attn` | `focus_duration` | Better memory = longer focus |
| (external) | Long-term memory | Consolidated knowledge |
