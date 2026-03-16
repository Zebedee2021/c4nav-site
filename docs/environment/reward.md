# Reward Function Design

## Terminal Rewards (Competition-Aligned)

```python
if reached:     return +100.0
if collided:    return -100.0
if out_of_bounds: return -100.0
reward = -0.1  # step penalty (encourages efficiency)
```

## Reward Shaping (Optional, `enable_shaping: true`)

Sparse terminal rewards make learning difficult. Shaping provides intermediate guidance:

### Progress Reward
```python
progress = prev_dist_to_target - dist_to_target
reward += progress * 2.0  # progress_scale
```
Positive when moving closer, negative when moving away.

### Safety Penalty
```python
if dist_to_nearest_obs < safety_distance:  # 3.0m
    ratio = 1.0 - dist_to_nearest_obs / safety_distance
    reward -= 0.5 * ratio  # proximity_penalty
```
Penalizes getting close to obstacles, proportional to proximity.

### Heading Alignment
```python
reward += 0.3 * cos(heading_to_target)  # heading_scale
```
Rewards facing the target direction.

## Ablation Results

| Configuration | Success Rate | Avg Reward |
|--------------|:-----------:|:----------:|
| ID3QN (all components) | **83%** | **84.0** |
| w/o reward shaping | 2% | -101.2 |
| w/o heading reward | 72% | 68.5 |
| w/o PER | 75% | 72.1 |
| w/o soft update | 78% | 76.3 |
| w/o LR schedule | 80% | 79.8 |

Reward shaping is the single most impactful component.
