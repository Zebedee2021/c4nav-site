# Embodied Space Theory

## Background

Traditional information systems (CPS architecture) model physical space and cyber space, but lack explicit modeling of:

1. Task-environment coupling
2. Behavioral constraints and strategy space
3. Unified learning space for intelligent agents

Embodied AI research (Brooks, Clark) argues that intelligence exists in the **body + environment + action loop**, but lacks engineering modeling methods.

## Formal Definition

$$\mathcal{E} = (S, A, T, C, P)$$

| Symbol | Meaning |
|--------|---------|
| S | State Space |
| A | Action Space |
| T | Task Space |
| C | Constraint Space |
| P | Physical Environment Field |

**Embodied Space = Environment + State + Action + Task + Constraint**

## GEM Structure Model

```
G -- Geometry Space (maps, 3D scenes, structures)
E -- Environment Field (wind, water, electromagnetic, temperature)
M -- Mission Space (tasks, behavioral rules, operational norms)
```

## C4 Platform GEM Mapping

| GEM Element | SpaceR-USV Status | Completeness |
|-------------|------------------|:------------:|
| **G** (Geometry) | Unity 3D scenes, obstacles, water boundaries, ship model | Complete |
| **E** (Environment) | Simulink wind/wave/current, hydrodynamics | Complete |
| **M** (Mission) | **Missing** -- only hardcoded "navigate to target" | Insufficient |

**C4 has G and E, but lacks M.**

## Upgrade Path: C4 -> Embodied Space Platform

### Current C4

Single task: `navigate(A -> B)` with obstacle avoidance.

### Upgraded C4 with Mission Space

**T (Task Graph):**
```
         +-- patrol (area cruise)
         |
start ---+-- search (target search)
         |
         +-- avoid (dynamic obstacles) <-- trigger: obstacle detected
         |
         +-- track (target tracking)   <-- trigger: target found
         |
         +-- return (return to base)   <-- trigger: low battery / mission complete
```

**C (Constraint Set):**

| Constraint Type | Content | Current C4 | Upgraded |
|----------------|---------|:----------:|:--------:|
| Safety | Min distance to obstacles >= 2m | Collision only | Explicit safety modeling |
| Navigation rules | COLREGS (give way to starboard) | None | Rule engine |
| Energy | Battery level affects strategy | None | Energy model |
| Communication | Autonomous decisions beyond comm range | None | Comm range model |

## Relationship to VLA + DRL

Embodied Space theory provides the formal foundation for the VLA + DRL hierarchical architecture:

```
VLA layer -- operates in T (Task) and C (Constraint) spaces
  "Currently on patrol, vessel ahead, COLREGS requires starboard turn,
   battery at 40%, return to base after completion"
              |
              v
DRL layer -- operates in G (Geometry) and E (Environment) spaces
  "Turn right 22.5 deg, speed 1.2 m/s"
```

## Evolution Path

```
Control Systems -> CPS -> AI-CPS -> Embodied Space
```

The Embodied Space framework elevates C4 from an "RL training environment for competitions" to "a general-purpose platform for validating embodied intelligence theory."

## From RL Training Platform to Embodied Intelligence Platform

| Dimension | Current C4 | Embodied Space C4 |
|-----------|-----------|-------------------|
| What it trains | Single-point navigation | Multi-task autonomous decision-making |
| Agent type | Pure RL (cerebellum) | VLA + RL (brain + cerebellum) |
| Environment expression | G + E | G + E + M |
| Academic positioning | RL training platform | Embodied intelligence research platform |
| Industrial value | Competition tool | General USV simulation framework |
