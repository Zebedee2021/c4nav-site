# c4nav-core 架构

## 模块结构

```
c4nav-core/
├── envs/              # 仿真环境
│   ├── usv_env.py     # 主环境（Gym 接口，184D 观测，5 个动作）
│   ├── usv_dynamics.py# 一阶惯性动力学模型
│   ├── lidar_sensor.py# 360° LiDAR 射线投射（180 条射线）
│   ├── scene.py       # 随机场景生成（障碍物、目标、出生点）
│   └── reward.py      # 奖励计算器，含可选塑形项
│
├── agents/            # 强化学习算法
│   ├── dqn_agent.py   # 统一 DQN 智能体（4 种变体，配置切换）
│   ├── network.py     # QNetwork + DuelingQNetwork
│   ├── replay_buffer.py
│   └── per_buffer.py  # 优先经验回放（SumTree）
│
├── config/            # YAML 配置文件
│   ├── baseline.yaml  # DQN
│   ├── ddqn.yaml      # Double DQN
│   ├── d3qn.yaml      # Dueling Double DQN
│   └── improved_d3qn.yaml # ID3QN（本文方法）
│
├── scripts/           # 训练与测试
│   ├── train.py
│   ├── train_compare.py  # 4 种算法对比
│   ├── test.py
│   └── run_ablation.py
│
└── viz/               # 可视化
    ├── plot_comparison.py
    ├── plot_trajectory.py
    └── draw_paper_figs.py
```

## 数据流

```
YAML 配置文件
    |
    v
USVNavEnv(config)              DQNAgent(state_dim, action_dim, config)
    |                                   |
    |-- reset() --> obs (184D)          |
    |                    |              |
    |               select_action() <---+
    |                    |
    |-- step(action) --> obs, reward, done, info
    |                    |
    |               store_transition()
    |               learn() --> loss
    |                    |
    |            update_epsilon()
    |            update_target_network()
    v                    v
 轨迹数据            model.pth
```

## 关键设计决策

**环境与智能体通过 `state_dim` 参数解耦。** 智能体的网络自动适应任意观测维度：

```python
# 环境侧：state_dim 由配置计算
self.state_dim = lidar_cfg.get("num_rays", 180) + 4  # 184

# 智能体侧：接收任意 state_dim
agent = DQNAgent(state_dim=env.state_dim, action_dim=env.action_dim, config=train_cfg)
# 首层：Linear(state_dim, hidden[0]) -- 自动适配
```

这意味着观测空间扩展（例如为 PIRL 添加物理特征）**无需修改任何智能体代码**。
