# Architecture & Ontology

> **See also:** [ONTOLOGY.md](../ONTOLOGY.md) for the full taxonomy of entities

---

## The Five Kinds of Things

| # | Category | Role | Maps To |
|---|----------|------|---------|
| 1 | **Environment** | External world state | `src/environment.rs` |
| 2 | **Activities** | User inputs (verbs) | `src/activities.rs` |
| 3 | **Sensors** | Boundary translation | `src/sensors.rs` |
| 4 | **Internal** | Body systems + state | `src/systems/`, `src/state.rs` |
| 5 | **Dimensions** | Dashboard (derived) | `src/dimensions.rs` |

---

## The Tick Cycle (8 Passes)

Each tick = 1 minute of simulation time.

```
┌─────────────────────────────────────────────────────────────────────┐
│                         TICK CYCLE                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  1. RHYTHM PASS          Update circadian & ultradian phases        │
│         │                                                           │
│         ▼                                                           │
│  2. INPUT PASS           Apply activities → environment/body        │
│         │                                                           │
│         ▼                                                           │
│  3. SENSOR PASS          Read environment + state into sensors      │
│         │                                                           │
│         ▼                                                           │
│  4. REACTION PASS        Systems respond to inputs                  │
│         │                 (hormone release, metabolic processing)   │
│         ▼                                                           │
│  5. HOMEOSTASIS PASS     Feedback loops correct deviations          │
│         │                 (pH, temp, glucose, pressure)             │
│         ▼                                                           │
│  6. EFFECTOR PASS        Body takes corrective actions              │
│         │                 (sweat, shiver, breathe faster)           │
│         ▼                                                           │
│  7. COGNITIVE PASS       Update attention, perception, memory       │
│         │                                                           │
│         ▼                                                           │
│  8. TIME PASS            Advance clock, decay transients,           │
│                          accumulate debts, store history            │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Pass Details

### 1. Rhythm Pass
```rust
fn rhythm_pass(state: &mut BodyState, env: &Environment) {
    // Circadian phase advances based on clock time
    state.circadian.phase = calculate_phase(env.clock_time);

    // Ultradian 90-min cycles
    state.ultradian.brac_phase = (state.time_awake_min % 90) as f32 / 90.0;
}
```

### 2. Input Pass
```rust
fn input_pass(activities: &[Activity], env: &mut Environment, state: &mut BodyState) {
    for activity in activities {
        match activity.target() {
            ActivityTarget::Environment(change) => env.apply(change),
            ActivityTarget::Body(effect) => state.apply(effect),
            ActivityTarget::Both(change, effect) => {
                env.apply(change);
                state.apply(effect);
            }
        }
    }
}
```

### 5. Homeostasis Pass (Critical)
```rust
fn homeostasis_pass(state: &mut BodyState) {
    // pH: Body MUST maintain 7.35-7.45
    if state.respiratory.blood_ph < 7.35 {
        state.respiratory.rate += 2;  // Hyperventilate to blow off CO2
    }

    // Glucose: Maintain 70-100 mg/dL
    if state.metabolic.glucose < 70.0 {
        state.hormonal.glucagon += 0.1;  // Release stored glucose
    }

    // Temperature: Maintain 36.5-37.5°C
    if state.thermo.core_temp > 37.5 {
        state.thermo.sweat_rate += 0.2;  // Cool down
    }

    // Blood Pressure: Maintain ~120/80
    if state.cardio.systolic < 90 {
        state.renal.renin += 0.1;  // RAAS cascade
    }

    // Fluid: Maintain osmolality 280-295
    if state.hydration.osmolality > 295 {
        state.renal.adh += 0.1;  // Concentrate urine
    }
}
```

---

## Error States (Death Conditions)

The simulation should flag critical states:

| Condition | Threshold | What Happens |
|-----------|-----------|--------------|
| Hypoglycemia (severe) | glucose < 40 mg/dL | Loss of consciousness |
| Acidosis | pH < 7.0 | Organ failure |
| Hypothermia | core_temp < 32°C | Cardiac arrest |
| Hyperthermia | core_temp > 42°C | Brain damage |
| Dehydration (severe) | water_deficit > 10% | Shock |
| Hyperkalemia | potassium > 6.5 mEq/L | Cardiac arrhythmia |

---

## System Dependencies (Execution Order)

Systems must run in dependency order. A system can only read from systems that ran before it.

```
Level 0 (Independent):
├── CircadianSystem
├── UltradianSystem
└── EnvironmentSystem

Level 1 (Depends on L0):
├── RespiratorySystem (needs: circadian)
├── RenalSystem (needs: nothing yet)
└── DigestionSystem (needs: nothing yet)

Level 2 (Depends on L1):
├── MetabolicSystem (needs: respiratory, digestive)
├── HydrationSystem (needs: renal)
└── CardiovascularSystem (needs: respiratory)

Level 3 (Depends on L2):
├── HormonalSystem (needs: metabolic, cardiovascular)
├── AutonomicSystem (needs: metabolic, cardiovascular)
└── ThermoregulationSystem (needs: metabolic)

Level 4 (Depends on L3):
├── EnergySystem (needs: metabolic, hormonal)
├── HungerSystem (needs: metabolic, hormonal)
└── ImmuneSystem (needs: hormonal)

Level 5 (Depends on L4):
├── AttentionSystem (needs: energy, hormonal)
├── MemorySystem (needs: hormonal, sleep)
└── AllostaticSystem (needs: all stress signals)
```

---

## Immutability & Time Travel

State is **immutable** after each tick. This enables:

1. **Branching:** "What if I had eaten carbs instead of fasting?"
2. **Replay:** Step forward/backward through history
3. **MCTS:** Monte Carlo Tree Search for optimal protocols

```rust
struct Timeline {
    id: Uuid,
    parent_id: Option<Uuid>,
    branch_point: u64,           // Tick where we branched
    states: Vec<BodyState>,      // State at each tick (immutable)
    scheduled: BTreeMap<u64, Vec<Activity>>,  // Future events
}

fn branch(timeline: &Timeline, from_tick: u64) -> Timeline {
    Timeline {
        id: Uuid::new_v4(),
        parent_id: Some(timeline.id),
        branch_point: from_tick,
        states: timeline.states[..=from_tick].to_vec(),
        scheduled: BTreeMap::new(),
    }
}
```

---

## System Interactions (Non-Obvious Connections)

These cross-system effects emerge from the simulation but aren't obvious from individual systems.

### Retention + Keto = Redundant
```
Both suppress appetite through overlapping pathways:
- Both increase sympathetic tone
- Both reduce dopamine spikes (stable energy)
- Keto makes retention EASIER (suppresses urges)

BUT: Keto DURING stable retention removes energy
- Pick ONE as primary
- Optimal: Keto to ENTER retention → carbs once stable
```

### Dopamine → Immune System
```
Immune cells have dopamine receptors.
ADHD = more pronounced effect.

Observation on retention:
- WITH: No sicknesses, virus stays "on surface"
- WITHOUT: Sick every ~3 weeks

Keto also improves immunity (via inflammation reduction).
```

### Retention → Sleep Reduction
```
WHY less sleep needed:
- Orexin/hypocretin upregulation (same as keto)
- More efficient glymphatic clearing
- Less neural waste production
- Better dopamine = less recovery needed

This is REAL reduced need, not forced wakefulness.
Mechanism: Different temporal niche when not in "mating mode"
```

### Coordination Enhancement (Retention)
```
Observable effects on retention:
- Catch things normally dropped
- Navigate crowds smoothly
- Type faster with fewer errors
- Feel weight of objects precisely
- Move with deliberate intention

WHY:
- Dopamine controls motor precision
- Higher dopamine = finer motor control
- Testosterone enhances proprioception
- Cerebellum processing speed increases
```

### Keto vs Carbs for Different Modes
```
KETO better for:
- Business meetings
- Deep work sessions
- Structured tasks
- Tunnel focus

CARBS better for:
- Meeting friends
- Spontaneous social
- Parties
- Quick social bursts

Trade-off: Keto reduces social reactivity (good for anxiety, bad for spontaneity)
```

---

## File Map

| Spec File | Rust Module | Purpose |
|-----------|-------------|---------|
| `00_ARCHITECTURE.md` | `src/lib.rs`, `src/simulation.rs` | Tick loop, orchestration |
| `01_ENGINE_CORE.md` | `src/systems/{renal,respiratory,metabolic}.rs` | Homeostatic machinery |
| `02_CONTROLLERS.md` | `src/systems/{circadian,hormonal,autonomic}.rs` | Regulatory systems |
| `03_ACTIVITIES.md` | `src/activities.rs`, `src/environment.rs` | Input layer |
| `04_AXES.md` | `src/axes.rs` | Output/UX layer - 7 physiological axes |
| `06_ZONES.md` | `src/zones/*.rs` | **Physical compartments and mass flow** |
| `DATA_DICTIONARY.md` | `src/state.rs` | All variable definitions |

> **See also:** [GRAPH.md](../GRAPH.md) - Fan-out analysis and dependency maps (reference doc, not a spec)

---

## Architecture Note: Physics vs Rules

This simulation uses a **Physics-based** approach, not rule-based:

```
RULES (what we avoid):          PHYSICS (what we do):
────────────────────            ─────────────────────
if eating { drowsy = true }     Blood routes to gut
if exercise { focus -= 0.2 }    → Less blood to brain
if glucose < 50 { confuse }     → Less glucose delivery
                                → Reduced function
"Teleporting" effects           EMERGENT effects
```

See `06_ZONES.md` for the physical compartment model.
