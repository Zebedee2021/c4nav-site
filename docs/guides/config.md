# YAML 参数配置参考

所有训练参数通过 `config/` 目录下的 YAML 配置文件控制。

## 示例：`improved_d3qn.yaml`

```yaml
scene:
  area_size: 20.0
  num_obstacles_min: 3
  num_obstacles_max: 5
  obstacle_radius: 0.8
  target_radius: 1.5
  spawn_margin: 2.5
  min_start_target_dist: 8.0
  min_obstacle_spacing: 2.5

usv:
  max_speed: 1.8
  cruise_speed: 1.4
  max_turn_rate: 45.0
  speed_damping: 0.95
  turn_damping: 0.90
  collision_radius: 0.65

lidar:
  num_rays: 180
  max_range: 18.0
  fov_degrees: 360.0

reward:
  reach_target: 100.0
  collision: -100.0
  step_penalty: -0.1
  enable_shaping: true
  progress_scale: 2.0
  safety_distance: 3.0
  proximity_penalty: 0.5
  heading_scale: 0.3

episode:
  max_steps: 500
  dt: 0.1

training:
  num_episodes: 2000
  lr: 0.0005
  gamma: 0.99
  epsilon_start: 1.0
  epsilon_end: 0.02
  epsilon_decay: 0.998
  batch_size: 128
  buffer_capacity: 200000
  hidden_dims: [256, 256]
  network_type: "dueling"
  use_double_dqn: true
  use_per: true
  use_soft_update: true
  tau: 0.005
  loss_fn: "huber"
  lr_schedule: "step"
  lr_step_size: 500
  lr_gamma: 0.5
```

## 参数分组

### 场景参数
控制水域面积和障碍物生成，与 C4 竞赛规格对齐。

### USV 参数
船体动力学参数，数值与真实 USV130 实船匹配。

### LiDAR 参数
虚拟传感器配置：180 条射线覆盖 360 度，最大量程 18m。

### 奖励参数
终端奖励与竞赛评分规则对齐，奖励塑形分量为可选的训练辅助项。

### 训练参数
算法超参数。算法变体由开关组合决定（`use_double_dqn`、`network_type`、`use_per` 等）。
