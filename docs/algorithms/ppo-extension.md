# PPO 扩展

## 为什么引入 PPO？

### 两大技术路线

强化学习算法分为两大技术路线：

| 路线 | 代表算法 | 学什么 | 通俗类比 |
|------|---------|-------|---------|
| **基于价值** | DQN、DDQN、D3QN | 学"每个动作值多少分"（Q 值），选分最高的 | 考试填选择题，给每个选项打分 |
| **基于策略** | PPO、A2C、TRPO | 直接学"应该做什么"（策略概率分布） | 学开车，直接学肌肉记忆 |

c4nav-core 目前只实现了基于价值的 DQN 系列。引入 PPO 可构成完整的路线对比：

- **基于价值**：DQN -> DDQN -> D3QN -> ID3QN（渐进改进）
- **基于策略**：PPO（标准基线）

当审稿人质疑"为什么只有 DQN 变体？"时，跨路线对比可以增强论文的说服力。

### 为什么在同一平台上比较？

不同仿真平台之间的算法结果**不可直接比较**（场景、物理、观测、奖励都不同），因此跨路线对比必须在**同一个平台**上进行：

| 对比方式 | 是否有效 | 原因 |
|---------|:-------:|------|
| c4nav-core 上 DQN vs PPO | 有效 | 同一环境、同一观测、同一奖励，唯一变量是算法 |
| c4nav-core DQN vs highway-env PPO | **无效** | 环境不同，不知道差异来自算法还是环境 |
| SpaceR-USV DQN vs c4nav-core DQN | **无效** | 物理模型不同，任务难度不同 |

这也是 c4nav-core 遵循标准 Gymnasium 接口的重要原因 -- 只有接口标准化，才能在**同一个环境**里公平比较不同算法，而不是每换一个算法就得重写环境。

## 离散与连续动作空间兼容性

| 动作空间 | 兼容算法 | 不兼容 |
|---------|---------|--------|
| **离散** `Discrete(5)` (c4nav-core) | DQN 系列、PPO、A2C | DDPG、TD3、SAC |
| **连续** `Box(2,)` | PPO、A2C、SAC、TD3、DDPG | DQN 系列 |

c4nav-core 使用 `Discrete(5)`，因此 PPO 无需修改动作空间即可直接使用。DQN 只能处理离散动作，PPO 两种都可以 -- 这是 PPO 被广泛使用的原因之一。

!!! info "未来扩展"
    如果将 c4nav-core 的动作空间从离散 (5) 改为连续 `Box(2,)`（舵角 + 油门连续控制），则可以引入 SAC、TD3 等连续动作算法。但 DQN 系列将不再适用，需要全部替换为连续动作算法。

## 实现：Gym 封装类（约 20 行）

c4nav-core 已遵循 Gymnasium 的 `reset()` / `step()` 接口，只需补充 `observation_space` 和 `action_space` 的声明即可接入 SB3：

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

Stable-Baselines3 (SB3) 是最主流的 RL 算法库，内置 PPO、A2C、SAC、TD3 等算法。通过上面的封装类，所有 SB3 算法都可以一行代码替换：

```python
from stable_baselines3 import A2C
model = A2C("MlpPolicy", env, verbose=1)  # 只改这一行
```

## 预期性能

| 算法 | 总交互步数 | 训练时间（RTX 4060） |
|------|:--------:|:------------------:|
| DQN 系列 | ~1M（2000 轮 x 500 步） | ~25 分钟 |
| PPO | ~0.5-1M | ~15-30 分钟 |

PPO 支持向量化环境（`n_envs=4`，同时跑 4 个环境实例），可进一步加速训练。
