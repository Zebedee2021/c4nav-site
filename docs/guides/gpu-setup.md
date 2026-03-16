# GPU 配置

## 检查 GPU 可用性

```bash
nvidia-smi -L
# 预期输出：GPU 0: NVIDIA GeForce RTX 4060 (UUID: ...)
```

## 安装 CUDA 版 PyTorch

推荐 Python 3.12（3.14 尚无 CUDA wheels）：

```bash
# 使用 Python 3.12 创建虚拟环境
python3.12 -m venv venv
source venv/bin/activate  # Windows 下使用 venv\Scripts\activate

# 安装 CUDA 12.4 版 PyTorch
pip install torch torchvision --index-url https://download.pytorch.org/whl/cu124
```

## 验证

```python
import torch
print(f"CUDA 可用: {torch.cuda.is_available()}")
print(f"设备: {torch.cuda.get_device_name(0)}")
# 预期输出：CUDA 可用: True
# 预期输出：设备: NVIDIA GeForce RTX 4060
```

## 训练加速效果

c4nav-core 的模型较小（约 180K 参数），GPU 加速主要体现在经验回放缓冲区的批量运算上：

| 硬件 | 2000 轮训练耗时 |
|------|:--------------:|
| 仅 CPU (i7) | ~40 分钟 |
| RTX 4060 GPU | ~25 分钟 |

该模型规模下加速幅度适中（约 1.6 倍）。使用更大的网络或更大的批量时，加速效果更明显。
