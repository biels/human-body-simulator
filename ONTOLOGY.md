# Human Body Simulator - Ontology

A taxonomy of all entities in the simulation and how they relate.

---

## Overview: The Five Kinds of Things

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         1. ENVIRONMENT                                   │
│                    (External World State)                                │
│   Sunlight, Temperature, Altitude, Social context, Food availability    │
└────────────────────────────────┬────────────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                         2. ACTIVITIES                                    │
│                      (The Verbs / User Input)                            │
│   Go outside, Eat meal, Exercise, Sleep, Breathe, Wait                  │
│                                                                          │
│   Activities either:                                                     │
│   • Change ENVIRONMENT (go_outside → light ↑)                           │
│   • Move resources across boundary (eat → food enters body)             │
│   • Trigger internal processes (exercise → metabolic demand)            │
└────────────────────────────────┬────────────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                         3. INTERFACE / SENSORS                           │
│              (The Boundary Between World and Body)                       │
│                                                                          │
│   • Retina: Detects light → signals to Circadian system                 │
│   • Skin thermoreceptors: Detects temp → signals to Thermoregulation    │
│   • Gut chemoreceptors: Detects nutrients → signals to Metabolic        │
│   • Baroreceptors: Detects blood pressure → signals to Cardiovascular   │
│   • Proprioceptors: Detects body position → signals to Vestibular       │
└────────────────────────────────┬────────────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    4. INTERNAL (Body)                                    │
│                                                                          │
│   ┌─────────────────────────┐    ┌─────────────────────────┐            │
│   │   4A. SYSTEMS (Logic)   │    │  4B. STATE (Variables)  │            │
│   │   ─────────────────     │    │  ───────────────────    │            │
│   │   The machinery that    │◄──►│  The raw numbers the    │            │
│   │   processes inputs and  │    │  systems read & write   │            │
│   │   maintains homeostasis │    │                         │            │
│   │                         │    │  bloodGlucose: 95       │            │
│   │   • CircadianSystem     │    │  cortisol: 0.7          │            │
│   │   • MetabolicSystem     │    │  coreTemp: 37.1         │            │
│   │   • RenalSystem         │    │  dopamine: 0.6          │            │
│   │   • etc (22 systems)    │    │  etc (110+ variables)   │            │
│   └─────────────────────────┘    └─────────────────────────┘            │
│                                                                          │
└────────────────────────────────┬────────────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                         5. AXES                                          │
│              (Felt Physiological Pathways)                               │
│                                                                          │
│   7 fundamental dimensions you can FEEL, mapped to real pathways.       │
│   These answer: "How do I feel?"                                        │
│                                                                          │
│   • Androgenic      = Testosterone pathway (drive, libido, confidence)  │
│   • Dopaminergic    = Dopamine baseline (content vs seeking)            │
│   • Metabolic       = Keto vs Glycolytic (hunger, clarity, stability)   │
│   • Adenosinergic   = Sleep pressure (alert vs sleepy)                  │
│   • Autonomic       = Symp/Para balance (stressed vs calm)              │
│   • Immune          = Inflammation (resilient vs run down)              │
│   • Thermoregulatory = BAT/cold adaptation (cold tolerance)             │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

**Key Insight:** The user NEVER sets internal variables directly. You can't "set" your dopamine—you must do an activity that triggers it. Axes are READ-ONLY views calculated from state.

---

## 1. EXTERNAL ENVIRONMENT (Context)

Things outside the body that affect internal state. The agent can modify some of these through activities.

### 1.1 Physical Environment

| Variable | Unit | Modifiable By | Affects |
|----------|------|---------------|---------|
| `ambientLight` | lux | location, time, lights | Circadian, Visual, Melatonin |
| `ambientTemp` | °C | location, clothing, HVAC | Thermoregulation, BAT |
| `altitude` | meters | travel | Respiratory (pO2), Cardiovascular |
| `humidity` | % | location | Thermoregulation (sweating efficiency) |
| `airQuality` | AQI | location, windows | Respiratory |
| `noise` | dB | location | Stress, Sleep quality |

### 1.2 Temporal Context

| Variable | Unit | Notes |
|----------|------|-------|
| `clockTime` | 0-1440 | Minutes since midnight (external clock) |
| `dayOfYear` | 1-365 | Affects daylight duration |
| `season` | enum | winter/spring/summer/fall |
| `timezone` | offset | For jet lag simulation |

### 1.3 Social Environment

| Variable | Unit | Notes |
|----------|------|-------|
| `socialContext` | enum | alone/small_group/crowd/intimate |
| `socialSupport` | 0-1 | Quality of relationships |
| `socialStress` | 0-1 | Conflict, pressure |
| `workDemand` | 0-1 | Cognitive/time pressure |

### 1.4 Resource Availability

| Variable | Unit | Notes |
|----------|------|-------|
| `foodAvailable` | boolean | Can eat? |
| `waterAvailable` | boolean | Can drink? |
| `sleepOpportunity` | boolean | Can sleep? (bed, dark, quiet) |
| `exerciseOpportunity` | boolean | Gym, space, equipment |
| `coldWaterAccess` | boolean | For cold exposure |

---

## 2. ACTIVITIES (Actions)

What the agent can DO. Activities are the primary input to the simulation.

### 2.1 Environment-Changing Activities

These primarily modify the EXTERNAL environment, which then affects internal state.

| Activity | Changes | Example |
|----------|---------|---------|
| `go_outside` | ambientLight ↑↑, ambientTemp = weather | Morning sun exposure |
| `go_inside` | ambientLight ↓, ambientTemp = room | Return from walk |
| `travel` | altitude, timezone, all environment vars | Fly to mountains |
| `turn_on_lights` | ambientLight ↑ | Evening artificial light |
| `dim_lights` | ambientLight ↓ | Pre-sleep routine |
| `open_window` | ambientTemp → outdoor, airQuality | Fresh air |
| `set_thermostat` | ambientTemp | Climate control |
| `enter_sauna` | ambientTemp ↑↑ | Heat exposure |
| `enter_cold_plunge` | ambientTemp ↓↓ (local) | Cold exposure |
| `put_on_clothing` | insulation ↑ | Temperature regulation |
| `remove_clothing` | insulation ↓ | Allow cooling |

### 2.2 Direct Biological Activities

These directly affect INTERNAL state (bypass environment).

| Activity | Primary Systems Affected |
|----------|--------------------------|
| **Nutrition** | |
| `eat_meal` | Metabolic, Digestive, Hunger, Insulin |
| `drink_water` | Hydration, Renal |
| `take_supplement` | Various (depends on supplement) |
| `fast` | Metabolic (ketones ↑), Hunger, Autophagy |
| **Movement** | |
| `exercise_HIIT` | Metabolic, Cardiovascular, Hormonal, Lactate |
| `exercise_zone2` | Metabolic (fat oxidation), Cardiovascular |
| `exercise_strength` | Anabolic (mTOR), Hormonal (testosterone) |
| `walk` | Autonomic (↓stress), Visual (optic flow) |
| `stretch/yoga` | Autonomic (parasympathetic), Flexibility |
| **Rest & Recovery** | |
| `sleep` | Energy (adenosine clear), Hormonal (GH), Memory |
| `nap` | Energy (partial adenosine clear) |
| `NSDR` | Hormonal (dopamine +65%), Autonomic |
| `meditate` | Autonomic (parasympathetic), Cognitive |
| **Breathing** | |
| `physiological_sigh` | Respiratory (CO2 ↓), Autonomic (calm) |
| `box_breathing` | Respiratory (balanced), Autonomic |
| `wim_hof_breathing` | Respiratory (alkalosis), Hormonal (adrenaline) |
| `breath_hold` | Respiratory (CO2 tolerance), Autonomic |
| **Sexual** | |
| `ejaculation` | Hormonal (prolactin ↑, dopamine ↓), Energy |
| `retain` | Hormonal (testosterone sensitivity ↑) |
| **Visual** | |
| `focus_near` | Visual (focal mode), Autonomic (sympathetic ↑) |
| `gaze_horizon` | Visual (panoramic), Autonomic (parasympathetic ↑) |
| `close_eyes` | Visual (remove input), enables NSDR |

### 2.3 Consumption Activities

Specific inputs with parameters.

| Activity | Parameters | Effects |
|----------|------------|---------|
| `eat` | carbs, protein, fat, fiber, GI | Metabolic cascade |
| `drink` | liters, electrolytes | Hydration, Renal |
| `caffeine` | mg, time_since_waking | Energy (adenosine block) |
| `supplement` | type, dose | Various |

### 2.4 Passive/Time Activities

Sometimes the best action is inaction.

| Activity | Duration | What Happens |
|----------|----------|--------------|
| `wait` | minutes | Time passes, systems update |
| `rest` | minutes | Parasympathetic recovery |
| `do_nothing` | minutes | Default state, no intervention |

---

## 3. INTERNAL STATE (Systems)

The body's physiological machinery. Organized by function.

### 3.1 System Categories

```
FOUNDATIONAL (The "Engine")
├── Renal (Kidneys) - Electrolyte/fluid regulation
├── Respiratory - Gas exchange, pH
├── Body Composition - Mass storage
└── Visual/Vestibular - Sensory gateway

METABOLIC & ENERGY
├── Metabolic - Fuel processing
├── Energy & Fatigue - ATP, adenosine
├── Hunger & Satiety - Ghrelin, leptin
├── Anabolic/MPS - Muscle building
└── Digestive - Food processing

REGULATORY
├── Autonomic NS - Fight/flight vs rest
├── Hydration - Water/electrolytes
├── Thermoregulation - Temperature
└── Cardiovascular - Blood flow

HORMONAL & IMMUNE
├── Hormonal - Signaling molecules
├── Immune - Defense
└── Gut-Brain - Microbiome

COGNITIVE & BEHAVIORAL
├── Attention - Focus, flow
├── Perceptual - Frame rate
└── Memory - Learning

RHYTHM
├── Circadian - 24-hour cycles
└── Ultradian - 90-min cycles

LONG-TERM
└── Allostatic Load - Cumulative stress
```

### 3.2 Variable Types Within Systems

Each system contains different types of variables:

| Type | Description | Examples |
|------|-------------|----------|
| **Levels** | Current amount of something | `bloodGlucose`, `adenosine`, `water` |
| **Rates** | Speed of change | `heartRate`, `respiratoryRate`, `GFR` |
| **Modes** | Discrete states | `metabolicMode`, `visualMode`, `sleepStage` |
| **Sensitivities** | Responsiveness | `insulinSensitivity`, `leptinSensitivity` |
| **Capacities** | Maximum potential | `glycogenCapacity`, `BAT.mass` |
| **Debts** | Accumulated deficits | `sleepDebt`, `recoveryDebt` |
| **Loads** | Accumulated burden | `allostaticLoad` |

---

## 4. INFORMATION FLOW

How changes propagate through the simulation.

### 4.1 Activity → Environment → Internal

```
Activity: go_outside
    ↓
Environment: ambientLight = 100,000 lux
    ↓
Internal:
    ├── Circadian: ↑cortisol spike
    ├── Visual: mode = panoramic (naturally)
    ├── Hormonal: ↑melatonin suppression
    └── Autonomic: ↑alertness
```

### 4.2 Activity → Internal (Direct)

```
Activity: eat_meal(carbs=100, protein=30, fat=20)
    ↓
Internal:
    ├── Digestive: stomachFullness ↑
    ├── Metabolic: bloodGlucose ↑ → insulin ↑
    ├── Hunger: ghrelin ↓, satiety ↑
    ├── Hormonal: orexin ↓ (drowsy)
    └── Energy: perceived energy changes over time
```

### 4.3 Internal → Internal (Cross-System)

```
Metabolic: insulin drops (fasting)
    ↓
Renal: aldosterone ↓
    ↓
Hydration: sodium excretion ↑
    ↓
Cardiovascular: blood volume ↓
    ↓
Autonomic: heart rate ↑ (compensation)
```

### 4.4 Time → Internal (Rhythms)

```
Time: 6:00 AM (circadian phase = early_morning)
    ↓
Internal:
    ├── Hormonal: cortisol ↑ (peak)
    ├── Thermoregulation: coreTemp rising
    ├── Energy: adenosine still clearing
    └── Cognitive: alertness ramping up
```

---

## 5. CAUSALITY CHAINS

Common sequences in the simulation.

### 5.1 Morning Protocol Chain

```
Activity: wake_up
    ↓
Activity: go_outside (within 30 min)
    ↓
Environment: ambientLight = 100,000 lux
    ↓
Circadian: cortisol spike +50%
    ↓
Activity: wait 90 min
    ↓
Energy: adenosine naturally cleared by cortisol
    ↓
Activity: drink_caffeine
    ↓
Energy: adenosine receptors blocked (no buildup behind)
    ↓
Result: Sustained energy, no afternoon crash
```

### 5.2 Keto Adaptation Chain

```
Activity: eat_meal(carbs=20, protein=100, fat=150) daily
    ↓
Metabolic: bloodGlucose stable low → insulin low
    ↓
Metabolic: glycogen depletes → ketone production begins
    ↓
Renal: aldosterone drops → sodium excreted (whoosh)
    ↓
Hydration: water follows sodium → weight loss
    ↓
(If not supplementing sodium)
Symptoms: headache, fatigue, cramps = "Keto Flu"
    ↓
(After 2-4 weeks)
Metabolic: fatAdaptation → 1.0
    ↓
Result: Stable energy, mental clarity, reduced hunger
```

### 5.3 Cold Exposure Chain

```
Activity: enter_cold_plunge (temp=10°C, duration=3min)
    ↓
Thermoregulation: coreTemp threat detected
    ↓
Autonomic: sympathetic ↑↑↑
    ↓
Hormonal: norepinephrine +530%, dopamine +250%
    ↓
Thermoregulation: BAT.activation ↑
    ↓
Thermoregulation: vasoconstriction → vasodilation rebound
    ↓
(Long-term)
Thermoregulation: BAT.mass ↑
    ↓
Result: Better cold tolerance, elevated mood for hours
```

---

## 6. TAXONOMY SUMMARY

| Category | What It Is | Examples |
|----------|------------|----------|
| **Environment** | External context | Light, temperature, altitude, social |
| **Activities** | Agent actions | Eat, exercise, sleep, go outside |
| **Systems** | Internal machinery | Metabolic, Hormonal, Renal |
| **Variables** | Measurable state | bloodGlucose, cortisol, heartRate |
| **Transitions** | State changes | High carb → ↑insulin → ↓ketones |
| **Homeostasis** | Feedback loops | Low BP → ↑RAAS → retain sodium |
| **Rhythms** | Time-based patterns | Circadian (24h), Ultradian (90m) |

---

## 7. IMPLEMENTATION NOTES

### Data Flow Architecture

```rust
// ═══════════════════════════════════════════════════════════════════════
// 1. ENVIRONMENT (External World)
// ═══════════════════════════════════════════════════════════════════════
struct Environment {
    light_lux: f32,
    temperature_c: f32,
    altitude_m: f32,
    humidity_pct: f32,
    social_context: SocialContext,
    clock_time: u16,              // minutes since midnight
    // Resources available
    food_available: bool,
    water_available: bool,
    sleep_opportunity: bool,
}

// ═══════════════════════════════════════════════════════════════════════
// 2. ACTIVITIES (The Verbs - User Input)
// ═══════════════════════════════════════════════════════════════════════
enum Activity {
    // Environment-changing
    GoOutside,
    GoInside,
    DimLights,
    SetThermostat(f32),

    // Resource transfer (moves things across boundary)
    Eat(Meal),
    Drink { liters: f32, sodium_mg: f32 },
    TakeSupplement(Supplement),

    // Internal triggers
    Exercise { mode: ExerciseMode, intensity: f32, duration: u16 },
    Sleep,
    Breathe(BreathingTechnique),

    // Passive
    Wait(u16),  // minutes
}

// ═══════════════════════════════════════════════════════════════════════
// 3. INTERFACE / SENSORS (Boundary Translation)
// ═══════════════════════════════════════════════════════════════════════
struct SensorInputs {
    // From environment
    retinal_light: f32,           // What the eyes detect
    skin_temperature: f32,        // What skin feels

    // From internal state
    baroreceptor_pressure: f32,   // Blood pressure sensors
    chemoreceptor_co2: f32,       // CO2 levels in blood
    osmoreceptor_concentration: f32,  // Fluid concentration

    // Proprioception
    body_position: BodyPosition,
    movement_velocity: f32,
}

impl SensorInputs {
    fn from_environment_and_state(env: &Environment, state: &BodyState) -> Self {
        Self {
            retinal_light: env.light_lux,
            skin_temperature: env.temperature_c,  // Modified by clothing insulation
            baroreceptor_pressure: state.cardiovascular.systolic_bp,
            chemoreceptor_co2: state.respiratory.pco2,
            osmoreceptor_concentration: state.hydration.osmolality,
            body_position: state.vestibular.position,
            movement_velocity: state.vestibular.velocity,
        }
    }
}

// ═══════════════════════════════════════════════════════════════════════
// 4A. SYSTEMS (The Logic / Processors)
// ═══════════════════════════════════════════════════════════════════════
trait System {
    /// Each system reads sensors + state, writes to state
    fn tick(&self, sensors: &SensorInputs, state: &mut BodyState);
}

struct CircadianSystem;
impl System for CircadianSystem {
    fn tick(&self, sensors: &SensorInputs, state: &mut BodyState) {
        // Light > 10,000 lux within 2 hours of wake → cortisol spike
        if sensors.retinal_light > 10_000.0 && state.circadian.hours_since_wake < 2.0 {
            state.hormonal.cortisol += 0.5;  // Morning spike
        }
        // etc.
    }
}

// ═══════════════════════════════════════════════════════════════════════
// 4B. STATE (The Variables - All Raw Numbers)
// ═══════════════════════════════════════════════════════════════════════
struct BodyState {
    metabolic: MetabolicState,
    hormonal: HormonalState,
    autonomic: AutonomicState,
    hydration: HydrationState,
    respiratory: RespiratoryState,
    cardiovascular: CardiovascularState,
    circadian: CircadianState,
    energy: EnergyState,
    // ... all 22 subsystems
}

// ═══════════════════════════════════════════════════════════════════════
// 5. AXES (Felt Physiological Pathways - Derived Read-Only)
// ═══════════════════════════════════════════════════════════════════════
struct AxisSnapshot {
    androgenic: f32,       // 0.0-1.0 (drive, libido, confidence)
    dopaminergic: f32,     // 0.0-1.0 (content vs seeking)
    metabolic: FuelMode,   // Glycolytic/Hybrid/Ketogenic
    adenosinergic: f32,    // 0.0-1.0 (alertness)
    autonomic: f32,        // -1.0 (stressed) to 1.0 (calm)
    immune: f32,           // 0.0-1.0 (resilience)
    thermoregulatory: f32, // 0.0-1.0 (cold tolerance)
}

impl AxisSnapshot {
    /// PURE FUNCTION: Axes are ALWAYS calculated from state
    fn from_state(state: &BodyState) -> Self {
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
        let t_factor = (s.hormonal.testosterone / 800.0).clamp(0.0, 1.0);
        let retention_bonus = match s.hormonal.retention_days as u16 {
            0..=2 => -0.3,
            3..=6 => 0.0,
            7..=14 => 0.2,
            _ => 0.15,
        };
        (t_factor + retention_bonus - s.hormonal.prolactin * 0.4).clamp(0.0, 1.0)
    }

    fn calc_dopaminergic(s: &BodyState) -> f32 {
        (s.hormonal.dopamine_baseline - s.hormonal.prolactin * 0.5).clamp(0.0, 1.0)
    }

    fn calc_autonomic(s: &BodyState) -> f32 {
        s.autonomic.parasympathetic - s.autonomic.sympathetic
    }

    // ... other axis calculations (see specs/04_AXES.md)
}

// ═══════════════════════════════════════════════════════════════════════
// THE SIMULATION (Orchestrator)
// ═══════════════════════════════════════════════════════════════════════
struct Simulation {
    environment: Environment,
    state: BodyState,
    systems: Vec<Box<dyn System>>,
    time: SimTime,
    history: Timeline,
}

impl Simulation {
    fn tick(&mut self, activities: Vec<Activity>) {
        // 1. Activities modify environment OR transfer resources
        for activity in &activities {
            self.apply_activity(activity);
        }

        // 2. Sensors read environment + state
        let sensors = SensorInputs::from_environment_and_state(
            &self.environment,
            &self.state
        );

        // 3. All systems process (in dependency order)
        for system in &self.systems {
            system.tick(&sensors, &mut self.state);
        }

        // 4. Homeostasis pass (feedback corrections)
        self.homeostasis_pass();

        // 5. Advance time
        self.time.advance();

        // 6. Store for branching/replay
        self.history.push(self.state.clone());
    }

    /// Get current axes (read-only view for UI)
    fn axes(&self) -> AxisSnapshot {
        AxisSnapshot::from_state(&self.state)
    }
}
