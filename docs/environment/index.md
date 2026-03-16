# Simulation Environment

c4nav-core 仿真环境的设计：从竞赛规则到 MDP 建模。

## Design Principle

C4 竞赛规则不是直接写进算法的，而是通过 **MDP 建模**间接编码到 RL 框架的四个要素中：

| Competition Rule | MDP Element | Code Module |
|-----------------|-------------|-------------|
| Water area specification | Initial state distribution | `scene.py` |
| Ship specification | State transition function | `usv_dynamics.py` |
| Scoring rules | Reward function | `reward.py` |
| Control method | Action space | `ACTION_MAP` in `usv_env.py` |
| Sensor capability | Observation space | `lidar_sensor.py` + `usv_env.py` |

## Sections

- [Rules to MDP Mapping](mdp-mapping.md) -- how competition rules are encoded
- [Dynamics Model](dynamics.md) -- 1st-order inertia vs Fossen 3-DOF
- [Reward Function](reward.md) -- terminal rewards + shaping components
