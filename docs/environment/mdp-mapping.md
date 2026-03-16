# Competition Rules to MDP Mapping

## Scene Rules --> State Space + Scene Generation

| Competition Rule | Code | Config Value |
|-----------------|------|-------------|
| Water area 20m x 20m | `area_size: 20.0` | Walls at +/-10m |
| 3-5 random obstacles | `num_obstacles_min/max: 3/5` | Randomized each episode |
| Obstacle spacing >= 2.5m | `min_obstacle_spacing: 2.5` | Prevent unsolvable scenes |
| Start-to-target dist >= 8m | `min_start_target_dist: 8.0` | Meaningful navigation |
| Target reach radius 1.5m | `target_radius: 1.5` | Arrival detection |

## Ship Specification --> Dynamics Model (Transition Function)

| Competition Rule | Code | Effect |
|-----------------|------|--------|
| Max speed 1.8 m/s | `max_speed: 1.8` | Agent cannot exceed |
| Max turn rate 45 deg/s | `max_turn_rate: 45.0` | Agent cannot turn instantly |
| Ship has inertia | `speed_damping: 0.95` | Speed response is delayed |
| Turn delay | `turn_damping: 0.90` | Turn response is delayed |
| Hull collision radius 0.65m | `collision_radius: 0.65` | Collision detection size |

## Scoring Rules --> Reward Function

| Competition Scoring | Reward Design | Code |
|--------------------|---------------|------|
| Reach target: score | `+100.0` | `if reached` |
| Collision: penalty | `-100.0` | `if collided` |
| Out of bounds: penalty | `-100.0` | `if out_of_bounds` |
| Less time is better | `-0.1` per step | `step_penalty` |

## Reward Shaping (engineering aid, not competition rules)

| Shaping Term | Formula | Purpose |
|-------------|---------|---------|
| Progress | `(prev_dist - curr_dist) * 2.0` | Guide agent toward target |
| Safety | `-0.5 * (1 - obs_dist/safety_dist)` | Maintain safe distance |
| Heading | `+0.3 * cos(heading_to_target)` | Face toward target |

## Control Method --> Action Space

5 discrete actions defined in `ACTION_MAP`:

| Action | Turn | Speed | Usage |
|--------|------|-------|-------|
| 0 | Left 45 deg | 70% cruise | Emergency avoidance |
| 1 | Left 22.5 deg | 100% cruise | Fine adjustment |
| 2 | Straight | 100% cruise | Normal cruise |
| 3 | Right 22.5 deg | 100% cruise | Fine adjustment |
| 4 | Right 45 deg | 70% cruise | Emergency avoidance |

Hard turns automatically reduce speed (speed_factor: 0.7), matching real ship operations.

## Observation Space

184-dimensional vector:

| Index | Content | Normalization |
|-------|---------|---------------|
| [0:180] | LiDAR readings | 0=close, 1=clear (by max_range) |
| [180] | Distance to target | / diagonal |
| [181] | Angle to target | / pi |
| [182] | Distance to nearest obstacle | / max_range |
| [183] | Angle to nearest obstacle | / pi |
