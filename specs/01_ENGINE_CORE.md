# Engine Core - Homeostatic Machinery

The foundational systems that maintain physiological balance. These are the "engines" that all other systems depend on.

> **Key Concept:** These systems "fight" to maintain setpoints. When a variable drifts, they actively correct it.

---

## System: Renal (Kidneys)

Regulates water, electrolytes, and blood pressure via urine production and hormone cascades.

### State Variables
| Variable | Rust Path | Notes |
|----------|-----------|-------|
| `gfr` | `renal.gfr` | Glomerular filtration rate |
| `urine_production` | `renal.urine_production` | mL/hour output |
| `urine_concentration` | `renal.urine_concentration` | mOsm/kg |
| `aldosterone` | `renal.aldosterone` | Na+ retention hormone |
| `adh` | `renal.adh` | Water retention hormone |
| `renin` | `renal.renin` | RAAS cascade start |
| `angiotensin_ii` | `renal.angiotensin_ii` | Vasoconstriction |

### Inputs (Dependencies)
| Reads From | Variable | Purpose |
|------------|----------|---------|
| `meta` | `insulin` | Low insulin → reduced aldosterone |
| `hydra` | `osmolality` | Triggers ADH response |
| `cardio` | `systolic` | Triggers RAAS if low |
| `hydra` | `potassium` | High K+ → aldosterone release |

### Homeostasis Logic

```rust
fn renal_homeostasis(state: &mut BodyState) {
    // RAAS CASCADE - Low BP triggers sodium retention
    if state.cardio.systolic < 90.0 {
        state.renal.renin += 0.1;
        state.renal.angiotensin_ii = state.renal.renin * 0.8;
        state.renal.aldosterone += state.renal.angiotensin_ii * 0.5;
    }

    // INSULIN-ALDOSTERONE LINK (Keto Whoosh)
    // Low insulin → kidneys dump sodium
    if state.meta.insulin < 10.0 {
        state.renal.aldosterone *= 0.9;  // Reduced retention
    }

    // OSMOLALITY → ADH
    if state.hydra.osmolality > 295.0 {
        state.renal.adh += 0.1;
        state.renal.urine_concentration += 50.0;
        state.renal.urine_production *= 0.8;
    } else if state.hydra.osmolality < 275.0 {
        state.renal.adh -= 0.1;
        state.renal.urine_concentration -= 100.0;
        state.renal.urine_production *= 1.3;
    }

    // HIGH POTASSIUM → Excrete it (protective)
    if state.hydra.potassium > 5.0 {
        state.renal.aldosterone += 0.1;
    }
}
```

### Outputs (Modifies)
| Writes To | Variable | Effect |
|-----------|----------|--------|
| `hydra` | `sodium` | Aldosterone retains/excretes |
| `hydra` | `total_water` | ADH retains/excretes |
| `cardio` | `blood_volume` | Via water retention |

### Keto Whoosh Effect
```
Carb restriction
    ↓
↓ Insulin
    ↓
↓ Aldosterone (insulin normally stimulates it)
    ↓
↓ Sodium retention → Kidneys DUMP sodium
    ↓
Water follows sodium → Rapid water weight loss
    ↓
If not supplementing: "Keto Flu" (headache, fatigue, cramps)
```

---

## System: Respiratory & pH

Controls breathing rate, gas exchange, and blood pH. Critical for distinguishing ketosis (safe) from ketoacidosis (deadly).

### State Variables
| Variable | Rust Path | Notes |
|----------|-----------|-------|
| `blood_ph` | `resp.blood_ph` | TIGHT: 7.35-7.45 |
| `pco2` | `resp.pco2` | Primary breathing driver |
| `po2` | `resp.po2` | Oxygen level |
| `o2_sat` | `resp.o2_sat` | Hemoglobin binding |
| `bicarbonate` | `resp.bicarbonate` | Buffer system |
| `rate` | `resp.rate` | Breaths per minute |
| `tidal_volume` | `resp.tidal_volume` | Air per breath |
| `co2_tolerance` | `resp.co2_tolerance` | Anxiety threshold |

### Inputs (Dependencies)
| Reads From | Variable | Purpose |
|------------|----------|---------|
| `meta` | `ketones` | High ketones → acid load |
| `meta` | `lactate` | Exercise acid |
| `env` | `altitude_m` | Lower pO2 at altitude |
| `thermo` | `core_temp` | Fever increases rate |

### Homeostasis Logic

```rust
fn respiratory_homeostasis(state: &mut BodyState) {
    // pH BUFFER EQUATION: CO2 + H2O ⇌ H2CO3 ⇌ H+ + HCO3-

    // ACIDOSIS (pH too low) → Hyperventilate to blow off CO2
    if state.resp.blood_ph < 7.35 {
        state.resp.rate += 2.0;  // Kussmaul breathing
        state.resp.pco2 -= 2.0;
    }

    // ALKALOSIS (pH too high) → Slow breathing
    if state.resp.blood_ph > 7.45 {
        state.resp.rate -= 1.0;
        state.resp.pco2 += 1.0;
    }

    // METABOLIC ACID LOAD (ketones, lactate)
    let acid_load = state.meta.ketones * 0.1 + state.meta.lactate * 0.05;
    if acid_load > 0.3 {
        state.resp.blood_ph -= acid_load * 0.01;
        // Bicarbonate buffers first
        state.resp.bicarbonate -= acid_load * 0.5;
    }

    // ALTITUDE COMPENSATION
    if state.env.altitude_m > 2500.0 {
        let altitude_factor = (state.env.altitude_m - 2500.0) / 1000.0;
        state.resp.po2 -= altitude_factor * 10.0;
        state.resp.rate += altitude_factor * 2.0;  // Compensate
    }

    // pH bounds check
    state.resp.blood_ph = state.resp.blood_ph.clamp(6.8, 7.8);
}
```

### Outputs (Modifies)
| Writes To | Variable | Effect |
|-----------|----------|--------|
| `resp` | `blood_ph` | Central output |
| `resp` | `pco2` | Via breathing rate |
| `ans` | `sympathetic` | Low O2 → stress response |

### Ketosis vs Ketoacidosis
| State | Ketones | pH | Insulin | Danger |
|-------|---------|-----|---------|--------|
| Nutritional Ketosis | 0.5-3.0 | 7.35-7.45 | Low but present | None |
| Starvation Ketosis | 3-5 | 7.30-7.40 | Very low | Minimal |
| Diabetic Ketoacidosis | >10 | <7.30 | ABSENT | Life-threatening |

**Key:** Even trace insulin prevents runaway ketone production.

### Breathing Techniques
| Technique | CO2 Effect | pH Effect | Mechanism |
|-----------|------------|-----------|-----------|
| Physiological Sigh | ↓↓ quickly | ↑ alkaline | Vagal activation |
| Box Breathing | Balanced | Stable | Equilibrium |
| Wim Hof | ↓↓↓ | ↑↑ alkalosis | Adrenaline surge |
| Breath Holds | ↑ | ↓ | CO2 tolerance training |

---

## System: Body Composition

Energy storage accounting - where mass comes from and goes to.

### State Variables
| Variable | Rust Path | Notes |
|----------|-----------|-------|
| `adipose_mass` | `comp.adipose_mass` | Total fat (kg) |
| `visceral_fat` | `comp.visceral_fat` | Inflammatory fat |
| `subcutaneous_fat` | `comp.subcutaneous_fat` | Less harmful |
| `lean_mass` | `comp.lean_mass` | Muscle (kg) |
| `glycogen_mass` | `comp.glycogen_mass` | Stored carbs (kg) |
| `body_weight` | `comp.body_weight` | Sum total |
| `bmr` | `comp.bmr` | Basal metabolic rate |
| `tdee` | `comp.tdee` | Total daily expenditure |

### Inputs (Dependencies)
| Reads From | Variable | Purpose |
|------------|----------|---------|
| `meta` | `insulin` | High → store, Low → mobilize |
| `meta` | `ketones` | Fat burning indicator |
| `anab` | `mps`, `mpb` | Muscle gain/loss |
| Activities | `calories_consumed` | Energy in |
| Activities | `activity_expenditure` | Energy out |

### Energy Balance Logic

```rust
fn body_composition_update(state: &mut BodyState, calories_in: f32, calories_out: f32) {
    let balance = calories_in - calories_out;

    if balance > 0.0 {
        // SURPLUS: Store energy
        if state.meta.insulin > 30.0 {
            // High insulin → prioritize glycogen, then fat
            let glycogen_room = 0.5 - state.comp.glycogen_mass;
            let to_glycogen = (balance / 4000.0).min(glycogen_room);  // 4 kcal/g
            state.comp.glycogen_mass += to_glycogen;

            let remaining = balance - (to_glycogen * 4000.0);
            state.comp.adipose_mass += remaining / 9000.0;  // 9 kcal/g fat
        }
    } else {
        // DEFICIT: Mobilize energy
        let deficit = -balance;

        // Priority 1: Glycogen
        if state.comp.glycogen_mass > 0.0 {
            let from_glycogen = (deficit / 4000.0).min(state.comp.glycogen_mass);
            state.comp.glycogen_mass -= from_glycogen;
            deficit -= from_glycogen * 4000.0;
        }

        // Priority 2: Fat (if ketones available)
        if deficit > 0.0 && state.meta.ketones > 0.3 {
            state.comp.adipose_mass -= deficit / 9000.0;
        }
        // Priority 3: Muscle (only if desperate)
        else if deficit > 0.0 {
            state.comp.lean_mass -= deficit / 4000.0 * 0.3;  // Partial muscle loss
        }
    }

    // Update weight
    state.comp.body_weight = state.comp.adipose_mass
                           + state.comp.lean_mass
                           + state.comp.glycogen_mass
                           + 42.0;  // ~42L water baseline

    // BMR scales with lean mass
    state.comp.bmr = 370.0 + (21.6 * state.comp.lean_mass);
}
```

### Storage Capacities
| Compartment | Capacity | Energy Density | Total Energy |
|-------------|----------|----------------|--------------|
| Blood glucose | ~5g | 4 kcal/g | ~20 kcal |
| Liver glycogen | ~100g | 4 kcal/g | ~400 kcal |
| Muscle glycogen | ~400g | 4 kcal/g | ~1,600 kcal |
| Adipose tissue | ~15kg avg | 9 kcal/g | ~135,000 kcal |

### Outputs (Modifies)
| Writes To | Variable | Effect |
|-----------|----------|--------|
| `comp` | `body_weight` | Sum of compartments |
| `comp` | `bmr` | Scales with lean mass |
| `hunger` | `leptin` | Proportional to fat mass |

---

## System: Visual & Vestibular

The sensory gateway to autonomic state. Huberman's primary lever for stress control.

### State Variables
| Variable | Rust Path | Notes |
|----------|-----------|-------|
| `mode` | `visual.mode` | focal/panoramic |
| `gaze_target` | `visual.gaze_target` | near/mid/far/horizon |
| `optic_flow` | `visual.optic_flow` | Forward motion perception |
| `light_input` | `visual.light_input` | Current lux exposure |
| `vestibular` | `visual.vestibular` | Balance system activation |

### Inputs (Dependencies)
| Reads From | Variable | Purpose |
|------------|----------|---------|
| `env` | `light_lux` | Light exposure |
| Activities | `location` | Indoor/outdoor |
| Activities | `movement` | Walking/running → optic flow |

### Visual-Autonomic Logic

```rust
fn visual_autonomic_effects(state: &mut BodyState) {
    // FOCAL MODE → Sympathetic activation
    if state.visual.mode == VisualMode::Focal {
        state.ans.sympathetic += 0.05;
        state.ans.parasympathetic -= 0.02;
    }

    // PANORAMIC MODE → Parasympathetic activation
    if state.visual.mode == VisualMode::Panoramic {
        state.ans.parasympathetic += 0.05;
        state.ans.sympathetic -= 0.02;
    }

    // OPTIC FLOW → Amygdala quieting
    // Forward movement creates visual flow → reduces anxiety
    if state.visual.optic_flow > 0.5 {
        state.ans.sympathetic -= 0.1;
        // Evolutionarily: if moving forward safely, threats behind
    }

    // GAZE DIRECTION
    match state.visual.gaze_target {
        GazeTarget::Upward => state.horm.norepinephrine += 0.02,   // ↑ Alertness
        GazeTarget::Downward => state.horm.norepinephrine -= 0.02, // ↓ Alertness
        GazeTarget::Horizon => { /* Neutral, calming */ }
        _ => {}
    }
}
```

### Key Mechanisms

**Focal vs Panoramic:**
| Mode | Description | ANS Effect | Use Case |
|------|-------------|------------|----------|
| **Focal** | Narrow attention, near object | ↑Sympathetic | Screen work, hunting |
| **Panoramic** | Wide peripheral awareness | ↑Parasympathetic | Horizon gazing, rest |

**Optic Flow:**
```
Forward movement (walking, running, driving):
- Visual field flows past periphery
- Quiets the Amygdala
- ↓Anxiety, ↓rumination

Static visual input:
- No flow = scanning for threats
- ↑Amygdala activation
- ↑Vigilance, potential anxiety
```

### Outputs (Modifies)
| Writes To | Variable | Effect |
|-----------|----------|--------|
| `ans` | `sympathetic` | Focal ↑, panoramic ↓ |
| `ans` | `parasympathetic` | Panoramic ↑, focal ↓ |
| `circ` | (via light) | Morning light → cortisol |

---

## System: Metabolic (Core Fuel Processing)

The central fuel processing system - glucose, ketones, insulin dynamics.

### State Variables
| Variable | Rust Path | Notes |
|----------|-----------|-------|
| `glucose` | `meta.glucose` | Blood sugar (mg/dL) |
| `ketones` | `meta.ketones` | Alternative fuel (mmol/L) |
| `insulin` | `meta.insulin` | Master metabolic switch |
| `insulin_sensitivity` | `meta.insulin_sensitivity` | Degrades with chronic high carb |
| `glucagon` | `meta.glucagon` | Counter-regulatory |
| `liver_glycogen` | `meta.liver_glycogen` | Stored glucose (grams) |
| `muscle_glycogen` | `meta.muscle_glycogen` | Local muscle fuel |
| `ffa` | `meta.ffa` | Free fatty acids |
| `mode` | `meta.mode` | glycolytic/ketogenic/hybrid |
| `fat_adaptation` | `meta.fat_adaptation` | Takes 2-4 weeks |
| `lactate` | `meta.lactate` | Bridge fuel |

### Inputs (Dependencies)
| Reads From | Variable | Purpose |
|------------|----------|---------|
| Activities | `meal` | Carbs, protein, fat intake |
| `comp` | `glycogen_mass` | Available storage |
| `circ` | `phase` | Morning insulin sensitivity |

### Metabolic Logic

```rust
fn metabolic_tick(state: &mut BodyState) {
    // GLUCOSE-INSULIN DYNAMICS
    if state.meta.glucose > 140.0 {
        state.meta.insulin += (state.meta.glucose - 100.0) * 0.3 * state.meta.insulin_sensitivity;
    }

    // HIGH INSULIN → Suppress ketones, store glucose
    if state.meta.insulin > 25.0 {
        state.meta.ketones *= 0.9;  // Ketone production suppressed
        state.meta.ffa *= 0.8;       // Fat release suppressed
        state.meta.mode = MetabolicMode::Glycolytic;
    }

    // LOW INSULIN → Ketone production
    if state.meta.insulin < 10.0 && state.meta.liver_glycogen < 20.0 {
        state.meta.ketones += 0.1 * state.meta.fat_adaptation;
        state.meta.ffa += 0.1;
        state.meta.mode = MetabolicMode::Ketogenic;
    }

    // GLYCOGEN DEPLETION → Triggers ketogenesis
    if state.meta.liver_glycogen < 10.0 {
        state.meta.glucagon += 0.1;
        state.meta.ketones += 0.05;
    }

    // FAT ADAPTATION (slow process)
    if state.meta.mode == MetabolicMode::Ketogenic {
        state.meta.fat_adaptation += 0.001;  // ~2-4 weeks to fully adapt
    } else {
        state.meta.fat_adaptation *= 0.999;  // Slowly loses adaptation
    }

    // LACTATE DYNAMICS
    // Lactate is valuable fuel for heart/brain
    state.meta.lactate *= 0.95;  // Continuous clearance
}
```

### Outputs (Modifies)
| Writes To | Variable | Effect |
|-----------|----------|--------|
| `resp` | (acid load) | Ketones/lactate → lower pH |
| `energy` | `atp` | Fuel availability |
| `horm` | `orexin` | Low glucose → ↓orexin |

### Key Transitions
| Trigger | Effect |
|---------|--------|
| High carb meal | ↑glucose → ↑insulin → ↓ketones |
| Fasting 12-16h | ↓glucose → ↓insulin → ↑ketones |
| HIIT exercise | ↓glycogen → ↑lactate → ↑glucagon |
| Weeks on keto | ↑fat_adaptation |

---

## Homeostasis Pass (Integration)

All homeostatic corrections run together after the reaction systems.

```rust
fn homeostasis_pass(state: &mut BodyState) {
    // 1. pH REGULATION (Respiratory)
    if state.resp.blood_ph < 7.35 {
        state.resp.rate += 2.0;  // Hyperventilate
    } else if state.resp.blood_ph > 7.45 {
        state.resp.rate -= 1.0;
    }

    // 2. FLUID BALANCE (Renal)
    if state.hydra.osmolality > 295.0 {
        state.renal.adh += 0.1;
        state.renal.urine_concentration += 50.0;
    } else if state.hydra.osmolality < 275.0 {
        state.renal.adh -= 0.1;
        state.renal.urine_production += 10.0;
    }

    // 3. BLOOD PRESSURE (RAAS)
    if state.cardio.systolic < 90.0 {
        state.renal.renin += 0.1;
        state.renal.aldosterone += 0.05;
    } else if state.cardio.systolic > 140.0 {
        state.renal.renin -= 0.05;
    }

    // 4. GLUCOSE (Pancreatic)
    if state.meta.glucose > 140.0 {
        state.meta.insulin += 5.0;
    } else if state.meta.glucose < 70.0 {
        state.meta.glucagon += 0.1;
        if state.meta.glucose < 55.0 {
            state.ans.cortisol += 0.2;  // Emergency
            state.horm.adrenaline += 0.3;
        }
    }

    // 5. TEMPERATURE
    if state.thermo.core_temp > 37.5 {
        state.thermo.sweating += 0.2;
        state.thermo.vasodilation += 0.1;
    } else if state.thermo.core_temp < 36.5 {
        state.thermo.shivering += 0.1;
        state.thermo.bat_activation += 0.2;
        state.thermo.vasoconstriction += 0.1;
    }

    // 6. ELECTROLYTE (Potassium emergency)
    if state.hydra.potassium > 5.5 {
        state.renal.aldosterone += 0.1;  // Excrete K+
    }
}
```

---

## Error States

| Condition | Threshold | Consequence |
|-----------|-----------|-------------|
| Severe hypoglycemia | glucose < 40 | Loss of consciousness |
| Acidosis | pH < 7.0 | Organ failure |
| Hypothermia | core_temp < 32°C | Cardiac arrest |
| Hyperthermia | core_temp > 42°C | Brain damage |
| Severe dehydration | water_deficit > 10% | Shock |
| Hyperkalemia | potassium > 6.5 | Cardiac arrhythmia |
| Hyponatremia | sodium < 120 | Seizures |
