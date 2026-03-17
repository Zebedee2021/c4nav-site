# 论文写作指南

**训练数据如何变成一篇论文**

!!! info "完成学习闭环"
    你已经了解了算法原理、触发了训练、理解了结果 -- 现在来看这些结果如何写成一篇论文。

---

## 全局视角

```
+----------------------------------------------------------------------+
|                    演示闭环 (Demo Closed Loop)                         |
|                                                                       |
|   1. 算法原理      2. 云端训练      3. 理解结果      4. 撰写论文       |
|                                                                       |
|   Section 3        GitHub Actions   Charts & CSV     Section 4        |
|   (ID3QN 设计)     (触发训练)       (图表与数据)     (实验分析)        |
|        |                |                |               |             |
|        v                v                v               v             |
|   paper/section3 -> c4nav-core -> c4nav-data/ -> paper/section4       |
+----------------------------------------------------------------------+
```

---

## 论文结构概览

论文采用 IEEE Transactions 标准格式：

| 章节 | 标题 | 核心内容 | 数据来源 |
|:---:|---|---|---|
| 1 | 引言 (Introduction) | 研究背景、动机 | 文献综述 |
| 2 | 问题建模 (Problem Formulation) | 环境、状态/动作空间 | `figures/fig1_environment.png` |
| **3** | **算法设计 (Proposed Method)** | **ID3QN 算法设计** | **`paper/section3_proposed_method.md`** |
| **4** | **实验分析 (Experiments)** | **结果与分析** | **`paper/section4_experiments.md`** |
| 5 | 结论 (Conclusion) | 总结与未来工作 | -- |

**第三章和第四章是核心章节**，其 Markdown 草稿在 [c4nav-data/paper/](https://github.com/Zebedee2021/c4nav-data/tree/main/paper) 中。

---

## 数据 -> 图表 -> 论文

### 图表生成流程

```
training_log.csv (原始数据)
        |
        v
   viz/ scripts (可视化脚本)
        |
        +-->  fig5_training_curves.png    --> Paper Fig.5
        +-->  fig6_test_performance.png   --> Paper Fig.6
        +-->  fig7_trajectories.png       --> Paper Fig.7
        +-->  fig8_ablation.png           --> Paper Fig.8
```

所有 8 张出版级图片 (300 dpi) 在 [c4nav-data/figures/](https://github.com/Zebedee2021/c4nav-data/tree/main/figures)。

### 图表与论文的完整映射

| 图 | 文件 | 论文中用途 | 数据来源 |
|---|---|---|---|
| Fig.1 | `fig1_environment.png` | Section 2: 仿真环境示意图 | 环境配置 |
| Fig.2 | `fig2_architecture.png` | Section 3.1: 系统架构图 | 架构设计 |
| Fig.3 | `fig3_network_structure.png` | Section 3.2: Dueling 网络结构图 | 网络设计 |
| Fig.4 | `fig4_reward_shaping.png` | Section 3: 奖励塑形示意 | 奖励函数 |
| **Fig.5** | **`fig5_training_curves.png`** | **Section 4.2.1: 训练曲线 (3 子图)** | **`compare/*/training_log.csv`** |
| **Fig.6** | **`fig6_test_performance.png`** | **Section 4.2.2: 测试性能柱状图** | **`compare/*/training_log.csv` (后 100 轮)** |
| **Fig.7** | **`fig7_trajectories.png`** | **Section 4.2.4: 轨迹对比 (2x2)** | **测试 rollout 数据** |
| **Fig.8** | **`fig8_ablation.png`** | **Section 4.3: 消融实验** | **`ablation/*/training_log.csv`** |

---

## 数据 -> 表格 -> 论文

### 表格与数据的映射

| 表 | 论文内容 | 计算方式 |
|---|---|---|
| **Table 10** | 四种算法测试对比 | `compare/*/training_log.csv` 后 100 轮 |
| **Table 11** | 终止原因统计 | 统计 `success/collision/out_of_bounds/timeout` 列 |
| **Table 13** | 消融实验结果 | `ablation/*/training_log.csv` 后 50 轮 |

### 示例：从 CSV 计算 Table 10

```python
import pandas as pd

algorithms = ['baseline', 'ddqn', 'd3qn', 'improved']
for alg in algorithms:
    df = pd.read_csv(f'compare/{alg}_seed0/training_log.csv')
    last100 = df.tail(100)
    sr = last100['success'].mean() * 100
    avg_r = last100['reward'].mean()
    avg_s = last100['steps'].mean()
    print(f"{alg:>12s}: SR={sr:.1f}%  AvgR={avg_r:.1f}  AvgSteps={avg_s:.0f}")
```

预期输出 (2000 轮, seed=0)：

```
    baseline: SR=20.0%  AvgR=-57.4  AvgSteps=275
        ddqn: SR=4.0%   AvgR=-101.4 AvgSteps=115
        d3qn: SR=2.0%   AvgR=-101.2 AvgSteps=163
    improved: SR=83.0%  AvgR=84.0   AvgSteps=86
```

这些数字直接填入论文的 **Table 10**。

---

## 论文写作流程

面向初学者，推荐以下写作流程：

### 步骤 1：阅读第三章草稿

下载 [section3_proposed_method.md](https://github.com/Zebedee2021/c4nav-data/blob/main/paper/section3_proposed_method.md) 学习算法设计：

- 3.1 DQN -> DDQN -> D3QN 理论基础
- 3.2 网络架构（Dueling, 参数量）
- 3.3-3.6 四项关键改进（PER, Huber 损失, 软更新, 学习率调度）
- 3.7 完整训练算法（Algorithm 1）
- 3.8 递进式对比设计（4 种算法，特性表）

### 步骤 2：运行训练

按照 [训练方式](training-methods.md) 页面选择本地、GitHub Actions 或 Colab 触发训练。

### 步骤 3：下载并分析结果

按照 [训练结果解读](understanding-results.md) 页面解读图表和 CSV 数据。

### 步骤 4：填写第四章

使用 [section4_experiments.md](https://github.com/Zebedee2021/c4nav-data/blob/main/paper/section4_experiments.md) 作为模板，将占位数字替换为你的实际结果：

- 4.2.1 训练曲线分析 -> 描述你的 Fig.5
- 4.2.2 测试性能 -> 用你的 CSV 数据填入 Table 10
- 4.2.3 终止原因统计 -> 填入 Table 11
- 4.2.4 轨迹分析 -> 描述你的 Fig.7

### 步骤 5：生成 PDF

LaTeX 源文件 (`main.tex`) 在 `c4nav-core/paper/`，编译命令：

```bash
cd c4nav-core/paper
pdflatex main.tex
bibtex main
pdflatex main.tex
pdflatex main.tex
```

---

## 初学者要点

| 要点 | 说明 |
|---|---|
| **先有数据再写文章** | 所有结论必须有可复现的数据支撑 |
| **每张图讲一个故事** | Fig.5 展示学习过程, Fig.6 展示最终性能 |
| **表格概括, 图表展示** | 两者结合做到全面呈现 |
| **可复现性是关键** | 固定种子, 公开代码, 公开数据 |
| **递进式对比** | DQN -> DDQN -> D3QN -> ID3QN 展示每项改进的价值 |
