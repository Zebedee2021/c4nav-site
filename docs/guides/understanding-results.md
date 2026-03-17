# 训练结果解读

**如何解读和使用训练输出**

!!! info "本页以真实 Demo 为例"
    以一次云端训练结果 (`ep500-seed42`) 为例，逐一讲解每个输出文件和图表。

---

## 你会拿到什么

Train Compare 工作流完成后，下载 ZIP 解压，目录结构如下：

```
compare-results-ep500-seed42/           <-- 本次 demo 的完整结果
|
+-- comparison_curves.png               <-- 图 A: 训练曲线 (4 种算法叠加)
+-- comparison_bars.png                 <-- 图 B: 柱状对比图 (最终性能)
+-- comparison_trajectories.png         <-- 图 C: 轨迹对比图 (同一场景)
|
+-- baseline_seed42/                    <-- DQN 基线
|   +-- training_log.csv                <-- 500 行，每行 = 1 回合
|   +-- models/
|       +-- best.pth                    <-- 训练过程中奖励最高的模型
|       +-- ep100.pth ~ ep500.pth       <-- 每 100 回合的检查点
|       +-- final.pth                   <-- 最后一回合的模型
+-- ddqn_seed42/                        <-- Double DQN
|   +-- ...
+-- d3qn_seed42/                        <-- Dueling Double DQN
|   +-- ...
+-- improved_seed42/                    <-- ID3QN (本文算法)
    +-- ...
```

---

## 图 A：训练曲线

![训练曲线](https://raw.githubusercontent.com/Zebedee2021/c4nav-data/main/demo-ep500-seed42/comparison_curves.png)

这张图有 **3 个子图**，展示 4 种算法在 500 回合中的学习过程。

### 颜色对照

| 颜色 | 算法 | 说明 |
|:---:|---|---|
| **灰色** | DQN (Baseline) | 最简单，无改进 |
| **蓝色** | Double DQN | + 双 Q 估计 |
| **绿色** | D3QN | + 对决网络 |
| **红色** | ID3QN (本文) | + PER + 奖励塑形 + 软更新 + 学习率调度 |

### 子图 1：回合奖励

| 看什么 | 本次结果 |
|--------|---------|
| **红线上升** = ID3QN 在学习 | 红线从 -80 上升到 +25 附近 |
| **灰/蓝/绿平坦** = 未学到有效策略 | 三条线在 -70~-110 徘徊 |
| **目标线 (>= 80)** = 论文目标 | 500 回合尚未达标，需 2000 回合 |

**解读**：奖励 > 0 表示无人艇持续成功到达目标。负值 = 碰撞/出界/超时。

### 子图 2：成功率

| 看什么 | 本次结果 |
|--------|---------|
| **红线攀升** | ID3QN: 0% -> 峰值 ~70%, 稳定在 ~50-60% |
| **其他接近 0%** | Baseline ~10%, DDQN/D3QN < 10% |
| **目标线 (80%)** | 尚未达到，需更多回合 |

**解读**：这是最重要的指标。ID3QN 在第 200 回合左右明显超越其他算法。

### 子图 3：回合步数

| 模式 | 含义 |
|------|------|
| 步数短 + 成功 | 高效导航 |
| 步数短 + 失败 | 快速碰撞（不好） |
| 步数长 + 失败 | 无目的漫游 |

**本次结果**：红线稳定在 80-100 步左右，其他算法波动较大，说明 ID3QN 找到了稳定高效的路径。

---

## 图 B：性能柱状图

![柱状对比图](https://raw.githubusercontent.com/Zebedee2021/c4nav-data/main/demo-ep500-seed42/comparison_bars.png)

这张图对比了 4 种算法**测试阶段**（训练后跑 20 个测试回合）的表现。

### 4 个指标详解

| 指标 | 越高越好？ | DQN | DDQN | D3QN | **ID3QN** | 解释 |
|---|:---:|:---:|:---:|:---:|:---:|---|
| **成功率 (%)** | 是 | 10.0 | 5.0 | 25.0 | **65.0** | 到达目标的回合占比 |
| **平均奖励** | 是 | -87.9 | -97.0 | -58.0 | **49.1** | 每回合平均总奖励 |
| **平均步数** | 视情况 | 79.8 | 70.6 | 80.6 | **95.2** | 步数高 = 完整导航而非碰撞 |
| **路径效率** | 是 | 9.3 | 8.3 | 16.0 | **10.9** | 路径质量指标 |

### 解读要点

- **成功率**是最决定性的指标。ID3QN 的 65%（红色柱）远超其他
- **平均奖励**：只有 ID3QN 获得**正奖励** (49.1)，其他都是深度负值
- **平均步数**：ID3QN 步数最多 (95.2) 是因为活得久能完成导航，而非缺点
- **D3QN 路径效率 (16.0)**：看起来高但有误导性 -- 它只成功了 25%，样本量很小

---

## 图 C：轨迹对比图

![轨迹对比图](https://raw.githubusercontent.com/Zebedee2021/c4nav-data/main/demo-ep500-seed42/comparison_trajectories.png)

这是 2x2 网格图，展示 4 种算法在**完全相同的场景**下的导航结果。

### 场景元素

| 元素 | 符号 | 含义 |
|------|:----:|------|
| 绿点 | ● | 出发点 |
| 黄星 | ☆ | 目标点 |
| 粉色圆 | ◯ | 障碍物 |
| 彩色线 | — | 无人艇轨迹 |

### 解读每个面板

| 面板 | 算法 | 结果 | 奖励 | 分析 |
|:---:|---|---|:---:|---|
| 左上 | **DQN (基线)** | 出界 (OOB) | R=-114 | 直接飘出地图边界，未学会避障 |
| 右上 | **Double DQN** | 碰撞 | R=-106 | 朝目标前进但撞上障碍物 |
| 左下 | **D3QN** | 到达 | R=92 | 成功到达！但路径弯曲，不够高效 |
| 右下 | **ID3QN (本文)** | 到达 | R=128 | 平滑高效地绕过障碍物，奖励最高 |

### 关键观察

1. **DQN** 未学到有效行为 -- 漫游出界
2. **DDQN** 有目标导向行为（朝目标移动）但无法避障
3. **D3QN** 能到达目标但路径弯曲低效
4. **ID3QN** 展示了最平滑、最直接的轨迹，奖励最高 (128 vs 92)

!!! tip "这就是奖励塑形的作用"
    ID3QN 的复合奖励（距离+航向+安全）教会它**高效导航**，而不仅是到达目标。

---

## CSV 数据详解

每个算法文件夹包含 `training_log.csv`，以下是 `improved_seed42` 的前几行：

```
episode  reward   steps  success  collision  out_of_bounds  timeout  epsilon  avg_loss  path_length  dist_to_target  lr
0        143.11   142    True     False      False          False    0.998    0.0546    15.99        1.50            0.0005
1       -110.81   154    False    False      True           False    0.996    1.6229    17.31        8.57            0.0005
2        -97.24    65    False    False      True           False    0.994    1.5586     7.34        8.04            0.0005
```

### 逐列解读

| 列名 | 示例 | 含义 |
|------|------|------|
| `episode` | 0 | 回合编号（从 0 开始） |
| `reward` | 143.11 | 总奖励，**正值 = 好** |
| `steps` | 142 | 无人艇走了多少步 |
| `success` | True | 是否到达目标 |
| `collision` | False | 是否碰撞障碍物 |
| `out_of_bounds` | False | 是否出界 |
| `timeout` | False | 是否超时 |
| `epsilon` | 0.998 | 探索率，从 1.0 衰减到 ~0.01 |
| `avg_loss` | 0.0546 | 神经网络训练损失 |
| `path_length` | 15.99 | 实际行驶距离（米） |
| `dist_to_target` | 1.50 | 结束时到目标的距离 |
| `lr` | 0.0005 | 当前学习率 |

### 回合结果判定

每个回合以下列**其中一种**结果结束：

```
                              +-- success=True      --> 到达 (奖励 +100 ~ +150)
                              |
  回合结束 -------------------+-- collision=True     --> 碰撞 (惩罚 -100)
                              |
                              +-- out_of_bounds=True --> 出界 (惩罚 -100)
                              |
                              +-- timeout=True       --> 超时 (惩罚 -50)
```

### 典型奖励范围

| 结果 | 奖励范围 | 说明 |
|------|:-------:|------|
| 成功 | +50 ~ +150 | 基础奖励 + 距离奖励 + 航向奖励 |
| 碰撞 | -100 ~ -120 | 重罚 |
| 出界 | -100 ~ -120 | 重罚 |
| 超时 | -40 ~ -70 | 中等惩罚 |

---

## 500 vs 2000 回合：为什么需要更多训练

本次 Demo 仅用了 500 回合。对比 2000 回合的完整基准：

| 指标 | 500 回合 (Demo) | 2000 回合 (完整) | 提升 |
|------|:---:|:---:|---|
| ID3QN 成功率 | 65% | **83%** | +18pp，更多训练有用 |
| ID3QN 平均奖励 | 49.1 | **84.0** | 几乎翻倍 |
| ID3QN 平均步数 | 95.2 | **86** | 训练越多越高效 |

从训练曲线可以看到红线在第 500 回合时**仍在上升**，尚未收敛。2000 回合才足够让算法收敛。

---

## 使用模型权重

```python
from agents.dqn_agent import DQNAgent
from envs.usv_env import USVNavEnv
import yaml

# 加载配置
with open("config/improved_d3qn.yaml") as f:
    config = yaml.safe_load(f)

# 创建环境和智能体
env = USVNavEnv(config)
agent = DQNAgent(state_dim=env.state_dim, action_dim=env.action_dim,
                 config=config["training"])

# 加载最优模型
agent.load("improved_seed42/models/best.pth")
agent.epsilon = 0.0  # 贪婪模式，不探索

# 运行一个回合
state, info = env.reset(seed=42)
for step in range(200):
    action = agent.select_action(state, training=False)
    state, reward, terminated, truncated, info = env.step(action)
    if terminated or truncated:
        break

print(f"结果: {'成功' if info.get('reached') else '失败'}")
```

---

## 快速 Python 分析

```python
import pandas as pd

# 加载 4 种算法数据
algos = ['baseline', 'ddqn', 'd3qn', 'improved']
for alg in algos:
    df = pd.read_csv(f"{alg}_seed42/training_log.csv")
    last_100 = df.tail(100)
    sr = last_100['success'].mean() * 100
    avg_r = last_100['reward'].mean()
    print(f"{alg:>12s}: SR={sr:.0f}%  AvgR={avg_r:.1f}")
```

预期输出：

```
    baseline: SR=12%  AvgR=-85.3
        ddqn: SR=6%   AvgR=-95.2
        d3qn: SR=18%  AvgR=-72.1
    improved: SR=58%  AvgR=35.7
```
