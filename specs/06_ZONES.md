# Physical Zones - The Biological Physics Engine

> **This replaces rule-based simulation with physics-based flow.**
> Instead of 500 "if-then" rules, we have 6 zones and 1 master transfer equation.

---

## Philosophy: From Storyteller to Laboratory

```
OLD MODEL (Rules):                    NEW MODEL (Physics):
──────────────────                    ────────────────────

eat_meal() {                          eat_meal() {
  glucose += carbs * 0.5;               lumen.stomach_carbs += carbs_g;
  insulin += glucose * 0.3;             // That's it. Physics handles the rest.
  if (glucose > 140) { ... }          }
  if (exercise) { ... }
}                                     tick() {
                                        // Mass flows through pipes
"Teleporting" + Rules                   // based on concentrations and valves
                                        // Effects EMERGE from physics
                                      }
```

---

## The 6 Physical Zones

Each zone is a **physical compartment** with storage capacity and gates.

### Zone 1: LUMEN (GI Tract)

> Technically "outside" the body - the tube from mouth to anus.

```rust
struct Lumen {
    // Storage (grams of mass)
    stomach_carbs: f32,      // Recently eaten, being processed
    stomach_protein: f32,
    stomach_fat: f32,
    stomach_volume: f32,     // mL (capacity ~1000mL)

    intestine_carbs: f32,    // Ready for absorption
    intestine_protein: f32,
    intestine_fat: f32,

    // Rates
    gastric_emptying_rate: f32,  // mL/min (slowed by fat, stress)
    absorption_rate: f32,         // g/min (requires blood flow)
}
```

**Capacities:**
| Compartment | Max Capacity | Normal Transit |
|-------------|--------------|----------------|
| Stomach | ~1000 mL | 2-4 hours |
| Small intestine | ~400g food mass | 3-5 hours |

**Gates (Valves):**
- `pyloric_sphincter`: Stomach → Intestine (slowed by fat, stress)
- `absorption_gate`: Intestine → Blood (requires blood flow to gut)

---

### Zone 2: VASCULAR (Blood)

> The highway. Finite volume (~5L). Everything competes for delivery.

```rust
struct Vascular {
    // Volume
    plasma_volume: f32,       // mL (normally ~3000mL)
    total_blood_volume: f32,  // mL (normally ~5000mL)

    // Concentrations (mass per volume)
    glucose_concentration: f32,    // mg/dL (normal: 70-100)
    ketone_concentration: f32,     // mmol/L
    sodium_concentration: f32,     // mEq/L
    potassium_concentration: f32,  // mEq/L
    oxygen_saturation: f32,        // % (normal: 95-100)

    // Flow
    cardiac_output: f32,           // L/min (rest: 5, exercise: 20+)

    // Partitioning (must sum to 1.0)
    flow_to_brain: f32,     // ~15% at rest
    flow_to_gut: f32,       // ~25% at rest, ↑ digesting
    flow_to_muscle: f32,    // ~20% at rest, ↑↑ exercise
    flow_to_liver: f32,     // ~25% at rest
    flow_to_kidney: f32,    // ~20% at rest
    flow_to_skin: f32,      // Variable (thermoregulation)
}
```

**The Critical Constraint:**
```rust
assert!(
    flow_to_brain + flow_to_gut + flow_to_muscle
    + flow_to_liver + flow_to_kidney + flow_to_skin
    == 1.0,
    "Blood flow partitioning must sum to 1.0!"
);
```

**Why this matters:**
- If `flow_to_muscle` spikes during exercise, other organs MUST get less
- Brain fog during exercise = physics, not a rule
- Post-meal drowsiness = blood routed to gut, less to brain

---

### Zone 3: HEPATIC (Liver)

> The reservoir and factory. Stores glycogen, produces ketones.

```rust
struct Hepatic {
    // Storage
    glycogen_stored: f32,        // grams (max ~100g)
    fat_stored: f32,             // grams (lipid droplets)

    // Production capacity
    gluconeogenesis_rate: f32,   // g/hour (making glucose from protein)
    ketogenesis_rate: f32,       // mmol/hour (making ketones from fat)

    // Valves (hormone-controlled)
    glycogen_release_valve: f32,  // 0-1 (controlled by glucagon)
    glycogen_uptake_valve: f32,   // 0-1 (controlled by insulin)
    ketone_production_valve: f32, // 0-1 (opens when insulin low)
}
```

**Capacities:**
| Storage | Max Capacity | Depletion Time |
|---------|--------------|----------------|
| Glycogen | ~100g | 12-18 hours fasted |
| Fat buffer | Variable | Days |

**Key behaviors:**
- Low blood glucose → Glucagon opens `glycogen_release_valve`
- High blood glucose → Insulin opens `glycogen_uptake_valve`
- Glycogen depleted + low insulin → `ketone_production_valve` opens

---

### Zone 4: MYO (Skeletal Muscle)

> The engines. Variable demand. Local glycogen storage.

```rust
struct Myo {
    // Storage (per-muscle-group average)
    glycogen_local: f32,         // grams (max ~400g total)

    // Demand
    atp_demand: f32,             // ATP/min required
    atp_production: f32,         // ATP/min being produced

    // Fuel sources
    glucose_uptake_rate: f32,    // g/min from blood
    fat_oxidation_rate: f32,     // g/min from blood FFA

    // Valves
    insulin_receptor_density: f32,  // 0-1 (exercise increases this)
    glut4_translocation: f32,       // 0-1 (insulin + contraction)

    // State
    lactate_accumulation: f32,   // mmol/L local
    fatigue_level: f32,          // 0-1
}
```

**ATP Demand by Activity:**
| Activity | ATP Demand | Primary Fuel |
|----------|------------|--------------|
| Rest | 1x | Fatty acids |
| Walking | 3x | Mixed |
| Zone 2 | 6x | Mostly fat |
| HIIT | 15x+ | Glucose → Lactate |
| Strength | 10x burst | Phosphocreatine → Glucose |

---

### Zone 5: CORTEX (Brain)

> The control room. Constant high demand. Cannot store fuel.

```rust
struct Cortex {
    // NO storage - completely dependent on blood delivery

    // Demand (constant but variable by region)
    atp_demand: f32,             // ATP/min (high, ~20% of body total)

    // Delivery (what actually arrives)
    glucose_delivery: f32,       // mg/min arriving via blood
    ketone_delivery: f32,        // mmol/min arriving via blood
    oxygen_delivery: f32,        // mL/min arriving via blood

    // Function (emerges from delivery vs demand)
    electrical_potential: f32,   // 0-1 (nerve function)

    // The critical calculation:
    // If delivery < demand → function degrades

    // Regional demands (for cognitive hierarchy)
    prefrontal_demand: f32,      // Executive function (high)
    social_processing_demand: f32, // Highest cost, first to fail
    motor_cortex_demand: f32,    // Protected, last to fail
}
```

**The Brain Priority:**
```rust
// When glucose_delivery is insufficient, triage:
if glucose_delivery < atp_demand {
    let deficit = atp_demand - glucose_delivery;

    // Cut expensive functions first
    social_processing -= deficit * 2.0;    // First to go
    prefrontal_function -= deficit * 1.5;  // Second
    language_function -= deficit * 1.0;    // Third
    motor_function -= deficit * 0.3;       // Protected
}
```

---

### Zone 6: RENAL (Kidneys)

> The filter. Removes waste. Controls electrolyte balance.

```rust
struct Renal {
    // Filtration
    filtration_rate: f32,        // mL/min (GFR, normal ~120)

    // Gates
    sodium_reabsorption_gate: f32,  // 0-1 (aldosterone)
    water_reabsorption_gate: f32,   // 0-1 (ADH)
    potassium_excretion_gate: f32,  // 0-1

    // Output (leaves simulation)
    urine_volume: f32,           // mL/hour
    sodium_excreted: f32,        // mEq/hour
    waste_excreted: f32,         // g/hour
}
```

---

## The Master Transfer Equation

Every tick, for every zone, calculate delivery:

```rust
/// The physics that replaces all the rules
fn calculate_delivery(
    vascular: &Vascular,
    zone: Zone,
    valve_opening: f32,  // 0-1, controlled by hormones
) -> f32 {
    let concentration = vascular.glucose_concentration;
    let blood_flow = vascular.cardiac_output * vascular.flow_to(zone);

    // Delivery = Concentration × Flow × Valve
    concentration * blood_flow * valve_opening
}
```

**Why this is powerful:**

```
SCENARIO: Post-meal drowsiness ("Food Coma")
───────────────────────────────────────────

1. Eat large meal
   → lumen.stomach_carbs = 100g

2. Digestion starts
   → vascular.flow_to_gut increases to 40%
   → vascular.flow_to_brain DECREASES (must sum to 1.0)

3. Brain delivery drops
   → cortex.glucose_delivery = concentration × (cardiac_output × 0.10)
   → Less than cortex.atp_demand

4. Brain function degrades
   → EMERGENT drowsiness

No rule says "if eating then drowsy"
It EMERGES from the physics of blood routing
```

---

## Hormones as Valves

Hormones don't "do things" - they open/close gates between zones.

```rust
struct HormoneValves {
    // INSULIN: Blood → Muscle glucose gate
    insulin: f32,  // 0-1
    // High insulin = muscle soaks up glucose from blood

    // GLUCAGON: Liver → Blood glucose gate
    glucagon: f32,  // 0-1
    // High glucagon = liver releases glycogen to blood

    // ADRENALINE: Routing override
    adrenaline: f32,  // 0-1
    // High adrenaline = blood to muscle/brain, away from gut/kidney

    // CORTISOL: Pressure maintainer
    cortisol: f32,  // 0-1
    // High cortisol = maintains vascular pressure even when volume low

    // ALDOSTERONE: Kidney sodium gate
    aldosterone: f32,  // 0-1
    // High aldosterone = kidney reabsorbs sodium (retains it)

    // ADH: Kidney water gate
    adh: f32,  // 0-1
    // High ADH = kidney reabsorbs water (concentrates urine)
}

fn apply_hormone_valves(zones: &mut Zones, hormones: &HormoneValves) {
    // Insulin opens muscle glucose uptake
    zones.myo.glut4_translocation = hormones.insulin * zones.myo.insulin_receptor_density;

    // Glucagon opens liver glycogen release
    zones.hepatic.glycogen_release_valve = hormones.glucagon;

    // Adrenaline redistributes blood flow
    if hormones.adrenaline > 0.5 {
        zones.vascular.flow_to_muscle += 0.2;
        zones.vascular.flow_to_brain += 0.1;
        zones.vascular.flow_to_gut -= 0.2;
        zones.vascular.flow_to_kidney -= 0.1;
        // Normalize to sum to 1.0
        normalize_flow(&mut zones.vascular);
    }
}
```

---

## Mass Conservation Auditor

The debugger that ensures physics is correct:

```rust
fn audit_mass(state: &SimulationState) -> Result<(), MassLeak> {
    // Track all carbohydrate mass in the system
    let carbs_in_lumen = state.lumen.stomach_carbs
                       + state.lumen.intestine_carbs;

    let carbs_in_blood = state.vascular.glucose_concentration
                       * state.vascular.plasma_volume
                       / 100.0;  // Convert mg/dL to grams

    let carbs_in_liver = state.hepatic.glycogen_stored;

    let carbs_in_muscle = state.myo.glycogen_local;

    let carbs_burned = state.total_calories_burned / 4.0;  // 4 kcal/g carb

    let carbs_excreted = state.renal.glucose_excreted;  // Rare, diabetic

    let total_accounted = carbs_in_lumen
                        + carbs_in_blood
                        + carbs_in_liver
                        + carbs_in_muscle
                        + carbs_burned
                        + carbs_excreted;

    let total_ingested = state.cumulative_carbs_eaten;

    let difference = (total_accounted - total_ingested).abs();

    if difference > 0.1 {  // Allow 0.1g rounding error
        return Err(MassLeak {
            expected: total_ingested,
            found: total_accounted,
            difference,
        });
    }

    Ok(())
}
```

---

## The New Tick Cycle

```rust
fn tick(state: &mut SimulationState) {
    // 1. DIGESTION: Move mass through lumen
    digest_tick(&mut state.lumen, &state.vascular);

    // 2. ABSORPTION: Lumen → Blood (requires gut blood flow)
    absorb_tick(&mut state.lumen, &mut state.vascular);

    // 3. HORMONE SIGNALS: Update valve positions
    hormone_tick(&mut state.hormones, &state.vascular);

    // 4. APPLY VALVES: Hormones control gates
    apply_valves(&mut state.zones, &state.hormones);

    // 5. BLOOD ROUTING: ANS controls partitioning
    route_blood(&mut state.vascular, &state.ans);

    // 6. DELIVERY: Calculate what each zone receives
    for zone in &mut state.zones {
        zone.delivery = calculate_delivery(&state.vascular, zone);
    }

    // 7. METABOLISM: Each zone processes delivery vs demand
    hepatic_tick(&mut state.hepatic, &mut state.vascular);
    myo_tick(&mut state.myo, &mut state.vascular);
    cortex_tick(&mut state.cortex, &state.vascular);
    renal_tick(&mut state.renal, &mut state.vascular);

    // 8. AUDIT: Verify mass conservation
    audit_mass(&state).expect("Mass leak detected!");

    // 9. DERIVE: Calculate axes from zone states
    state.axes = derive_axes(&state.zones);
}
```

---

## Emergent Effects (No Rules Needed)

| Effect | Old Model (Rule) | New Model (Physics) |
|--------|------------------|---------------------|
| Food coma | `if eating { drowsy = true }` | Blood routes to gut → brain delivery ↓ |
| Exercise brain fog | `if exercising { focus -= 0.2 }` | Blood routes to muscle → brain delivery ↓ |
| Keto flu | `if ketones < 0.5 { fatigue = true }` | Glycogen depleted + ketone valve closed → brain starved |
| Hypoglycemia confusion | `if glucose < 50 { confusion = true }` | Glucose concentration ↓ → brain delivery ↓ → function ↓ |
| Dehydration fatigue | `if hydration < 0.7 { tired = true }` | Plasma volume ↓ → cardiac output ↓ → all delivery ↓ |
| Adrenaline focus | `if adrenaline > 0.8 { focus += 0.3 }` | Blood routes to brain → brain delivery ↑ |

---

## Migration Path

### Phase 1: Define Zones (This file)
✓ Define 6 zones with components and capacities

### Phase 2: Implement Vascular Routing
- Add blood flow partitioning to cardiovascular system
- Implement `flow_to_[organ]` variables
- Add constraint that partitions sum to 1.0

### Phase 3: Convert Eat Activity
- Change `eat_meal()` to only add mass to `lumen.stomach`
- Let digestion/absorption pipeline move it naturally

### Phase 4: Add Mass Auditor
- Track all mass entering system
- Verify conservation on every tick
- Fail loudly if physics breaks

### Phase 5: Migrate Other Systems
- Convert each system to use zone-based delivery
- Remove rule-based effects that now emerge from physics

---

## File Map Update

| Spec File | Purpose |
|-----------|---------|
| `00_ARCHITECTURE.md` | Tick loop, overall flow |
| `01_ENGINE_CORE.md` | Homeostatic laws (keep) |
| `02_CONTROLLERS.md` | Regulatory systems (keep, but hormones become valves) |
| `03_ACTIVITIES.md` | Input layer (simplify - just add mass to lumen) |
| `04_AXES.md` | Output layer (now derived from zone states) |
| **`06_ZONES.md`** | **NEW: Physical compartments and mass flow** |
| `DATA_DICTIONARY.md` | Update with zone-based variables |
