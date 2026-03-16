# PPO Extension

## Why Add PPO?

Adding PPO creates a complete Value-based vs Policy-based comparison:

- **Value-based**: DQN -> DDQN -> D3QN -> ID3QN (progressive improvement)
- **Policy-based**: PPO (standard baseline)

This strengthens paper review resilience when reviewers ask "why only DQN variants?"

## Discrete vs Continuous Action Space Compatibility

| Action Space | Compatible Algorithms | Incompatible |
|-------------|----------------------|-------------|
| **Discrete** `Discrete(5)` (c4nav-core) | DQN family, PPO, A2C | DDPG, TD3 |
| **Continuous** `Box(2,)` | PPO, A2C, SAC, TD3, DDPG | DQN family |

c4nav-core uses `Discrete(5)`, so PPO is compatible without changing the action space.

## Implementation: Gym Wrapper (~20 lines)

```python
import gymnasium as gym
from gymnasium import spaces
import numpy as np

class USVNavGymWrapper(gym.Env):
    def __init__(self, config):
        super().__init__()
        self.env = USVNavEnv(config)
        self.observation_space = spaces.Box(
            low=-1.0, high=1.0,
            shape=(self.env.state_dim,), dtype=np.float32
        )
        self.action_space = spaces.Discrete(self.env.action_dim)

    def reset(self, seed=None, options=None):
        return self.env.reset(seed=seed)

    def step(self, action):
        return self.env.step(int(action))
```

## Training with SB3

```python
from stable_baselines3 import PPO

env = USVNavGymWrapper(config)
model = PPO("MlpPolicy", env, verbose=1)
model.learn(total_timesteps=500_000)
```

## Expected Performance

| Algorithm | Total Interaction Steps | Training Time (RTX 4060) |
|-----------|:----------------------:|:------------------------:|
| DQN family | ~1M (2000 ep x 500 steps) | ~25 min |
| PPO | ~0.5-1M | ~15-30 min |

PPO supports vectorized environments (`n_envs=4`), which can further speed up training.
