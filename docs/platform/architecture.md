# c4nav-core Architecture

## Module Structure

```
c4nav-core/
├── envs/              # Simulation Environment
│   ├── usv_env.py     # Main env (Gym interface, 184D obs, 5 actions)
│   ├── usv_dynamics.py# 1st-order inertia dynamics model
│   ├── lidar_sensor.py# 360° LiDAR ray casting (180 rays)
│   ├── scene.py       # Random scene generation (obstacles, target, spawn)
│   └── reward.py      # Reward calculator with optional shaping
│
├── agents/            # RL Algorithms
│   ├── dqn_agent.py   # Unified DQN agent (4 variants via config)
│   ├── network.py     # QNetwork + DuelingQNetwork
│   ├── replay_buffer.py
│   └── per_buffer.py  # Prioritized Experience Replay (SumTree)
│
├── config/            # YAML Configurations
│   ├── baseline.yaml  # DQN
│   ├── ddqn.yaml      # Double DQN
│   ├── d3qn.yaml      # Dueling Double DQN
│   └── improved_d3qn.yaml # ID3QN (proposed method)
│
├── scripts/           # Training & Testing
│   ├── train.py
│   ├── train_compare.py  # 4-algorithm comparison
│   ├── test.py
│   └── run_ablation.py
│
└── viz/               # Visualization
    ├── plot_comparison.py
    ├── plot_trajectory.py
    └── draw_paper_figs.py
```

## Data Flow

```
YAML Config
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
 trajectory          model.pth
```

## Key Design Decisions

**Environment and Agent are decoupled via `state_dim` parameter.** The agent's network auto-adapts to any observation dimension:

```python
# env side: state_dim computed from config
self.state_dim = lidar_cfg.get("num_rays", 180) + 4  # 184

# agent side: accepts any state_dim
agent = DQNAgent(state_dim=env.state_dim, action_dim=env.action_dim, config=train_cfg)
# First layer: Linear(state_dim, hidden[0]) -- auto-adapts
```

This means observation expansion (e.g., adding physics features for PIRL) requires **zero changes to agent code**.
