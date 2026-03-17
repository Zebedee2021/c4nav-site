# 云端训练

## 架构

```
本地                          GitHub 云端                  公开资产
┌─────────────┐   git push   ┌──────────────────┐  自动推送  ┌─────────────┐
│  c4nav-core │ ────────────>│  GitHub Actions   │─────────>│  c4nav-data │
│  (代码仓库)  │              │  Ubuntu 虚拟机     │           │  (数据仓库)  │
└─────────────┘              └──────────────────┘           └─────────────┘
      │                              │                             │
      │  手动触发                     │ Artifact (ZIP)              v
      └────────────────────────────>│                        c4nav-demo
                                                             c4nav-site
```

训练在 GitHub 免费 Ubuntu 虚拟机上运行（2 核 CPU、7 GB 内存），无需本地 GPU。

!!! tip "GitHub Actions 相当于一台免费的远程电脑"
    你把代码推上去，它帮你跑，跑完把结果还给你。代码是你的，电脑是 GitHub 借你的。

## 三个工作流

| 工作流 | 触发方式 | 用途 | 超时 |
|--------|---------|------|------|
| **Smoke Test** | 推送自动触发 | 10 轮快速验证，确认代码无误 | 15 分钟 |
| **Train Compare** | 手动触发 | 4 种算法对比训练 | 180 分钟 |
| **Train Ablation** | 手动触发 | 6 种变体消融实验 | 240 分钟 |

## 如何触发训练（7 步）

### 步骤 1：打开仓库

访问 `https://github.com/Zebedee2021/c4nav-core`

### 步骤 2：点击 Actions 选项卡

在顶部导航栏找到 **Actions** 并点击。

### 步骤 3：选择 Train Compare 工作流

在左侧栏中点击 **Train Compare**。

### 步骤 4：点击 Run workflow

右侧出现蓝色横幅，点击 **Run workflow** 按钮。

### 步骤 5：填写参数

| 参数 | 选项 | 说明 |
|------|------|------|
| algorithms | `all` / `baseline` / `ddqn` / `d3qn` / `improved` | 训练哪些算法 |
| episodes | `500` / `1000` / `2000` | 训练回合数 |
| seed | 任意整数 | 随机种子（保证可复现） |
| upload_to_data_repo | 勾选与否 | 是否自动推送结果到 c4nav-data |

**推荐配置：**

- **首次测试**（约 8 分钟）：algorithms = `improved`，episodes = `500`
- **正式论文**（约 90 分钟）：algorithms = `all`，episodes = `2000`，upload = 勾选

### 步骤 6：监控进度

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

### 步骤 7：下载结果

训练完成后，页面底部出现 Artifact ZIP，点击下载：

```
compare-results-ep2000-seed0/
├── comparison_curves.png       # 训练曲线
├── comparison_bars.png         # 柱状对比图
├── baseline_seed0/training_log.csv
├── ddqn_seed0/training_log.csv
├── d3qn_seed0/training_log.csv
└── improved_seed0/
    ├── training_log.csv
    └── models/best.pth         # 最优模型
```

## 耗时参考

| 回合数 | 单个算法 | 全部 4 个 |
|--------|:-------:|:--------:|
| 500 | ~8 分钟 | ~30 分钟 |
| 1000 | ~15 分钟 | ~60 分钟 |
| 2000 | ~25 分钟 | ~90 分钟 |

免费计划私有仓库每月 **2,000 分钟**。一次完整对比训练约 90 分钟，额度充裕。

## 本地训练

如果有本地 GPU 或不想使用云端，也可以直接运行：

```bash
pip install -r requirements.txt

# 单个算法
python scripts/train_compare.py --algorithms improved --episodes 2000 --seed 0

# 全部算法
python scripts/train_compare.py --algorithms all --episodes 2000 --seed 0

# 消融实验
python scripts/run_ablation.py --episodes 500 --seed 0
```

## 算法与训练框架的分离

```
c4nav-core/
├── config/                    ← 你改这里（算法行为开关）
├── agents/                    ← 需要时改这里（新增技术）
├── envs/                      ← 稳定层（一般不改）
├── scripts/                   ← 稳定层（几乎不改）
├── viz/                       ← 稳定层（几乎不改）
└── .github/workflows/         ← 稳定层（几乎不改）
```

| 层 | 内容 | 谁改 | 频率 |
|---|---|---|---|
| `config/` | 算法行为开关 | 算法开发者 | 每次实验 |
| `agents/` | 网络 + 智能体代码 | 算法开发者 | 新增技术时 |
| `envs/` | 仿真环境 | 很少 | 环境规则变更时 |
| `scripts/` | 训练循环 | 几乎不改 | 框架级变更 |

**核心理念：你只需要关注算法设计，训练、对比、可视化已经完全自动化。**
