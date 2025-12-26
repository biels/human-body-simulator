# Data Dictionary

Flat index of ALL variables in the simulation. This is the **Single Source of Truth** for naming.

> **Rule:** If it's not in this dictionary, it doesn't exist in the codebase.

---

## Quick Reference

| Prefix | System | Rust Module |
|--------|--------|-------------|
| `renal.` | Renal System | `systems/renal.rs` |
| `resp.` | Respiratory & pH | `systems/respiratory.rs` |
| `comp.` | Body Composition | `systems/composition.rs` |
| `visual.` | Visual & Vestibular | `systems/visual.rs` |
| `meta.` | Metabolic | `systems/metabolic.rs` |
| `ans.` | Autonomic NS | `systems/autonomic.rs` |
| `hydra.` | Hydration | `systems/hydration.rs` |
| `horm.` | Hormonal | `systems/hormonal.rs` |
| `energy.` | Energy & Fatigue | `systems/energy.rs` |
| `circ.` | Circadian | `systems/circadian.rs` |
| `thermo.` | Thermoregulation | `systems/thermoregulation.rs` |
| `immune.` | Immune | `systems/immune.rs` |
| `attn.` | Attention/Cognitive | `systems/attention.rs` |
| `percept.` | Perceptual | `systems/perceptual.rs` |
| `ultra.` | Ultradian | `systems/ultradian.rs` |
| `hunger.` | Hunger/Satiety | `systems/hunger.rs` |
| `anab.` | Anabolic/MPS | `systems/anabolic.rs` |
| `allo.` | Allostatic Load | `systems/allostatic.rs` |
| `gut.` | Gut-Brain Axis | `systems/gut_brain.rs` |
| `cardio.` | Cardiovascular | `systems/cardiovascular.rs` |
| `digest.` | Digestive | `systems/digestive.rs` |
| `memory.` | Memory/Learning | `systems/memory.rs` |
| `env.` | Environment | `environment.rs` |
| `dim.` | Dimensions | `dimensions.rs` |

---

## Environment Variables

| Variable | Type | Unit | Range | Rust Path | Notes |
|----------|------|------|-------|-----------|-------|
| Light level | `f32` | lux | 0-150,000 | `env.light_lux` | 100k = bright sun, 500 = office |
| Temperature | `f32` | °C | -20 to 50 | `env.temperature_c` | Ambient temp |
| Altitude | `f32` | meters | 0-8848 | `env.altitude_m` | Affects pO2 |
| Humidity | `f32` | % | 0-100 | `env.humidity_pct` | Affects sweating efficiency |
| Clock time | `u16` | minutes | 0-1439 | `env.clock_time` | Minutes since midnight |
| Day of year | `u16` | day | 1-365 | `env.day_of_year` | For daylight calculation |
| Social context | `enum` | - | alone/small/crowd/intimate | `env.social_context` | |
| Food available | `bool` | - | - | `env.food_available` | Can eat? |
| Water available | `bool` | - | - | `env.water_available` | Can drink? |
| Sleep opportunity | `bool` | - | - | `env.sleep_opportunity` | Can sleep? |

---

## Renal System (Kidneys)

| Variable | Type | Unit | Range | Rust Path | Notes |
|----------|------|------|-------|-----------|-------|
| GFR | `f32` | mL/min | 60-120 | `renal.gfr` | Glomerular filtration rate |
| Urine production | `f32` | mL/hour | 20-100 | `renal.urine_production` | Output rate |
| Urine concentration | `f32` | mOsm/kg | 300-1200 | `renal.urine_concentration` | Dilute vs concentrated |
| Aldosterone | `f32` | normalized | 0.0-1.0 | `renal.aldosterone` | Na+ retention |
| ADH/Vasopressin | `f32` | normalized | 0.0-1.0 | `renal.adh` | H2O retention |
| Renin | `f32` | normalized | 0.0-1.0 | `renal.renin` | RAAS cascade start |
| Angiotensin II | `f32` | normalized | 0.0-1.0 | `renal.angiotensin_ii` | Vasoconstriction |

---

## Respiratory & pH System

| Variable | Type | Unit | Range | Rust Path | Notes |
|----------|------|------|-------|-----------|-------|
| Blood pH | `f32` | pH | 6.8-7.8 | `resp.blood_ph` | Normal: 7.35-7.45 |
| pCO2 | `f32` | mmHg | 20-60 | `resp.pco2` | Normal: 35-45 |
| pO2 | `f32` | mmHg | 60-100 | `resp.po2` | Normal: 80-100 |
| O2 saturation | `f32` | % | 80-100 | `resp.o2_sat` | Hemoglobin binding |
| Bicarbonate | `f32` | mEq/L | 18-30 | `resp.bicarbonate` | Buffer system |
| Respiratory rate | `f32` | breaths/min | 8-30 | `resp.rate` | Primary CO2 control |
| Tidal volume | `f32` | mL | 300-800 | `resp.tidal_volume` | Air per breath |
| CO2 tolerance | `f32` | normalized | 0.0-1.0 | `resp.co2_tolerance` | Anxiety threshold |

---

## Body Composition

| Variable | Type | Unit | Range | Rust Path | Notes |
|----------|------|------|-------|-----------|-------|
| Adipose mass | `f32` | kg | 3-100 | `comp.adipose_mass` | Total fat |
| Visceral fat | `f32` | kg | 0-20 | `comp.visceral_fat` | Inflammatory fat |
| Subcutaneous fat | `f32` | kg | 3-80 | `comp.subcutaneous_fat` | Less harmful |
| Lean muscle mass | `f32` | kg | 20-60 | `comp.lean_mass` | Metabolically active |
| Glycogen mass | `f32` | kg | 0-0.5 | `comp.glycogen_mass` | Liver + muscle |
| Body weight | `f32` | kg | 40-200 | `comp.body_weight` | Sum of all |
| BMR | `f32` | kcal/day | 1200-2500 | `comp.bmr` | Basal metabolic rate |
| TDEE | `f32` | kcal/day | 1500-4000 | `comp.tdee` | Total daily expenditure |

---

## Visual & Vestibular System

| Variable | Type | Unit | Range | Rust Path | Notes |
|----------|------|------|-------|-----------|-------|
| Visual mode | `enum` | - | focal/panoramic | `visual.mode` | Huberman lever |
| Gaze target | `enum` | - | near/mid/far/horizon | `visual.gaze_target` | |
| Optic flow | `f32` | normalized | 0.0-1.0 | `visual.optic_flow` | Forward motion |
| Light input | `f32` | lux | 0-150,000 | `visual.light_input` | What eyes detect |
| Vestibular activation | `f32` | normalized | 0.0-1.0 | `visual.vestibular` | Balance system |

---

## Metabolic System

| Variable | Type | Unit | Range | Rust Path | Notes |
|----------|------|------|-------|-----------|-------|
| Blood glucose | `f32` | mg/dL | 40-300 | `meta.glucose` | Normal: 70-100 |
| Blood ketones | `f32` | mmol/L | 0.0-10.0 | `meta.ketones` | Keto: 0.5-3.0 |
| Liver glycogen | `f32` | grams | 0-100 | `meta.liver_glycogen` | Depletes 10-12h fast |
| Muscle glycogen | `f32` | grams | 0-400 | `meta.muscle_glycogen` | Local muscle fuel |
| Insulin | `f32` | μU/mL | 0-150 | `meta.insulin` | Master switch |
| Insulin sensitivity | `f32` | normalized | 0.0-1.0 | `meta.insulin_sensitivity` | Higher = better |
| Glucagon | `f32` | normalized | 0.0-1.0 | `meta.glucagon` | Counter-regulatory |
| Free fatty acids | `f32` | mmol/L | 0.1-2.0 | `meta.ffa` | Released when insulin low |
| Metabolic mode | `enum` | - | glycolytic/ketogenic/hybrid | `meta.mode` | Current fuel |
| Fat adaptation | `f32` | normalized | 0.0-1.0 | `meta.fat_adaptation` | 2-4 weeks to build |
| Lactate | `f32` | mmol/L | 0.5-20.0 | `meta.lactate` | Bridge fuel |

---

## Autonomic Nervous System

| Variable | Type | Unit | Range | Rust Path | Notes |
|----------|------|------|-------|-----------|-------|
| Sympathetic tone | `f32` | normalized | 0.0-1.0 | `ans.sympathetic` | Fight/flight |
| Parasympathetic tone | `f32` | normalized | 0.0-1.0 | `ans.parasympathetic` | Rest/digest |
| HRV (RMSSD) | `f32` | ms | 10-150 | `ans.hrv` | Higher = better |
| Cortisol | `f32` | normalized | 0.0-1.0 | `ans.cortisol` | Stress hormone |

---

## Hydration & Electrolytes

| Variable | Type | Unit | Range | Rust Path | Notes |
|----------|------|------|-------|-----------|-------|
| Total body water | `f32` | liters | 30-50 | `hydra.total_water` | ~60% body weight |
| Intracellular water | `f32` | liters | 20-35 | `hydra.icf` | 2/3 of total |
| Extracellular water | `f32` | liters | 10-18 | `hydra.ecf` | 1/3 of total |
| Sodium | `f32` | mEq/L | 125-150 | `hydra.sodium` | Normal: 136-145 |
| Potassium | `f32` | mEq/L | 2.5-6.5 | `hydra.potassium` | Normal: 3.5-5.0 |
| Magnesium | `f32` | mg/dL | 1.0-3.0 | `hydra.magnesium` | Normal: 1.7-2.2 |
| Zinc | `f32` | μg/dL | 50-150 | `hydra.zinc` | Normal: 80-120 |
| Osmolality | `f32` | mOsm/kg | 260-310 | `hydra.osmolality` | Normal: 275-295 |

---

## Hormonal System

| Variable | Type | Unit | Range | Rust Path | Notes |
|----------|------|------|-------|-----------|-------|
| Dopamine baseline | `f32` | normalized | 0.0-1.0 | `horm.dopamine_baseline` | Tonic level (setpoint) |
| Dopamine phasic | `f32` | normalized | 0.0-2.0 | `horm.dopamine_phasic` | Acute spikes above baseline |
| Dopamine reuptake rate | `f32` | normalized | 0.5-2.0 | `horm.dopamine_reuptake` | DAT activity - ADHD: higher = clears faster |
| Receptor density | `f32` | normalized | 0.0-1.0 | `horm.receptor_density` | Number of D2 receptors - downregulates with frequent stimulation |
| Receptor sensitivity | `f32` | normalized | 0.0-1.0 | `horm.receptor_sensitivity` | How responsive each receptor is - upregulates with abstinence |
| Seeking behavior | `f32` | normalized | 0.0-1.0 | `horm.seeking` | Drive to find dopamine hits = f(1 - baseline × sensitivity) |
| Last reward time | `u32` | minutes | 0-∞ | `horm.last_reward_time` | Minutes since last high-reward activity |
| Norepinephrine | `f32` | normalized | 0.0-1.0 | `horm.norepinephrine` | Alertness |
| Adrenaline | `f32` | normalized | 0.0-1.0 | `horm.adrenaline` | Acute stress |
| Orexin | `f32` | normalized | 0.0-1.0 | `horm.orexin` | Wakefulness |
| Testosterone | `f32` | ng/dL | 200-1200 | `horm.testosterone` | |
| DHT | `f32` | normalized | 0.0-1.0 | `horm.dht` | Potent androgen |
| Prolactin | `f32` | multiplier | 0.0-4.0 | `horm.prolactin` | 1.0=baseline, 4.0=400% spike post-ejac, decays ~2 weeks |
| Retention days | `u16` | days | 0-365 | `horm.retention_days` | Since ejaculation |
| Serotonin | `f32` | normalized | 0.0-1.0 | `horm.serotonin` | Mood, satiety |
| Melatonin | `f32` | normalized | 0.0-1.0 | `horm.melatonin` | Sleep pressure |
| BDNF | `f32` | normalized | 0.0-1.0 | `horm.bdnf` | Brain plasticity |
| IGF-1 | `f32` | ng/mL | 100-400 | `horm.igf1` | Growth/recovery |
| Growth hormone | `f32` | ng/mL | 0-20 | `horm.gh` | Peaks sleep/fasting |

---

## Energy & Fatigue System

| Variable | Type | Unit | Range | Rust Path | Notes |
|----------|------|------|-------|-----------|-------|
| ATP | `f32` | normalized | 0.0-1.0 | `energy.atp` | Cellular energy |
| Perceived energy | `f32` | score | 0-100 | `energy.perceived` | Subjective |
| Sleep debt | `f32` | hours | 0-48 | `energy.sleep_debt` | Accumulated deficit |
| Adenosine | `f32` | normalized | 0.0-1.0 | `energy.adenosine` | Sleep pressure |
| Adenosine receptors available | `f32` | normalized | 0.0-1.0 | `energy.receptors_available` | Free receptors |
| Adenosine receptors blocked | `f32` | normalized | 0.0-1.0 | `energy.receptors_blocked` | By caffeine |
| Adenosine buildup | `f32` | normalized | 0.0-2.0 | `energy.adenosine_buildup` | Behind caffeine |
| Muscle fatigue | `f32` | normalized | 0.0-1.0 | `energy.muscle_fatigue` | Per-group avg |
| Central fatigue | `f32` | normalized | 0.0-1.0 | `energy.central_fatigue` | CNS fatigue |

---

## Circadian System

| Variable | Type | Unit | Range | Rust Path | Notes |
|----------|------|------|-------|-----------|-------|
| Time of day | `u16` | minutes | 0-1439 | `circ.time_of_day` | Since midnight |
| Phase | `enum` | - | 8 phases | `circ.phase` | early_morning, etc. |
| Light exposure today | `f32` | lux-hours | 0-∞ | `circ.light_exposure` | Accumulated |
| Core body temp | `f32` | °C | 35.5-38.5 | `circ.core_temp` | Circadian rhythm |
| Cortisol rhythm | `f32` | multiplier | 0.5-1.5 | `circ.cortisol_mult` | Circadian mod |
| Hours since wake | `f32` | hours | 0-24 | `circ.hours_awake` | For caffeine timing |

---

## Thermoregulation System

| Variable | Type | Unit | Range | Rust Path | Notes |
|----------|------|------|-------|-----------|-------|
| Core temp | `f32` | °C | 35-42 | `thermo.core_temp` | Normal: 36.5-37.5 |
| Skin temp | `f32` | °C | 28-38 | `thermo.skin_temp` | Varies by location |
| Sweating | `f32` | normalized | 0.0-1.0 | `thermo.sweating` | Activation |
| Shivering | `f32` | normalized | 0.0-1.0 | `thermo.shivering` | Activation |
| Vasodilation | `f32` | normalized | 0.0-1.0 | `thermo.vasodilation` | Heat dissipation |
| Vasoconstriction | `f32` | normalized | 0.0-1.0 | `thermo.vasoconstriction` | Heat conservation |
| Yawning | `f32` | normalized | 0.0-1.0 | `thermo.yawning` | Brain cooling |
| BAT activation | `f32` | normalized | 0.0-1.0 | `thermo.bat_activation` | Thermogenesis |
| BAT mass | `f32` | grams | 20-300 | `thermo.bat_mass` | Trainable |

---

## Immune System

| Variable | Type | Unit | Range | Rust Path | Notes |
|----------|------|------|-------|-----------|-------|
| Status | `enum` | - | healthy/fighting_bacterial/fighting_viral/inflamed/recovering | `immune.status` | |
| Inflammation | `f32` | normalized | 0.0-1.0 | `immune.inflammation` | General level |
| WBC count | `f32` | normalized | 0.0-1.0 | `immune.wbc` | Immune cells |
| Pro-inflammatory cytokines | `f32` | normalized | 0.0-1.0 | `immune.cytokines_pro` | IL-6, TNF-α |
| Anti-inflammatory cytokines | `f32` | normalized | 0.0-1.0 | `immune.cytokines_anti` | IL-10 |
| Fever response | `f32` | normalized | 0.0-1.0 | `immune.fever` | Intentional temp↑ |

---

## Attention & Cognitive System

| Variable | Type | Unit | Range | Rust Path | Notes |
|----------|------|------|-------|-----------|-------|
| Gamma | `f32` | exponent | 1.0-5.0 | `attn.gamma` | Motivation curve - ADHD: 2.5-4.0 |
| Flow state | `bool` | - | - | `attn.in_flow` | Currently in flow |
| Activation energy | `f32` | normalized | 0.0-1.0 | `attn.activation_energy` | Task start threshold |
| Focus duration | `f32` | minutes | 0-180 | `attn.focus_duration` | Sustained capacity |
| Exploration drive | `f32` | normalized | 0.0-1.0 | `attn.exploration` | Novelty-seeking - ADHD: naturally high |
| Exploitation capacity | `f32` | normalized | 0.0-1.0 | `attn.exploitation` | Ability to extract value from known path |
| Current mode | `enum` | - | exploring/exploiting/stuck | `attn.mode` | Current cognitive strategy |
| Discount rate | `f32` | normalized | 0.0-1.0 | `attn.discount_rate` | Present bias - higher = more impulsive |
| Time horizon | `f32` | hours | 0-∞ | `attn.time_horizon` | How far ahead planning |
| Social capacity | `f32` | normalized | 0.0-1.0 | `attn.social_capacity` | FIRST to degrade when tired |
| Executive function | `f32` | normalized | 0.0-1.0 | `attn.executive_function` | System 2 capacity |
| Motor precision | `f32` | normalized | 0.0-1.0 | `attn.motor_precision` | LAST to degrade when tired |

---

## Perceptual System

| Variable | Type | Unit | Range | Rust Path | Notes |
|----------|------|------|-------|-----------|-------|
| Frame rate | `f32` | fps equiv | 20-200 | `percept.frame_rate` | Perceptual sampling |
| Quantization mode | `enum` | - | binary/gradient | `percept.quantization` | 1-bit vs smooth |

---

## Ultradian System

| Variable | Type | Unit | Range | Rust Path | Notes |
|----------|------|------|-------|-----------|-------|
| Phase | `u8` | minutes | 0-90 | `ultra.phase` | Into current cycle |
| Cycle state | `enum` | - | peak/plateau/trough/recovery | `ultra.state` | |
| Focus capacity | `f32` | normalized | 0.0-1.0 | `ultra.focus_capacity` | Current in cycle |
| Rest need | `f32` | normalized | 0.0-1.0 | `ultra.rest_need` | Accumulated |

---

## Hunger & Satiety System

| Variable | Type | Unit | Range | Rust Path | Notes |
|----------|------|------|-------|-----------|-------|
| Ghrelin | `f32` | normalized | 0.0-1.0 | `hunger.ghrelin` | Hunger hormone |
| Leptin | `f32` | normalized | 0.0-1.0 | `hunger.leptin` | Satiety hormone |
| Hunger score | `f32` | score | 0-100 | `hunger.hunger_score` | Perceived |
| Satiety score | `f32` | score | 0-100 | `hunger.satiety_score` | Perceived |
| Leptin sensitivity | `f32` | normalized | 0.0-1.0 | `hunger.leptin_sensitivity` | Resistance? |

---

## Anabolic / MPS System

| Variable | Type | Unit | Range | Rust Path | Notes |
|----------|------|------|-------|-----------|-------|
| mTOR | `f32` | normalized | 0.0-1.0 | `anab.mtor` | Master switch |
| MPS | `f32` | normalized | 0.0-1.0 | `anab.mps` | Muscle protein synthesis |
| MPB | `f32` | normalized | 0.0-1.0 | `anab.mpb` | Muscle protein breakdown |
| Net protein balance | `f32` | signed | -1.0 to 1.0 | `anab.net_balance` | MPS - MPB |
| Anabolic window | `bool` | - | - | `anab.in_window` | Elevated MPS |
| Leucine level | `f32` | normalized | 0.0-1.0 | `anab.leucine` | Key amino |

---

## Allostatic Load System

| Variable | Type | Unit | Range | Rust Path | Notes |
|----------|------|------|-------|-----------|-------|
| Allostatic load | `f32` | score | 0-100 | `allo.load` | Cumulative burden |
| Adaptive capacity | `f32` | normalized | 0.0-1.0 | `allo.capacity` | Ability to handle |
| Recovery debt | `f32` | normalized | 0.0-1.0 | `allo.recovery_debt` | Deficit |

---

## Gut-Brain Axis System

| Variable | Type | Unit | Range | Rust Path | Notes |
|----------|------|------|-------|-----------|-------|
| Microbiome diversity | `f32` | normalized | 0.0-1.0 | `gut.diversity` | Higher = healthier |
| Microbiome balance | `f32` | signed | -1.0 to 1.0 | `gut.balance` | Good vs bad ratio |
| Gut permeability | `f32` | normalized | 0.0-1.0 | `gut.permeability` | Leaky gut |
| Gut serotonin | `f32` | normalized | 0.0-1.0 | `gut.serotonin` | 90% made here |
| Vagal tone | `f32` | normalized | 0.0-1.0 | `gut.vagal_tone` | Gut-brain strength |

---

## Cardiovascular System

| Variable | Type | Unit | Range | Rust Path | Notes |
|----------|------|------|-------|-----------|-------|
| Heart rate | `f32` | bpm | 40-200 | `cardio.hr` | Rest: 60-100 |
| Systolic BP | `f32` | mmHg | 70-200 | `cardio.systolic` | Normal: 90-120 |
| Diastolic BP | `f32` | mmHg | 40-120 | `cardio.diastolic` | Normal: 60-80 |
| Stroke volume | `f32` | mL | 50-150 | `cardio.stroke_volume` | Per beat |
| Cardiac output | `f32` | L/min | 3-20 | `cardio.cardiac_output` | HR × SV |
| Blood volume | `f32` | liters | 3.5-6.5 | `cardio.blood_volume` | Total blood |

---

## Digestive System

| Variable | Type | Unit | Range | Rust Path | Notes |
|----------|------|------|-------|-----------|-------|
| Stomach fullness | `f32` | normalized | 0.0-1.0 | `digest.fullness` | Mechanical stretch |
| Gastric emptying time | `f32` | hours | 0.5-6.0 | `digest.emptying_time` | Current food |
| Intestinal motility | `f32` | normalized | 0.0-1.0 | `digest.motility` | Movement rate |
| Digestive state | `enum` | - | fasted/digesting/absorbing/fasted_deep | `digest.state` | |
| Blood flow to gut | `f32` | normalized | 0.0-1.0 | `digest.blood_flow` | Para = more |

---

## Memory & Learning System

| Variable | Type | Unit | Range | Rust Path | Notes |
|----------|------|------|-------|-----------|-------|
| Working memory | `f32` | normalized | 0.0-1.0 | `memory.working` | Active capacity |
| Consolidation state | `enum` | - | encoding/consolidating/retrieving | `memory.state` | |
| Neuroplasticity window | `bool` | - | - | `memory.plasticity_window` | Enhanced learning |

---

## Dimensions (Dashboard - READ ONLY)

| Dimension | Type | Range | Formula | Rust Path |
|-----------|------|-------|---------|-----------|
| Focus | `f32` | 0-100 | `f(dopamine, norepinephrine, glucose, adenosine)` | `dim.focus` |
| Energy | `f32` | 0-100 | `f(ATP, glycogen, sleepDebt, caffeine)` | `dim.energy` |
| Mood | `f32` | 0-100 | `f(serotonin, dopamine, cortisol, social)` | `dim.mood` |
| Libido | `f32` | 0-100 | `f(testosterone, dopamine, prolactin, stress)` | `dim.libido` |
| Readiness | `f32` | 0-100 | `f(glycogen, muscleFatigue, HRV, sleep)` | `dim.readiness` |
| Hunger | `f32` | 0-100 | `f(ghrelin, leptin, glucose, fullness)` | `dim.hunger` |
| Stress | `f32` | 0-100 | `f(cortisol, sympathetic, allostaticLoad)` | `dim.stress` |
| Clarity | `f32` | 0-100 | `f(ketones, glucose, hydration, sleepDebt)` | `dim.clarity` |

---

## Summary Stats

| Metric | Count |
|--------|-------|
| Environment variables | 10 |
| Internal state variables | 110+ |
| Dimensions (derived) | 8 |
| Total systems | 22 |
| Rust modules | 25+ |
