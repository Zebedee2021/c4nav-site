# PPO 扩展

## 为什么引入 PPO？

引入 PPO 可构成完整的基于价值 vs 基于策略的算法对比：

- **基于价值**：DQN -> DDQN -> D3QN -> ID3QN（渐进改进）
- **基于策略**：PPO（标准基线）

当审稿人质疑"为什么只有 DQN 变体？"时，这可以增强论文的说服力。

## 离散与连续动作空间兼容性

| 动作空间 | 兼容算法 | 不兼容 |
|---------|---------|--------|
| **离散** `Discrete(5)` (c4nav-core) | DQN 系列、PPO、A2C | DDPG、TD3 |
| **连续** `Box(2,)` | PPO、A2C、SAC、TD3、DDPG | DQN 系列 |

c4nav-core 使用 `Discrete(5)`，因此 PPO 无需修改动作空间即可直接使用。

## 实现：Gym 封装类（约 20 行）

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

## 使用 SB3 训练

```python
from stable_baselines3 import PPO

env = USVNavGymWrapper(config)
model = PPO("MlpPolicy", env, verbose=1)
model.learn(total_timesteps=500_000)
```

## 预期性能

| 算法 | 总交互步数 | 训练时间（RTX 4060） |
|------|:--------:|:------------------:|
| DQN 系列 | ~1M（2000 轮 x 500 步） | ~25 分钟 |
| PPO | ~0.5-1M | ~15-30 分钟 |

PPO 支持向量化环境（`n_envs=4`），可进一步加速训练。
