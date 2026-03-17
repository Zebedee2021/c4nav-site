# 训练方式

c4nav-core 支持 3 种训练方式，适合不同场景：

| 方式 | 硬件要求 | 适合场景 | GPU 加速 | 上手难度 |
|------|---------|---------|:--------:|:-------:|
| **本地训练** | 本地电脑 | 日常开发调试、有 GPU 的机器 | 有 GPU 时自动启用 | 低 |
| **GitHub Actions** | 无（GitHub 提供） | 正式对比实验、论文出图 | 无 (CPU) | 低 |
| **Google Colab** | 无（Google 提供） | 需要免费 GPU、大规模训练 | 免费 T4 GPU | 中 |

---

## 方式一：本地训练

最直接的方式，代码在本地机器上运行。

### 环境准备

```bash
git clone https://github.com/Zebedee2021/c4nav-core.git
cd c4nav-core
python -m venv venv
# Windows
venv\Scripts\activate
# Linux/Mac
source venv/bin/activate
pip install -r requirements.txt
```

如果有 NVIDIA 显卡，额外安装 CUDA 版 PyTorch（详见 [GPU 配置](gpu-setup.md)）：

```bash
pip install torch torchvision --index-url https://download.pytorch.org/whl/cu124
```

### 运行训练

```bash
# 单个算法快速测试（约 8 分钟）
python scripts/train_compare.py --algorithms improved --episodes 500 --seed 0

# 全部 4 种算法对比（约 25-90 分钟，取决于硬件）
python scripts/train_compare.py --algorithms all --episodes 2000 --seed 0

# 消融实验
python scripts/run_ablation.py --episodes 500 --seed 0
```

### 耗时参考（本地）

| 回合数 | 仅 CPU (i7) | RTX 4060 GPU |
|--------|:-----------:|:------------:|
| 500 | ~12 分钟 | ~8 分钟 |
| 2000 | ~40 分钟 | ~25 分钟 |

!!! tip "什么时候用本地训练"
    - 修改代码后快速验证（500 轮 Smoke Test）
    - 有 GPU 时训练速度最快
    - 不需要联网

---

## 方式二：GitHub Actions 云端训练

训练在 GitHub 免费 Ubuntu 虚拟机上运行（2 核 CPU、7 GB 内存），无需本地 GPU。

### 架构

```
本地                          GitHub 云端                  公开资产
+-------------+   git push   +------------------+  自动推送  +-------------+
|  c4nav-core | ------------>|  GitHub Actions   |--------->|  c4nav-data |
|  (代码仓库)  |              |  Ubuntu 虚拟机     |           |  (数据仓库)  |
+-------------+              +------------------+           +-------------+
      |                              |                             |
      |  手动触发                     | Artifact (ZIP)              v
      +---------------------------->|                        c4nav-demo
                                                             c4nav-site
```

!!! tip "GitHub Actions 相当于一台免费的远程电脑"
    你把代码推上去，它帮你跑，跑完把结果还给你。代码是你的，电脑是 GitHub 借你的。

### 三个工作流

| 工作流 | 触发方式 | 用途 | 超时 |
|--------|---------|------|------|
| **Smoke Test** | 推送自动触发 | 10 轮快速验证，确认代码无误 | 15 分钟 |
| **Train Compare** | 手动触发 | 4 种算法对比训练 | 180 分钟 |
| **Train Ablation** | 手动触发 | 6 种变体消融实验 | 240 分钟 |

### 如何触发训练（7 步）

**步骤 1**：访问 `https://github.com/Zebedee2021/c4nav-core`

**步骤 2**：点击顶部导航栏的 **Actions** 选项卡

**步骤 3**：在左侧栏中点击 **Train Compare**

**步骤 4**：点击右侧蓝色横幅中的 **Run workflow** 按钮

**步骤 5**：填写参数

| 参数 | 选项 | 说明 |
|------|------|------|
| algorithms | `all` / `baseline` / `ddqn` / `d3qn` / `improved` | 训练哪些算法 |
| episodes | `500` / `1000` / `2000` | 训练回合数 |
| seed | 任意整数 | 随机种子（保证可复现） |
| upload_to_data_repo | 勾选与否 | 是否自动推送结果到 c4nav-data |

**推荐配置：**

- **首次测试**（约 8 分钟）：algorithms = `improved`，episodes = `500`
- **正式论文**（约 90 分钟）：algorithms = `all`，episodes = `2000`，upload = 勾选

**步骤 6**：监控进度

| 图标 | 含义 |
|------|------|
| 橙色圆点 | 运行中 |
| 绿色对勾 | 成功 |
| 红色叉 | 失败 |

日志字段解读：

```
[DQN (Bas) Ep 200/2000 | R: -112.1 | Avg50: -87.8 | SR: 10.0% | Eps: 0.670 | 73s
```

| 字段 | 含义 |
|------|------|
| `Ep 200/2000` | 当前/总回合 |
| `R: -112.1` | 本回合奖励 |
| `Avg50: -87.8` | 近 50 回合均值 |
| `SR: 10.0%` | 成功率 |
| `Eps: 0.670` | 探索率（从 1.0 衰减到 ~0.01） |

**步骤 7**：下载结果

训练完成后，页面底部出现 Artifact ZIP，点击下载：

```
compare-results-ep2000-seed0/
+-- comparison_curves.png       # 训练曲线
+-- comparison_bars.png         # 柱状对比图
+-- baseline_seed0/training_log.csv
+-- ddqn_seed0/training_log.csv
+-- d3qn_seed0/training_log.csv
+-- improved_seed0/
    +-- training_log.csv
    +-- models/best.pth         # 最优模型
```

### 耗时参考（GitHub Actions）

| 回合数 | 单个算法 | 全部 4 个 |
|--------|:-------:|:--------:|
| 500 | ~8 分钟 | ~30 分钟 |
| 1000 | ~15 分钟 | ~60 分钟 |
| 2000 | ~25 分钟 | ~90 分钟 |

免费计划私有仓库每月 **2,000 分钟**。一次完整对比训练约 90 分钟，额度充裕。

### 跨仓库自动推送

当步骤 5 中勾选 `upload_to_data_repo = true` 时，训练完成后结果会通过 **SSH Deploy Key** 自动推送到 c4nav-data 仓库：

```
GitHub Actions VM
  |
  +-- 训练完成，生成 CSV / 图表 / 模型
  |
  +-- 使用 SSH Deploy Key 认证
  |   (密钥存储在 c4nav-core 的 Repository Secrets 中)
  |
  +-- git push 到 c4nav-data 仓库
      |
      +-- c4nav-demo 自动更新（读取 c4nav-data 的图表）
      +-- c4nav-site 文档可引用最新数据
```

不勾选则仅生成 Artifact ZIP，不影响 c4nav-data 仓库。

!!! tip "什么时候用 GitHub Actions"
    - 正式论文出图（结果自动归档到 c4nav-data）
    - 没有本地 GPU 时的主要训练方式
    - 需要可复现的标准化训练流程

---

## 方式三：Google Colab

Google Colab 提供免费的 GPU 运行时（通常是 Tesla T4, 15 GB 显存），适合需要 GPU 加速但本地没有显卡的情况。

### 快速开始

在 Colab 中新建 Notebook，依次运行以下 Cell：

**Cell 1：克隆代码并安装依赖**

```python
!git clone https://github.com/Zebedee2021/c4nav-core.git
%cd c4nav-core
!pip install -r requirements.txt -q
```

**Cell 2：验证 GPU**

```python
import torch
print(f"CUDA 可用: {torch.cuda.is_available()}")
if torch.cuda.is_available():
    print(f"GPU: {torch.cuda.get_device_name(0)}")
    print(f"显存: {torch.cuda.get_device_properties(0).total_mem / 1024**3:.1f} GB")
```

**Cell 3：运行训练**

```python
# 单个算法快速测试
!python scripts/train_compare.py --algorithms improved --episodes 500 --seed 0

# 全部 4 种算法对比
# !python scripts/train_compare.py --algorithms all --episodes 2000 --seed 0
```

**Cell 4：下载结果**

```python
# 打包结果为 ZIP
import shutil
shutil.make_archive('results', 'zip', 'outputs/compare')

# 下载到本地
from google.colab import files
files.download('results.zip')
```

### 注意事项

| 项目 | 说明 |
|------|------|
| **运行时类型** | 菜单 -> 运行时 -> 更改运行时类型 -> 选择 **T4 GPU** |
| **超时断开** | 免费版空闲 ~90 分钟后断开，保持页面活跃 |
| **会话时长** | 免费版单次最多 ~12 小时 |
| **存储** | 断开后文件丢失，训练完成后及时下载结果 |
| **GPU 配额** | 免费版有每日 GPU 使用时长限制，用完后降级为 CPU |

### 耗时参考（Colab T4 GPU）

| 回合数 | 单个算法 | 全部 4 个 |
|--------|:-------:|:--------:|
| 500 | ~5 分钟 | ~18 分钟 |
| 2000 | ~15 分钟 | ~55 分钟 |

!!! tip "什么时候用 Colab"
    - 本地没有 GPU，但需要 GPU 加速
    - 需要跑大量回合的训练
    - 临时使用，不想配置本地环境

---

## 三种方式对比

| 维度 | 本地训练 | GitHub Actions | Google Colab |
|------|---------|---------------|-------------|
| **硬件** | 自己的电脑 | GitHub 提供 (2 核 CPU) | Google 提供 (T4 GPU) |
| **GPU** | 有则用 | 无 | 免费 T4 |
| **2000 轮耗时** | 25~40 分钟 | ~90 分钟 | ~55 分钟 |
| **结果归档** | 手动管理 | 自动推送到 c4nav-data | 手动下载 |
| **可复现性** | 取决于本地环境 | 标准化 Ubuntu 环境 | 环境可能变化 |
| **网络要求** | 不需要 | 需要 | 需要 |
| **使用限制** | 无 | 2,000 分钟/月（私有仓库） | 每日 GPU 配额 |
| **最适合** | 日常开发调试 | 正式论文实验 | GPU 加速训练 |

---

## 算法与训练框架的分离

无论使用哪种训练方式，运行的都是同一套代码：

### 详细文件架构

```
c4nav-core/
+-- config/                        <-- 你改这里（算法行为开关）
|   +-- baseline.yaml               <-- Algorithm 1: DQN
|   +-- ddqn.yaml                   <-- Algorithm 2: Double DQN
|   +-- d3qn.yaml                   <-- Algorithm 3: D3QN
|   +-- improved_d3qn.yaml          <-- Algorithm 4: ID3QN (你的改进)
|
+-- agents/                        <-- 需要时改这里（新增技术）
|   +-- network.py                  <-- 神经网络结构 (QNetwork + DuelingQNetwork)
|   +-- dqn_agent.py                <-- 智能体逻辑 (特性开关)
|   +-- per_buffer.py               <-- 优先经验回放 (SumTree)
|   +-- replay_buffer.py            <-- 标准经验回放
|
+-- envs/                          <-- 稳定层（一般不改）
|   +-- usv_env.py                  <-- Gym 仿真环境
|   +-- usv_dynamics.py             <-- 无人艇动力学
|   +-- lidar_sensor.py             <-- 虚拟 LiDAR
|   +-- reward.py                   <-- 奖励函数
|   +-- scene.py                    <-- 障碍物场景生成
|
+-- scripts/                       <-- 稳定层（几乎不改）
|   +-- train_compare.py            <-- 训练循环、日志、CSV 导出
|
+-- viz/                           <-- 稳定层（几乎不改）
|   +-- plot_comparison.py          <-- 图表生成
|
+-- .github/workflows/             <-- 稳定层（几乎不改）
    +-- smoke-test.yml              <-- 推送自动测试
    +-- train-compare.yml           <-- 4 算法对比训练
    +-- train-ablation.yml          <-- 消融实验
```

### 分层原则

| 层 | 内容 | 谁改 | 频率 |
|---|---|---|---|
| `config/` | 算法行为开关 | 算法开发者 | 每次实验 |
| `agents/` | 网络 + 智能体代码 | 算法开发者 | 新增技术时 |
| `envs/` | 仿真环境 | 很少 | 环境规则变更时 |
| `scripts/` | 训练循环 | 几乎不改 | 框架级变更 |
| `viz/` | 图表代码 | 几乎不改 | 样式调整时 |
| `.github/workflows/` | 云端自动化 | 几乎不改 | 基础设施变更时 |

### 配置如何控制算法

**YAML 配置**是核心接口。仅通过切换开关，就能定义完全不同的算法，**无需修改任何 Python 代码**：

```yaml
# ── DQN Baseline ──               # ── ID3QN (Improved) ──
network_type: "q"                   network_type: "dueling"       # Dueling 网络
use_double_dqn: false               use_double_dqn: true          # Double DQN
use_per: false                      use_per: true                 # 优先回放
use_soft_update: false              use_soft_update: true         # Polyak 软更新
loss_fn: "mse"                      loss_fn: "huber"              # Huber 损失
lr_schedule: "none"                 lr_schedule: "step"           # 学习率衰减
lr: 0.001                           lr: 0.0005                    # 更小的初始 LR

# 奖励塑形 (reward 节)
enable_shaping: false               enable_shaping: true          # 密集奖励
heading_scale: 0.0                  heading_scale: 0.3            # 航向对齐
```

### 开发者工作流

```
+-----------------------------------------------------------+
|  开发者工作流                                                |
|                                                            |
|  1. 修改 config/ YAML 或 agents/ 代码                       |
|     (调整算法参数或新增技术)                                   |
|                                                            |
|  2. git push                                               |
|     +---> Smoke Test 自动运行 (10 轮, ~2 分钟)               |
|           确认代码无误                                       |
|                                                            |
|  3. 手动触发 Train Compare                                   |
|     +---> 4 种算法自动训练 + 对比                             |
|           生成 CSV / 图表 / 模型权重                          |
|                                                            |
|  4. 下载结果，查看图表，判断算法优劣                            |
|                                                            |
|  5. 迭代：回到第 1 步继续改进                                 |
+-----------------------------------------------------------+
```

### 示例：新增一个算法变体

如果你想测试一个新想法（如加入 Noisy Networks），只需要 3 步：

**步骤 1**：创建 `config/my_variant.yaml`（从 `improved_d3qn.yaml` 复制，修改开关）

**步骤 2**：如果需要，在 `agents/network.py` 中添加新的网络类

**步骤 3**：在 `scripts/train_compare.py` 中注册：

```python
ALGORITHM_CONFIGS = {
    "baseline": "baseline.yaml",
    "ddqn": "ddqn.yaml",
    "d3qn": "d3qn.yaml",
    "improved": "improved_d3qn.yaml",
    "my_variant": "my_variant.yaml",     # <-- 加一行
}
```

然后 `git push` -> 触发训练 -> 对比结果。训练循环、日志记录、CSV 导出、图表生成、云端自动化**全部自动适配**你的新变体。

**核心理念：你只需要关注算法设计，训练、对比、可视化已经完全自动化。**
