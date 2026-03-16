# 快速开始

## 前提条件

- Python 3.10+（推荐 3.12）
- pip
- （可选）NVIDIA 显卡 + CUDA，用于加速训练

## 步骤一：克隆与配置

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

## 步骤二：训练

```bash
# 训练全部 4 种算法（每种 2000 轮，总计约 90 分钟）
python scripts/train_compare.py --algorithms all --seed 0

# 单独训练某一种算法（更快）
python scripts/train_compare.py --algorithms improved --seed 0 --episodes 500
```

## 步骤三：测试

```bash
python scripts/test.py --config config/improved_d3qn.yaml --model outputs/compare/improved_seed0/models/best.pth
```

输出包括：成功率、平均奖励、轨迹可视化，以及最佳回合的动画。
