<div align="center">

<img src="https://img.shields.io/badge/C4Nav-USV%20Autonomous%20Navigation-0066cc?style=for-the-badge" alt="C4Nav" />

# C4Nav Project

**全国海洋航行器设计与制作大赛 C4 智能导航赛道 -- 无人艇自主导航解决方案**

**National Marine Vehicle Design & Construction Competition C4 Track -- USV Autonomous Navigation**

Improved Dueling Double DQN (ID3QN) with Prioritized Experience Replay and Reward Shaping

<br/>

<a href="https://Zebedee2021.github.io/c4nav-site/"><img src="https://img.shields.io/badge/Project%20Homepage-GitHub%20Pages-1890ff?style=flat-square" /></a>&nbsp;
<a href="https://Zebedee2021.github.io/c4nav-demo/"><img src="https://img.shields.io/badge/Interactive%20Demo-ECharts-1890ff?style=flat-square" /></a>&nbsp;
<a href="https://github.com/Zebedee2021/c4nav-site/wiki"><img src="https://img.shields.io/badge/Wiki-6%20Pages-1890ff?style=flat-square" /></a>&nbsp;
<img src="https://img.shields.io/badge/ID3QN-83%25%20SR-e74c3c?style=flat-square" />&nbsp;
<img src="https://img.shields.io/badge/License-MIT-green?style=flat-square" />

</div>

---

## About / 关于本项目

C4Nav 是面向 C4 智能导航赛道的完整解决方案，涵盖**算法开发、云端训练、数据分析、论文写作**的全流程闭环。项目采用 4 仓库架构，职责清晰分离，适合教学、科研和竞赛备战。

C4Nav is a complete solution for the C4 Intelligent Navigation track, covering the full loop of **algorithm development, cloud training, data analysis, and paper writing**. The project uses a 4-repository architecture with clear separation of concerns.

### Competition Background / 赛事背景

**全国海洋航行器设计与制作大赛**是我国船舶与海洋工程领域内层次最高、规模最大的国家级竞赛。C4 智能导航赛道要求：

- 基于组委会提供的 Unity 仿真平台，采用**深度强化学习**算法训练无人艇
- 使无人艇能够灵活**躲避障碍物**并到达指定**目标点**
- 评分标准：成功率、完成用时、路径长度（客观）+ 算法新颖性、报告质量、答辩（主观）

本项目构建了等效的轻量级 Python/Gym 环境，算法设计和训练方法可直接迁移至官方平台。

---

## 4-Repository Architecture / 四仓库架构

```
                            ┌──────────────────────────────────┐
                            │        c4nav-site (Public)       │
                            │      项目主页 + Wiki (6 pages)    │
                            │      Homepage + Wiki             │
                            │    ★ You are here / 你在这里 ★    │
                            └──────────┬───────────────────────┘
                                       │
                    ┌──────────────────┼──────────────────┐
                    │                  │                  │
          ┌─────────▼──────┐  ┌───────▼────────┐  ┌─────▼──────────┐
          │  c4nav-core    │  │  c4nav-data    │  │  c4nav-demo    │
          │  (Private)     │  │  (Public)      │  │  (Public)      │
          │                │  │                │  │                │
          │  Algorithms    │  │  Training data │  │  Interactive   │
          │  Environment   │──│  Paper PDF     │──│  ECharts viz   │
          │  CI Workflows  │  │  Figures (8)   │  │  Bilingual     │
          │  LaTeX paper   │  │  Demo results  │  │                │
          └────────────────┘  └────────────────┘  └────────────────┘
               Private            Public               Public
```

| Repo | Visibility | Role / 职责 | Key Contents / 核心内容 |
|------|:----------:|-------------|------------------------|
| **[c4nav-site](https://Zebedee2021.github.io/c4nav-site/)** | Public | Project hub / 项目门户 | Homepage, Wiki (6 pages), learning path |
| **[c4nav-core](https://github.com/Zebedee2021/c4nav-core)** | Private | Algorithm & training / 算法与训练 | DQN agents, Gym env, CI workflows, LaTeX paper |
| **[c4nav-data](https://github.com/Zebedee2021/c4nav-data)** | Public | Data & paper / 数据与论文 | CSVs, model weights, 8 figures, paper PDF |
| **[c4nav-demo](https://Zebedee2021.github.io/c4nav-demo/)** | Public | Visualization / 可视化 | Interactive ECharts, bilingual CN/EN |

### Why 4 Repos? / 为什么要分 4 个仓库？

| Concern | Solution |
|---------|----------|
| Algorithm code is proprietary / 算法代码需要保密 | c4nav-core is **private** |
| Training data should be public & downloadable / 训练数据应公开可下载 | c4nav-data is **public** |
| Demo needs fast hosting / 演示需要快速托管 | c4nav-demo on GitHub Pages |
| Need a discoverable entry point / 需要一个可发现的入口 | c4nav-site as the hub |
| CI auto-pushes results across repos / CI 自动跨仓库推送结果 | GitHub Actions + SSH deploy key |

---

## Algorithm: ID3QN / 算法：ID3QN

The project implements **4 progressive DQN variants**, each building on the previous:

项目实现了 **4 种递进式 DQN 变体**，层层递进：

```
DQN (Baseline)  ──→  DDQN (+Double)  ──→  D3QN (+Dueling)  ──→  ID3QN (+PER +Shaping +SoftUpdate +LR)
  SR: 20%              SR: 4%               SR: 2%                SR: 83%
```

| Feature | DQN | DDQN | D3QN | **ID3QN** |
|---------|:---:|:----:|:----:|:---------:|
| Network | Q-Net 114K | Q-Net 114K | Dueling 135K | Dueling 135K |
| Double DQN | - | ✓ | ✓ | ✓ |
| PER (SumTree) | - | - | - | ✓ |
| Reward Shaping | - | - | - | ✓ (3 components) |
| Soft Target Update | - | - | - | ✓ (τ=0.005) |
| LR Schedule | Fixed 1e-3 | Fixed 1e-3 | Fixed 1e-3 | StepLR 5e-4 |

| Metric | DQN | DDQN | D3QN | **ID3QN** |
|--------|:---:|:----:|:----:|:---------:|
| Success Rate | 20.0% | 4.0% | 2.0% | **83.0%** |
| Avg Reward | -57.4 | -101.4 | -101.2 | **84.0** |
| Avg Steps | 275 | 115 | 163 | **86** |

---

## Simulation Environment / 仿真环境

| Parameter | Value |
|-----------|-------|
| Water area / 水域 | 20m x 20m |
| Obstacles / 障碍物 | 3-5 random static circles (r=0.8m) |
| Perception / 感知 | 180-ray LiDAR, 360° FOV, 18m range |
| State space / 状态空间 | 184-dim (180 LiDAR + 4 navigation) |
| Action space / 动作空间 | 5 discrete (hard left / soft left / straight / soft right / hard right) |
| USV dynamics / 动力学 | First-order inertia, max speed 1.8 m/s |
| Reward / 奖励 | +100 reach, -100 collision, -0.1/step + optional shaping |

---

## Cloud Training Pipeline / 云端训练流水线

```
Developer / 开发者                    GitHub Cloud / 云端                 Public / 公开
┌──────────────┐     git push     ┌────────────────────┐   auto-push   ┌──────────────┐
│  c4nav-core  │ ───────────────▶ │  GitHub Actions    │ ────────────▶ │  c4nav-data  │
│  (edit code  │                  │  Ubuntu VM (free)  │               │  (CSV, PNG,  │
│   & config)  │  manual trigger  │  3 workflows:      │               │   .pth)      │
└──────────────┘ ───────────────▶ │  • Smoke Test      │               └──────────────┘
                                  │  • Train Compare   │
                                  │  • Train Ablation  │
                                  └────────────────────┘
```

| Workflow | Trigger | Duration | Purpose |
|----------|---------|:--------:|---------|
| **Smoke Test** | Auto on push | ~3 min | 10-ep validation, catch errors |
| **Train Compare** | Manual | ~90 min | 4-algorithm comparison (2000 ep) |
| **Train Ablation** | Manual | ~60 min | 6-variant ablation study |

**Config-driven separation**: Algorithm developers only edit `config/*.yaml` and `agents/`. Training loop, logging, visualization, and CI automation remain stable. See [Training Guide (Wiki)](https://github.com/Zebedee2021/c4nav-site/wiki/Training-Guide) for details.

---

## Demo Closed Loop / 演示闭环

The Wiki provides a **4-step learning path** from algorithm theory to published paper:

Wiki 提供从算法原理到发表论文的 **4 步学习路径**：

```
① Algorithm Deep Dive     ② Trigger Training       ③ Understand Results    ④ Write Paper
   算法架构详解               触发云端训练               理解训练结果            撰写论文

┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐
│ DQN→DDQN→D3QN  │   │ GitHub Actions  │   │ Charts, CSV,    │   │ Data → Figures  │
│ →ID3QN with     │──▶│ step-by-step    │──▶│ model weights   │──▶│ → Tables →      │
│ code mapping    │   │ visual guide    │   │ interpretation  │   │ Paper sections  │
└─────────────────┘   └─────────────────┘   └─────────────────┘   └─────────────────┘
```

### Wiki Pages (6 pages, bilingual CN/EN)

| Page | Content |
|------|---------|
| [**Home**](https://github.com/Zebedee2021/c4nav-site/wiki) | 4-step closed-loop navigation + all page index |
| [**Algorithm Deep Dive**](https://github.com/Zebedee2021/c4nav-site/wiki/Algorithm-Deep-Dive) | DQN→ID3QN progressive design with code mapping (~550 lines) |
| [**How to Trigger Training**](https://github.com/Zebedee2021/c4nav-site/wiki/How-to-Trigger-Training) | Visual step-by-step guide + GitHub Actions explanation |
| [**Training Guide**](https://github.com/Zebedee2021/c4nav-site/wiki/Training-Guide) | Pipeline reference + algorithm/framework separation |
| [**Understanding Results**](https://github.com/Zebedee2021/c4nav-site/wiki/Understanding-Results) | Chart & CSV interpretation with real demo data |
| [**Paper Writing Guide**](https://github.com/Zebedee2021/c4nav-site/wiki/Paper-Writing-Guide) | Data → figures → tables → paper section mapping |

---

## Quick Links / 快速链接

| Resource | Link | Description |
|----------|------|-------------|
| Project Homepage / 项目主页 | [c4nav-site](https://Zebedee2021.github.io/c4nav-site/) | Bilingual homepage with competition background |
| Interactive Demo / 在线演示 | [c4nav-demo](https://Zebedee2021.github.io/c4nav-demo/) | ECharts visualization of all 8 paper figures |
| Core Code / 核心代码 | [c4nav-core](https://github.com/Zebedee2021/c4nav-core) | Algorithms, env, training scripts, CI workflows |
| Data & Paper / 数据与论文 | [c4nav-data](https://github.com/Zebedee2021/c4nav-data) | CSVs, model weights, figures, paper PDF |
| Paper PDF / 论文 PDF | [ID3QN_USV_Navigation.pdf](https://github.com/Zebedee2021/c4nav-data/blob/main/paper/ID3QN_USV_Navigation.pdf) | Full IEEE format paper |
| Demo Results / 教学结果 | [demo-ep500-seed42](https://github.com/Zebedee2021/c4nav-data/tree/main/demo-ep500-seed42) | Downloadable 500-ep demo for learning |
| Wiki / 知识库 | [Wiki Home](https://github.com/Zebedee2021/c4nav-site/wiki) | 6-page bilingual learning guide |

---

## Tech Stack / 技术栈

| Layer | Technology |
|-------|-----------|
| Algorithm | Python 3.10, PyTorch 2.x, NumPy |
| Environment | Custom Gym-style (USV dynamics, LiDAR, reward shaping) |
| Training | GitHub Actions CI/CD (Ubuntu VM, free tier) |
| Visualization | ECharts (interactive), Matplotlib (paper figures) |
| Website | HTML/CSS/JS, GitHub Pages, bilingual i18n |
| Paper | LaTeX (IEEE Transactions format) |
| Data | CSV training logs, .pth model weights, 300dpi PNG |

---

## Citation

```bibtex
@article{author2026id3qn,
  title={Improved Dueling Double DQN with Prioritized Experience Replay
         and Reward Shaping for USV Autonomous Navigation},
  author={Author},
  journal={TBD},
  year={2026}
}
```

## License

MIT License
