# Algorithms

c4nav-core 已实现的算法和可扩展方向。

## Implemented: DQN Family (4 Variants)

All 4 variants are implemented in a single `DQNAgent` class, differentiated by config switches:

| Algorithm | Config | Key Switches |
|-----------|--------|-------------|
| DQN (Baseline) | `baseline.yaml` | All off, MSE loss |
| Double DQN | `ddqn.yaml` | `use_double_dqn: true` |
| D3QN | `d3qn.yaml` | + `network_type: dueling` |
| ID3QN (Ours) | `improved_d3qn.yaml` | + PER + soft update + shaping + Huber + LR schedule |

## Extensible: PPO via SB3

The environment follows standard Gymnasium interface. Adding PPO requires only a ~20-line wrapper class. See [PPO Extension](ppo-extension.md).

## Algorithm-Environment Separation

The environment and algorithm are **fully decoupled**:

- Environment defines `state_dim` and `action_dim`
- Agent accepts these as constructor parameters
- Changing observation dimension requires **zero agent-side code changes**

## Sections

- [DQN Family](dqn-family.md) -- 4 variants in detail
- [PPO Extension](ppo-extension.md) -- how to add PPO support
- [Model & Inference](model-inference.md) -- model structure, storage, loading, and inference
