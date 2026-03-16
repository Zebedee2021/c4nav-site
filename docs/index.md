# C4Nav - USV Autonomous Navigation

**全国海洋航行器设计与制作大赛 C4 智能导航赛道 -- 无人艇自主导航研究平台**

---

## Project Overview

C4Nav 是一个面向无人艇 (USV) 自主导航研究的开源项目，围绕 C4 智能导航赛道构建了从算法研究到实船部署的完整技术栈。

核心组件 **c4nav-core** 是一个基于 Gymnasium 接口标准构建的轻量级仿真环境，集成了简化 USV 动力学模型、360° LiDAR 感知仿真和竞赛规则对齐的奖励体系，在保持训练效率的同时提供了足够的领域特异性，支持多种 DRL 算法的快速对比验证。

## Repository Architecture

```
                        c4nav-site
                    Documentation Hub
                    (MkDocs + GitHub Pages)
                           |
            +--------------+--------------+
            |              |              |
        c4nav-core     c4nav-data     c4nav-demo
        (Private)      (Public)       (Public)
        Algorithms     Training data  Interactive
        Environment    Paper PDF      ECharts viz
        CI Workflows   Figures        Bilingual
```

| Repo | Role | Key Contents |
|------|------|-------------|
| [c4nav-site](https://zebedee2021.github.io/c4nav-site/) | Documentation Hub | MkDocs site, research notes |
| [c4nav-core](https://github.com/Zebedee2021/c4nav-core) | Algorithm & Training | DQN agents, Gym env, training scripts |
| [c4nav-data](https://github.com/Zebedee2021/c4nav-data) | Data & Paper | CSVs, model weights, figures, paper PDF |
| [c4nav-demo](https://zebedee2021.github.io/c4nav-demo/) | Visualization | Interactive ECharts demo |

## Documentation Structure

### :material-compass: [Platform](platform/index.md)
Project positioning, c4nav-core architecture, relationship with C4 official SpaceR-USV platform.

### :material-waves: [Environment](environment/index.md)
Simulation environment design: competition rules to MDP mapping, dynamics model, LiDAR sensing, reward function.

### :material-brain: [Algorithms](algorithms/index.md)
Implemented algorithms (DQN family), extensibility (PPO), model structure and inference.

### :material-flask: [Research](research/index.md)
Research directions: Physics-Informed RL, hierarchical sim transfer framework, 4-phase roadmap, Embodied Space theory.

### :material-rocket-launch: [Guides](guides/index.md)
Quick start, GPU setup, YAML configuration reference.

### :material-bookshelf: [References](references/index.md)
Related papers, technical glossary.

## Key Results

| Algorithm | Success Rate | Avg Reward | Avg Steps | Convergence |
|-----------|:-----------:|:----------:|:---------:|:-----------:|
| DQN (Baseline) | 20.0% | -57.4 | 275 | - |
| Double DQN | 4.0% | -101.4 | 115 | - |
| D3QN | 2.0% | -101.2 | 163 | - |
| **ID3QN (Ours)** | **83.0%** | **84.0** | **86** | **Ep 294** |

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Algorithm | Python 3.10+, PyTorch 2.x, NumPy |
| Environment | Custom Gymnasium-style (USV dynamics, LiDAR, reward shaping) |
| Training | GitHub Actions CI/CD, local GPU (RTX 4060) |
| Documentation | MkDocs Material, GitHub Pages |
| Visualization | ECharts (interactive), Matplotlib (paper figures) |
| Paper | LaTeX (IEEE Transactions format) |
