# DQN 系列：从 DQN 到 ID3QN

**逐层递进讲解 + 代码对照**

!!! info "阅读说明"
    本页解释每项改进**为什么**存在、**改了什么**、**在代码哪里**实现。无需 RL 基础。

## 算法来源

| 算法 | 来源 | 提出时间 | 原始论文 |
|------|------|:-------:|---------|
| **DQN** | DeepMind | 2015 | Mnih et al., "Human-level control through deep RL", *Nature* |
| **DDQN** | DeepMind | 2016 | van Hasselt et al., "Deep RL with Double Q-learning", *AAAI* |
| **D3QN** | 学术界组合 | ~2016 | Dueling (Wang et al., *ICML* 2016) + Double 的组合，无单一论文 |
| **ID3QN** | **本文提出** | - | D3QN + PER + 奖励塑形 + 软更新 + Huber + 学习率调度 |

其中 DQN、DDQN、D3QN 是已发表的成熟算法。ID3QN 是本文提出的**工程组合方案**，其中每一项单独技术都是已有的：

| 技术 | 原始来源 |
|------|---------|
| PER（优先经验回放） | Schaul et al., 2016 (DeepMind) |
| 奖励塑形 | Ng et al., 1999（经典方法） |
| 软更新 (τ) | Lillicrap et al., 2016（DDPG 论文） |
| Huber 损失 | Huber, 1964（统计学） |
| 学习率调度 | 深度学习标准实践 |

!!! note "本文贡献的定位"
    ID3QN 不是算法创新，而是**针对 USV 导航场景的工程组合创新**。本文的贡献是：在 USV 导航这个具体场景下，系统性地验证了哪些组件有用、哪些没用、组合效果如何。

---

## 递进路线图

不是 4 个独立算法，而是**同一个算法框架的 4 个配置等级**：

```
  Level 1              Level 2             Level 3             Level 4
  +-----------+   +-------------+   +-------------+   +------------------+
  |    DQN    |-->|    DDQN     |-->|    D3QN     |-->|   ID3QN (Ours)   |
  |   基线     |   |  +Double    |   |  +Dueling   |   |  +PER +Shaping   |
  |           |   |             |   |             |   |  +SoftUpdate +LR |
  +-----------+   +-------------+   +-------------+   +------------------+
       |                |                 |                    |
   Q值过高估计        消除过估计        值-优势分离          全面增强
   训练不稳定        但网络结构不变     更好的泛化          稀疏奖励下收敛
       |                |                 |                    |
   SR: 20%          SR: 4%           SR: 2%             SR: 83%
```

**为什么 DDQN/D3QN 反而比 DQN 差？** 这看起来违反直觉，后文有详细解释。

---

## Level 1: DQN 基线

### 核心思想

DQN 用神经网络逼近最优 Q 值函数 Q*(s,a)："在状态 s 下，执行动作 a 有多好？"

```
State s (184-dim)                          Q-values (5 actions)
+--------------+    +----------+    +----------+    +------------+
| 180 LiDAR    |    |  FC 256  |    |  FC 256  |    | Q(s, left) |
| dist_target  |--->|  + ReLU  |--->|  + ReLU  |--->| Q(s, fwd)  |
| angle_target |    |          |    |          |    | Q(s, right)|
| dist_obs     |    |  (47K)   |    |  (66K)   |    | ...        |
| angle_obs    |    +----------+    +----------+    +------------+
+--------------+                                     Total: ~114K params
```

### 两个关键机制

**经验回放 (Experience Replay)**：将转移 (s, a, r, s') 存入缓冲区，随机采样训练，打破样本间的时序相关性。

**目标网络 (Target Network)**：Q 网络的一个副本，更新频率更低，提供稳定的学习目标。

### 训练循环

```
训练循环:
  1. 智能体观察状态 s
  2. 选动作: a = argmax Q(s, a; θ)  (ε-greedy 探索)
  3. 执行 a，得到奖励 r 和下一状态 s'
  4. 存储 (s, a, r, s', done) 到回放缓冲区
  5. 从缓冲区随机采样一批数据
  6. 计算目标: y = r + γ · max_a' Q(s', a'; θ⁻)      <-- 目标网络 θ⁻
  7. 最小化损失: L = (Q(s, a; θ) - y)²                 <-- MSE 损失
  8. 每 50 轮: θ⁻ ← θ                                  <-- 硬拷贝
```

### DQN 的问题

第 6 步的 `max` 运算同时承担了"选动作"和"估值"两个职责，导致系统性的 **Q 值过高估计** -- 网络总觉得某些动作比实际好。

### 代码对照

| 概念 | 文件 | 位置 |
|------|------|------|
| Q-Network (114K) | `agents/network.py` | `class QNetwork` |
| ε-greedy 选动作 | `agents/dqn_agent.py` | `select_action()` |
| 经验回放缓冲区 | `agents/replay_buffer.py` | `class ReplayBuffer` |
| 硬目标更新 (50 轮) | `agents/dqn_agent.py` | `update_target_network()` |
| MSE 损失 | `agents/dqn_agent.py` | `loss_fn = nn.MSELoss` |
| 配置文件 | `config/baseline.yaml` | `use_double_dqn: false, network_type: "q"` |

---

## Level 2: Double DQN (DDQN)

### 改了什么

DDQN 通过**解耦**动作选择和价值评估来解决过估计问题：

```
DQN  目标: y = r + γ · max_a' Q(s', a'; θ⁻)
                        ^
                        同一个网络既选又评

DDQN 目标: y = r + γ · Q(s', argmax_a' Q(s', a'; θ), θ⁻)
                               ^                      ^
                               在线网络选               目标网络评
```

!!! tip "类比"
    两个独立评委。即使评委 A 高估了某个动作，评委 B 会给出更保守的评估，防止盲目乐观。

### 代码变化：仅 4 行

```python
# agents/dqn_agent.py -> learn() 方法

with torch.no_grad():
    if self.use_double_dqn:    # <-- DDQN 路径
        # 第 1 步：在线网络选出最优动作
        best_actions = self.q_network(next_states_t).argmax(dim=1)
        # 第 2 步：目标网络评估该动作的价值
        next_q_values = self.target_network(next_states_t).gather(
            1, best_actions.unsqueeze(1)).squeeze(1)
    else:                       # <-- DQN 路径
        # 同一个网络同时选和评
        next_q_values = self.target_network(next_states_t).max(dim=1)[0]
```

### Config Diff: DQN -> DDQN

```diff
  # 仅 2 个开关变化：
- use_double_dqn: false        -> use_double_dqn: true
- loss_fn: "mse"               -> loss_fn: "huber"
  # 其他完全一样：同样的网络、同样的缓冲区、同样的更新方式
```

### 代码对照

| 概念 | 文件 | 位置 |
|------|------|------|
| Double DQN 逻辑 | `agents/dqn_agent.py` | `learn()` -> `if self.use_double_dqn` |
| 开关 | `config/ddqn.yaml` | `use_double_dqn: true` |
| Huber 损失（新增） | `agents/dqn_agent.py` | `nn.SmoothL1Loss` |

---

## Level 3: Dueling D3QN

### 改了什么

标准 Q 网络直接输出 Q(s,a)。但在很多状态下，**动作选择并不重要**（如 USV 在远离障碍物的开阔水域）。Dueling 架构将 Q 分解为两个分量：

```
标准 Q-Network (DQN/DDQN):             Dueling Q-Network (D3QN/ID3QN):

State --> [FC 256] --> [FC 256] --> Q(s,a)   State --> [FC 256] --> [FC 256] --+-> [FC 128] -> V(s)
                                              (共享特征)                       |    (价值: 1 标量)
           114K params                                                        |
                                                                              +-> [FC 128] -> A(s,a)
                                                                                   (优势: 5 个值)
                                                            135K params                 |
                                                                                        v
                                                                        Q(s,a) = V(s) + A(s,a) - mean(A)
```

- **V(s)** = "这个状态本身好不好？" (1 个标量)
- **A(s,a)** = "这个动作比平均好多少？" (5 个值)

!!! tip "直觉"
    开阔水域中所有动作差不多 -> V(s) 捕获大部分价值，A(s,a) 接近零。靠近障碍物时，特定动作很关键 -> A(s,a) 区分它们。这种分离让网络更高效地学习状态价值。

### 网络代码

```python
# agents/network.py -> class DuelingQNetwork

class DuelingQNetwork(nn.Module):
    def __init__(self, state_dim=184, action_dim=5,
                 hidden_dims=[256,256], stream_hidden=128):
        # 共享特征提取器（和标准 Q 网络一样）
        self.shared = nn.Sequential(
            nn.Linear(184, 256), nn.ReLU(),
            nn.Linear(256, 256), nn.ReLU(),
        )
        # 价值流: shared -> 128 -> 1
        self.value_stream = nn.Sequential(
            nn.Linear(256, 128), nn.ReLU(), nn.Linear(128, 1)
        )
        # 优势流: shared -> 128 -> 5
        self.advantage_stream = nn.Sequential(
            nn.Linear(256, 128), nn.ReLU(), nn.Linear(128, 5)
        )

    def forward(self, x):
        features = self.shared(x)
        value = self.value_stream(features)              # (batch, 1)
        advantage = self.advantage_stream(features)      # (batch, 5)
        # 减去平均优势以保证可辨识性
        q = value + advantage - advantage.mean(dim=-1, keepdim=True)
        return q
```

### 参数量对比

| 组件 | 标准 Q-Net | Dueling Q-Net |
|------|:---------:|:------------:|
| 共享 FC1 (184->256) | 47,360 | 47,360 |
| 共享 FC2 (256->256) | 65,792 | 65,792 |
| 输出层 (256->5) | 1,285 | -- |
| 价值流 (256->128->1) | -- | 32,897 |
| 优势流 (256->128->5) | -- | 33,413 |
| **总计** | **~114K** | **~135K** |

参数量仅增加 18%，对训练速度没有显著影响。

### Config Diff: DDQN -> D3QN

```diff
  # 仅 1 个开关变化：
- network_type: "q"            -> network_type: "dueling"
  # Double DQN、Huber 损失、硬更新 -- 全部保持不变
```

### 代码对照

| 概念 | 文件 | 位置 |
|------|------|------|
| DuelingQNetwork (135K) | `agents/network.py` | `class DuelingQNetwork` |
| V + A - mean(A) 聚合 | `agents/network.py` | `DuelingQNetwork.forward()` |
| 按配置选择网络 | `agents/dqn_agent.py` | `__init__()` -> `if network_type == "dueling"` |
| 配置文件 | `config/d3qn.yaml` | `network_type: "dueling"` |

---

## Level 4: ID3QN (本文算法)

### 改了什么

ID3QN 在 D3QN 基础上添加**四项改进**，每项针对一个具体弱点：

```
D3QN 的弱点                            ID3QN 的解决方案
------------------------------         --------------------------------
1. 稀疏奖励: 智能体很难从              1. 奖励塑形: 密集信号引导早期探索
   每步 -0.1 中学到东西

2. 均匀采样: 稀有的成功经验            2. PER (SumTree): 高 TD 误差的
   和无用经验被同等对待                     转移被更频繁回放

3. 硬目标更新: θ⁻ 每 50 轮             3. 软更新 (Polyak): τ=0.005
   跳变一次，引起训练振荡                  每步平滑混合，消除跳变

4. 固定学习率: 同一个 LR               4. StepLR: 每 500 轮减半
   无法兼顾探索和精调阶段                  (5e-4 -> 2.5e-4 -> 1.25e-4 -> 6.25e-5)
```

### 改进 1：奖励塑形 (Reward Shaping)

```
无塑形:                           有塑形:
  Step 1: r = -0.1                Step 1: r = -0.1 + 0.3 (靠近) + 0.2 (朝向目标)
  Step 2: r = -0.1                Step 2: r = -0.1 + 0.1 (靠近) - 0.2 (靠近障碍物)
  Step 3: r = -0.1                Step 3: r = -0.1 + 0.4 (靠近) + 0.3 (朝向目标)
  ...                             ...
  Step 86: r = +100 (到达!)       Step 86: r = +100 (到达!)

  问题: 85 步 -0.1 几乎无          优势: 每一步都有有意义的
  学习信号                          梯度信号
```

**三个塑形分量：**

| 分量 | 公式 | 作用 | 代码位置 |
|------|------|------|---------|
| **R_prog** (进度) | Δd × 2.0 | 靠近目标给正奖励 | `envs/reward.py` line 41 |
| **R_safe** (安全) | -(1 - d/3.0) × 0.5 | 靠近障碍物给惩罚 | `envs/reward.py` line 44-46 |
| **R_head** (航向) | 0.3 × cos(θ) | 朝向目标给正奖励 | `envs/reward.py` line 49-50 |

```python
# envs/reward.py -> RewardCalculator.compute()

if self.enable_shaping:
    # R_prog: 靠近目标 = 正奖励
    progress = prev_dist_to_target - dist_to_target
    reward += progress * self.progress_scale           # Δd × 2.0

    # R_safe: 太靠近障碍物 = 惩罚
    if dist_to_nearest_obs < self.safety_distance:     # < 3.0m
        ratio = 1.0 - dist_to_nearest_obs / self.safety_distance
        reward -= self.proximity_penalty * ratio       # -(1-d/3) × 0.5

    # R_head: 朝向目标 = 奖励
    if heading_to_target is not None and self.heading_scale > 0:
        reward += self.heading_scale * math.cos(heading_to_target)  # 0.3 × cos(θ)
```

### 改进 2：优先经验回放 (PER)

标准回放均匀采样所有转移。但稀有事件（成功导航、目标附近碰撞）比普通开阔水域步骤携带更多学习信号。

PER 按 TD 误差 |δ|（"惊讶程度"）的比例采样：

```
标准回放:                         PER:

所有转移等概率采样                  高 TD 误差的转移被更频繁采样

  +-------------------------+     +-------------------------+
  | ░░░░░░░░░░░░░░░░░░░░░░ |     | ░░░░████░░░░░░████████ |
  | routine    routine      |     | routine  collision  success |
  | P = 1/N    P = 1/N      |     | P ∝ |δ|^α  (α=0.6)    |
  +-------------------------+     +-------------------------+
```

**SumTree 数据结构：**

```
                [Total = 12.0]              <-- O(1) 总和查询
               /              \
           [7.0]             [5.0]          <-- 内部求和节点
          /     \           /     \
       [3.0]  [4.0]     [2.0]  [3.0]       <-- 叶节点优先级 (|δ| + ε)^α
        t1     t2        t3     t4

  插入/更新: O(log N)  <-- 从叶节点向上传播
  采样:      O(log N)  <-- 从根节点向下遍历
  vs. 线性扫描: O(N)   <-- N=200,000 时快 10³ 倍
```

**重要性采样 (IS) 校正**：PER 改变了数据分布，引入偏差。IS 权重校正此偏差：

w_i = (1 / (N × P(i)))^β，其中 β 从 0.4 退火到 1.0。

```python
# agents/dqn_agent.py -> learn() 方法

# IS 权重参与损失计算，校正 PER 引入的采样偏差
loss = (is_weights_t * element_loss).mean()

# 训练后用新的 TD 误差更新优先级
self.replay_buffer.update_priorities(indices, td_errors)
```

| 代码文件 | 内容 |
|---------|------|
| `agents/per_buffer.py` | `class SumTree` -- O(log N) 操作的二叉树 |
| `agents/per_buffer.py` | `class PrioritizedReplayBuffer` -- 比例采样 + IS 权重 |
| `agents/dqn_agent.py` | `learn()` -> `is_weights_t * element_loss` -- 加权损失 |
| `agents/dqn_agent.py` | `learn()` -> `update_priorities(indices, td_errors)` -- 更新优先级 |

### 改进 3：软目标网络更新

DQN/DDQN/D3QN 每 50 轮**硬拷贝**整个网络到目标网络，导致目标 Q 值突变：

```
硬更新 (DQN/DDQN/D3QN):              软更新 (ID3QN):

θ⁻  --------+  --------+  -----     θ⁻  -------------------------
             |          |                     (平滑混合)
             +----------+------     θ⁻ <- 0.005·θ + 0.995·θ⁻
             ^          ^                ^  每次 learn 都微调
          ep 50      ep 100              小幅推动，无跳变
          大跳变      大跳变
```

```python
# agents/dqn_agent.py -> _soft_update()

def _soft_update(self):
    """Polyak averaging: θ⁻ = τ·θ + (1-τ)·θ⁻"""
    for target_param, param in zip(
        self.target_network.parameters(), self.q_network.parameters()
    ):
        target_param.data.copy_(
            self.tau * param.data + (1 - self.tau) * target_param.data
        )   # τ = 0.005: 每步仅混入 0.5% 的在线网络
```

### 改进 4：学习率调度

固定学习率无法兼顾快速收敛（需大 LR）和精细调优（需小 LR）。

```
Episode:    0        500       1000      1500      2000
LR:        5e-4 --> 2.5e-4 --> 1.25e-4 --> 6.25e-5
阶段:      探索      收敛       精调       打磨
```

```python
# agents/dqn_agent.py -> __init__()

if lr_schedule == "step":
    self.scheduler = StepLR(self.optimizer, step_size=500, gamma=0.5)
    # 每 500 轮: lr *= 0.5
```

### Config Diff: D3QN -> ID3QN (全部变化)

```diff
  training:
    network_type: "dueling"             # 不变
    use_double_dqn: true                # 不变
-   use_per: false                    -> use_per: true
-   use_soft_update: false            -> use_soft_update: true
+   tau: 0.005                          # 新增: 软更新系数
    loss_fn: "huber"                    # 不变
-   lr: 0.001                         -> lr: 0.0005
-   lr_schedule: "none"               -> lr_schedule: "step"
+   lr_step_size: 500                   # 新增: 每 500 轮衰减
+   lr_gamma: 0.5                       # 新增: 减半
+   per_alpha: 0.6                      # 新增: PER 优先化程度
+   per_beta_start: 0.4                 # 新增: IS 权重退火起点
+   per_beta_end: 1.0                   # 新增: IS 权重退火终点

  reward:
-   enable_shaping: false             -> enable_shaping: true
+   progress_scale: 2.0                 # 新增: R_prog 权重
+   safety_distance: 3.0                # 新增: R_safe 距离阈值
+   proximity_penalty: 0.5              # 新增: R_safe 惩罚系数
+   heading_scale: 0.3                  # 新增: R_head 权重
```

---

## 为什么 DDQN/D3QN 反而更差

这是最违反直觉的结果。DDQN/D3QN 理论上优于 DQN，但成功率 (4%, 2%) 反而低于 DQN (20%)。

**根因：极端稀疏奖励**

在 C4 环境中，USV 只有到达目标 (+100) 或碰撞 (-100) 时才有明确信号，每步仅扣 0.1。没有奖励塑形时：

- **DQN** 过高估计 Q 值 -> 意外地探索更激进 -> 碰巧到达目标 20%
- **DDQN** 纠正了过估计 -> 更保守 -> 探索不足 -> 成功率仅 4%
- **D3QN** 更精确的 Q 估计 -> 更保守 -> 更难找到目标 -> 仅 2%

!!! tip "类比理解"
    DQN 像一个盲目乐观的新手，什么都敢试，偶尔误打误撞成功。DDQN/D3QN 是"纠正了乐观"的理性人，在没有路标（密集奖励）的情况下反而更谨小慎微，不敢探索。

**ID3QN 的突破**：奖励塑形提供了密集引导信号（R_prog: "越来越近是好事", R_safe: "太靠近障碍物是坏事", R_head: "朝向目标是好事"），把稀疏奖励问题转化为可学习的信号。

---

## 完整对照表

### 特性矩阵

| 特性 | DQN | DDQN | D3QN | ID3QN |
|------|:---:|:----:|:----:|:-----:|
| 网络 | Q-Net (114K) | Q-Net (114K) | Dueling (135K) | Dueling (135K) |
| Double DQN | - | Y | Y | Y |
| PER (SumTree) | - | - | - | Y |
| 奖励塑形 | - | - | - | Y (3 分量) |
| 损失函数 | MSE | Huber | Huber | Huber |
| 目标更新 | 硬拷贝/50轮 | 硬拷贝/50轮 | 硬拷贝/50轮 | 软更新 τ=0.005 |
| 学习率 | 固定 1e-3 | 固定 1e-3 | 固定 1e-3 | StepLR 5e-4 |
| 梯度裁剪 | 5.0 | 5.0 | 5.0 | 5.0 |

### 完整性能

| 指标 | DQN | DDQN | D3QN | **ID3QN** |
|------|:---:|:----:|:----:|:---------:|
| 成功率 | 20.0% | 4.0% | 2.0% | **83.0%** |
| 平均奖励 | -57.4 | -101.4 | -101.2 | **84.0** |
| 平均步数 | 275 | 115 | 163 | **86** |
| 收敛轮次 | N/A | N/A | N/A | **Ep 294** |
| 碰撞率 | 33.2% | 49.7% | 50.1% | **19.3%** |
| 出界率 | 37.2% | 43.9% | 40.9% | **8.3%** |

### 结果解读

DDQN 和 D3QN 比 DQN 基线**还差**（4% 和 2% vs 20%），这不是 bug，而是一个重要发现：

- 在 USV 导航这种**稀疏奖励**问题中，单纯改进网络结构或降低过估计**不够**
- 真正的突破来自 ID3QN 的**工程组合拳**：PER 让智能体优先复习失败经验，奖励塑形提供中间引导信号，两者缺一不可
- 消融实验也证实了这一点：去掉奖励塑形，成功率从 83% 直接跌到 2%（详见[奖励函数](../environment/reward.md)页面的消融结果）

这个实证结论对 USV 领域有参考价值 -- 在稀疏奖励的导航任务中，**奖励工程比网络架构更重要**。

---

## 消融实验：哪项改进最重要

消融实验每次禁用一项改进（500 轮），观察成功率变化：

```
影响力排序:

  奖励塑形  ████████████████████████████████████ 最关键
  软更新    ████████████████████
  PER      ████████████████
  学习率调度 ██████████████
  航向奖励  ██████████              最小影响
```

!!! warning "关键洞察"
    没有奖励塑形，ID3QN 降至 12% 成功率（比 DQN 基线还差）。塑形是**基石**，使所有其他改进得以发挥作用。在稀疏奖励的导航任务中，提供密集引导信号是最重要的设计选择。

---

## 算法开关

4 种算法在**同一份代码**（1 个 `DQNAgent` 类）中实现，通过 YAML 配置开关组合切换：

| 开关 | DQN | DDQN | D3QN | ID3QN |
|------|:---:|:----:|:----:|:-----:|
| `use_double_dqn` | - | Y | Y | Y |
| `network_type: dueling` | - | - | Y | Y |
| `use_per` | - | - | - | Y |
| `enable_shaping` | - | - | - | Y |
| `use_soft_update` | - | - | - | Y |
| `lr_schedule: step` | - | - | - | Y |
| `loss_fn: huber` | - | Y | Y | Y |

---

## 代码文件速查

| 文件 | 实现内容 |
|------|---------|
| `agents/network.py` | `QNetwork` (DQN/DDQN) + `DuelingQNetwork` (D3QN/ID3QN) |
| `agents/dqn_agent.py` | 统一智能体：ε-greedy, learn(), 软/硬更新, 全部开关 |
| `agents/replay_buffer.py` | 标准均匀回放 (DQN/DDQN/D3QN) |
| `agents/per_buffer.py` | SumTree + 优先回放 (ID3QN) |
| `envs/reward.py` | 稀疏奖励 + 可选塑形 (R_prog, R_safe, R_head) |
| `config/baseline.yaml` | DQN：全部开关关闭 |
| `config/ddqn.yaml` | +Double DQN, +Huber |
| `config/d3qn.yaml` | +Dueling 网络 |
| `config/improved_d3qn.yaml` | +PER, +塑形, +软更新, +学习率调度 |
