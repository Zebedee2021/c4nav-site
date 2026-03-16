# Four-Phase Research Roadmap

## Overview

| Phase | Goal | Code Changes | Output |
|-------|------|:------------:|--------|
| 1 | Finalize baseline paper | None | Paper: ID3QN_USV_Navigation |
| 2 | PIRL upgrade + comparison | 3 files modified | Paper: PIRL for USV DRL |
| 3 | Baseline sim-to-real | ROS wrapper | Paper: Sim-to-real comparison |
| 4 | PIRL sim-to-real | Combine Phase 2+3 | Paper: Hierarchical transfer |

## Dependency and Parallelism

```
Phase 1 (paper refinement) -----------------> Submit
      |
      +-- Phase 2 (PIRL code changes) ------> Phase 2 paper
      |         |
      |         +-----> Phase 4 (PIRL + real ship)
      |
      +-- Phase 3 (baseline + real ship) ----> Phase 3 paper
```

- **Phase 1 first**: no code changes needed, can start immediately
- **Phase 2 and 3 in parallel**: Phase 2 modifies code, Phase 3 prepares real ship
- **Phase 4 depends on Phase 2**: needs PIRL version completed

## Phase-by-Phase Physics Coverage

| Phase | A: Transition | B: Observation | C: Reward | D: Network |
|-------|:---:|:---:|:---:|:---:|
| Phase 1 | 1st-order inertia | 184D geometric | distance+safety+heading | None |
| Phase 2 | 1st-order (unchanged) | **190D + physics** | **+ energy + jerk** | None |
| Phase 3 | **Fossen + wind/wave** | Per SpaceR-USV | Per SpaceR-USV | None |
| Phase 4 | **Fossen + wind/wave** | **+ physics** | **+ energy + jerk** | None |

## Developer Summary

We are conducting systematic research based on the c4nav-core simulation environment, in four phases:

**Phase 1** uses the existing c4nav-core environment and ID3QN algorithm to complete the baseline paper. No code changes involved.

**Phase 2** introduces Physics-Informed RL into c4nav-core: exposing dynamics model's internal physics quantities (linear/angular acceleration) in the observation vector (184D -> 190D), and adding energy cost and jerk penalty to the reward function. Changes are concentrated in 3 files; the agent network requires zero modification. Comprehensive before/after comparison experiments follow.

**Phase 3** deploys current baseline policies and SpaceR-USV (official Unity) policies to the real USV130 ship for sim-to-real transfer comparison. The core difference is physics fidelity: c4nav-core uses simplified 1st-order inertia, SpaceR-USV uses Fossen 3-DOF with wind/wave/current.

**Phase 4** repeats Phase 3 testing with the PIRL-upgraded version from Phase 2, comparing baseline vs physics-informed transfer quality.

The significance of this work: c4nav-core, as a lightweight pure-Python simulation, trains an order of magnitude faster than the Unity environment. By progressively introducing physics knowledge and conducting real-ship validation, we address a practical question -- whether physics-informed observation expansion and reward shaping can effectively reduce the sim-to-real gap without significantly increasing simulation complexity.
