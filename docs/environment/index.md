# 仿真环境

c4nav-core 仿真环境的设计：从竞赛规则到 MDP 建模。

## 设计原则

C4 竞赛规则不是直接写进算法的，而是通过 **MDP 建模**间接编码到 RL 框架的四个要素中：

| 竞赛规则 | MDP 要素 | 代码模块 |
|---------|---------|---------|
| 水域规格 | 初始状态分布 | `scene.py` |
| 船体参数 | 状态转移函数 | `usv_dynamics.py` |
| 评分规则 | 奖励函数 | `reward.py` |
| 控制方式 | 动作空间 | `usv_env.py` 中的 `ACTION_MAP` |
| 传感器能力 | 观测空间 | `lidar_sensor.py` + `usv_env.py` |

## 章节

- [竞赛规则到 MDP 映射](mdp-mapping.md) -- 竞赛规则如何编码
- [动力学模型](dynamics.md) -- 一阶惯性 vs Fossen 三自由度
- [奖励函数](reward.md) -- 终端奖励 + 奖励塑形
