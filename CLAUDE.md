# Human Body Simulator

## What This Is
A discrete-event physiological simulator with branching timelines. Simulate "how would I feel if I did X vs Y?" with 1-minute tick resolution.

## Core Intent
- Model the body as interconnected systems that respond to activities
- Enable time-travel / branching to compare protocols
- Answer: "What's the optimal morning routine?" or "Keto vs carbs for focus?"

## Key Concepts
- **Activities** are the only input (eat, sleep, exercise, cold plunge)
- **Systems** process inputs and maintain homeostasis
- **Axes** are the output - 7 felt physiological pathways:
  1. Androgenic (retention/testosterone)
  2. Dopaminergic (content vs seeking)
  3. Metabolic (keto vs glycolytic)
  4. Adenosinergic (alertness vs sleepy)
  5. Autonomic (calm vs stressed)
  6. Immune (resilient vs run down)
  7. Thermoregulatory (cold tolerance)
- User never sets internal state directly—must do activities that trigger it

## Sources
- Huberman Lab protocols (light, cold, dopamine, focus)
- Keto/metabolic research
- Semen retention effects
- "What I've Learned" (WIL) videos

## Specs Structure
- `specs/DATA_DICTIONARY.md` - SSOT for all variables
- `specs/00_ARCHITECTURE.md` - Tick loop, system order
- `specs/01_ENGINE_CORE.md` - Homeostatic machinery
- `specs/02_CONTROLLERS.md` - Regulatory systems
- `specs/03_ACTIVITIES.md` - All activities
- `specs/04_AXES.md` - 7 physiological axes (the "how do I feel" output)

## Implementation
- Language: Rust (for immutability, ECS pattern)
- State is immutable per tick → enables branching/replay
- Systems run in dependency order (8 passes per tick)
