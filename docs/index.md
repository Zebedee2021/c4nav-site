# C4Nav - 无人艇自主导航

**全国海洋航行器设计与制作大赛 C4 智能导航赛道 -- 无人艇自主导航研究平台**

---

## 项目概述

C4Nav 是一个面向无人艇 (USV) 自主导航研究的开源项目，围绕 C4 智能导航赛道构建了从算法研究到实船部署的完整技术栈。

核心组件 **c4nav-core** 是一个基于 Gymnasium 接口标准构建的轻量级仿真环境，集成了简化 USV 动力学模型、360° LiDAR 感知仿真和竞赛规则对齐的奖励体系，在保持训练效率的同时提供了足够的领域特异性，支持多种 DRL 算法的快速对比验证。

## 仓库架构

```
                        c4nav-site
                      文档中心
                    (MkDocs + GitHub Pages)
                           |
            +--------------+--------------+
            |              |              |
        c4nav-core     c4nav-data     c4nav-demo
        (私有)         (公开)         (公开)
        算法与环境     训练数据       交互式
        训练脚本       论文 PDF       ECharts 可视化
        CI 工作流      图表           双语
```

| 仓库 | 角色 | 主要内容 |
|------|------|---------|
| [c4nav-site](https://zebedee2021.github.io/c4nav-site/) | 文档中心 | MkDocs 站点、研究笔记 |
| [c4nav-core](https://github.com/Zebedee2021/c4nav-core) | 算法与训练 | DQN 智能体、Gym 环境、训练脚本 |
| [c4nav-data](https://github.com/Zebedee2021/c4nav-data) | 数据与论文 | CSV 数据、模型权重、图表、论文 PDF |
| [c4nav-demo](https://zebedee2021.github.io/c4nav-demo/) | 可视化 | 交互式 ECharts 演示 |

## 文档结构

### :material-compass: [平台](platform/index.md)
项目定位、c4nav-core 架构设计、与 C4 官方 SpaceR-USV 平台的关系。

### :material-waves: [仿真环境](environment/index.md)
仿真环境设计：竞赛规则到 MDP 映射、动力学模型、LiDAR 感知、奖励函数。

### :material-brain: [算法](algorithms/index.md)
已实现算法（DQN 系列）、可扩展性（PPO）、模型结构与推理。

### :material-flask: [研究方向](research/index.md)
研究方向：物理信息强化学习、分层仿真迁移框架、四阶段路线图、具身空间理论。

### :material-rocket-launch: [使用指南](guides/index.md)
快速开始、GPU 配置、YAML 参数配置参考。

### :material-bookshelf: [参考资料](references/index.md)
相关论文、术语表。

## 核心结果

| 算法 | 成功率 | 平均奖励 | 平均步数 | 收敛轮次 |
|------|:------:|:-------:|:-------:|:-------:|
| DQN (基线) | 20.0% | -57.4 | 275 | - |
| Double DQN | 4.0% | -101.4 | 115 | - |
| D3QN | 2.0% | -101.2 | 163 | - |
| **ID3QN (本文)** | **83.0%** | **84.0** | **86** | **Ep 294** |

以上结果基于 c4nav-core 仿真环境（2000 轮训练，相同随机种子），评估同一算法框架下各组件的增量贡献。跨平台的 sim-to-real 对比属于 [第三阶段研究](research/roadmap.md)。

## 仿真平台横向对比

| 维度 | c4nav-core | SpaceR-USV (C4 官方) | highway-env | MiniGrid |
|------|-----------|---------------------|-------------|----------|
| **领域** | USV 导航避障 | USV 导航避障 | 高速公路驾驶 | 网格世界导航 |
| **物理模型** | 一阶惯性（简化） | Fossen 3-DOF + 风/浪/流 | 运动学（无动力学） | 无（离散网格） |
| **传感器** | 360° LiDAR (180 射线) | 相机/LiDAR/IMU/GPS/深度 | 车道级观测 | 局部网格视野 |
| **观测维度** | 184D 连续 | 多模态（可配置） | ~25D 连续 | 7x7x3 图像 |
| **动作空间** | 离散 (5) | 连续/离散（ROS 接口） | 离散 (5) | 离散 (7) |
| **算法接口** | Gymnasium | ROS（算法无关） | Gymnasium | Gymnasium |
| **扰动仿真** | 无 | 风、浪、流 | 无 | 无 |
| **训练速度** | ~2000 步/秒 | ~50 步/秒（跨进程通信） | ~5000 步/秒 | ~10000 步/秒 |
| **部署依赖** | Python (单进程) | Unity + MATLAB + Docker | Python (单进程) | Python (单进程) |
| **实船对接** | 需 ROS 封装 | 原生 ROS | 不适用 | 不适用 |
| **开源** | 是 | 部分开源 | 是 | 是 |

**c4nav-core 的定位**：在通用轻量环境（highway-env、MiniGrid）和工程级仿真平台（SpaceR-USV）之间，提供一个兼顾 USV 领域特异性与训练效率的中间层。通用环境缺乏水面导航的物理特性，工程平台部署门槛高、迭代慢；c4nav-core 在两者之间取得平衡。

## 技术栈

| 层级 | 技术 |
|------|------|
| 算法 | Python 3.10+, PyTorch 2.x, NumPy |
| 仿真环境 | 自定义 Gymnasium 风格（USV 动力学、LiDAR、奖励塑形） |
| 训练 | GitHub Actions CI/CD，本地 GPU（RTX 4060） |
| 文档 | MkDocs Material, GitHub Pages |
| 可视化 | ECharts（交互式）、Matplotlib（论文图表） |
| 论文 | LaTeX（IEEE Transactions 格式） |
