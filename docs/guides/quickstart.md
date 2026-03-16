# Quick Start

## Prerequisites

- Python 3.10+ (3.12 recommended)
- pip
- (Optional) NVIDIA GPU with CUDA for accelerated training

## Step 1: Clone and Setup

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

## Step 2: Train

```bash
# Train all 4 algorithms (2000 episodes each, ~90 min total)
python scripts/train_compare.py --algorithms all --seed 0

# Train single algorithm (faster)
python scripts/train_compare.py --algorithms improved --seed 0 --episodes 500
```

## Step 3: Test

```bash
python scripts/test.py --config config/improved_d3qn.yaml --model outputs/compare/improved_seed0/models/best.pth
```

Output includes success rate, average reward, trajectory visualizations, and animation of the best episode.
