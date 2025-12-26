# Human Body Simulator - Architecture

## Core Concept

Discrete-event simulator with branching timelines (inspired by mcts-simple).
Each "tick" = 1 minute. State is immutable → enables time-travel.

## Subsystems

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
│             ┌─────────────────────┐                                        │
│             │   ENERGY / FATIGUE  │                                        │
│             │   ───────────────── │                                        │
│             │   ATP               │                                        │
│             │   muscleGlycogen    │                                        │
│             │   perceivedEnergy   │                                        │
│             │   sleepDebt         │                                        │
│             └─────────────────────┘                                        │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Subsystem Details

### 1. Metabolic Subsystem
```typescript
interface MetabolicState {
  bloodGlucose: number;      // mg/dL (normal: 70-100)
  bloodKetones: number;      // mmol/L (keto: 0.5-3.0)
  liverGlycogen: number;     // grams (max ~100g)
  muscleGlycogen: number;    // grams (max ~400g)
  insulin: number;           // normalized 0-1
  glucagon: number;          // normalized 0-1
  freefattyAcids: number;    // mmol/L
}
```

**Transitions:**
- High carb meal → ↑glucose → ↑insulin → ↓ketones
- Fasting/keto → ↓glucose → ↓insulin → ↑ketones → ↑FFA
- Exercise → ↓glycogen → ↑glucagon

### 2. Autonomic Nervous System
```typescript
interface AutonomicState {
  sympathetic: number;       // 0-1 (fight/flight activation)
  parasympathetic: number;   // 0-1 (rest/digest activation)
  hrv: number;               // ms (higher = better recovery)
  cortisol: number;          // normalized, follows circadian
}
```

**Transitions:**
- HIIT → ↑sympathetic → ↓HRV → ↑cortisol
- Rest/meditation → ↑parasympathetic → ↑HRV
- Chronic stress → sustained ↑cortisol → ↓HRV baseline

**HRV Calculation:**
```
HRV = baselineHRV * parasympathetic / (sympathetic + 0.1)
```

### 3. Hydration & Electrolytes
```typescript
interface HydrationState {
  totalBodyWater: number;        // liters (typically ~42L for 70kg)
  intracellularWater: number;    // ~28L
  extracellularWater: number;    // ~14L
  sodium: number;                // mEq/L (normal: 136-145)
  potassium: number;             // mEq/L (normal: 3.5-5.0)
  magnesium: number;             // mg/dL (normal: 1.7-2.2)
  osmolality: number;            // mOsm/kg
}
```

**Keto-specific effects:**
- ↓insulin → ↑sodium excretion (natriuresis)
- Glycogen depletion → water loss (3g water per 1g glycogen)
- Need ~4-5g sodium/day in ketosis vs ~2g normally

### 4. Hormonal Pathways
```typescript
interface HormonalState {
  // Androgens
  testosterone: number;      // ng/dL
  freeTesto: number;         // pg/mL
  SHBG: number;              // nmol/L

  // Neurotransmitters (simplified as hormones)
  dopamine: number;          // normalized 0-1 (motivation/reward)
  serotonin: number;         // normalized 0-1 (mood/satiety)

  // Catecholamines
  adrenaline: number;        // normalized 0-1
  noradrenaline: number;     // normalized 0-1

  // Growth factors
  BDNF: number;              // normalized (brain plasticity)
  IGF1: number;              // ng/mL
}
```

**Transitions:**
- HIIT → ↑testosterone (acute), ↑adrenaline, ↑BDNF
- Fasting → ↑noradrenaline, ↑BDNF
- Sleep deprivation → ↓testosterone, ↓dopamine sensitivity
- Cold exposure → ↑noradrenaline, ↑dopamine

### 5. Energy & Fatigue
```typescript
interface EnergyState {
  ATP: number;               // normalized cellular energy
  perceivedEnergy: number;   // 0-100 subjective
  sleepDebt: number;         // hours
  muscleFatigue: number;     // 0-1 per muscle group
  centralFatigue: number;    // 0-1 CNS fatigue
}
```

## Activities (Inputs)

```typescript
type Activity =
  | { type: 'meal'; carbs: number; protein: number; fat: number; fiber: number }
  | { type: 'water'; liters: number; sodium?: number; potassium?: number }
  | { type: 'supplement'; name: 'sodium' | 'potassium' | 'magnesium' | 'caffeine'; amount: number }
  | { type: 'exercise'; mode: 'HIIT' | 'steady' | 'strength' | 'walk'; duration: number; intensity: number }
  | { type: 'sleep'; quality: number; duration: number }
  | { type: 'stress'; level: number; duration: number }
  | { type: 'cold_exposure'; temp: number; duration: number }
  | { type: 'meditation'; duration: number }
  | { type: 'fasting' }  // no-op, just no food
```

## Effects Matrix (Simplified)

| Activity | Glucose | Ketones | Sympathetic | HRV | Dopamine | Water |
|----------|---------|---------|-------------|-----|----------|-------|
| High-carb meal | ↑↑ | ↓↓ | ↓ | ↑ | ↑ (temp) | ↑ |
| Keto meal | → | ↑ | → | → | → | → |
| HIIT | ↓↓ | ↑ | ↑↑↑ | ↓↓ | ↑↑ | ↓↓ |
| Fasting | ↓ | ↑↑ | ↑ | varies | ↑ | ↓ |
| Sleep | → | varies | ↓↓ | ↑↑ | reset | → |
| Cold | → | → | ↑↑ | ↓→↑ | ↑↑↑ | → |
| Caffeine | ↑ | → | ↑ | ↓ | ↑ | ↓ |

## State Transitions

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
  state = updateEnergy(state);         // fatigue, perceived energy

  // 4. Apply cross-system effects
  state = applyCrossEffects(state);

  return state;
}
```

## Timeline / Branching (from mcts-simple)

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
```

## UI Concept

```
┌─────────────────────────────────────────────────────────────────┐
│  Timeline: [■■■■■■■■■■■■■■■░░░░░░░░░░░░░░░░░░░░░]  Day 1, 14:30 │
│            ↑ branch here                                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Glucose ████████░░░░░░░░░░  85 mg/dL                         │
│   Ketones ██████████████░░░░  1.8 mmol/L  ← in ketosis         │
│   HRV     ████████████░░░░░░  62 ms                            │
│   Energy  ██████████████████  89/100                           │
│                                                                 │
│   Sodium  ██████░░░░░░░░░░░░  LOW - need supplement            │
│   Water   ████████████░░░░░░  OK                               │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│  [+ Add Activity]                                               │
│                                                                 │
│  Scheduled:                                                     │
│    15:00  HIIT workout (20 min)                                │
│    16:00  Keto meal (30g protein, 40g fat)                     │
│    18:00  Magnesium supplement                                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Implementation Phases

### Phase 1: Core Loop
- [ ] State types and tick function
- [ ] Basic metabolic (glucose only)
- [ ] Event queue
- [ ] Simple CLI output

### Phase 2: Expand Subsystems
- [ ] Full metabolic (ketones, glycogen)
- [ ] Hydration + electrolytes
- [ ] Autonomic (sympathetic/para, HRV)

### Phase 3: Hormonal & Energy
- [ ] Dopamine, testosterone
- [ ] Fatigue model
- [ ] Circadian rhythms

### Phase 4: Timeline & UI
- [ ] Branching/time-travel
- [ ] React UI with visualizations
- [ ] Scheduler integration

## References

- CarbMetSim (glucose model): https://github.com/mukulgoyalmke/CarbMetSim
- Huberman Lab (protocols): various episodes on dopamine, testosterone, HRV
- Keto electrolytes: ~4-5g sodium, 1-3g potassium, 300-400mg magnesium daily
