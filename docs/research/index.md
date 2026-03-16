# Research Directions

围绕 c4nav-core 的四个研究方向。

## Overview

| Direction | Core Question | Status |
|-----------|--------------|--------|
| [Physics-Informed RL](pirl.md) | Can physics knowledge improve strategies without high-fidelity simulation? | Phase 2 planned |
| [Transfer Framework](transfer.md) | Can low-fidelity pre-training accelerate high-fidelity convergence? | Novel in USV domain |
| [4-Phase Roadmap](roadmap.md) | Systematic research plan from baseline paper to real-world validation | In progress |
| [Embodied Space](embodied-space.md) | Unified modeling framework for intelligent agent behavior | Theoretical framework |

## Evolution Path

```
Phase 1: Baseline paper (current c4nav-core, no code changes)
    |
    +-- Phase 2: PIRL upgrade (add physics to observation + reward)
    |       |
    |       +-- Phase 4: PIRL + real ship testing
    |
    +-- Phase 3: Baseline + SpaceR-USV + real ship testing
            |
            +-- Future: VLA + DRL hierarchical architecture
                    |
                    +-- Embodied Space: unified theoretical foundation
```
