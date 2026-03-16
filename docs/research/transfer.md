# Hierarchical Sim Transfer Framework

## Core Idea

Pre-train in lightweight simulation, transfer to high-fidelity simulation, deploy to real ship.

```
Stage 1: c4nav-core (low-fidelity, fast)
  | Export model weights
  v
Stage 2: SpaceR-USV (high-fidelity, Fossen physics)
  | Same ROS node
  v
Stage 3: Real USV130 ship
```

## Novelty Assessment

**This approach has NOT been done in the USV domain.** Survey results:

| Domain | Representative Work | Approach |
|--------|-------------------|----------|
| USV sim-to-real | Batista et al. (IROS 2024) | Single sim -> real (not hierarchical) |
| Quadrotor | Ryou et al. (MIT, IJRR 2025) | Multi-fidelity Bayesian optimization |
| Autonomous driving | TU Delft (Duckietown) | CARLA -> Gym -> real car |
| Multi-robot | Qiu et al. (Tsinghua, IEEE ACCESS 2021) | Low-fi sampling + high-fi fine-tuning |
| Curriculum transfer | Shukla (Tufts, ICRA 2022) | Low-fi curriculum -> high-fi transfer |

**USV domain is completely blank** for hierarchical sim transfer.

## Three Connection Modes

| Mode | Description | Purpose |
|------|-------------|---------|
| **Direct test** | Load c4nav-core weights -> run in SpaceR-USV (no training) | Quantify transfer gap |
| **Fine-tuning** | Load weights -> continue training in SpaceR-USV | Accelerate convergence |
| **Domain randomization** | Randomize physics params during c4nav-core training | Train robust policies |

## Complete Experiment Design

| Experiment | Method | Measures |
|-----------|--------|----------|
| A | SpaceR-USV train from scratch (baseline) | Full training time |
| B | c4nav-core pretrain -> SpaceR-USV direct test | Transfer loss |
| C | c4nav-core pretrain -> SpaceR-USV fine-tune | Speedup ratio |
| D | c4nav-core + domain randomization -> SpaceR-USV direct test | Robustness gain |

## Why It Works

The ROS-based architecture of SpaceR-USV enables seamless connection:

```
Training phase:   RL algorithm -- ROS --> Unity simulator (virtual sensors)
Deployment phase: RL algorithm -- ROS --> Real ship hardware (real sensors)
```

The RL algorithm node doesn't change between simulation and real ship. Only the ROS data source switches.
