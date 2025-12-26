# Activities & Environment - The Input Layer

Activities are the **only** way the user interacts with the simulation. They modify the environment or move resources across the body boundary.

---

## Activity Types

| Type | Description | Example |
|------|-------------|---------|
| **Environment-Changing** | Modifies external context | Go outside, dim lights |
| **Resource Transfer** | Moves things into body | Eat, drink, supplement |
| **Internal Trigger** | Directly activates systems | Exercise, breathe, sleep |
| **Passive** | Time passes, no action | Wait, rest |

---

## Environment Variables

The external world state. Activities modify these, then the body responds.

```rust
struct Environment {
    // Physical
    light_lux: f32,          // 0-150,000 (sun = 100k, office = 500)
    temperature_c: f32,       // -20 to 50
    altitude_m: f32,          // 0-8848
    humidity_pct: f32,        // 0-100

    // Temporal
    clock_time: u16,          // 0-1439 (minutes since midnight)
    day_of_year: u16,         // 1-365

    // Social
    social_context: SocialContext,  // Alone, SmallGroup, Crowd, Intimate

    // Resource availability
    food_available: bool,
    water_available: bool,
    sleep_opportunity: bool,
}

enum SocialContext {
    Alone,
    SmallGroup,   // 2-5 people
    Crowd,        // Many people
    Intimate,     // Partner/close friend
}
```

---

## Environment-Changing Activities

These modify `Environment`, which then affects the body indirectly.

### Activity: Go Outside

**Parameters:** None

**Immediate Effects:**
```rust
fn go_outside(env: &mut Environment, time_of_day: u16) {
    // Light based on time (simplified)
    env.light_lux = match time_of_day {
        360..=480 => 10_000.0,    // Early morning
        480..=600 => 50_000.0,    // Morning
        600..=1080 => 100_000.0,  // Midday
        1080..=1200 => 50_000.0,  // Afternoon
        1200..=1320 => 10_000.0,  // Evening
        _ => 0.1,                  // Night
    };

    // Temperature would come from weather system
    env.temperature_c = 20.0;  // Placeholder
}
```

**Downstream Effects:**
- Light → Circadian system → cortisol spike (if morning)
- Light → Melatonin suppression
- Temperature → Thermoregulation

---

### Activity: Go Inside

**Parameters:** None

```rust
fn go_inside(env: &mut Environment) {
    env.light_lux = 500.0;      // Typical indoor
    env.temperature_c = 22.0;   // Climate controlled
}
```

---

### Activity: Dim Lights

**Parameters:** `target_lux: f32`

```rust
fn dim_lights(env: &mut Environment, target_lux: f32) {
    env.light_lux = target_lux.clamp(0.1, 500.0);
}
```

**Use Case:** Pre-sleep routine. Target < 100 lux allows melatonin onset.

---

### Activity: Set Thermostat

**Parameters:** `target_temp: f32`

```rust
fn set_thermostat(env: &mut Environment, target_temp: f32) {
    env.temperature_c = target_temp.clamp(15.0, 30.0);
}
```

---

### Activity: Enter Cold Plunge

**Parameters:** `water_temp: f32`, `duration_min: u16`

```rust
fn enter_cold_plunge(env: &mut Environment, water_temp: f32) {
    // Skin experiences water temp directly
    // Core temp responds via thermoregulation system
    env.temperature_c = water_temp;  // 2-15°C typically
}
```

**Effects (per minute in cold):**
| Variable | Change | Notes |
|----------|--------|-------|
| `horm.norepinephrine` | +0.05-0.20 | Up to +530% |
| `horm.dopamine_baseline` | +0.02-0.08 | Up to +250% |
| `ans.sympathetic` | +0.15 | Acute stress |
| `thermo.vasoconstriction` | +0.2 | Heat conservation |
| `thermo.bat_activation` | +0.1 | Non-shivering thermogenesis |

---

### Activity: Enter Sauna

**Parameters:** `temp: f32`, `duration_min: u16`

```rust
fn enter_sauna(env: &mut Environment, temp: f32) {
    env.temperature_c = temp;  // 80-100°C typical
    env.humidity_pct = 20.0;   // Dry sauna
}
```

**Effects (per minute):**
| Variable | Change | Notes |
|----------|--------|-------|
| `thermo.core_temp` | +0.01-0.03 | Rises slowly |
| `thermo.sweating` | +0.15 | Cooling response |
| `horm.gh` | +0.05 | Growth hormone boost |
| `cardio.hr` | +5-10 bpm | Cardiovascular demand |

---

## Resource Transfer Activities

These move substances across the body boundary.

### Activity: Eat Meal

**Parameters:**
```rust
struct Meal {
    carbs_g: f32,      // Grams of carbohydrates
    protein_g: f32,    // Grams of protein
    fat_g: f32,        // Grams of fat
    fiber_g: f32,      // Grams of fiber
    glycemic_index: f32, // 0-100 (affects glucose spike speed)
}
```

**Immediate Effects:**
```rust
fn eat_meal(state: &mut BodyState, meal: &Meal) {
    // Stomach fills
    let volume = (meal.carbs_g + meal.protein_g + meal.fat_g) / 500.0;
    state.digest.fullness += volume.min(1.0);

    // Ghrelin drops (delayed 20 min in reality)
    state.hunger.ghrelin *= 0.7;

    // Calories for energy accounting
    let calories = meal.carbs_g * 4.0 + meal.protein_g * 4.0 + meal.fat_g * 9.0;
    // Handled in body composition

    // Macros affect metabolic state over time (digestion system)
}
```

**Continuous Effects (as digested):**
| Macro | Effect | Timing |
|-------|--------|--------|
| Carbs | ↑glucose → ↑insulin → ↓ketones | 15-60 min |
| Protein | ↑leucine → ↑mTOR, moderate insulin | 1-3 hours |
| Fat | ↓gastric emptying, stable glucose | 3-5 hours |
| Fiber | ↑microbiome, ↓glucose spike | Hours |

**High Carb Meal Specifics:**
```rust
if meal.carbs_g > 50.0 {
    state.horm.orexin *= 0.8;  // Post-meal drowsiness
}
```

---

### Activity: Drink

**Parameters:**
```rust
struct Drink {
    liters: f32,
    sodium_mg: f32,
    potassium_mg: f32,
    magnesium_mg: f32,
}
```

**Immediate Effects:**
```rust
fn drink(state: &mut BodyState, drink: &Drink) {
    state.hydra.total_water += drink.liters;
    state.hydra.sodium += drink.sodium_mg / 23.0 / 14.0;  // Convert to mEq/L
    state.hydra.potassium += drink.potassium_mg / 39.0 / 14.0;
    state.hydra.magnesium += drink.magnesium_mg / 24.3 / 42.0;

    // Recalculate osmolality
    state.hydra.osmolality = calculate_osmolality(&state.hydra);
}
```

---

### Activity: Take Supplement

**Parameters:** `supplement: Supplement`, `dose_mg: f32`

```rust
enum Supplement {
    // Electrolytes
    Sodium,
    Potassium,
    Magnesium,
    Zinc,

    // Stimulants
    Caffeine,

    // Amino acids
    LTyrosine,
    AlphaGPC,
    Creatine,

    // Hormonal support
    VitaminD,
    TongkatAli,
    FadogiaAgrestis,

    // Other
    Omega3 { epa_mg: f32, dha_mg: f32 },
}
```

**Supplement Effects:**

| Supplement | Primary Effect | Onset |
|------------|----------------|-------|
| Caffeine | Block adenosine receptors | 20-45 min |
| L-Tyrosine | ↑dopamine precursor | 30-60 min |
| Alpha-GPC | ↑acetylcholine (focus) | 30-60 min |
| Creatine | ↑ATP availability | Chronic |
| Omega-3 (EPA) | ↓inflammation | Chronic |
| Omega-3 (DHA) | Brain structure | Chronic |
| Vitamin D | Hormone precursor | Chronic |

**Caffeine Specifics:**
```rust
fn take_caffeine(state: &mut BodyState, mg: f32) {
    // Block adenosine receptors
    let blocking_power = (mg / 100.0).min(0.8);
    state.energy.receptors_blocked += blocking_power;
    state.energy.receptors_available -= blocking_power;

    // But adenosine keeps building
    // (handled in energy system tick)
}
```

---

## Internal Trigger Activities

These directly activate internal processes.

### Activity: Exercise

**Parameters:**
```rust
struct Exercise {
    mode: ExerciseMode,
    intensity: f32,      // 0.0-1.0
    duration_min: u16,
}

enum ExerciseMode {
    HIIT,       // High intensity intervals
    Zone2,      // Aerobic endurance
    Strength,   // Resistance training
    Walk,       // Low intensity
    Sprint,     // Max effort bursts
    Yoga,       // Flexibility + parasympathetic
}
```

**HIIT Effects (per minute):**
```rust
fn hiit_tick(state: &mut BodyState, intensity: f32) {
    state.meta.muscle_glycogen -= 2.0 * intensity;
    state.meta.lactate += 0.5 * intensity;
    state.ans.sympathetic += 0.1 * intensity;
    state.horm.adrenaline += 0.1 * intensity;
    state.horm.norepinephrine += 0.08 * intensity;
    state.horm.bdnf += 0.02 * intensity;
    state.thermo.core_temp += 0.01 * intensity;
    state.energy.muscle_fatigue += 0.05 * intensity;
    state.cardio.hr += 30.0 * intensity;
}
```

**Zone 2 Effects (per minute):**
```rust
fn zone2_tick(state: &mut BodyState) {
    state.meta.ffa += 0.02;  // Fat oxidation
    state.meta.muscle_glycogen -= 0.5;
    state.ans.sympathetic += 0.02;
    state.cardio.hr += 15.0;
    // Long-term: ↑mitochondria, ↑fat adaptation
}
```

**Strength Effects (per set):**
```rust
fn strength_set(state: &mut BodyState, intensity: f32) {
    state.meta.muscle_glycogen -= 1.0 * intensity;
    state.anab.mtor += 0.1 * intensity;
    state.horm.testosterone += 5.0 * intensity;  // ng/dL
    state.energy.muscle_fatigue += 0.1 * intensity;
}
```

**Walk Effects (per minute):**
```rust
fn walk_tick(state: &mut BodyState) {
    state.ans.parasympathetic += 0.01;
    state.ans.sympathetic -= 0.005;
    state.visual.optic_flow = 0.7;  // Forward movement
    // Optic flow → amygdala quieting → ↓anxiety
}
```

---

### Activity: Sleep

**Parameters:**
```rust
struct Sleep {
    target_duration_hours: f32,
    quality: f32,  // 0.0-1.0
}
```

**Sleep Stage Cycle (per 90 min):**
| Stage | Duration | Effects |
|-------|----------|---------|
| Light (N1/N2) | ~50% | Transition, some memory |
| Deep (N3) | ~20% | ↑GH, physical repair, declarative memory |
| REM | ~25% | Emotional processing, procedural memory |

**Effects (per minute of sleep):**
```rust
fn sleep_tick(state: &mut BodyState, quality: f32) {
    state.is_sleeping = true;

    // Adenosine clearing
    state.energy.adenosine *= 0.98 * quality;
    state.energy.adenosine_buildup *= 0.95;

    // Sleep debt reduction
    state.energy.sleep_debt -= 0.01 * quality;

    // Hormonal
    if is_deep_sleep(state) {
        state.horm.gh += 0.05;
        state.horm.testosterone += 1.0;
    }

    // Memory consolidation
    state.memory.state = MemoryState::Consolidating;

    // Parasympathetic dominance
    state.ans.parasympathetic = 0.8;
    state.ans.sympathetic = 0.2;
}
```

---

### Activity: NSDR (Non-Sleep Deep Rest)

**Parameters:** `duration_min: u16`

**Effects:**
```rust
fn nsdr_tick(state: &mut BodyState) {
    // Huberman: 65% dopamine restoration
    state.horm.dopamine_baseline += 0.01;  // +65% over 20 min

    // Parasympathetic activation
    state.ans.parasympathetic += 0.05;
    state.ans.cortisol *= 0.98;

    // Partial adenosine clearing
    state.energy.adenosine *= 0.99;

    // Eyes closed → removes visual input
    state.visual.mode = VisualMode::Closed;
}
```

---

### Activity: Meditation

**Parameters:** `duration_min: u16`, `technique: MeditationType`

```rust
enum MeditationType {
    Focused,      // Single-point focus
    OpenMonitoring,  // Awareness of all
    LovingKindness,  // Compassion focus
}
```

**Effects (per minute):**
```rust
fn meditate_tick(state: &mut BodyState) {
    state.ans.parasympathetic += 0.02;
    state.ans.sympathetic -= 0.01;
    state.ans.cortisol *= 0.99;
    state.attn.focus_duration += 0.5;  // Training effect
}
```

---

### Activity: Breathing Technique

**Parameters:** `technique: BreathingTechnique`, `duration_min: u16`

```rust
enum BreathingTechnique {
    PhysiologicalSigh,  // Double inhale + long exhale
    BoxBreathing,       // 4-4-4-4
    WimHof,             // 30 breaths + hold
    NasalOnly,          // Slow nasal breathing
}
```

**Physiological Sigh (per rep):**
```rust
fn physiological_sigh(state: &mut BodyState) {
    // Fastest real-time stress reduction
    state.resp.pco2 -= 2.0;
    state.resp.blood_ph += 0.01;  // Slight alkaline shift
    state.ans.sympathetic -= 0.15;
    state.ans.parasympathetic += 0.1;
}
```

**Wim Hof (per round):**
```rust
fn wim_hof_round(state: &mut BodyState) {
    // 30 deep breaths → alkalosis
    state.resp.pco2 -= 10.0;
    state.resp.blood_ph += 0.05;

    // Adrenaline surge
    state.horm.adrenaline += 0.3;
    state.horm.norepinephrine += 0.2;

    // Increased cold tolerance temporarily
    state.thermo.cold_tolerance += 0.1;
}
```

---

### Activity: Ejaculation

**Parameters:** None

**Immediate Effects:**
```rust
fn ejaculation(state: &mut BodyState) {
    // Prolactin spike
    state.horm.prolactin += 0.8;

    // Dopamine crash
    state.horm.dopamine_baseline *= 0.6;
    state.horm.dopamine_phasic = 0.0;

    // Reset retention counter
    state.horm.retention_days = 0.0;

    // Zinc loss
    state.hydra.zinc -= 3.0;  // ~3mg per emission

    // Brief parasympathetic surge then fatigue
    state.ans.parasympathetic += 0.3;
    state.energy.perceived -= 20.0;
}
```

**Recovery Timeline:**
| Day | Prolactin | Dopamine | Testosterone |
|-----|-----------|----------|--------------|
| 0 | High | Crashed | Normal |
| 1-2 | Declining | Recovering | Normal |
| 3-6 | Normal | Normal | Rising |
| 7 | Normal | Normal | Peak (+45%) |
| 8+ | Normal | Normal | Elevated |

---

## Learning Activities

These affect the memory/learning system. The encoding strength depends on HOW you learn.

### Activity: Study (Passive)

**Parameters:** `duration_min: u16`, `material_difficulty: f32`

Passive learning - reading, watching lectures. Low retention.

```rust
fn study_passive(state: &mut BodyState, duration: u16, difficulty: f32) {
    // Passive consumption = 1.0x multiplier (baseline)
    state.memory.testing_mult = 1.0;
    state.memory.last_encoding = state.time;

    // Add to uncommitted load based on encoding strength
    let encoded = state.memory.encoding_strength * duration as f32 * difficulty * 0.01;
    state.memory.uncommitted_load += encoded;

    // Mild fatigue from sustained attention
    state.attn.executive_function *= 0.995;
}
```

---

### Activity: Study (Active Recall)

**Parameters:** `duration_min: u16`, `material_difficulty: f32`

Self-testing, flashcards, practice problems. 4x better retention.

```rust
fn study_active(state: &mut BodyState, duration: u16, difficulty: f32) {
    // Active recall = 3.5x multiplier
    state.memory.testing_mult = 3.5;
    state.memory.last_encoding = state.time;

    // Much more encoding per minute
    let encoded = state.memory.encoding_strength * duration as f32 * difficulty * 0.04;
    state.memory.uncommitted_load += encoded;

    // More tiring - uses executive function heavily
    state.attn.executive_function *= 0.99;

    // But also more rewarding (dopamine from "getting it right")
    state.horm.dopamine_phasic += 0.1;
}
```

---

### Activity: Teach/Explain

**Parameters:** `duration_min: u16`

Teaching others or explaining out loud. Forces deep processing.

```rust
fn teach(state: &mut BodyState, duration: u16) {
    // Teaching = 2.5x multiplier
    state.memory.testing_mult = 2.5;
    state.memory.last_encoding = state.time;

    // Social + cognitive engagement
    let encoded = state.memory.encoding_strength * duration as f32 * 0.03;
    state.memory.uncommitted_load += encoded;

    // Uses social capacity (degrades first when tired)
    state.attn.social_capacity *= 0.98;
}
```

---

### Activity: Take Break (Gap Effect)

**Parameters:** `duration_sec: u16` (10-30 seconds optimal)

Micro-break during learning. Brain replays at 10-20x speed.

```rust
fn learning_break(state: &mut BodyState, duration_sec: u16) {
    // Short breaks = replay consolidation
    if duration_sec >= 10 && duration_sec <= 60 {
        // Brain uses gap to rehearse recently encoded material
        // Reduces forgetting rate temporarily
        state.memory.forgetting_rate *= 0.5;

        // Small consolidation happens even awake
        let micro_consolidation = state.memory.uncommitted_load * 0.02;
        state.memory.uncommitted_load -= micro_consolidation;
    }

    // Executive function recovery
    state.attn.executive_function += 0.01;
}
```

---

### Activity: Spaced Repetition Session

**Parameters:** `material_id: string`, `session_number: u8`

Reviewing material at optimal spacing intervals.

```rust
fn spaced_repetition(state: &mut BodyState, session_number: u8) {
    // Spacing factor increases with proper intervals
    // Session 1: immediate, Session 2: next day, Session 3: 3 days, etc.
    state.memory.spacing_factor = match session_number {
        1 => 1.0,   // First exposure
        2 => 1.5,   // Next day
        3 => 1.8,   // 3 days later
        4 => 2.0,   // 1 week later
        _ => 2.0,   // Maintaining
    };

    // Combined with active recall
    state.memory.testing_mult = 3.0;

    // Very efficient encoding
    let encoded = state.memory.encoding_strength * 10.0 * state.memory.spacing_factor;
    state.memory.uncommitted_load += encoded;
}
```

---

### Activity: Deep Work Session

**Parameters:** `duration_min: u16` (90 min optimal = ultradian cycle)

Sustained focused work. Combines encoding with flow potential.

```rust
fn deep_work(state: &mut BodyState, duration: u16) {
    // First 5-10 minutes are painful (normal)
    // Then focus locks in

    if state.ultra.state == UltradianState::Peak
        || state.ultra.state == UltradianState::Plateau
    {
        // Aligned with ultradian cycle = optimal
        state.memory.testing_mult = 2.0;

        // BDNF can rise from sustained focus
        state.horm.bdnf += 0.001 * duration as f32;
    } else {
        // Against the cycle = harder
        state.memory.testing_mult = 1.2;
    }

    state.memory.last_encoding = state.time;

    let encoded = state.memory.encoding_strength * duration as f32 * 0.02;
    state.memory.uncommitted_load += encoded;

    // Must rest after (ultradian trough)
    state.ultra.rest_need += duration as f32 / 90.0;
}
```

---

## Passive Activities

### Activity: Wait

**Parameters:** `duration_min: u16`

```rust
fn wait(state: &mut BodyState, duration: u16) {
    for _ in 0..duration {
        tick(state, vec![]);  // Empty activity list
    }
}
```

Time passes, all systems update normally.

---

### Activity: Rest

**Parameters:** `duration_min: u16`

```rust
fn rest(state: &mut BodyState) {
    // Slightly enhanced recovery vs just waiting
    state.ans.parasympathetic += 0.01;
    state.energy.muscle_fatigue *= 0.995;
    state.ultra.rest_need -= 0.02;
}
```

---

## Activity Scheduling

Activities can be scheduled for future ticks:

```rust
struct ScheduledActivity {
    tick: u64,
    activity: Activity,
}

struct Timeline {
    scheduled: BTreeMap<u64, Vec<Activity>>,
    // ...
}

fn schedule(timeline: &mut Timeline, at_tick: u64, activity: Activity) {
    timeline.scheduled
        .entry(at_tick)
        .or_insert_with(Vec::new)
        .push(activity);
}
```

---

## Activity Classification Summary

| Activity | Target | Primary Systems |
|----------|--------|-----------------|
| Go outside | Environment | Circadian, Visual |
| Go inside | Environment | Circadian |
| Dim lights | Environment | Circadian |
| Cold plunge | Environment + Body | Thermo, Hormonal |
| Sauna | Environment + Body | Thermo, Cardio |
| Eat meal | Body | Metabolic, Digestive, Hunger |
| Drink | Body | Hydration |
| Supplement | Body | Various |
| HIIT | Body | Metabolic, Hormonal, Cardio |
| Zone 2 | Body | Metabolic, Cardio |
| Strength | Body | Anabolic, Hormonal |
| Walk | Body | Autonomic, Visual |
| Sleep | Body | Energy, Hormonal, Memory |
| NSDR | Body | Hormonal, Autonomic |
| Meditate | Body | Autonomic, Attention |
| Breathe | Body | Respiratory, Autonomic |
| Ejaculation | Body | Hormonal |
| Study (passive) | Body | Memory, Attention |
| Study (active) | Body | Memory, Attention, Hormonal |
| Teach/Explain | Body | Memory, Attention, Social |
| Learning Break | Body | Memory, Attention |
| Spaced Repetition | Body | Memory |
| Deep Work | Body | Memory, Attention, Ultradian |
| Wait | None | Time passes |
