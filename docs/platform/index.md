# Platform Overview

c4nav-core 的定位、架构设计，以及与 C4 官方 SpaceR-USV 平台的关系。

## Core Positioning

c4nav-core 是一个**面向 USV 导航的轻量级算法实验台**，而非通用算法库。

- **环境是核心贡献** -- USV 动力学 + LiDAR 仿真 + 竞赛规则对齐
- **算法是标准实现** -- DQN 全系列为已有技术的组合，非算法创新
- **Gymnasium 接口** -- 遵循 `reset()` / `step()` 标准，可接入任意兼容算法

## Why Not Just Use Gymnasium?

Gymnasium 自带环境（CartPole、LunarLander 等）同样轻量、快速。区别在于**领域特异性**：

| Gym Environment | Gap with USV Navigation |
|----------------|------------------------|
| LunarLander | 2D control, no navigation or obstacle avoidance |
| CarRacing | Track following, not free-space navigation |
| highway-env | Structured lanes, not open water |
| MiniGrid | Discrete grid, no continuous dynamics |

c4nav-core 在 Gym 生态中补了一个空位：**USV 导航避障仿真环境**。

## Applicable Scenarios

c4nav-core 的核心抽象（2D LiDAR + 点到点导航 + 惯性动力学）可迁移到：

- **室内移动机器人** -- 仓储 AGV、配送机器人（改参数即可）
- **无人机低空导航** -- 固定高度平面避障
- **自动驾驶简化场景** -- 停车场、园区低速场景
- **多智能体协同** -- 需扩展为多 Agent（改动较大）
