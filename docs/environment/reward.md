# 奖励函数设计

## 终端奖励（对齐竞赛规则）

```python
if reached:     return +100.0
if collided:    return -100.0
if out_of_bounds: return -100.0
reward = -0.1  # 步惩罚（鼓励高效）
```

## 奖励塑形（可选，`enable_shaping: true`）

稀疏的终端奖励使学习非常困难。奖励塑形提供中间引导信号：

### 进度奖励
```python
progress = prev_dist_to_target - dist_to_target
reward += progress * 2.0  # progress_scale
```
靠近目标时为正，远离时为负。

### 安全惩罚
```python
if dist_to_nearest_obs < safety_distance:  # 3.0m
    ratio = 1.0 - dist_to_nearest_obs / safety_distance
    reward -= 0.5 * ratio  # proximity_penalty
```
接近障碍物时给予惩罚，与距离成正比。

### 航向对齐
```python
reward += 0.3 * cos(heading_to_target)  # heading_scale
```
朝向目标方向时给予奖励。

## 消融实验结果

| 配置 | 成功率 | 平均奖励 |
|------|:------:|:-------:|
| ID3QN（全部组件） | **83%** | **84.0** |
| 去除奖励塑形 | 2% | -101.2 |
| 去除航向奖励 | 72% | 68.5 |
| 去除 PER | 75% | 72.1 |
| 去除软更新 | 78% | 76.3 |
| 去除学习率调度 | 80% | 79.8 |

奖励塑形是影响最大的单一组件。
