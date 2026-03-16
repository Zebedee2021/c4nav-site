# 算法

c4nav-core 已实现的算法和可扩展方向。

## 已实现：DQN 系列（4 种变体）

全部 4 种变体在同一个 `DQNAgent` 类中实现，通过配置开关区分：

| 算法 | 配置文件 | 关键开关 |
|------|---------|---------|
| DQN (基线) | `baseline.yaml` | 全部关闭，MSE 损失 |
| Double DQN | `ddqn.yaml` | `use_double_dqn: true` |
| D3QN | `d3qn.yaml` | + `network_type: dueling` |
| ID3QN (本文) | `improved_d3qn.yaml` | + PER + 软更新 + 奖励塑形 + Huber + 学习率调度 |

## 可扩展：通过 SB3 接入 PPO

环境遵循标准 Gymnasium 接口，添加 PPO 仅需约 20 行封装代码。详见 [PPO 扩展](ppo-extension.md)。

## 算法-环境解耦

环境与算法**完全解耦**：

- 环境定义 `state_dim` 和 `action_dim`
- 智能体在构造函数中接收这些参数
- 修改观测维度**无需改动任何智能体代码**

## 章节

- [DQN 系列](dqn-family.md) -- 4 种变体详解
- [PPO 扩展](ppo-extension.md) -- 如何添加 PPO 支持
- [模型与推理](model-inference.md) -- 模型结构、存储、加载与推理
