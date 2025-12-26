# System Dependencies (Reference Document)

> **This is NOT runtime code.** This documents which variables affect which others, for human understanding only. The actual simulation uses simple ECS - the graph is implicit in the code.

---

## Why This Document Exists

When tweaking the simulation, we need to understand:
- **What affects what** - If sodium is low, what breaks?
- **Why some variables matter more** - Fan-out determines impact
- **Where to look when debugging** - Trace the paths

---

## The ~30 Truly Stateful Variables

These persist across ticks. Everything else is derived.

### Slow-Drift (High Cascade Impact)

| Variable | Rust Path | Timescale | Fan-Out | Notes |
|----------|-----------|-----------|---------|-------|
| Receptor density | `horm.receptor_density` | Weeks | 8+ | Affects ALL dopamine calculations |
| Fat adaptation | `meta.fat_adaptation` | 2-4 weeks | 6+ | Determines metabolic mode ceiling |
| BAT mass | `thermo.bat_mass` | Months | 4+ | Cold tolerance ceiling |
| Sleep debt | `energy.sleep_debt` | Days | 10+ | Accumulates, affects everything |
| Retention days | `horm.retention_days` | Days | 7+ | Compounds into many downstream vars |
| Leptin sensitivity | `hunger.leptin_sensitivity` | Weeks | 3+ | Can develop resistance |

### Medium-Drift

| Variable | Rust Path | Timescale | Fan-Out | Notes |
|----------|-----------|-----------|---------|-------|
| Receptor sensitivity | `horm.receptor_sensitivity` | Days | 6+ | Faster than density |
| Prolactin | `horm.prolactin` | Days | 5+ | 4-day half-life |
| Adenosine | `energy.adenosine` | Hours | 4+ | Clears with sleep |
| Adenosine buildup | `energy.adenosine_buildup` | Hours | 3+ | Behind caffeine block |
| Liver glycogen | `meta.liver_glycogen` | Hours | 4+ | Depletes/replenishes |
| Muscle glycogen | `meta.muscle_glycogen` | Hours | 3+ | Per-muscle fuel |

### Fast-Drift (Moment-to-Moment)

| Variable | Rust Path | Timescale | Fan-Out | Notes |
|----------|-----------|-----------|---------|-------|
| Sodium | `hydra.sodium` | Hours | 12+ | HIGHEST fan-out |
| Potassium | `hydra.potassium` | Hours | 8+ | Nerve/muscle function |
| Magnesium | `hydra.magnesium` | Hours | 6+ | Parasympathetic |
| Hydration | `hydra.total_water` | Hours | 9+ | Everything needs it |
| Glucose | `meta.glucose` | Minutes | 6+ | Rapid swings |
| Caffeine | `energy.receptors_blocked` | Hours | 4+ | 5-6h half-life |
| Sympathetic | `ans.sympathetic` | Minutes | 5+ | Fight/flight |
| Parasympathetic | `ans.parasympathetic` | Minutes | 5+ | Rest/digest |
| Cortisol | `ans.cortisol` | Hours | 6+ | Stress hormone |
| Norepinephrine | `horm.norepinephrine` | Minutes | 5+ | Alertness |
| Core temp | `thermo.core_temp` | Minutes | 4+ | Thermoregulation |
| Testosterone | `horm.testosterone` | Hours | 4+ | Daily rhythm |
| Melatonin | `horm.melatonin` | Hours | 3+ | Circadian |

---

## Fan-Out Analysis

**Fan-out = how many downstream calculations use this variable**

Higher fan-out = more impact when this variable changes.

| Variable | Direct Deps | Total Reach | Effective Weight |
|----------|-------------|-------------|------------------|
| Sleep (quality/debt) | 6 | 15+ | Very High |
| Retention state | 7 | 13+ | Very High |
| Sodium | 6 | 12+ | Very High |
| Hydration | 5 | 9+ | Very High |
| Receptor density | 4 | 8+ | High |
| Dopamine baseline | 3 | 7+ | High |
| Glucose stability | 3 | 6+ | High |
| Sympathetic/Para | 4 | 5+ | Medium |
| Norepinephrine | 2 | 4+ | Medium |
| Focus (derived) | 0 | 0 | Output |
| Axes (derived) | 0 | 0 | Output |

**The insight:** Fixing high-fan-out variables fixes many things at once. Fixing low-fan-out (like focus directly) doesn't stick - it's being steered by upstream state.

---

## Dependency Maps

### Electrolytes → Everything

```
Sodium ─────┬──→ Nerve Conduction ──┬──→ Dopamine Release ──→ Focus
            │                       │
            ├──→ Muscle Function ───┼──→ Exercise Capacity
            │                       │
            ├──→ Kidney Function ───┼──→ Hydration Regulation
            │                       │
            └──→ Cardiac Function ──┘

Why 5 cents of salt can fix "everything feels off"
```

### Retention → Multiple Pathways

```
Retention ──┬──→ Prolactin (suppressed) ──→ Dopamine Signaling ──┐
            │                                                     │
            ├──→ Receptor Sensitivity ────────────────────────────┼──→ Focus
            │                                                     │
            ├──→ Testosterone Utilization ──→ Confidence          │
            │                                                     │
            ├──→ Sleep Efficiency ──→ Recovery ───────────────────┘
            │
            └──→ Immune Function

Why it feels like a "superpower" - 5+ pathways activated simultaneously
```

### Sleep → Highest Fan-Out

```
Sleep ──┬──→ Adenosine Clearance ──→ Alertness ──┐
        │                                         │
        ├──→ Memory Consolidation                 │
        │                                         │
        ├──→ Hormone Regulation ──┬──→ Cortisol   ├──→ All Cognitive
        │                         │               │    Functions
        │                         └──→ GH         │
        │                                         │
        ├──→ Glymphatic Clearance ────────────────┤
        │                                         │
        └──→ Immune Function                      │

Fan-out: 15+ (highest of any single variable)
```

### Dopamine System (Internal)

```
                    ┌─────────────────────────────────────┐
                    │         EFFECTIVE DOPAMINE          │
                    │  = baseline × density × sensitivity │
                    │              / reuptake             │
                    └─────────────────────────────────────┘
                                      │
        ┌─────────────────────────────┼─────────────────────────────┐
        │                             │                             │
        ▼                             ▼                             ▼
   ┌─────────┐                 ┌─────────────┐              ┌──────────────┐
   │ Seeking │                 │ Exploration/│              │    Time      │
   │ (inverse)│                │ Exploitation│              │  Preference  │
   └─────────┘                 └─────────────┘              └──────────────┘
        │                             │                             │
        ▼                             ▼                             ▼
   Scrolling,                  Can I stick with              Present bias
   restlessness                this task?                    vs future

ADHD: High reuptake → same input, less effect → need "Level 10" for "Level 5"
```

---

## Pathway Examples

### "Why do I feel foggy?"

Trace backwards from focus:

```
Focus is low
    ↑
Effective dopamine is low
    ↑
Check: baseline? density? sensitivity? reuptake?
    ↑
receptor_density degraded (weeks of overstimulation)
    OR
prolactin elevated (recent ejaculation, 4-day half-life)
    OR
sleep_debt accumulated (adenosine still high)
    OR
sodium low (nerve conduction impaired)

FIX: Identify which upstream variable, address THAT
```

### "Why does cold shower help focus?"

Trace forward from cold exposure:

```
Cold exposure
    │
    ├──→ Norepinephrine +300%
    │         │
    │         ├──→ Alertness ↑
    │         │
    │         └──→ Dopamine baseline +250% (lasts hours)
    │                   │
    │                   └──→ Focus ↑, Seeking ↓
    │
    └──→ Sympathetic activation ──→ Wakefulness

Multiple paths to focus, all boosted simultaneously
```

### "Why does the carb crash happen?"

```
Fat-adapted state (low insulin normally)
    │
    ▼
Eat carbs
    │
    ├──→ Insulin spikes (body not used to this)
    │
    ├──→ Wants to retain sodium, BUT sodium reserves depleted from keto
    │         │
    │         └──→ Electrolyte gap ──→ "Crash" feeling
    │
    └──→ Glucose spike then crash (reactive hypoglycemia)

FIX: Pre-load sodium 2h before carb meal, not "the carbs"
```

---

## Stateful vs Derived Summary

```
┌─────────────────────────────────────────────────────────────────┐
│  STATEFUL (~30 variables)                                       │
│  - Persist between ticks                                        │
│  - Have decay/accumulation functions                            │
│  - This is what gets saved when branching timelines             │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼ pure computation
┌─────────────────────────────────────────────────────────────────┐
│  DERIVED (~80 variables)                                        │
│  - Computed fresh each tick from stateful vars                  │
│  - effective_dopamine, hunger_score, etc.                       │
│  - Could be recomputed anytime from state snapshot              │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼ aggregation
┌─────────────────────────────────────────────────────────────────┐
│  AXES (7)                                                       │
│  - Pure aggregations of derived values                          │
│  - The "dashboard" - what you feel                              │
│  - NOT stateful, just a view                                    │
└─────────────────────────────────────────────────────────────────┘
```

---

## Using This Document

**When debugging the simulation:**
1. Identify which output/axis is wrong
2. Look up its dependencies in the maps above
3. Trace upstream to find which stateful variable is off
4. Check if that stateful variable's update logic is correct

**When adding new variables:**
1. Is it stateful (persists) or derived (computed)?
2. What does it depend on? (inputs)
3. What depends on it? (outputs → fan-out)
4. Add to appropriate section in DATA_DICTIONARY.md
5. Update dependency maps here if high fan-out

**When tweaking the model:**
1. High fan-out variables have more impact
2. Slow-drift variables compound over time
3. Fix upstream, not downstream
