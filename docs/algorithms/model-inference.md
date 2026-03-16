# 模型结构、存储与推理

## 什么是"模型"？

训练好的"模型"是一个含有约 180K 浮点数（权重和偏置）的神经网络。训练完成后，这些权重编码了从数十万次试错交互中学到的导航策略。

## 存储格式

模型保存为 `.pth` 文件（PyTorch 标准格式）：

| 文件 | 大小 | 内容 |
|------|------|------|
| `baseline_seed0/final.pth` | 1.8 MB | QNetwork + 优化器状态 |
| `improved_seed0/final.pth` | 2.8 MB | DuelingQNetwork + 目标网络 + 优化器 + epsilon |

2.8 MB 包含：

- Q 网络权重（约 0.7 MB）
- 目标网络权重（约 0.7 MB，一份副本）
- 优化器状态（约 1.2 MB，Adam 动量/方差）
- Epsilon 值（1 个浮点数）

仅用于推理时，Q 网络权重（约 0.7 MB）即可。

## 加载与运行

```python
# 步骤 1：创建空网络（随机权重）
agent = DQNAgent(state_dim=184, action_dim=5, config=train_cfg)

# 步骤 2：加载训练好的权重
agent.load("outputs/compare/improved_seed0/models/final.pth")

# 步骤 3：运行推理循环
state, info = env.reset()
for step in range(500):
    action = agent.select_action(state, training=False)  # 推理
    state, reward, terminated, truncated, info = env.step(action)
    if terminated or truncated:
        break
```

## 推理过程详解

一次推理调用 = 一连串矩阵乘法：

```
184 个输入数值
    x 权重矩阵 --> 256 个数值
    x 权重矩阵 --> 256 个数值
    --> V 分支 --> 1 个数值
    --> A 分支 --> 5 个数值
    Q = V + (A - mean(A)) --> 5 个 Q 值
    argmax --> 动作 (0-4)
```

耗时：< 1 毫秒。

## 训练 vs 推理

| | 训练 | 推理 |
|---|---|---|
| 功能 | 试错，调整权重 | 使用固定权重做决策 |
| 计算量 | 较重（前向 + 反向 + 梯度更新） | 很轻（仅前向） |
| 耗时 | 2000 轮约 25 分钟 | 每步 < 1 毫秒 |
| 是否需要 GPU | 推荐（加速训练） | 不需要 |
| 经验回放缓冲区 | 需要（存储数十万条经验） | 不需要 |

## 推理硬件要求

| 硬件 | 能否推理？ | 单步延迟 |
|------|:---------:|---------|
| RTX 4060 GPU | 可以 | < 0.1 ms |
| 笔记本 CPU | 可以 | < 1 ms |
| Raspberry Pi 4 | 可以 | ~5 ms |
| Jetson Nano（船载嵌入式） | 可以 | ~2 ms |

环境每步模拟 0.1 秒实时时间，推理耗时 < 1ms，在任何硬件上都能轻松实现实时运行。
