# YAML Configuration Reference

All training parameters are controlled via YAML config files in `config/`.

## Example: `improved_d3qn.yaml`

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

## Parameter Groups

### Scene Parameters
Control the water area and obstacle generation. Aligned with C4 competition specifications.

### USV Parameters
Ship dynamics parameters. Values match the real USV130 ship.

### LiDAR Parameters
Virtual sensor configuration. 180 rays covering 360 degrees with 18m range.

### Reward Parameters
Terminal rewards match competition scoring. Shaping components are optional aids for training.

### Training Parameters
Algorithm hyperparameters. The algorithm variant is determined by the combination of switches (`use_double_dqn`, `network_type`, `use_per`, etc.).
