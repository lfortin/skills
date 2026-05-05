---
name: create-stellar-system
description: Use this skill when the user asks to design, simulate, or document a physically coherent stellar system, including stars, planets, orbital stability, thermal zones, and chemical composition/distribution for worldbuilding or hard-sci-fi settings.
license: MIT
metadata:
  author: lfortin
  version: "1.4.0"
  updated: 2026-05-05
  category: Worldbuilding
  tags: [astronomy, physics, scifi, simulation, worldbuilding, procedural-generation]
---

# Create Stellar System

Design a stellar system that is internally consistent with basic orbital and thermal physics. Keep outputs clear, structured, and reusable for downstream lore or simulation work.

## Workflow

1. Define the central star and its primary constraints.
2. Establish orbital architecture and stability rules.
3. Map thermal zones and energy flows.
4. Assign plausible planetary compositions by orbital region.
5. Produce a final system sheet using the output template.

## 1) Stellar Core Logic

The star is the primary gravitational and thermal driver.

- Choose **classification** (for example, M-class dwarf or G-class main sequence).
- Estimate **luminosity/flux behavior** across AU distances.
- Note **stellar wind and magnetosphere pressure** to inform atmospheric stripping and auroral intensity.
- Identify the likely **habitable zone** and rough **frost line**.

## 2) Orbital Mechanics And Stability

All major bodies must satisfy long-term stability assumptions.

- Compute or estimate the **tidal locking radius** from stellar mass.
- Set **eccentricity** targets:
  - Low (`e < 0.01`) for stable, near-circular orbits.
  - Higher (`e > 0.1`) only with explicit perturbation causes (resonance forcing, collisions, shepherd masses).
- Use **orbital resonances** (for example, 2:1 or 3:2) to justify compact multi-body systems.

## 3) Thermal Gradient And Energy Budget

Model the system as heat flow from source to sink.

- **Primary source:** stellar radiation.
- **Secondary sources:** tidal heating, radiogenic decay, residual contraction, or exotic sources if justified.
- **Sinks:** albedo reflection, atmospheric transport, radiation to space, or other setting-specific sinks.
- Place the **frost line** where volatile compounds can condense.

## 4) Planetary Compositional Logic

Tie composition to formation zone and migration history.

- **Inner refractory zone:** iron, nickel, silicates, dense rocky bodies.
- **Outer volatile zone:** ices and gas-rich compositions (`H2O`, `CO2`, `N2`, `CH4`).
- Include accretion dynamics: massive shepherd bodies can clear disk gaps and create ring/belt structure.

## 5) Triple Observational Standard

For complete mapping, describe system observations from three vantage points:

- **Perihelion Watch:** high-energy/star-proximal phenomena.
- **Median Watch:** temperate-zone traffic, intersections, and stable operations.
- **Aphelion Watch:** deep-system anomalies and boundary dynamics.

## Output Template

Use this template for each major body:

```markdown
### [Object Name]
- **Classification:** (Terrestrial / Metallic / Gas Giant / Ice Giant / Singularity)
- **Mass:** [Value in Earth masses or kg]
- **Diameter (average):** [X] m or km
- **Semi-Major Axis:** [X] AU
- **Orbital Period:** [X] Earth Years
- **Orbital Speed (average):** [X] km/s
- **Eccentricity (e):** [X.XXX]
- **Tidal Status:** (Locked / Resonant / Asynchronous)
- **Rotation Period:** [X] Hours or Days
- **Axial Tilt:** [X] Degrees
- **Surface Gravity:** [X] g
- **Hill Sphere Radius:** [X] km or AU
- **Surface Temperature (average):** [X] K
- **Atmospheric Profile:** [Pressure in bars] / [Chemical composition]
- **Geological Activity:** (Tectonic / Cryovolcanic / Inert)
- **Chemical Composition:** (for example, heavy metals, liquid hydrocarbons, rare isotopes)
```

## Quality Checks

Before finalizing, verify:

- Every body's orbit is physically plausible with the defined star mass.
- Thermal zones match star output and orbital distances.
- Composition matches formation region or clearly documented migration history.
- The final write-up uses consistent units and terminology.